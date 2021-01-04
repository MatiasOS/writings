# DIDI-SSI-Mouro
DIDI SSI Mouro es un fork de uPort Trust Graph Server (aka EdgeServer). Es responsabl de almacenar los datos y la información del usuario.


Mouro es un gestor de base de datos, el cual se comunica con una base de datos ‘sqlite ’
para almacenar las credenciales emitidas por el didi-server y los distintos issuers, permite el
acceso a los usuarios de didi a sus credenciales y verificar el estado de los mismos.

Otra dato interesente, es que se crea un archivo de DB SQLite por cada did.
## Llamadas

### Types
Las query cuentan con 2 tipos. 
#### Edge
Representa una credencial emitida.

```
type Edge {
  hash: ID!
  jwt: String!
  from: Identity!
  to: Identity!
  type: String
  time: Int!
  tag: String
  visibility: VisibilityEnum
  retention: Int
  data: String
}
```

#### Identity
Contiene un identificador decentralizado.
```
type Identity {
  did: String!
}
```
### Headers
las llamadas requieren de la generación de un token vacío firmado porel DID del usuario que realiza la llamada, este se ingresa en la sección “HTTPHEADERS” y tiene el siguiente formato:
```json
{  
  "Authorization": "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NkstUiJ9.eyJpYXQiOjE1ODAyMjY0NzEsImV4cCI6MTU4MDIyNjk3MS4xMTIsImlzcyI6ImRpZDpldGhyOjB4REZBNTE4Y2VhRWQxYmZlNmY3MDRFNTFBMThkNGJCMEExNDcxNGNkMiJ9.NdihoBy4uEMsCLaitRIETQ-fnB2SGJLyxfApked_42T9lZHJkIcGRQmxweOaer-UuG3A4R7-LOQYN76MivlQCgA" 
}
```

### Query
#### me
Retorna la info del usuario a partir del token:

`me: Identity`

Ejemplo:
```
query {
  me() {
    did
  }
}
```
#### hash
Dado un DID, retorna el hash de recuperación desde swarm

`hash(did: String): String`

Ejemplo:
```
query {
  hash(did: “did:ethr:...”)
}
```

Si el user que relealiza la llamada es DIDI-SERVER. Este puede obtener cuaquier hash, en cambio, un user solo puede obtener su propio hash de swarm.

#### edgeByJwt
Obtiene un certificado a partir de su jwt y did.  
`edgeByJwt(edgeJWT: String, did: String): Edge`

Ejemplo:
```
query {
  edgeByJwt(edgeJWT: “...”, did: “did:ethr:....”) {
  jwt,
  hash
  }
}
```
Si el user que relealiza la llamada es DIDI-SERVER. Este puede obtener cuaquier Edge(credencial), en cambio, un user solo puede obtener su propio Edge.

#### findEdges
Retorna los certificados emitidos a toDID.

`findEdges(toDID: String, type: [String], since: Int tag: [String], revoked: Int = 0 ): [Edge!]!`

Además, por cada Edge realiza una comparación en la que el toDID debe coincidir con el user del token de autorización o el DIDI-Server.
### Mutations
#### addEdge
Guarda un nuevo certificado en el archivo de SQLite perteneciente al did del parametro. El único autorizado a realizar esta accion es DIDI-Server. Es decir que el mismo usuario no puede guardar una credencial emiitda hacia su did.

`addEdge(edgeJWT: String, did: String):Edge`
Ejemplo:
```
mutation {
  addEdge(edgeJWT: “...”, did: “did:ethr:....”) {
    jwt,
    hash
  }
}
```
#### removeEdge
Revoca un jwt previamente emitido. Solo funciona si el did se corresponde con el del dueño del token o el DIDI-Server.

`removeEdge(hash: String, did: String):String`