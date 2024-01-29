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
4. Usamos
   ```bash
   docker ps
   ```
   Nos debería dar un resultado como el de ![Terminal con los contenedores corriendo](https://github.com/andresalmeida/BD_EXA_P2/blob/main/Imgs_Readme/docker%20ps.png)


### Database

No hay necesidad de instalación adicional para la carpeta `database`.

### Frontend

1. Abre una terminal en la carpeta `frontend`.
2. Ejecuta `npx create-vite@latest .` para inicializar un proyecto Vite.
3. Luego, ejecuta `npm install` para instalar las dependencias de React y Vite.
4. Ejecuta `npm install axios` para instalar la dependencia Axios que se utilizará en el frontend.
5. Ejecuta `npm run dev` para iniciar la aplicación de React con Vite.

### Carpeta del Proyecto

1. Abre una terminal en la carpeta del proyecto.

## Uso

1. Accede a la aplicación en tu navegador usando la URL proporcionada por Vite, generalmente `http://localhost:3000`.
2. En la pantalla de bienvenida, ingresa el nombre del maestro y haz clic en "Enter".
3. Si es la primera vez que ingresaste, serás redirigido a la página de creación de evaluación. De lo contrario, verás la página con las opciones de "Crear Nueva Evaluación" y "Mis Evaluaciones".
4. Para crear una nueva evaluación, ingresa el título, las preguntas y las opciones de respuesta. Luego, haz clic en "Create Evaluation".
5. Para ver las evaluaciones creadas, haz clic en "Mis Evaluaciones".

## Estructura del Proyecto

- `backend`: Contiene el servidor Express y las rutas para manejar las solicitudes.
- `database`: Almacena archivos de datos simulados para perros, adoptantes y adopciones.
- `frontend`: La interfaz de usuario de la aplicación React construida con Vite.

## Tecnologías Utilizadas

- Frontend: React con Vite.
- Backend: Node.js con Express.
- Base de Datos: Datos simulados almacenados en archivos JSON.

## Contribuciones

Las contribuciones son bienvenidas. Si encuentras errores o mejoras posibles, ¡no dudes en abrir un problema o enviar una solicitud de extracción!

## Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo `LICENSE` para obtener más detalles.
