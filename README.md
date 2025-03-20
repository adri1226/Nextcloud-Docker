# Nextcloud-Docker
How to make a nexcloud server in local using docker in windows.

## Prerequisites
1. Have [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) activated in windows.
2. Have installed [docker desktop](https://www.docker.com/products/docker-desktop/) in windows.

## 0 - Make the nexcloud server
We use [docker compose](https://docs.docker.com/compose/) to create our nexcloud server because we need two containers, one is our nextcloud container and the other is a database container.<br>

## 1 - Data base container
First, we configure the data base container in the docker-compose.yml file.

```yml
dbName:
    image: mariadb:latest
    container_name: dbContainer
    restart: always
    environment:
        MYSQL_ROOT_PASSWORD: rootpassword
        MYSQL_DATABASE: nextcloud
        MYSQL_USER: nextclouduser
        MYSQL_PASSWORD: nextcloudpass
    volumes:
        - ./db:/var/lib/mysql
```

### 1.1 - Restart policy
We use the restart policy in always because we want that the server to restart automatically if its showt down for any reason.<br>
You can learn more about the restart policy [here](https://github.com/compose-spec/compose-spec/blob/main/deploy.md#restart_policy).

### 1.2 - Volumes
For the database, we are not using an external volume, which means that all the database files will be created in the same path where the docker-compose.yml file is located in the *db* folder.<br>
This is for better control of where our database files are located. If we do not use this volume, the files will be created inside the container and will be more difficult to access.

### 1.3 - Enviroment
We must to configurate some envriomental variables for MariaDB correct working.<br>
These enviromental variables are defined in his [official site](https://mariadb.com/kb/en/mariadb-server-docker-official-image-environment-variables/).

