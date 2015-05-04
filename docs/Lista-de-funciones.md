#Lista de Funciones
###getChanges()
Esta es la función principal del Plug-in, la más importante, y normalmente la que más código dedica. Su cometido es el de recibir información de los sucesos de la nube, procesar esos eventos, y traducirlos en cambios de Clowds (CloudStorageChange), para que el programa sepa qué hacer con ellos. 
Devuelve un  ```QHash<QString, QList<CloudStorageChange> >```

el cual contendrá un registro de los cambios que se van a realizar. La clave será la ruta del fichero en `Qstring`, y su valor un `CloudStorageChange`, que es el cambio que ha sufrido.

El primer paso será hacer la petición HTTP para obtener la lista de los eventos ocurridos en la nube e ir guardándolos en un QHash, por ejemplo allEvents (ver ANEXO 2). Lo importante es que el QHash almacene los metadatos (normalmente en JSON).

En el caso de que la nube pueda devolver más de un cambio por archivo, una buena manera de tratar esta lista de cambios sería hacer otro QHash, como myChanges (Ver ANEXO 2), en el que en cada valor almacene un QMap. Este QMap sería una lista de los eventos (metadatos), indexada por fecha.

`QHash < file_ID, QMap<fecha, evento> >`

Con esto se conseguirá que por cada fichero haya una lista de todos sus cambios ordenados cronológicamente.

La mecánica será recorrer todos esos cambios ordenados por fecha para ir procesando por cada fichero sus cambios uno a uno, del más antiguo al más reciente. Según los metadatos de cada evento, debemos ir asignando las 'Flags' en función de qué clase de evento se trate: subir o crear es 'New', renombrar o mover es 'Moved', etc.

###downloadFile(QString remotePath, QString localPath)
Esta función descarga desde la nube cada archivo que Clowds considere que debe descargar al sincronizar.

Aquí sólo debes preocuparte de introducir la petición HTTP que requiera el sistema que estemos implementando para hacer la descarga de un sólo archivo.

Hay que tener en cuenta que cada vez que Clowds ejecute la función, pasará por parámetro las variables remotePath y localPath. Generalmente son iguales, salvo que el sistema con el que estemos trabajando cifre los directorios. En ese caso, remotePath estará cifrada. El único momento en el que se ha de utilizar localPath, es cuando le indicamos la ruta de descarga.
Para descargar el archivo:
resources_->shrGet(QUrl, QFile);
		QUrl - URL de la petición.
		QFile - El archivo que se descargará en la carpeta de sincronización.
Dependiendo del sistema con el que estemos trabajando, es posible que se requieran cabeceras, o parámetros en la URL.

###uploadFile(QString remotePath, QString localPath)
Esta función se encarga de subir los archivos que Clowds detecte como 'new'  en la carpeta de sincronización. Al sincronizar Clowds, se encargará de subir a la nube los archivos creados en local.

Recordemos que cada vez que Clowds ejecute la función, pasará por parámetro las variables remotePath y localPath. Generalmente son iguales, salvo que el sistema con el que estemos trabajando cifre los directorios. En ese caso, remotePath estará cifrada. El único momento en el que se ha de utilizar localPath, es cuando le indicamos la ruta donde está el fichero local que se desea subir.

Normalmente, los archivos se suben mediante Multipart-POST.
resources->shrPost(QUrl, QMap, QFile);
QUrl - URL de la petición HTTP POST para subir un fichero.
QMap - Parámetros que pueda requerir la petición HTTP, normalmente datos sobre el fichero que va a ser subido.
QFile - Archivo a subir.

Es posible que alguna nube requiera el uso del verbo PUT para subir archivos, en vez de POST. La implementación que Clowds suministra es
`resources->shrPut(QUrl url, QFile *localFile, qint64 chunkIndex = -1, qint64 chunkSize = 10485760);`
###moveFile(QString from, QString to)

Mueve un archivo en la nube que había sido movido en la carpeta de sincronización.
Parámetros:

* from - ruta local anterior del archivo.
* to - ruta local nueva del archivo.

Hacer la petición HTTP correspondiente. Si no hay una petición exclusiva para mover un archivo, normalmente se hace actualizando la información del archivo, cambiando su padre (posiblemente el ID del padre).

* GET → `resources->shrGet();`
* POST → `resources->shrPost();`
* Otro verbo → `resources→shrSendCustomRequest();`

Se podría guardar el resultado de esta petición en un QByteArray para confirmar que se ha movido el archivo:

`QByteArray result = resources->shrSendCustomRequest();`

En caso de que se haya realizado con éxito devolver true; en otro caso, false.

###moveDir(QString from, QString to)

Mueve un directorio en la nube que había sido movido en la carpeta de sincronización.
Parámetros:

* from - ruta local anterior del directorio.
* to - ruta local nueva del directorio.

Hacer la petición HTTP correspondiente. Si no hay una petición exclusiva para mover un directorio, normalmente se hace actualizando la información del archivo, cambiando su padre (posiblemente el ID del padre).

* GET → `resources->shrGet();`
* POST → `resources->shrPost();`
* Otro verbo → `resources→shrSendCustomRequest();`

Nos puede interesar guardar el resultado de esta petición en un QByteArray para confirmar que se ha movido el directorio:
`QByteArray result = resources->shrSendCustomRequest();`

En caso de que se haya realizado con éxito devolver true; en otro caso, false.

###rmFile(QString path)

Borra un archivo en la nube que ha sido borrado en la carpeta de sincronización. La mayoría de servicios por defecto lo envían a la papelera. Siempre que tengamos la posibilidad, se recomienda usar la papelera para eliminar archivos, en vez de borrarlos directamente, para no poner en riesgo los archivos importantes de los usuarios.
Parámetros:

* path - La ruta del archivo local que ha sido eliminado.

Hacer la petición HTTP correspondiente, usando una de estas funciones, según el caso:

* GET → resources->shrGet();
* POST → resources->shrPost();
* Otro verbo → resources->shrSendCustomRequest();

Generalmente, se usa el verbo DELETE para borrar archivos. Por ejemplo:

`resources_->shrSendCustomRequest(“DELETE”, 	QUrl(http://www.example.org/endpoint));`

Nos puede interesar guardar el resultado de esta petición en un QByteArray para confirmar que se ha borrado el archivo:

`QByteArray result = resources->shrSendCustomRequest();`

En caso de que se haya realizado con éxito devolver true; en otro caso, false.

Es probable que necesites información del archivo a borrar, la cual podrás obtener a través de una estructura de datos propia, como `filesByPath.value(path)` (Ver ANEXO 2).

###mkDir(QString path)

Crea los directorios nuevos detectados por Clowds en la nube.
Parámetros:

path - La ruta del directorio local que ha sido creado.
NOTA: si el servicio cifra los ficheros, esta ruta estará cifrada.

Hacer la petición HTTP correspondiente, usando una de estas funciones, según el caso:

* GET → resources->shrGet();
* POST → resources->shrPost();
* Otro verbo → resources->shrSendCustomRequest();

###rmDir(QString path)

Borra un directorio en la nube que ha sido borrado en la carpeta de sincronización, la mayoría de servicios por defecto lo envían a la papelera.

Parámetros:
path - La ruta del directorio local que ha sido eliminado.

Hacer la petición HTTP correspondiente, usando una de estas funciones, según el caso:

* GET → resources->shrGet();
* POST → resources->shrPost();
* Otro verbo → resources->shrSendCustomRequest();

Generalmente, se usa el verbo DELETE para borrar archivos. Por ejemplo:

`resources_->shrSendCustomRequest(“DELETE”, 
	QUrl(http://www.example.org/endpoint));`

Nos puede interesar guardar el resultado de esta petición en un QByteArray para confirmar que se ha borrado el directorio:

`QByteArray result = resources->shrSendCustomRequest();`

En caso de que se haya realizado con éxito devolver true; en otro caso, false.

Es probable que necesites información del directorio a borrar, la cual podrás obtener a través de una estructura de datos propia, como `filesByPath.value(path)` (Ver ANEXO 2).



