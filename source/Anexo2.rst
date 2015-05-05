.. _Anexo-2-Clase-principal-campos-y-métodos-útiles:

Anexo 2. Clase principal: campos y métodos útiles
*************************************************

*Nada de lo descrito en este Anexo es obligatorio para el funcionamiento
de un Plug-in para Clowds. Son recomendaciones para facilitar su
desarrollo.*

*Se sugiere que el nombre de la clase principal guarde relación con el
nombre del servicio que se va a implementar. De aquí en adelante la
llamaremos myCloud.*

QString accessToken
~~~~~~~~~~~~~~~~~~~

**Access Token**

En caso de utilizar OAuth2, puede ser útil guardar en esta variable el
access token, ya que lo más normal es que tengas que introducirlo en
cada petición HTTP para la autorización.

QString nextChanges
~~~~~~~~~~~~~~~~~~~

**Marcador de eventos**

Generalmente el servicio debe ponernos a disposición alguna forma de
recibir los cambios desde un punto en concreto. En esta variable debemos
guardar cada vez que hay cambios la última posición de los mismos, para
tener un orden. Este valor debe registrarse además en la base de datos
para guardar la posición al cerrar el programa y la volvemos a recuperar
de la base de datos al iniciar de nuevo el programa.

``resources_->settingsSetValue(QString key, QString value);``

QHash  allEvents
~~~~~~~~~~~~~~~~

**Lista de todos los eventos**

En esta lista se almacenarán temporalmente (sin importar su contenido)
todos los cambios que reciba Clowds en cada llamada a getChanges().

-  Clave: ID del evento.
-  Valor: Objeto Change, con el QJsonObject entero del evento.

QHash > myChanges
~~~~~~~~~~~~~~~~~

**Lista de todos los cambios por cada archivo**

Este QHash es el que retorna getChanges(). De todos los eventos de
allEvents, solo se insertarán en myChanges aquellos eventos que sean
cambios válidos para Clowds (Insertar, modificar, borrar, mover,
restaurar de la papelera…). Eventos como marcar un archivo como
favorito, añadir o quitar etiquetas, son eventos inservibles para
Clowds.

Guardará de cada archivo, un QMap con todos sus cambios, indexados por
fecha (recomendable UNIX Time Stamp) para ordenarlos cronológicamente.

| Clave: ID del archivo. Valor: QMap
| { Clave: Fecha (UNIX Time Stamp). Valor: Objeto Change. }.

QHash filesById
~~~~~~~~~~~~~~~

**Lista de archivos indexada por ID**

En este QHash se irán insertando los archivos que se vayan descargando o
creando en la nube indexados por el ID del archivo, para poder tener un
registro de todos los archivos que tenemos almacenados, y así poder
conseguir información de cada uno de ellos en cualquier momento.

-  Clave: ID del archivo [File::getId()]
-  Valor: Archivo [File]

QHash filesByPath
~~~~~~~~~~~~~~~~~

**Lista de archivos indexada por la ruta del archivo**

En este QHash se irán insertando los archivos que se vayan descargando o
creando en la nube indexados por la ruta (path) del archivo, para poder
tener un registro de todos los archivos que tenemos almacenados, y así
poder conseguir información de cada uno de ellos en cualquier momento.

-  Clave: Ruta del archivo [File::getPath()]
-  Valor: Archivo [File]

QHash<QString, QList<CloudS
~~~~~~~~~~~~~~~~~~~~~~~~~~~
