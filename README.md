# Docker setup for Oro Commerce

This guide provides an overview of managing your docker setup that includes PHP, Nginx, OpenSearch, PostgreSQL, pgAdmin, Redis, Supervisor and MailCatcher. You will learn how to start and stop containers, access specific services, check the database and email systems, and more.

Feel free to update this README.md if you notice anything missing. If you need a different version of Nginx or PHP, please reach out to adnnor@gmail.com.

**Credits**
Some parts of this setup, especially the Elasticsearch configuration and the concepts of file management and mounts, are adapted from [Markshust's Docker Setup for Magento](https://github.com/markshust/docker-magento).

## The phpfpm service
Common PHP extensions: `bcmath`, `bz2`, `calendar`, `exif`, `ftp`, `gd`, `gettext`, `intl`, `mbstring`, `mysqli`, `opcache`, `pcntl`, `pdo_mysql`, `soap`, `sockets`, `sodium`, `sysvmsg`, `sysvsem`, `sysvshm`, `xsl`, `zip`
Redis
Imagick
SSH2
Swoole
Blackfire  
Node.js v20.x  
Composer v2.6.6  
Git v2.x  
Redis CLI  
Xdebug v3.3.2
nano and vi ;-)

## The app service
Nginx is configured to serve PHP applications, utilizing FastCGI for processing PHP files. The service supports both HTTP and HTTPS traffic.

1. **User and Permissions**  
   The Nginx service runs as a non-root user (`app`) for security. Ensure that the appropriate permissions are set on the mounted directories to avoid permission issues.

2. **Using with Magento**  
   If you are using this Nginx setup for Magento, uncomment the following line in your Docker Compose file to mount the sample Nginx configuration (only available in docker-compose.mysql-pms.yml):
   ```yaml
   #- ./src/nginx.conf.sample:/var/www/html/nginx.conf:cached
   ```

3. **Volume Mounting**  
   The volume `./src:/var/www/html:cached` is mounted, allowing your application files to be served from the `src` directory on your host machine. Ensure your application files are located in this directory for proper operation.

4. **Logs**  
   Access and error logs are located at `/var/log/nginx/access.log` and `/var/log/nginx/error.log`, respectively. Monitor these logs for troubleshooting and performance analysis.

## The supervisor service

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
**Useful supervisord commands**

- `supervisorctl reread` - Reloads the configuration files.
- `supervisorctl update` - Updates Supervisor to include any new or modified programs.
- `supervisorctl status` - Checks the status of the running programs.


## The rabbitmq service

RabbitMQ is a message broker that facilitates communication between different components of the Oro application. It enables asynchronous processing of tasks, allowing for efficient handling of background jobs, notifications, and real-time messaging between services.

**Ports**
- **Management UI:** Accessible at `http://localhost:15672`
- **Message Broker:** Uses port `5672` for communication.

**Environment Configuration**
RabbitMQ environment variables are loaded from the `env/rabbitmq.env` file. You can customize settings like user credentials and other configurations here. Default credentials are `rabbit` for both username and password.



#### Check the Status of the Background Processes

To check the status of the background processes, run:

```bash
supervisorctl status
```

You should see information similar to this:

```
oro_message_consumer:oro_message_consumer_00   RUNNING   pid 4847, uptime 0:05:36
oro_message_consumer:oro_message_consumer_01   RUNNING   pid 4846, uptime 0:05:36
oro_message_consumer:oro_message_consumer_02   RUNNING   pid 4845, uptime 0:05:36
oro_message_consumer:oro_message_consumer_03   RUNNING   pid 4844, uptime 0:05:36
oro_message_consumer:oro_message_consumer_04   RUNNING   pid 4843, uptime 0:05:36
oro_web_socket                                 RUNNING   pid 5163, uptime 0:00:05
```

## 1. Starting and Stopping Containers

**To start and stop the containers:**

```bash
docker-compose -f docker-compose.yml up -d --remove-orphans # to start containers
docker-compose -f docker-compose.yml down # to down containers
```
Note: The `--remove-orphans` option in the docker-compose up command is used to remove any containers that are not defined in the current docker-compose.yml file but were part of previous runs of Docker Compose.

## 2. Accessing Containers

To access any container, use the following command to get its name:
```bash
docker-compose ps
```
Then, enter a container using:
```bash
docker-compose exec <service-name> bash
# e.g
docker-compose exec phpfpm bash
```

You can also execute commands directly into the container without entering it by using:
```bash
docker ps # to get container ID

docker exec -it <container-id> [command]
# e.g.
docker exec -it <container-id> php -v"
docker exec -it <container-id> bin/console cache:clean --env=prod
```

## 3. PostgreSQL Database
This section covers the interaction with the database.
### Checking database service

To check if the PostgreSQL database is running, log into the database:
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

### Importing database dump

To import a `.sql` file:
```bash
cat path/to/your/file.sql | docker exec -i <container-id> psql -U root -d dev
```

For `.gz` archive:
```bash
gunzip < path/to/your/file.sql.gz | docker exec -i <container-id> psql -U root -d dev
```
### Export database dump
```bash
docker exec -t <container-id> pg_dump -U root -d dev > path/to/your/file.sql
```

## 4. pgAdmin

Access pgAdmin at:
```
http://localhost:1435
```

**pgAdmin Credentials:**
- **Default Email:** `root@adnanshahzad.me`
- **Password:** `root`

## 5. Checking Emails Using MailCatcher

To check emails using MailCatcher, open your web browser and navigate to:
```
http://localhost:1080
```

## 6. Checking ElasticSearch Health


```bash
curl -X GET "http://localhost:9200/_cluster/health"
```


## 7. Checking Logs

To view logs for a specific service, use:
```bash
docker-compose logs <service_name>
```
If you want to see logs for all services, simply run:
```bash
docker-compose logs
```

## 8. Additional Information

Other versions of PHP, Nginx, and Elasticsearch are available on Docker Hub under the `adnnor` repository, but only the following have been tested:

- **PHP Versions:**
    - `adnnor/php:8.2-fpm`
    - `adnnor/php:8.3-fpm`
- **Nginx Version:**
    - `adnnor/nginx:1.18`
- **ElasticSearch Versions:**
    - `adnnor/elasticsearch:8.4`

For installation, please refer to the official Oro Commerce documentation [here](https://doc.oroinc.com/backend/setup/installation/).
