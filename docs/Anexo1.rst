Anexo 1. Clases recomendables
=============================

*Nada de lo descrito en este Anexo es obligatorio para el funcionamiento
de un Plug-in para Clowds. Son recomendaciones para facilitar su
desarrollo.*

Clase JsonData
--------------

Esta clase sirve para tratar objetos QJsonObject.

Clase File
----------

Esta clase descompondrá los metadatos, y guardará los datos clave de
cada archivo -tipo de fichero, nombre, ID, directorio padre, etc.- de
forma que podamos trabajar con ellos fácilmente.

Normalmente los servicios dan los metadatos en formato JSON, de manera
que esta clase debe heredar de JsonData. En caso de que los metadatos
estén en otro formato, habría que hacer una clase similar a JsonData
para tratar ese formato, por ejemplo ``XmlData``.

Estructura de ejemplo:

-  ``QString id`` - ID del fichero.
-  ``QString path`` - Ruta completa del fichero, si es un directorio
   debe acabar en ‘/’.
-  ``bool isFolder`` - Booleano que indica si el fichero es un
   directorio. True es un directorio, false es un archivo.
-  ``QString parentId`` - ID del directorio padre del fichero.
-  ``QString title`` - Nombre del fichero.

Todos estos campos se rellenan en un constructor al que se le pasa un
QJsonObject, en el caso de que los metadatos sean en formato JSON.

Clase MyO2
----------

Esta clase hereda de O2.

Se encarga de realizar la autenticación y autorización mediante OAuth2,
si el servicio lo requiriera. En algunos servicios, como por ejemplo
“Bitcasa”, usa aspectos de OAuth1 y de OAuth2.

Clase Change
------------

Esta clase hereda de JsonData.

Debe guardar los metadatos de cada evento y, aparte, la fecha del propio
evento, para poder ordenar los cronológicamente. La forma de hacerlo
depende del servicio que estemos implementando, por que hay algunos que
ya los dan ordenados.

Estructura:

-  ``File file`` - Objeto ‘File’ del fichero afectado del evento.
-  ``QDateTime date`` - Fecha del evento.
