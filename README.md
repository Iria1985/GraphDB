## GraphDB
### INSTALAR GRAPHDB
* Situamos el zip de graphdb donde queramos.
* Hacemos unzip.
* Entramos en el archivo de configuración graphdb.conf dentro de la carpeta conf y modificamos la variable" graphdb.home".
* El contenido de esta variable es el path de la carpeta principal unzipeada de graphdb.

### METER LICENCIA EN GRAPHDB
Copiamos el archivo de licencia dentro de la carpeta conf de nuestra base graphdb y automáticamente sin hacer ninguna otra gestión ya se activa sola.
<pre>
/users/graphdb/conf/PSA_TRAINING_W3_GRAPHDB_ENTERPRISE_expires-18-10-2019_latest-18-10-2019.license
</pre>
También indicar que esta licencia puede ser cargada fácilmente en la gestión web vía Workbench

### ARRANCAR EL DEMONIO DE GRAPHDB

#### Arranque normal:
<pre>
/users/graphdb/bin/graphdb -d
</pre>

*Arranque decidiendo el nivel de logs:*
<pre>
/users/graphdb/bin/graphdb -d -Dgraphdb.logger.root.level=WARM 
</pre>

### PARAR GRAPHDB
La única manera de parar GraphDB es realizar un ps -aux|grep graphdb y quedarnos con el número de PID al que le realizaremos un kill -9 PID

## BACKUP FULL DEL CLUSTER

### Example 1: con nombre por defecto

Comando:
<pre>
curl -u usuario:pass -H 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/repositoryId", "operation":"backup", "arguments":[null]}' http://masterNode:7200/jolokia/
</pre>
Ejemplo:
<pre>
curl -H -u usuario:contraseña 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/master1", "operation":"backup", "arguments":[null]}' http://yval48k0:7200/jolokia/
</pre>
### Example 2: dándole un nombre

Comando:
<pre>
curl -H  -u usuario:contraseña 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/repositoryId", "operation":"backup", "arguments":["20190501"]}' http://masterNode:7200/jolokia/
</pre>
Ejemplo:
```
DATE=$(date +%Y%m%d)

curl -H 'content-type: application/json' -d "{\"type\":\"exec\", \"mbean\":\"ReplicationCluster:name=ClusterInfo\/master1\", \"operation\":\"backup\", \"arguments\":[\"$DATE\"]}" http://yval48k0:7200/jolokia/
```
Los backups se ejecutan en el master principal y se encuentran en el path: /users/graphdb/data/repositories/master1/backup

## RESTORE CLUSTER BACKUP

Comando:
<pre>
curl -H 'content-type: application/json' -d '{"type":"exec","mbean":"ReplicationCluster:name=ClusterInfo\/repositoryId","operation":"restoreFromImage","arguments":["20190504-sunday"]}' http://masterNode:7200/jolokia/
</pre>
Ejemplo:
<pre>
curl -H 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/master1", "operation":"restoreFromImage", "arguments":["20190918"]}' http://yval48k0:7200/jolokia/
</pre>
## BACKUP INCREMENTAL

### Ejemplo 1: Creamos un full 

Comando:
<pre>
curl -H 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/repositoryId", "operation":"backup", "arguments":["20190504-sunday"]}' http://masterNode:7200/jolokia/
</pre>
Ejemplo:
```
DATE=$(date +%Y%m%d)

curl -H 'content-type: application/json' -d "{\"type\":\"exec\", \"mbean\":\"ReplicationCluster:name=ClusterInfo\/master1\", \"operation\":\"backup\", \"arguments\":[\"$DATE\"]}" http://yval48k0:7200/jolokia/
```
El nombre de este backup sería : 20190918

### Ejemplo 2: Creamos incrementales desde el ultimo full

Comando:
<pre>
curl -H 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/repositoryId", "operation":"incrementalBackup", "arguments":["20190504-sunday","20190505-monday", false]}' http://masterNode:7200/jolokia/
</pre>
Ejemplo:
```
DATE2=$(date +%Y%m%d%H%M%S)

curl -H 'content-type: application/json' -d "{\"type\":\"exec\", \"mbean\":\"ReplicationCluster:name=ClusterInfo\/master1\", \"operation\":\"incrementalBackup\", \"arguments\":[\$DATE\",\"$DATE2\", false]}" http://yval48k0:7200/jolokia/
```
## RESTAURACIÓN DE UN BACKUP INCREMENTAL

1. Restauración el último backup full

Comando:
<pre>
curl -H 'content-type: application/json' -d '{"type":"exec", "mbean":"ReplicationCluster:name=ClusterInfo\/master1", "operation":"restoreFromImage", "arguments":["20190918"]}' http://yval48k0:7200/jolokia/
</pre>
2. Restauración del incremental desde el último backup full
Imaginemos que el nombre de mi backup es "20190918" y el nombre de mi incremental es "20190919inc".

Comando:
<pre>
curl -H 'content-type: application/json' -d '{"type":"exec","mbean":"ReplicationCluster:name=ClusterInfo\/master1","operation":"restoreFromImage","arguments":["20190919inc"]}' http://yval48k0:7200/jolokia/
</pre>
## VERIFICACIÓN

Hacer la verificación desde el master hay dos posibilidades

### Vía web:
Es siempre sin autorización:
<pre>
http://yval48k0.inetpsa.com:7200/repositories/master1/health?new
</pre>
### Vía curl:
Es la que necesitamos añadiéndole usuario y pass:
<pre>
curl -u usuario:pass http://yval3zz0.inetpsa.com:7200/repositories/muz00master1/health?new
</pre>

## CREACIÓN DE USUARIOS
### CREAR UN USUARIO ADMINISTRADOR

Comando:
<pre>
curl -X POST -H 'Content-Type: application/json' -H 'X-GraphDB-Password: {password}' -d '{ "grantedAuthorities": ["ROLE_ADMIN"]  }' 'http://{graphdb-url}/rest/security/user/{username}'
</pre>
### CREAR UN USUARIO CON PERMISOS EN EL REPOSITORIO

Comando:
<pre>
curl -X POST -H 'Content-Type: application/json' -H 'X-GraphDB-Password: {password}' -d '{ "grantedAuthorities": ["ROLE_USER", "READ_REPO_{repository-id}", "WRITE_REPO_{repository-id}"]  }' 'http://{graphdb-url}/rest/security/user/{username}'
</pre>
## LISTAR USUARIOS CREADOS Y SUS PERMISOS

Comando:
<pre>
curl 'http://{graphdb-url}/rest/security/user'
</pre>
## CAMBIAR LA CONTRASEÑA DE UN USUARIO

Comando:
<pre>
curl -X PUT -H 'Content-Type: application/json' -H 'X-GraphDB-Password: {password}' -d '{}' 'http://{graphdb-url}/rest/security/user/{username}'
</pre>
## CAMBIAR LOS PERMISOS DE UN USUARIO

Comando:
<pre>
curl -X PUT -H 'Content-Type: application/json' -d '{ "grantedAuthorities":["ROLE_REPO_MANAGER"] }' 'http://{graphdb-url}/rest/security/user/{username}'
</pre> 
## BORRAR UN USUARIO

Comando:
<pre>
curl -X DELETE 'http://{graphdb-url}/rest/security/user/{username}'
</pre>

## VERIFICACIONES
### VERIFICAMOS EL TAMAÑO DE LA BASE DE DATOS
<pre>
curl 'http://{graphdb-url}/repositories/{repository-id}/size' 
</pre>
## IMPORTAR DATOS
Hay que crear un archivo con la información y luego se importa
### Extensión *.rdf:
Comando:
<pre>
curl -X POST -H 'Content-Type: application/rdf+xml' 'http://{graphdb-url}/repositories/{repository-id}/statements' -d @wine.rdf
<pre>
Hay que crear un *.ttl con la información y luego se importa
</pre>
### Extensión *.ttl:
Contenido del ttl:
```
PREFIX dc: <http://purl.org/dc/elements/1.1/>
INSERT DATA
      {
      GRAPH <http://example> {
          <http://example/book1> dc:title "A new book" ;
                                 dc:creator "A.N.Other" .
          }
      }
``` 
Comando:
<pre>
curl -X POST -H 'Content-Type: text/turtle' 'http://{graphdb-url}/repositories/{repository-id}/statements' -d @prueba.ttl
</pre>

### VERIFICAMOS QUE EL TAMAÑO DE LA BASE DE DATOS ES DIFERENTE

Comando:
<pre>
curl 'http://{graphdb-url}/repositories/{repository-id}/size' 
</pre>
## BORRAMOS TODA LA INFORMACIÓN DE LA BASE DE DATOS
El tamaño del repositorio debería ser 0

Comando:
<pre>
curl -X POST -H 'Content-Type: application/sparql-update' 'http://{graphdb-url}/repositories/{repository-id}/statements' -d '
</pre>

### LOGIN VÍA REST API

#### Adquirir el authorized token vía:

Comando:
<pre>
curl -i -X POST -H 'X-GraphDB-Password: {password}' 'http://{graphdb-url}:7200/rest/login/{username}'
</pre>
#### Login con el authorized token:

Comando:
<pre>
curl -H 'Authorization: GDB eyJ1c2VybmFtZSI6ImFkbWluIiwiYXV0aGVudGljYXRlZEF0IjoxNTY4OTcwNTA3MjI0fQ==.6laSIHBxsigy9M97trVakHzEnVjXXjOxSUzgvzSnBhU=' http://{graphdb-url}:7200/rest/security/user
</pre>
