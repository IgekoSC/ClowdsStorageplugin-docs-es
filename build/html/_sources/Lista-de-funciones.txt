.. role:: cpp(code)
   :language: c++

.. _Lista-de-funciones:

Lista de Funciones
******************

La siguiente lista comprende las funciones mínimas que se necesitan implementar para que el plugin pueda comunicarse con Clowds y que este pueda sincronizar con el servicio en la nube que quiera implementar.


getChanges 
____________

Esta es la función principal del Plug-in, la más importante, y normalmente la que más código dedica. Su cometido es el de recibir información de los sucesos de la nube, procesar esos eventos, y traducirlos en cambios de Clowds (CloudStorageChange), para que el programa sepa qué hacer con ellos. 

Devuelve un
  
	:cpp:`QHash <QString, QList<CloudStorageChange> >`

el cual contendrá un registro de los cambios que se van a realizar. La clave será la ruta del fichero en `QString`, y su valor un `CloudStorageChange`, que es el cambio que ha sufrido. 

El primer paso será hacer la petición HTTP para obtener la lista de los eventos ocurridos en la nube e ir guardándolos en un `QHash`, por ejemplo `allEvents` (ver :ref:`Anexo 2 <"QHash<QString, Change> allEvents">`). Lo importante es que el `QHash` almacene los metadatos (normalmente en JSON). 

En el caso de que la nube pueda devolver más de un cambio por archivo, una buena manera de tratar esta lista de cambios sería hacer otro `QHash`, como `myChanges` (ver :ref:`Anexo 2 <"QHash<QString, QMap<uint, Change> > myChanges">`), donde cada valor corresponde a un QMap. Dicho QMap sería una lista de1 los eventos (metadatos), indexada por fecha. 

	:cpp:`QHash < file_ID, QMap<fecha, evento> >`

Con esto se conseguirá que por cada fichero haya una lista de todos sus cambios ordenados cronológicamente. 

La mecánica será recorrer todos esos cambios ordenados por fecha para ir procesando por cada fichero sus cambios uno a uno, del más antiguo al más reciente. Según los metadatos de cada evento, debemos ir asignando las 'Flags' en función de que clase de evento se trate: subir o crear es 'New', renombrar o mover es 'Moved', etc. 

downloadFile 
____________

Esta función descarga desde la nube cada archivo que Clowds considere que debe descargar al sincronizar. 

Parámetros:

	**remotePath** - Ruta remota del fichero.

	**localPath** - Ruta del fichero en el sistema de archivos local.

.. Note:: Si el cifrado **no** está activado, remotePath y localPath serán iguales. 

Aquí sólo debes preocuparte de introducir la petición HTTP que requiera el sistema que estemos implementando para hacer la descarga de un sólo archivo. 

Para descargar el archivo: 

	:cpp:`resources_->shrGet(QUrl, QFile);`

	**QUrl** - URL de la petición. 

	**QFile** - El archivo que se descargará en la carpeta de sincronización. 

Dependiendo del sistema con el que estemos trabajando, es posible que se requieran cabeceras, o parámetros en la URL. 

uploadFile 
__________

Esta función se encarga de subir los archivos que Clowds detecte como 'new'  en la carpeta de sincronización. Al sincronizar Clowds, se encargará de subir a la nube los archivos creados en local. 

Parámetros:

	**remotePath** - Ruta remota del fichero.

	**localPath** - Ruta del fichero en el sistema de archivos local.

.. Note:: Si el cifrado **no** está activado, remotePath y localPath serán iguales. Siempre usar remotepath con la url del endpoint del servicio y localPath como paramétro para abrir el fichero local para leer su contenido 
	
Normalmente, los archivos se suben mediante Multipart-POST o PUT (véase la documentación de cada servicio). 

	:cpp:`resources->shrPost(QUrl, QMap, QFile);`


	**QUrl** - URL de la petición HTTP POST para subir un fichero. 

	**QMap** - Parámetros que pueda requerir la petición HTTP, normalmente datos sobre el fichero que va a ser subido. 

	**QFile** - Archivo a subir. 

Si el servicio en la nube requiere el uso del verbo PUT para subir archivos, puede usar la implementación que Clowds suministra:

	:cpp:`resources->shrPut(QUrl url, QFile *localFile, qint64 chunkIndex = -1, qint64 chunkSize = 10485760);`
	
	**QUrl** - URL de la petición HTTP PUT para subir un fichero. 

	**QFile** - Archivo a subir. 

	**qint64 chunkIndex** - Número del chunk de fichero a partir del cual hay que subir al servicio en la nube

	**qint64 chunkSize** - Tamaño de cada uno de los chunk del fichero.

moveFile
________

Mueve un archivo en la nube que había sido movido en la carpeta de sincronización. 

Parámetros: 

from - ruta local anterior del archivo. 

to - ruta local nueva del archivo. 

.. Note:: Si el cifrado **no** está activado, remotePath y localPath serán iguales. Siempre usar remotepath con la url del endpoint del servicio y localPath como paramétro para abrir el fichero local para leer su contenido

Hacer la petición HTTP correspondiente. Si no hay una petición exclusiva para mover un archivo, normalmente se hace actualizando la información del archivo, cambiando su padre (posiblemente el ID del padre). 

**GET** → :cpp:`resources->shrGet();`

**POST** → :cpp:`resources->shrPost();`

**Otro verbo** → :cpp:`resources->shrSendCustomRequest();`

Se podría guardar el resultado de esta petición en un `QByteArray` para confirmar que se ha movido el archivo: 

.. code-block:: cpp
	
	QByteArray result = resources->shrSendCustomRequest(); 

En caso de que se haya realizado con éxito devolver true; en otro caso, false. 

moveDir 
_______

Mueve un directorio en la nube que había sido movido en la carpeta de sincronización. 

Parámetros: 

	**from** - ruta local anterior del directorio. 

	**to** - ruta local nueva del directorio. 

.. Note:: Si el cifrado **está** activado, tanto el *from* como el *to* estarán cifrados.

Hacer la petición HTTP correspondiente. Si no hay una petición exclusiva para mover un directorio, normalmente se hace actualizando la información del archivo, cambiando su padre (posiblemente el ID del padre). 

**GET** → :cpp:`resources->shrGet();`

**POST** → :cpp:`resources->shrPost();`

**Otro verbo** → :cpp:`resources->shrSendCustomRequest();`

Nos puede interesar guardar el resultado de esta petición en un `QByteArray` para confirmar que se ha movido el directorio: 

.. code-block:: cpp
	
	QByteArray result = resources->shrSendCustomRequest();

En caso de que se haya realizado con éxito devolver true; en otro caso, false. 

rmFile
______

Borra un archivo en la nube que ha sido borrado en la carpeta de sincronización. La mayoría de servicios por defecto lo envían a la papelera. Siempre que tengamos la posibilidad, se recomienda usar la papelera para eliminar archivos, en vez de borrarlos directamente, para no poner en riesgo los archivos importantes de los usuarios. 

Parámetros: 

	**path** - La ruta del archivo local que ha sido eliminado. 

.. Note:: Si el cifrado **está** activado, esta ruta estará cifrado.

Hacer la petición HTTP correspondiente, usando una de estas funciones, según el caso: 

**GET** → :cpp:`resources->shrGet();`

**POST** → :cpp:`resources->shrPost();`

**Otro verbo** → :cpp:`resources->shrSendCustomRequest();`

Generalmente, se usa el verbo DELETE para borrar archivos. Por ejemplo: 

.. code-block:: cpp

	resources_->shrSendCustomRequest(“DELETE”, QUrl(http://www.example.org/endpoint)); 

Nos puede interesar guardar el resultado de esta petición en un `QByteArray` para confirmar que se ha borrado el archivo: 

.. code-block:: cpp
	
	QByteArray result = resources->shrSendCustomRequest();

En caso de que se haya realizado con éxito devolver true; en otro caso, false. 

Es probable que necesites información del archivo a borrar, la cual podrás obtener a través de una estructura de datos propia, como :cpp:`filesByPath.value(path)` (ver :ref:`Anexo 2 <"QHash<QString, File> filesByPath">`). 

mkDir
_____

Crea los directorios nuevos detectados por Clowds en la nube. 

Parámetros: 

	**path** - La ruta del directorio local que ha sido creado. 

.. Note:: Si el cifrado **está** activado, esta ruta estará cifrada. 

Hacer la petición HTTP correspondiente, usando una de estas funciones, según el caso: 

**GET** → :cpp:`resources->shrGet();`

**POST** → :cpp:`resources->shrPost();`

**Otro verbo** → :cpp:`resources->shrSendCustomRequest();`

rmDir
_____

Borra un directorio en la nube que ha sido borrado en la carpeta de sincronización, la mayoría de servicios por defecto lo envían a la papelera. 

Parámetros: 

 **path** - La ruta del directorio local que ha sido eliminado. 

Hacer la petición HTTP correspondiente, usando una de estas funciones, según el caso: 

**GET** → :cpp:`resources->shrGet();`

**POST** → :cpp:`resources->shrPost();`

**Otro verbo** → :cpp:`resources->shrSendCustomRequest();`

Generalmente, se usa el verbo DELETE para borrar archivos. Por ejemplo: 

.. code-block:: cpp

	resources_->shrSendCustomRequest(“DELETE”, QUrl(http://www.example.org/endpoint)); 

Nos puede interesar guardar el resultado de esta petición en un `QByteArray` para confirmar que se ha borrado el directorio: 

.. code-block:: cpp

	QByteArray result = resources->shrSendCustomRequest(); 

En caso de que se haya realizado con éxito devolver true; en otro caso, false. 

Es probable que necesites información del directorio a borrar, la cual podrás obtener a través de una estructura de datos propia, como :cpp:`filesByPath.value(path)` (ver :ref:`Anexo 2 <"QHash<QString, File> filesByPath">`). 

