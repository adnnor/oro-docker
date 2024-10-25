# Docker setup for Oro Commerce

This guide provides an overview of managing your docker setup that includes PHP, Nginx, OpenSearch, PostgreSQL, pgAdmin, Redis, Supervisor and MailCatcher. You will learn how to start and s↑ containers, access specific services, check the database and email systems, and more.


## Table of Contents
- [File Structure](#file-structure)
  - [env](#env)
  - [src](#src)
  - [conf](#conf)
  - [docker-compose.yml](#docker-composeyml)  
- [Services](#services)
  - [app](#app)
  - [phpfpm](#phpfpm)
  - [supervisor](#supervisor)
  - [postgres](#postgres)
  - [pgadmin](#pgadmin)
  - [rabbitmq](#rabbitmq)
  - [mailcatcher](#mailcatcher)
- [Credits](#credits)
- [Extras](#extras)

Please feel free to update this README.md if you notice anything that is missing. If you need a different version of Nginx or PHP, kindly fill out the form at [this link](https://docs.google.com/forms/d/1PA9piAEZfOT3rp3SQqsq8ASz2dIYLldU8lQQ-fPzvlQ/prefill).

## File Structure

This section outlines the key directories and files in the project, including configuration files, environment variable files, and application source code. Understanding the structure will help you manage the setup, configuration, and operation of the Docker-based environment effectively.


#### env
Contains environment variable files for different services.

- `phpfpm.env`: Environment variables for the PHP-FPM service.
- `postgres.env`: Environment variables for the PostgreSQL service.
- `pgadmin.env`: Environment variables for pgAdmin.
- `elasticsearch.env`: Environment variables for Elasticsearch.
- `rabbitmq.env`: Environment variables for RabbitMQ.

#### conf
Contains configuration files for various services.

- `nginx.default.conf`: Nginx configuration file.
- `.env-app.local`: Environment file for Oro Commerce.
- `supervisord.conf`: Supervisor configuration to manage background processes.

#### src
The base directory where the application source code is located. Mounted to `/var/www/html` within containers.

## docker-compose.yml [↑](#table-of-contents)
Defines services, volumes, networks, and mounts used in the Docker setup.

Check the complete [Docker Cheat Sheet](./docker-cheats.md) for useful Docker commands.

#### Volumes
Volumes are used to persist data across container restarts:

- `postgres-data`: Stores PostgreSQL data.
- `pgadmin-data`: Stores pgAdmin data.
- `rabbitmqdata`: Stores RabbitMQ data.
- `sockdata`: Socket files for Nginx communication.

#### Networks
The `oro` network is defined as a `bridge` network, enabling inter-service communication.

#### Mounts
The following directories and files are mounted to the containers:

The **Oro code base** is mounted from `./src:/var/www/html:cached`. This ensures that the application source code on your local machine is synchronized with the container, allowing for smooth development and real-time updates.

The **nginx configuration** is copied from `./conf/nginx.default.conf:/etc/nginx/conf.d/default.conf:cached`. This sets up the server configuration inside the container using your local Nginx configuration file.

The **environment file for Oro** is mounted from `./conf/.env-app.local:/var/www/html/.env-app.local:cached`. This file provides the necessary environment variables for the Oro application to run properly.

Your **Composer configuration** is mounted from `~/.composer:/var/www/.composer:cached`. This allows the container to use your local Composer settings, enabling seamless dependency management for PHP projects.

Your **SSH keys** are mounted from `~/.ssh:/var/www/.ssh:cached`. This ensures the container has access to your SSH keys, which is useful for secure access to private repositories or servers.

Finally, the **known hosts file** is mounted from `~/.ssh/known_hosts:/var/www/.ssh/known_hosts:cached`. This provides a list of SSH hosts your machine has connected to, ensuring smoother SSH operations within the container.

## app [↑](#table-of-contents)
Nginx is configured to serve PHP applications, utilizing FastCGI for processing PHP files. The service supports both HTTP and HTTPS traffic.

1. **User and Permissions**  
   The Nginx service runs as a non-root user (`app`) for security. Ensure that the appropriate permissions are set on the mounted directories to avoid permission issues.

3. **Volume Mounting**  
   The volume `./src:/var/www/html:cached` is mounted, allowing your application files to be served from the `src` directory on your host machine. Ensure your application files are located in this directory for proper operation.

4. **Logs**  
   Access and error logs are located at `/var/log/nginx/access.log` and `/var/log/nginx/error.log`, respectively. Monitor these logs for troubleshooting and performance analysis.

## phpfpm [↑](#table-of-contents)

Installed PHP extensions are: `bcmath`, `bz2`, `calendar`, `exif`, `ftp`, `gd`, `gettext`, `intl`, `mbstring`, `mysqli`, `opcache`, `pcntl`, `pdo_mysql`, `soap`, `sockets`, `sodium`, `sysvmsg`, `sysvsem`, `sysvshm`, `xsl`, `zip`

- Redis (redis-cli is available)
- Node.js v20.x  
- Composer v2.6.6  
- Git v2.x
- Xdebug v3.3.2
- nano and vi/vim! for sure ;)


## supervisor [↑](#table-of-contents)

The required background processes are the following:

- **Message Queue Consumer:** Performs resource-consuming tasks in the background.
- **Web Socket Server:** Manages real-time messages between the application server and the user’s browser.

It is crucial to keep these two background processes running. To maintain their constant availability, supervisor service is provided in the docker-compose.

The `supervisord.conf` file is already available under the `conf` directory. You may update it according to your needs. This file is loaded as the default supervisor config under `docker-compose.yml`, as shown below:

```yaml
  supervisor:
    ...
    volumes:
      ...
      - ./supervisord.conf:/etc/supervisord.conf
```

[Configure the Supervisor - Oro Documentation](https://doc.oroinc.com/backend/setup/installation/#configure-the-supervisor)

**Useful supervisor commands**

```bash
# reloads the configuration files
supervisorctl reread

# update supervisor to include any new or modified programs
supervisorctl update

# check the status of the running programs
supervisorctl status

```

Executing `supervisorctl status` should display information similar to this:

```
oro_message_consumer:oro_message_consumer_00   RUNNING   pid 4847, uptime 0:05:36
oro_message_consumer:oro_message_consumer_01   RUNNING   pid 4846, uptime 0:05:36
oro_message_consumer:oro_message_consumer_02   RUNNING   pid 4845, uptime 0:05:36
oro_message_consumer:oro_message_consumer_03   RUNNING   pid 4844, uptime 0:05:36
oro_message_consumer:oro_message_consumer_04   RUNNING   pid 4843, uptime 0:05:36
oro_web_socket                                 RUNNING   pid 5163, uptime 0:00:05
```


## postgres [↑](#table-of-contents)
This section covers the PostgreSQL database activities.

To check if the database is running, log into the database:
```bash
docker exec -it <container-id> psql -U root -d dev
# or access the container
docker-compose -f docker-compose.yml exec postgres bash
# then execute 
psql -U root -d dev
```

**Default Database Credentials:**
- **Username:** `root`
- **Password:** `root`
- **Host:** `postgres`
- **Default Database:** `dev`

**Importing database dump**


```bash
# import a `.sql` file
cat path/to/your/file.sql | docker exec -i <container-id> psql -U root -d dev
# For `.gz` archive
gunzip < path/to/your/file.sql.gz | docker exec -i <container-id> psql -U root -d dev
# export database
docker exec -t <container-id> pg_dump -U root -d dev > path/to/your/file.sql
```

## pgadmin [↑](#table-of-contents)

Access pgAdmin at:
```
http://localhost:1435
```

**pgAdmin Credentials:**
- **Default Email:** `root@adnanshahzad.me`
- **Password:** `root`

You'll need to create a new server in pgAdmin to access the database. For database details, please refer to the [postgres](#postgres) section.


## rabbitmq [↑](#table-of-contents)

RabbitMQ is a message broker that facilitates communication between different components of the Oro application. It enables asynchronous processing of tasks, allowing for efficient handling of background jobs, notifications, and real-time messaging between services.

**Ports**
- **Management UI:** Accessible at `http://localhost:15672`
- **Message Broker:** Uses port `5672` for communication.

**Environment Configuration**
RabbitMQ environment variables are loaded from the `env/rabbitmq.env` file. You can customize settings like user credentials and other configurations here. Default credentials are `rabbit` for both username and password.


## mailcatcher

To check emails using MailCatcher, open your web browser and navigate to:
```
http://localhost:1080
```

## Credits [↑](#table-of-contents)

Some parts of this setup, especially the Elasticsearch configuration and the concepts of file management and mounts, are adapted from [Markshust's Docker Setup for Magento](https://github.com/markshust/docker-magento).

## Extras

Other versions of PHP, Nginx, and Elasticsearch are available on Docker Hub under the `adnnor` repository, but only the following have been tested:

- **PHP Versions:**
    - `adnnor/php:8.2-fpm`
    - `adnnor/php:8.3-fpm`
- **Nginx Version:**
    - `adnnor/nginx:1.18`
- **ElasticSearch Versions:**
    - `adnnor/elasticsearch:8.4`

For installation, please refer to the official Oro Commerce documentation [here](https://doc.oroinc.com/backend/setup/installation/).

