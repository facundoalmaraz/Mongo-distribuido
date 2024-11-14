# Sistema de Base de Datos Distribuida para una Universidad con MongoDB y Docker

Este proyecto implementa un sistema de base de datos distribuida utilizando MongoDB y Docker. La base de datos está configurada como un `replica set` con múltiples nodos, simulando distintos campus universitarios (Central, Norte, y Sur). La configuración asegura alta disponibilidad, replicación de datos y tolerancia a fallos, permitiendo que la universidad gestione estudiantes, cursos y profesores en una base de datos distribuida.

## Requisitos

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Estructura del Proyecto

- **campus_central**: Nodo principal (Primary).
- **campus_norte**: Nodo secundario.
- **campus_sur**: Nodo secundario.

## Configuración e Instalación

### Paso 1: Crear el Archivo `docker-compose.yml`

Crea un archivo `docker-compose.yml` en el directorio raíz del proyecto con el siguiente contenido:

```yaml
version: '3.8'
services:
  campus_central:
    image: mongo:latest
    container_name: campus_central
    ports:
      - "27017:27017"  # Puerto de campus central
    environment:
      - MONGO_INITDB_REPLICA_SET_NAME=rs0
    command: ["mongod", "--replSet", "rs0"]
    networks:
      - mongo-cluster

  campus_norte:
    image: mongo:latest
    container_name: campus_norte
    ports:
      - "27018:27017"  # Puerto de campus norte
    environment:
      - MONGO_INITDB_REPLICA_SET_NAME=rs0
    command: ["mongod", "--replSet", "rs0"]
    networks:
      - mongo-cluster

  campus_sur:
    image: mongo:latest
    container_name: campus_sur
    ports:
      - "27019:27017"  # Puerto de campus sur
    environment:
      - MONGO_INITDB_REPLICA_SET_NAME=rs0
    command: ["mongod", "--replSet", "rs0"]
    networks:
      - mongo-cluster

networks:
  mongo-cluster:
    driver: bridge
    
```

### Paso 2: Levantar los Contenedores
Ejecuta el siguiente comando para iniciar los contenedores de MongoDB:

```bash
    docker-compose up -d

```

### Paso 3: Configurar el Replica Set

```bash
    docker exec -it campus_central mongosh
```

En el shell de MongoDB, ejecuta el siguiente comando para iniciar el replica set y añadir los nodos:

```javascript
    rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "campus_central:27017" },
    { _id: 1, host: "campus_norte:27017" },
    { _id: 2, host: "campus_sur:27017" }
  ]
})

```

### Paso 4: Insertar Datos de Ejemplo
Con el replica set configurado, crea una base de datos de ejemplo y una colección para almacenar información de estudiantes:


```javascript
use UniversidadDB
db.Estudiantes.insertMany([
  { nombre: "Carlos Pérez", matricula: "U12345", carrera: "Ingeniería", campus: "Central" },
  { nombre: "Ana López", matricula: "U67890", carrera: "Medicina", campus: "Norte" },
  { nombre: "Luis Gomez", matricula: "U54321", carrera: "Derecho", campus: "Sur" }
])


```

### Paso 5: Verificar la Replicación

```bash
    docker exec -it campus_norte mongosh

```

Consulta la colección Estudiantes:

```javascript
use UniversidadDB
db.Estudiantes.find()

```
