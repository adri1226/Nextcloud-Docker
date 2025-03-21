# Nextcloud-Docker
How to make a nexcloud server in local using docker in windows.

## Prerequisites
1. Have [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) activated in windows.
2. Have installed [docker desktop](https://www.docker.com/products/docker-desktop/) in windows.

## 0 - Make the nexcloud server
We use [docker compose](https://docs.docker.com/compose/) to create our nexcloud server because we need two containers, one is our nextcloud container and the other is a database container.<br>

## 1 - Data base container configuration
First, we configure the data base container in the [`docker-compose.yml`](/docker-compose.yml) file.

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

### 1.2 - Enviroment
We need to configurate some envrioment variables for MariaDB correct working.<br>
These enviroment variables are defined in his [official site](https://mariadb.com/kb/en/mariadb-server-docker-official-image-environment-variables/).

### 1.3 - Volumes
We are not using an external volume, which means that all the database files will be created in the same path where the [`docker-compose.yml`](/docker-compose.yml) file is located in the *db* folder.<br>
This is for better control of where our database files are located. If we do not use this volume, the files will be created inside the container and will be more difficult to access.

## 2 - Nexcloud container configuration
Second, we configure the nexcloud container in the [`docker-compose.yml`](/docker-compose.yml) file.

```yml
app:
    image: nextcloud:latest
    container_name: nextcloud-app
    restart: unless-stopped
    ports:
        - "80:80"
        - "433:433"
    environment:
        NEXTCLOUD_ADMIN_USER: admin
        NEXTCLOUD_ADMIN_PASSWORD: adminpass
        NEXTCLOUD_TRUSTED_DOMAINS: localhost
        MYSQL_HOST: db
        MYSQL_DATABASE: nextcloud
        MYSQL_USER: nextclouduser
        MYSQL_PASSWORD: nextcloudpass
    volumes:
        - ./nextcloud/config:/var/www/html/config
        - ./nextcloud/data:/var/www/html/data
        - ./nextcloud/custom_apps:/var/www/html/custom_apps
        - ./nextcloud/themes:/var/www/html/themes
        - external_storage:/external
    depends_on:
        - db
```

### 2.1 - Restart policy
As explained in the [previous section](#11---restart-policy).

### 2.2 - Ports
To access the server, we need to map some ports between our PC and the Docker container.<br>
To make sure that the port we are mapping is not in use, we can use the command `Get-NetTCPConnection -LocalPort numberPort` (in Windows).<br>
We need to map the container's port 80 (for http) and 443 (for https) as these are the two ports we need to access our nextcloud server.

### 2.3 - Environment
We need to configurate some enviroment variables for Nextcloud server correct working.<br>
These enviroment variables are defined in [his official docker image](https://hub.docker.com/_/nextcloud).

### 2.4 - Volumes
For a better control of where our nextcloud files are located, we made different volumes (the folders are created in the same path where the [`docker-compose.yml`](/docker-compose.yml) file is located):
+ `./nextcloud/config`, stores Nextcloud’s main configuration file (`config.php`).
+ `./nextcloud/data`, holds user-uploaded files.
+ `./nextcloud/custom_apps`, allows installation and persistence of custom apps.
+ `./nextcloud/themes`, enables custom themes and branding.
+ `external_storage:/external`, manages external storage integration in Nextcloud. In this [section](#3---configure-docker-volumes-in-windows) it's explained how to configure a volume in windows.

If we do not use these volume, the files will be created inside the container and will be more difficult to access.

## 3 - Configure docker volumes in windows
To configure a volume in windows with docker compose the only field that is different is the device field, which you have to specify differently than in linux the path.<br>
In windows you have to use the paths as he calls them with back-slash.<br>

```yml
volumes:
  external_storage:
    driver: local
    driver_opts:
      type: none
      device: E:\
      o: bind
```

To learn more about Docker Volumes in Docker Compose, visit its [official website](https://docs.docker.com/reference/compose-file/volumes/).

## 4 - Nextcloud configuration
### 4.1 - How to configure HTTPS
The oficial guide for activate https is [here](https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html#enabling-ssl).<br>
```bash
a2enmod ssl
a2ensite default-ssl
service apache2 reload
```
If when restarting apache2 have a problem with `ssl-cert-snakeoil.pem` you have to
1. Reinstall the package ssl-cert:
    ```bash 
    apt install --reinstall ssl-cert
    ```
2. Regenerate the certificate:
    ```bash 
    make-ssl-cert generate-default-snakeoil
    ```
3. Restart apache2 again:
    ```bash 
    service apache2 reload
    ```
### 4.2 - How to configure an external volume
Precondition: Have added the volume in [`docker-compose.yml`](/docker-compose.yml) file.<br>
To add an external store to nextcloud follow the next steps in the nextcloud web page:
1. Enable *External storage support* app.
    1. Click in your profile.
    2. Click in *Apps*.
    3. Click in *Disabled apps*.
    4. Search *External storage support* and enable.
2. Configure the external device.
    1. Click in your profile.
    2. Click in *Personal settings*.
    3. In *Administration* secction, select *External storage*.
    4. In *Folder name* write the name you want for the folder in the nextcloud server.
    5. In *Add storage* select *Local*.
    6. In *Authentication* select *None*.
    7. In *Configuration* select the path of the volume mounted (written in the [`docker-compose.yml`](/docker-compose.yml) file, in the volume section of the nextcloud container, i.e `/external`).

### 4.3 - How to configure an ip to access into nextcloud server
To add an a IP to access into the nextcloud server have to modify the `config.php` file.<br>
This file is in the path `./nextcloud/config/config.php`<br>
In the `trusted_domain` section you have to add your pc ip address:
```php
  'trusted_domains' => 
  array (
    0 => 'localhost',
    1 => 'xxx.xxx.xxx.xxx'
  ),
```

## 5 - How to access our local Nextcloud server from outside the network
We can use the Tailscale app to create an VPN between our PC an other devices that we want and have access into the server.<br>
To obtain Tailscale app you can visit his [official website](https://tailscale.com/).<br>
And to have access we need to add the Tailscale ip address into the `trusted_domain` section in `config.php` (is explained in the [before section](#43---how-to-configure-an-ip-to-access-into-nextcloud-server)).

## 6 - Possible problemss
### 6.1 - Upgrade needed
If when you access to the Nextcloud server it says something like:
```
Upgrade needed
Please use the command line updater as upgrading via browser is disabled in your config.php.

For help, please refer to the documentation.
```

You need to get inside the nexcloud container to update it.<br>
To get inside the nexcloud container, we will use the command:
```bash
docker exec -ti containerName /bin/bash
```

And to update nextcloud we will use the following command:
```bash
php occ upgrade
```

To check that the update went well we can use the command:
```bash
php occ status
```
And we will see that it has been updated correctly.

Then we will have to restart the container.
```bash
docker container restart containerName
```

### 6.2 - Error when restart apache2
If you get this error when trying to restart apache2:
```bash
AH00111: Config variable ${APACHE_BODY_LIMIT} is not defined
```
This apache2 variable must be configured.
Edit the `/etc/apache2/envvars' file and add it:
```bash
export APACHE_BODY_LIMIT=10485760
```
The number is the amount of memory we want to allocate, in the example it is 10MB.

### 6.3 - Unable to initialize frontend
If you get this error:
```bash
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. a
t /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 78.)
debconf: falling back to frontend: Readline
```

The frontend must be disabled, which can be done by exporting the environment variable export `DEBIAN_FRONTEND`.
```bash
export DEBIAN_FRONTEND=noninteractive
```
