# BD_EXA_P2

## INSTALACIÓN Y USO MYSQL WORKBENCH (UBUNTU)

La aplicación desde Ubuntu se debe instalar de la siguiente forma:

1. Instalamos MySQL desde la línea de comandos:
   
   ```bash
   sudo apt update
   sudo apt install mysql-workbench
   ```
2. Luego, instalamos nuestra herramienta
   ```bash
   sudo apt update
   sudo apt install freetds-bin freetds-dev tdsodbc
   ```
3. Después, ingresaremos a esta configuración:
   ```bash
   sudo nano /etc/freetds/freetds.conf
   ```
4. Añadimos esta sección y la actualizamos con nuestra información:
   ```bash
   [mssql_server]
    host = your_sql_server_address
    port = your_sql_server_port
    tds version = 7.4  # or the version supported by your SQL Server
   ```
5. Ahora, corremos esto, con la dirección IP de la otra máquina, el usuario, la contraseña y nos conectaremos a SQL Server:
   ```bash
   tsql -S mssql_server -U your_sql_username -P your_sql_password
   ```
6. Para realizar consultas:
   ```bash
   1> Use Spanish;
   2> GO
   1> Select * from DimChannel;
   2> GO
   ```
7. Para realizar cambios:
   ```bash
   1> update [DimChannel] set [channeldescription] = 'Proveedor' where [ChannelLabel] = '07';
   2> GO

## Instalación y uso MongoDB

1. Instalamos MongoDB Community desde [MongoDB Community Server Download](https://www.mongodb.com/try/download/community).
2. Instalamos MongoDB Shell desde [MongoDB Shell Download](https://www.mongodb.com/try/download/shell).
3. Por último, y en caso de no tenerlo, instalamos [Docker Desktop](https://www.docker.com/products/docker-desktop/).

### Replicación de MongoDB con Docker

1. Seguimos el proceso de [Deploying a MongoDB Cluster with Docker](https://www.mongodb.com/compatibility/deploying-a-mongodb-cluster-with-docker).
2. Creamos una Red de Docker
   ```Mongsh
   docker network create mongoCluster
   ```
3. Inicializamos las instancias de MongoDB para nuestros nodos, empezando con el nodo principal, y después, los secundarios:
   
   ```Mongsh
   docker run -d -p 27017:27017 --name mongo1 --network mongoCluster mongo:5 mongod --replSet myReplicaSet --bind_ip localhost,mongo1

   docker run -d -p 27018:27017 --name mongo2 --network mongoCluster mongo:5 mongod --replSet myReplicaSet --bind_ip localhost,mongo2
 
   docker run -d -p 27019:27017 --name mongo3 --network mongoCluster mongo:5 mongod --replSet myReplicaSet --bind_ip localhost,mongo3
   ```
4. Usamos el siguiente script:
   ```bash
   docker ps
   ```
   Nos debería dar un resultado como el de ![Terminal con los contenedores corriendo](https://github.com/andresalmeida/BD_EXA_P2/blob/main/Imgs_Readme/docker%20ps.png)
   
5. Y en Docker Desktop tendríamos:
   
![Docker Desktop](https://github.com/andresalmeida/BD_EXA_P2/blob/main/Imgs_Readme/Docker%20con%20Contenedores.png)

6. Ahora, iniciamos el Replica Set:
   
   ```Mongsh
   docker exec -it <nombre_contenedor> mongosh --eval "rs.initiate({
    _id: 'myReplicaSet',
    members: [
      {_id: 0, host: 'mongo1'},
      {_id: 1, host: 'mongo2'},
      {_id: 2, host: 'mongo3'}
    ]
   })"
   ```
7. Si es que el proceso de replicación fue exitoso, deberíamos obtener el siguiente mensaje:
   ```Mongsh
   { ok: 1 }
   ```
8. Para verificar la replicación y como funcionan nuestros contenedores, corremos el siguiente script:
   ```Mongsh
   docker exec -it mongo1 mongosh --eval "rs.status()"
   ```
   Obtendríamos esto: ![Contenedores Replica](https://github.com/andresalmeida/BD_EXA_P2/blob/main/Imgs_Readme/Contenedores%20Replica.png)
   
9. Con eso configurado, importante que debemos tener instalado [Studio 3T](https://studio3t.com/es/download/)
10. 

## Tecnologías Utilizadas

- MySQL
- MySQL Workbench
- Docker Desktop
- MongoDB Community
- MongoDB Shell
- Studio 3T

## Contribuciones

Las contribuciones son bienvenidas. Si encuentras errores o mejoras posibles, ¡no dudes en abrir un problema o enviar una solicitud de extracción!

## Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo `LICENSE` para obtener más detalles.
