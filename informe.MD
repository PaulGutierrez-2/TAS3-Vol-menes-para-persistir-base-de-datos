
# TAS3 - Volúmenes para persistir base de datos
## 1. Título
**Persistencia de Datos en Contenedores Docker: Uso de Volúmenes en PostgreSQL**

## 2. Tiempo de duración
50 minutos

## 3. Fundamentos
La virtualización ligera mediante contenedores, popularizada por Docker, ha revolucionado el desarrollo y despliegue de software. Sin embargo, los contenedores son efímeros, significa que cualquier datos escrito dentro del contenedor se pierde cuando el contenedor es detenido y eliminado.

### Volúmenes de Docker
Un Volumen es el mecanismo preferido por Docker para preservar los datos generados por los contenedores. los volúmenes son gestionados completamente por Docker. Docker crea y gestiona la estructura de directorios en el sistema operativo del host (`/var/lib/docker/volumes/` ) y mapea este directorio a un punto de montaje dentro del contenedor.

#### Características Clave:
- **Persistencia**: Los datos en el volumen persisten incluso después de que el contenedor asociado se detiene o se elimina.
- **Aislamiento**: Permiten la compartición segura de datos entre contenedores sin exponer directamente el sistema de archivos del host.
- **Gestión de I/O**: Son más eficientes en operaciones de entrada/salida (I/O) que otros mecanismos de almacenamiento.

### Funcionamiento en PostgreSQL
La imagen oficial de PostgreSQL está configurada para almacenar todos los datos de la base de datos en el directorio interno del contenedor: `/var/lib/postgresql/data`.

- **Sin Volumen**: Si no se especifica un volumen, los datos se escriben en este directorio interno. Al eliminar el contenedor, se elimina el directorio y, por lo tanto, los datos (Parte 1 de la práctica).
- **Con Volumen**: Cuando se utiliza la opción `-v nombre_volumen:/var/lib/postgresql/data`, Docker monta el volumen externo en ese directorio. Los archivos de la base de datos se escriben en el volumen, garantizando su persistencia.

## 5. Objetivos a alcanzar
- Demostrar el comportamiento efímero de los datos en contenedores Docker por defecto.
- Implementar y manipular Volúmenes de Docker para lograr la persistencia de datos.
- Verificar la persistencia de los datos de una base de datos PostgreSQL después de la eliminación del contenedor.
- Manejar conflictos de puertos y volúmenes al desplegar múltiples contenedores de base de datos.

## 6. Equipo necesario
- Docker Desktop instalado y en ejecución.
- Terminal con soporte para PowerShell.
- Un Administrador de Bases de Datos (DataGrip, TablePlus, DBeaver, pgAdmin, etc.).

## 7. Material de apoyo
- Documentación oficial de Docker sobre Volúmenes.
- Guía de sintaxis para comandos de PowerShell.
- Documentación de la imagen oficial de Docker PostgreSQL.

## 8. Procedimiento

### Parte 1: Base de datos sin volumen (Demostración de efimeridad)

#### Paso 1: Creación del Contenedor (Sin Volumen)
Se ejecuta el contenedor PostgreSQL, mapeando el puerto 5432 del host al 5433 del contenedor  y estableciendo la contraseña.

```powershell
docker run --name server_db1 -e POSTGRES_PASSWORD=1234 -p 5433:5432 -d postgres
```

![Figura 8-1.](img/Screenshot%202025-10-25%20203051.png)

#### Paso 2: Conexión y Creación de Datos
Se utiliza el administrador de base de datos para conectarse a localhost:5432.

- **Host**: localhost
- **Port**: 5432
- **User**: postgres
- **Password**: 1234

![Figura 8-2.](img/Screenshot%202025-10-25%20203357.png)

Se ejecuta el siguiente SQL:

```sql
CREATE DATABASE test;
\c test;
CREATE TABLE customer (id INT PRIMARY KEY, fullname VARCHAR(100), status BOOLEAN);
INSERT INTO customer (id, fullname, status) VALUES (1, 'Alice Johnson', TRUE);
```

![Figura 8-3.](img/Screenshot%202025-10-25%20210805.png)

#### Paso 3: Eliminación del Contenedor
Se detiene y se elimina el contenedor. Los datos se borran junto con el contenedor.

```powershell
docker stop server_db1
docker rm server_db1
```

![Figura 8-1.](img/Screenshot%202025-10-25%20211022.png)

#### Paso 4: Verificación de Pérdida de Datos
Se vuelve a crear el contenedor con el mismo nombre y configuración:

```powershell
docker run --name server_db1 -e POSTGRES_PASSWORD=1234 -p 5432:5432 -d postgres
```

Al conectar el administrador de base de datos, se verifica que la base de datos test y todos los datos asociados no existen. La base de datos está en su estado inicial.

![Figura 8-1.](img/Screenshot%202025-10-25%20211526.png)

### Parte 2: Base de datos con volumen (Persistencia de datos)

#### Paso 5: Creación de Contenedor (Con Volumen)
Se utiliza el volumen pgdata (que Docker creará si no existe) y se usa el puerto 5434 para evitar conflictos con el puerto 5432 que podría estar ocupando el host o el contenedor anterior (aunque se eliminó, se usa como buena práctica).

```powershell
docker run --name server_db2 `
  -e POSTGRES_PASSWORD=1234 `
  -p 5434:5432 `
  -v pgdata:/var/lib/postgresql `
  -d postgres
```

![Figura 8-1.](img/Screenshot%202025-10-25%20211642.png)

#### Paso 6: Conexión y Creación de Datos
Se conecta el administrador de bases de datos, esta vez usando el puerto 5434.

- **Host**: localhost
- **Port**: 5434
- **User**: postgres
- **Password**: 1234

Se repite la creación de la base de datos y la inserción de datos:

```sql
CREATE DATABASE test;
\c test;
CREATE TABLE customer (id INT PRIMARY KEY, fullname VARCHAR(100), status BOOLEAN);
INSERT INTO customer (id, fullname, status) VALUES (1, 'Maximus Decimus', TRUE);
```

![Figura 8-1.](img/Screenshot%202025-10-25%20212901.png)

#### Paso 7: Eliminación del Contenedor (Persistencia)
Se detiene y elimina el contenedor server_db2. El volumen pgdata se mantiene intacto.

```powershell
docker stop server_db2
docker rm server_db2
```

#### Paso 8: Verificación de Persistencia de Datos
Se vuelve a crear el contenedor con el mismo nombre y el mismo volumen (pgdata):

```powershell
docker run --name server_db2 `
  -e POSTGRES_PASSWORD=1234 `
  -p 5434:5432 `
  -v pgdata:/var/lib/postgresql `
  -d postgres
```

![Figura 8-1.](img/Screenshot%202025-10-25%20213014.png)

Al conectarse a localhost:5434, se verifica que la base de datos test, la tabla customer y el registro con ID 1 SÍ persisten.

![Figura 8-1.](img/Screenshot%202025-10-25%20213235.png)

## 9. Resultados esperados
La práctica ha demostrado satisfactoriamente el mecanismo de persistencia de datos en Docker.

- **Parte 1 (Sin Volumen)**: Se verificó que al detener y eliminar el contenedor server_db1, todos los datos creados (Base de datos test) se perdieron, y el contenedor recreado inició desde una base de datos vacía.
- **Parte 2 (Con Volumen)**: Se verificó que al usar el volumen pgdata y al recrear el contenedor server_db2 (usando el puerto 5434), la base de datos test y todos los registros internos se recuperaron sin problemas, confirmando la persistencia de los datos.

Se puede observar la base de datos test y el registro insertado (Maria Lopez) después de la recreación del contenedor.

![Figura 8-1.](img/Screenshot%202025-10-25%20213235.png)

## 10. Bibliografía
- Docker. (2024). Manage data in Docker. Obtenido de https://docs.docker.com/storage/
- Oracle. (2024). Comandos básicos de SQL para el manejo de datos. (Referencia para CREATE, INSERT).