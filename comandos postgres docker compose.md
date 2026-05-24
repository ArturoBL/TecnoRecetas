# Instalación de PostgreSQL en un contenedor de Docker con Docker Compose

## 1. Instalación de Docker Engine

```Bash

# Actualizar paquetes
sudo apt update && sudo apt upgrade -y

# Instalar dependencias
sudo apt install -y ca-certificates curl gnupg lsb-release

# Agregar la clave GPG oficial de Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Agregar el repositorio de Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker y Docker Compose
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 2. Agregar tu usuario al grupo docker (opcional pero recomendado)
```Bash
sudo usermod -aG docker $USER
newgrp docker
```

## 3. Crear la carpeta del proyecto
```Bash
mkdir ~/postgres-docker &&  cd ~/postgres-docker
```
## 4. Crear el archivo `docker-compose.yml`
```Bash
nano docker-compose.yml
```
### Contenido:
```Yaml
services:
  postgres:
    image: postgres:16
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin123
      POSTGRES_DB: mi_base
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin123
    ports:
      - "8080:80"
    depends_on:
      - postgres

volumes:
  postgres_data:
  ```
  
## 5. Levantar contenedores.
```Bash
docker compose up -d
```

## 6. Verificar que estén corriendo.
```Bash
docker compose ps
```

## 7. Acceder a pgadmin4.
Abrir en el navegador:  [http://localhost:8080](http://localhost:8080)

Inicia sesión con:

-   **Email:** `admin@admin.com`
-   **Password:** `admin123`

## 8. Conectar pgAdmin a PostgreSQL

Dentro de pgAdmin:

1.  Click derecho en **Servers** → **Register → Server**
2.  En la pestaña **General**, pon un nombre (ej. `Mi Postgres`)
3.  En la pestaña **Connection**:
    -   **Host:** `postgres` _(nombre del servicio en docker-compose)_
    -   **Port:** `5432`
    -   **Database:** `mi_base`
    -   **Username:** `admin`
    -   **Password:** `admin123`
4.  Click **Save**

## Comandos útiles de mantenimiento

```Bash
# Ver logs
docker compose logs -f

# Detener los servicios
docker compose down

# Detener y eliminar volúmenes (¡borra los datos!)
docker compose down -v

# Reiniciar
docker compose restart
```


## Para eliminar contenedor:
```Bash
cd ~/postgres-docker
docker compose down
```
Para **eliminar los datos** (volúmenes):
```Bash
docker compose down -v
```

## Para modificar la configuración:
```Bash
nano docker-compose.yml
```

```Bash
docker compose up -d
```


## Otros flujos:
|Lo que quieres hacer| Comando |
|--|--|
| Solo cambiar config, conservar datos | `docker compose down` → editar yml → `docker compose up -d` |
| Cambiar config y borrar datos | `docker compose down -v` → editar yml → `docker compose up -d` |
| Cambiar imagen/versión de Postgres | `docker compose down -v` → editar yml → `docker compose up -d` |
| Solo reiniciar sin cambios | docker compose restart |
