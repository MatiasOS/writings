# DIDI SSI Issuer module - Backend

El issuer es una herramienta que permite generar y emitir credenciles verificables. Cuando el Issuer se da de alta con DIDI Server, las credenciales se almacenan en Mouro, y envidas a la app AIDI del DID correspondiente.

El backend se encuentra desarrollando en NodeJS, y  ExpressJs. Además, almacena sus datos en MongoDB.

Dentro del código, se encuentra el termino certificado, jwt que hacen referencia a las credenciales verificables o VC.

## Rutas

### /user
Este conjunto de rutas se crean, modifican y eliminan usuarios y administradores.
Dentro de estas rutas se encuentra el endpoint securaizado para crear el primer admin.

### /participant
Se conoce como participantes al los dids que luego se utilizan como input en los desplegables al emitir un credencial.


### /template
Se leen, crean, modifican y eliminan templates de credenciales.


### /cert
Estas rutas permites crear, modificar, elimnar u obtener credenciales.

### /delegate
TBA. Esperando implementación multiblockchain.

Coorinda la delegaciones, este servicio genera la tx a la blockchain correspondiente, y luego almacena en DB local la delegación.

Este servicio tiene un bug, que no reconoce las delegaciones realizdas por fuera de la app. Es decir, si se realiza una delegcion en blockchain, no queda registrada en la app.

### /register
Crea, edita y elimano un nuevo registro donde se almacena Nombre, did, y clave privada. 

### /default
Se utiliza para crear los valores por defecto de el template de credencial y la blockchain de verificación. 

### /profile
Crud de perfiles. Cada perfil es un conjunto de permisos para leer, crear/modificar y eliminar en un módulo.

## Servicios

Los servicios se encargan de coordinar una acción y llevar a cabo una acción dentro del sistema. Para
### Blockchain services
TBA. Esperando implementación multiblockchain.

### Cert Service
Este servcio se encrga de crear credenciales a partir de templates para luego almacenarlos en DB local. 
También modificar y eliminar credenciales

### Mouro service
Este servicio no es lo que su nombre indica, debe ser refactorizado.

Se encarga de crearcredenciles verificables, para luego enviarlas a DIDI Server, o ser elimnadas en DIDI Server.

Otra funcionalidad con la que cuenta es enviar shareRequest a DIDI-Server. [Explicar que es un shareRequest y para que sirve]

### Delegate service
Se encarga de almacenar en DB local las delegaciones. El impacto real de las delegaiones se realiza en blockchain. 

A este servicio, lo debería llamar otro servicio que se ecuentre escuchando eventos en las distintas blockchain.

### Default service
Se encarga de estables la blockchain y el template por defecto. 

### Token service
Este servicio es el más simple del Issuer. Se encarga de crear y verificar tokens vacios, firmdos por el Issuer.

### User service
Se encarga de crear, editar y elimnar usuarios. También se encrga de realizar el login

### Participant service
Los participantes son todos los dids sobre los qu se tiene información en el sistema. Este servico provee la funcionlidad para crear, edita y modificar. Además, para obtener listas de participntes por requestCode y templateId.
