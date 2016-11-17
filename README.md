[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-sugarcrm/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-sugarcrm/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/sugarcrm)](https://hub.docker.com/r/bitnami/sugarcrm/)

# What is SugarCRM?

> SugarCRM is a modern and flexible CRM for your entire team. With SugarCRM you can manage all your accounts and customers, connect with people across the company and start selling to your customer base. SugarCRM has integrations with scores of other services such as Google Apps, Pardot marketing automation, Box.com, and social sites such as LinkedIn, Twitter, and Facebook.

https://www.sugarcrm.com/

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recommended with a version 1.6.0 or later.

# How to use this image

## Run SugarCRM with a Database Container

Running SugarCRM with a database server is the recommended way. You can either use docker-compose or run the container manually.

### Run the application using Docker Compose

This is the recommended way to run SugarCRM. You can use the following docker compose template:

```
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  application:
    image: 'bitnami/sugarcrm:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'sugarcrm_data:/bitnami/sugarcrm'
      - 'php_data:/bitnami/php'
      - 'apache_data:/bitnami/apache'
    depends_on:
      - mariadb

volumes:
  mariadb_data:
    driver: local
  sugarcrm_data:
    driver: local
  apache_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```
  $ docker network create sugarcrm_network
  ```

2. Start a MariaDB database in the network generated:

  ```
   $ docker run -d --name mariadb --net=sugarcrm_network bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to SugarCRM to resolve the host

3. Run the SugarCRM container:

  ```
  $ docker run -d -p 80:80 --name sugarcrm --net=sugarcrm_network bitnami/sugarcrm
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove every container and volume all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed. 

If you are using docker-compose your data will be persistent as long as you don't remove `mariadb_data`, `sugarcrm_data` and `apache_data` volumes. 

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount persistent folders in the host using docker-compose

This requires a slight modification from the template previously shown:

```
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    volumes:
      - '/path/to/your/local/mariadb_data:/bitnami/mariadb'
  sugarcrm:
    image: 'bitnami/sugarcrm:latest'
    depends_on:
      - mariadb
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/sugarcrm-persistence:/bitnami/sugarcrm'
      - '/path/to/php-persistence:/bitnami/php'
      - '/path/to/apache-persistence:/bitnami/apache'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```
  $ docker network create sugarcrm-tier
  ```

2. Create a MariaDB container with host volume:

  ```
  $$ docker run -d --name mariadb \
    --net sugarcrm-tier \
    --volume /path/to/mariadb-persistence:/bitnami/mariadb \
    bitnami/mariadb:latest
  ```
   *Note:* You need to give the container a name in order to SugarCRM to resolve the host

3. Create the SugarCRM container with host volumes:

  ```
  $ docker run -d --name sugarcrm -p 80:80 -p 443:443 \
    --net sugarcrm-tier \
    --volume /path/to/sugarcrm-persistence:/bitnami/sugarcrm \
    --volume /path/to/apache-persistence:/bitnami/apache \
    --volume /path/to/php-persistence:/bitnami/php \
    bitnami/sugarcrm:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and SugarCRM, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the SugarCRM container. For the MariaDB upgrade you can take a look at https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```
  $ docker pull bitnami/sugarcrm:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop sugarcrm`
 * For manual execution: `$ docker stop sugarcrm`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the sugarcrm folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v sugarcrm`
 * For manual execution: `$ docker rm -v sugarcrm`

5. Run the new image

 * For docker-compose: `$ docker-compose start sugarcrm`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name sugarcrm bitnami/sugarcrm:latest`

# Configuration
## Environment variables
 When you start the SugarCRM image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```
application:
  image: bitnami/sugarcrm:latest
  ports:
    - 80:80
  environment:
    - SUGARCRM_PASSWORD=my_password
  volumes_from:
    - application_data
```

 * For manual execution add a `-e` option with each variable and value:

```
 $ docker run -d -e SUGARCRM_PASSWORD=my_password -p 80:80 --name sugarcrm -v /your/local/path/bitnami/sugarcrm:/bitnami/sugarcrm --net=sugarcrm_network bitnami/sugarcrm
```

Available variables:

 - `SUGARCRM_USERNAME`: SugarCRM application username. Default: **User**
 - `SUGARCRM_PASSWORD`: SugarCRM application password. Default: **bitnami**
 - `SUGARCRM_EMAIL`: SugarCRM application email. Default: **user@example.com**
 - `SUGARCRM_LASTNAME`: SugarCRM application last name. Default: **Name**
 - `SUGARCRM_HOST`: Host domain or IP.
 - `MARIADB_USER`: Root user for the MariaDB database. Default: **root**
 - `MARIADB_PASSWORD`: Root password for the MariaDB.
 - `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
 - `MARIADB_PORT`: Port used by MariaDB server. Default: **3306**

### SMTP Configuration

To configure SugarCMR to send email using SMTP you can set the following environment variables:

 - `SUGARCRM_SMTP_HOST`: SugarCRM SMTP host.
 - `SUGARCRM_SMTP_PORT`: SugarCRM SMTP port.
 - `SUGARCRM_SMTP_USER`: SugarCRM SMTP account user.
 - `SUGARCRM_SMTP_PASSWORD`: SugarCRM SMTP account password.
 - `SUGARCRM_SMTP_PROTOCOL`: SugarCRM SMTP protocol to use.

This would be an example of SMTP configuration using a Gmail account:

 * docker-compose:

```
  application:
    image: bitnami/sugarcrm:latest
    ports:
      - 80:80
    environment:
      - SUGARCRM_SMTP_HOST=smtp.gmail.com
      - SUGARCRM_SMTP_USER=your_email@gmail.com
      - SUGARCRM_SMTP_PASSWORD=your_password
      - SUGARCRM_SMTP_PROTOCOL=TLS
      - SUGARCRM_SMTP_PORT=587
```

 * For manual execution:

```
 $ docker run -d -e SUGARCRM_SMTP_HOST=smtp.gmail.com -e SUGARCRM_SMTP_PROTOCOL=TLS -e SUGARCRM_SMTP_PORT=587 -e SUGARCRM_SMTP_USER=your_email@gmail.com -e \
 SUGARCRM_SMTP_PASSWORD=your_password -p 80:80 --name sugarcrm -v /your/local/path/bitnami/sugarcrm:/bitnami/sugarcrm bitnami/sugarcrm
```

# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

  * For docker-compose: `$ docker-compose stop sugarcrm`
  * For manual execution: `$ docker stop sugarcrm`

2. Copy the SugarCRM data folder in the host:

  ```
  $ docker cp /your/local/path/bitnami:/bitnami/sugarcrm
  ```

# Restoring a backup

To restore your application using backed up data simply mount the folder with SugarCRM data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-sugarcrm/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-sugarcrm/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/bitnami-docker-sugarcrm/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# License

Copyright 2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
