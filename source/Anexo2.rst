.. _Anexo-2-Clase-principal-campos-y-métodos-útiles:

Anexo 2. Clase principal: campos y métodos útiles
*************************************************

*Nada de lo descrito en este Anexo es obligatorio para el funcionamiento de un Plug-in para Clowds. Son recomendaciones para facilitar su desarrollo.*

Se sugiere que el nombre de la clase principal guarde relación con el nombre del servicio que se va a implementar. De aquí en adelante la llamaremos myCloud. 

QString accessToken
___________________ 

*Access Token*

En caso de utilizar OAuth2, puede ser útil guardar en esta variable el access token, ya que lo más normal es que tengas que introducirlo en cada petición HTTP para la autorización. 

QString nextChanges
___________________ 

*Marcador de eventos*

Generalmente el servicio debe ponernos a disposición alguna forma de recibir los cambios desde un punto en concreto. En esta variable debemos guardar cada vez que hay cambios la última posición de los mismos, para tener un orden. Este valor debe registrarse además en la base de datos para guardar la posición al cerrar el programa y la volvemos a recuperar de la base de datos al iniciar de nuevo el programa. 

.. code-block:: cpp

	resources_->settingsSetValue(QString key, QString value);

**QString key** -> En este caso seria "token" , "cursor" o por el estilo

**QString value** -> Esto sería el valor que nos de el servicio, generalmente una cadena alfanumérica muy larga.


.. _"QHash<QString, Change> allEvents":

QHash<QString, Change> allEvents 
________________________________

*Lista de todos los eventos*

En esta lista se almacenarán temporalmente (sin importar su contenido) todos los cambios que reciba Clowds en cada llamada a getChanges(). 

+---------------+------------------------+
| Clave         |        Valor           |
+===============+========================+
| QString       |   Change		 |
+---------------+------------------------+
|ID del         |  Objeto Change         |
|evento         |  QJsonObject entero    | 
|               |  del evento            |
+---------------+------------------------+

.. _"QHash<QString, QMap<uint, Change> > myChanges":

QHash<QString, QMap<uint, Change> > myChanges
_____________________________________________

*Lista de todos los cambios por cada archivo*

Este `QHash` es el que retorna getChanges(). De todos los eventos de allEvents, solo se insertarán en myChanges aquellos eventos que sean cambios válidos para Clowds (Insertar, modificar, borrar, mover, restaurar de la papelera…). Eventos como marcar un archivo como favorito, añadir o quitar etiquetas, son eventos inservibles para Clowds. 

Guardará de cada archivo, un QMap con todos sus cambios, indexados por fecha (recomendable UNIX Time Stamp) para ordenarlos cronológicamente. 

+------------+------------------------+
| Clave      |        Valor           |
+============+============+===========+
| QString    |   QMap<uint, Change>   |
+------------+------------+-----------+
|            |    uint    |   Change  |
|            +------------+-----------+
|  	     | Fecha      |  Objeto   |
|            | (UNIX Time |  Change   |
|            | Stamp)     | 	      |
+------------+------------+-----------+
 
.. _"QHash<QString, File> filesById":

QHash<QString, File> filesById 
______________________________

*Lista de archivos indexada por ID*

En este QHash se irán insertando los archivos que se vayan descargando o creando en la nube indexados por el ID del archivo, para poder tener un registro de todos los archivos que tenemos almacenados, y así poder conseguir información de cada uno de ellos en cualquier momento. 

+---------------+------------------------+
| Clave         |        Valor           |
+===============+========================+
| QString       |   File		 |
+---------------+------------------------+
|ID del         |  Archivo               |
|archivo        |                        | 
|`File::getId()`|                        |
+---------------+------------------------+

.. _"QHash<QString, File> filesByPath":

QHash<QString, File> filesByPath
________________________________

*Lista de archivos indexada por la ruta del archivo*

En este QHash se irán insertando los archivos que se vayan descargando o creando en la nube indexados por la ruta (path) del archivo, para poder tener un registro de todos los archivos que tenemos almacenados, y así poder conseguir información de cada uno de ellos en cualquier momento. 

+-----------------+------------------------+
| Clave           |        Valor           |
+=================+========================+
| QString         |   File		   |
+-----------------+------------------------+
|Ruta del         |  Archivo               |
|archivo          |                        | 
|`File::getPath()`|                        |
+-----------------+------------------------+

.. _"QHash<QString, QList<CloudStorageChange> > ClowdsChanges ":

QHash<QString, QList<CloudStorageChange> > ClowdsChanges 
________________________________________________________

*Lista de los cambios que se harán efectivos*

Este QHash es el que devolveremos en getChanges() al finalizar de procesar todos los cambios que encuentre Clowds. 

+-----------------+----------------------------+
| Clave           |        Valor               |
+=================+============================+
| QString         |  QList<CloudStorageChange> |
+-----------------+----------------------------+
| Ruta del archivo|  Lista de cambios          |
| que sufre el	  |  CloudStorageChange        | 
| cambio          |                            |
+-----------------+----------------------------+

.. _"QHash<QString, QList<CloudStorageChange> > ClowdsChangesById":

QHash<QString, QList<CloudStorageChange> > ClowdsChangesById 
____________________________________________________________

*Lista de los cambios que se harán efectivos indexada por ID*

Este QHash es igual que ClowdsChanges, salvo que está indexado por ID del archivo. Esto sirve para evitar conflictos en caso de que se repitan las rutas, debido a que ClowdsChanges va indexado por la ruta del archivo (Podrían solaparse o anularse los cambios, así como repetirse archivos, y otras situaciones). 

+-----------------+----------------------------+
| Clave           |        Valor               |
+=================+============================+
| QString         |  QList<CloudStorageChange> |
+-----------------+----------------------------+
| ID del archivo  |  Lista de cambios          |
| que sufre el	  |  CloudStorageChange        | 
| cambio          |                            |
+-----------------+----------------------------+ 

.. _"bool updateParentsOf(File &file)": 

bool updateParentsOf(File &file) 
________________________________

*Construye el path en local mediante el id y el parent id del 'file' que recibe*

Esta función se usa en caso de que el servicio no dé información acerca del path o el path no esté como Clowds lo pide. Por cada 'file' que reciba esta función lo que hace es construir el path del 'file' mediante el id y el parent id, todo ello en un bucle que va obteniendo el nombre (title) de dicho id y el del parent id hasta llegar a root que en ese punto se sale del bucle. El resultado lo va insertando en el path de dicho 'file' y una vez obtenido, en filesByPath inserta el path y el 'file' y devuelve 'true' si la operación se ha hecho correctamente, por lo contrario 'false'. 

    Párametro: Objeto File. 

