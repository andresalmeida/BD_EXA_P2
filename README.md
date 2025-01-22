# BD_EXA_P2

## INSTALACIÓN Y USO MYSQL WORKBENCH (UBUNTU)

Tenemos el siguiente ![escenario](https://github.com/andresalmeida/BD_EXA_P2/blob/main/Imgs_Readme/Diagrama.png)

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
10. Ahora corremos el siguiente script, dependiendo del contenedor que tengamos y que sea el principal:
    ```Mongsh
    docker exec -it mongo1 mongo
    ```
11. Ahí, si es que logramos conectarnos satisfactoriamente al nodo principal, obtendremos los siguiente:
    ```Mongsh
    myReplicaSet:PRIMARY>
    ```
12. Ahí usaremos:
    ```Mongsh
    show databases # Para ver las bases de datos que tenemos después de nuestra migración de datos desde Studio 3T
    use Spanish # Para usar la base de datos que deseemos
    db.DimChannel.find().projection() # Para ver los campos e información de una colección en específico

### NOTA
Esta replicación requiere que, cada que haya un cambio en SQL Server, desde Studio 3T debemos correr la migración de datos para que las tablas se actualicen.

## Replicación x Python
1. Para esta replicación, deberemos, obviamente instalado Python. Lo verificamos en nuestro ordenador corriendo el siguiente script:

```python
python
```
Ahí tendremos la versión de Python que tenemos instalada.

2. Ahora instalaremos pyodbc:
```python
python -m pip install pyodbc
```

3. Luego instalaremos pymongo:
```python
python -m pip install pymongo
```

4. Usaremos lo siguiente, que es lo mismo que el código llamado fragmentación.py:
```python
import pyodbc
from pymongo import MongoClient
import decimal
from datetime import datetime, timedelta
import time

def convert_decimal(value):
    try:
        return int(value)
    except (ValueError, TypeError):
        try:
            return float(value)
        except (ValueError, TypeError):
            return value

def fetch_and_insert_data(sql_cursor, sql_query, mongo_collection, key_column, last_replication_collection):
    # Verificar la última vez que se replicó la tabla
    last_replication = last_replication_collection.find_one({"table": mongo_collection.name})
    if last_replication:
        print(f"La replicación para la tabla {mongo_collection.name} ya se realizó en {last_replication['timestamp']}.")
        return
    
    sql_cursor.execute(sql_query)
    data = sql_cursor.fetchall()

    # Eliminar solo el contenido de la colección
    mongo_collection.delete_many({})

    for row in data:
        document = {key[0]: convert_decimal(value) for key, value in zip(sql_cursor.description, row)}

        # Cambiar el nombre del campo "_id" a "id" y convertir su valor a int si está presente
        if "_id" in document:
            document["id"] = convert_decimal(document.pop("_id"))
            
        # Crear un nuevo _id como entero
        if key_column in document:
            document["_id"] = convert_decimal(document[key_column])
        
        mongo_collection.insert_one(document)

    # Registrar la última replicación para la tabla
    last_replication_collection.update_one(
        {"table": mongo_collection.name},
        {"$set": {"_id": mongo_collection.name, "timestamp": datetime.utcnow()}},
        upsert=True
    )

# Configuración de la conexión a SQL Server en el servidor remoto
sql_server_connection_string = "DRIVER={SQL Server};SERVER=192.168.1.17;DATABASE=Spanish;UID=sa;PWD=password"

# Configuración de la conexión a MongoDB
mongo_client = MongoClient("mongodb://localhost:27017")
mongo_db = mongo_client["Replicacion"]

# Crear colecciones para cada tabla
mongo_collection_DimAccount = mongo_db['DimAccount']
mongo_collection_DimChannel = mongo_db['DimChannel']
mongo_collection_DimCurrency = mongo_db['DimCurrency']
mongo_collection_DimCustomer = mongo_db['DimCustomer']
mongo_collection_DimEmployee = mongo_db['DimEmployee']
mongo_collection_DimEntity = mongo_db['DimEntity']
mongo_collection_DimProduct = mongo_db['DimProduct']
mongo_collection_DimProductCategory = mongo_db['DimProductCategory']
mongo_collection_DimStore = mongo_db['DimStore']

# Colección para almacenar la última vez que se replicó cada tabla
last_replication_collection = mongo_db['LastReplication']

# Bucle para ejecutar el proceso cada 7 segundos
while True:
    try:
        # Intentar la conexión a SQL Server
        with pyodbc.connect(sql_server_connection_string) as sql_server_connection:
            sql_cursor = sql_server_connection.cursor()

            # Consulta y replicación para cada tabla
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimAccount]", mongo_collection_DimAccount, "AccountKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimChannel]", mongo_collection_DimChannel, "ChannelKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimCurrency]", mongo_collection_DimCurrency, "CurrencyKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimCustomer]", mongo_collection_DimCustomer, "CustomerKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimEmployee]", mongo_collection_DimEmployee, "EmployeeKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimEntity]", mongo_collection_DimEntity, "EntityKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimProduct]", mongo_collection_DimProduct, "ProductKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimProductCategory]", mongo_collection_DimProductCategory, "ProductCategoryKey", last_replication_collection)
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimStore]", mongo_collection_DimStore, "StoreKey", last_replication_collection)

            print("Proceso ETL completado.")

    except Exception as e:
        print(f"Error en la conexión o proceso ETL: {e}")

    finally:
        # Eliminar entradas de LastReplication correspondientes a tablas que ya no existen
        existing_tables = set(mongo_db.list_collection_names())
        tables_to_remove = [table['_id'] for table in last_replication_collection.find() if table['_id'] not in existing_tables]
        last_replication_collection.delete_many({"_id": {"$in": tables_to_remove}})
        
        # Cerrar conexión a SQL Server
        sql_server_connection.close()

    # Esperar 7 segundos antes de la próxima ejecución
    time.sleep(1)

# Cerrar conexión a MongoDB fuera del bucle
mongo_client.close()
```
Aquí importante, deberemos cambiar el SERVER a la dirección IP en donde se encuentra nuestro SQL Server, el nombre de la BD, el ID de ingreso y la contraseña:
```python
# Configuración de la conexión a SQL Server en el servidor remoto
sql_server_connection_string = "DRIVER={SQL Server};SERVER=xxx.xxx.x.xx;DATABASE=X;UID=X;PWD=X"
```
Además, aquí Dim_Account y el resto, son el nombre de las tablas de nuestra base de datos, cambialas de acuerdo a tus necesidades:
```python
# Crear colecciones para cada tabla
mongo_collection_DimAccount = mongo_db['DimAccount']
mongo_collection_DimChannel = mongo_db['DimChannel']
mongo_collection_DimCurrency = mongo_db['DimCurrency']
mongo_collection_DimCustomer = mongo_db['DimCustomer']
mongo_collection_DimEmployee = mongo_db['DimEmployee']
mongo_collection_DimEntity = mongo_db['DimEntity']
mongo_collection_DimProduct = mongo_db['DimProduct']
mongo_collection_DimProductCategory = mongo_db['DimProductCategory']
mongo_collection_DimStore = mongo_db['DimStore']
```
Finalmente:
- [Spanish] es el nombre de la BD
- [dbo] es el nombre del schema
- [DimAccount] es el nombre exacto como se muestra en SQL Server
- "AccountKey" es la primary key de la tabla en específico
```python
 # Consulta y replicación para cada tabla
            fetch_and_insert_data(sql_cursor, "SELECT * FROM [Spanish].[dbo].[DimAccount]", mongo_collection_DimAccount, "AccountKey", last_replication_collection)
```
Corres al codigo desde la consola llamando al archivo y ya podrás ver los cambios en tu MongoDB, sugerimos usar MongoDB Compass para una vista más optimizada.

## Tecnologías Utilizadas

- MySQL
- MySQL Workbench
- Docker Desktop
- MongoDB Community
- MongoDB Shell
- Studio 3T
- Python

## Contribuciones

Las contribuciones son bienvenidas. Si encuentras errores o mejoras posibles, ¡no dudes en abrir un problema o enviar una solicitud de extracción!

## Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo `LICENSE` para obtener más detalles.
