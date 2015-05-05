Lista de Funciones
==================

getChanges()
~~~~~~~~~~~~

Esta es la función principal del Plug-in, la más importante, y
normalmente la que más código dedica. Su cometido es el de recibir
información de los sucesos de la nube, procesar esos eventos, y
traducirlos en cambios de Clowds (CloudStorageChange), para que el
programa sepa qué hacer con ellos. Devuelve un
``QHash<QString, QList<CloudStorageChange> >``

el cual contendrá un registro de los cambios que se van a realizar. La
clave será la ruta del fichero en ``Qstring``, y su valor un
``CloudStorageChange``, que es el cambio que ha sufrido.

El primer paso será hacer la petición HTTP para obtener la lista de los
eventos ocurridos en la nube e ir guardándolos en un QHash, por ejemplo
allEvents (ver `ANEXO 2`_). Lo importante es que el QHash almacene los
metadatos (normalmente en JSON).

En el caso de que la nube pueda devolver más de un cambio por archivo,
una buena manera de tratar esta lista de cambios sería hacer otro QHash,
como myChanges (Ver ANEXO 2), en el que en cada valor almacene un QMap.
Este QMap sería una lista de los eventos (metadatos), indexada por
fecha.

``QHash < file_ID, QMap<fecha, evento> >``

Con esto se conseguirá que por cada fichero haya una lista de todos sus
cambios ordenados cronológicamente.

La mecánica será recorrer todos esos cambios ordenados por fecha para ir
procesando por cada fichero sus cambios uno a uno, del más antiguo al
más reciente. Según los metadatos de cada evento, debemos ir asignando
las ‘Flags’ en función de qué clase de evento se trate: subir o crear es
‘New’, renombrar o mover es ‘Moved’, etc.

downloadFile(QString remotePath, QString localPath)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Esta función descarga desde la nube cada archivo que Clowds considere
que debe descargar al sincronizar.

Aquí sólo debes preocuparte de introducir la petición HTTP que requiera
el sistema que estemos implementando para hacer la descarga de un sólo
archivo.

Hay que tener en cuenta que cada vez que Clowds ejecute la función,
pasará por parámetro las variables remotePath y localPath. Generalmente
son iguales, salvo que el sistema con el que estemos trabajando cifre
los directorios. En ese caso, remotePath estará cifrada. El único
momento en el que se ha de utilizar localPath, es cuando le indicamos la
ruta de descarga. Para descargar el archivo: resources\_->shrGet(QUrl,
QFile); QUrl - URL de la petición. QFile - El archivo que se descargará
en la carpeta de sincronización. Dependiendo del sistema con el que
estemos trabajando, es posible que se requieran cabeceras, o parámetros
en la URL.

uploadFile(QString remotePath, QString localPath)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Esta función se encarga de subir los archivos que Clowds detecte como
‘new’ en la carpeta de sincronización. Al sincronizar Clowds, se
encargará de subir a la nube los archivos creados en local.

Recordemos que cada vez que Clowds ejecute la función, pasará por
parámetro las variables remotePath y localPath. Generalmente son
iguales, salvo que el sistema con el que estemos

.. _ANEXO 2: Anexo2
