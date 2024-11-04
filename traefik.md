# Multi-Project Setup with Traefik

This guide covers setting up multiple isolated Docker projects using Traefik for automated routing and load balancing. Starting with an example project, `mood`, you’ll quickly replicate the setup for additional projects like `vibe`, each with its unique URLs and independent configuration. Perfect for managing multiple Oro environments efficiently with Traefik.

When integrating Traefik into your Docker setup, it's important to note that your existing hostnames for services like PostgreSQL, Elasticsearch, Redis, and RabbitMQ will remain unchanged. You will continue to access:

- PostgreSQL at `postgres`
- Elasticsearch at `elasticsearch`
- Redis at `redis`
- RabbitMQ at `rabbitmq`

For example, when connecting to the PostgreSQL database, you can use the following command with the username and password set to `root`:

```bash
psql -h postgres -U root -d dev
```

This consistency ensures that while Traefik enhances your routing and management capabilities, your service URLs remain the same, preventing any confusion in your workflow.

## Table of Contents

- [Set up project](#set-up-project)
- [Set up another project](#set-up-another-project)

## Set up project

Let’s set up an example project named `mood` project, which will be accessible at `mood.local`.

First, create a directory to hold all your Oro projects, for example, `~/Downloads/www-docker/sites`.

Next, create a new directory named `mood`, clone this repository into it, and ensure you have the following file structure:

```
~/Downloads/www-dokcer/sites/mood/

├── conf/
│   ├── .env-app.local
│   ├── nginx.default.conf
│   └── supervisord.conf
├── env/
│   ├── elasticsearch.env
│   ├── mailtrap.env
│   ├── pgadmin.env
│   ├── phpfpm.env
│   ├── postgres.env
│   └── rabbitmq.env
├── src/
│   └── (empty)
├── backups/
│   └── (empty)
├── acme.json
├── docker-compose.yml
├── .env
├── README.md
└── traefik.yml
```

Add the following URLs to your `/etc/hosts` file:

- 127.0.0.1 mood.local
- 127.0.0.1 pgadmin.mood.local
- 127.0.0.1 traefik.mood.local
- 127.0.0.1 rabbitmq.mood.local
- 127.0.0.1 mailcatcher.mood.local
- 127.0.0.1 elasticsearch.mood.local

Run `docker-compose up -d` to start the containers. Ensure you have Docker set up on your machine; if not, please refer to the official documentation and **confirm that Docker is running without requiring sudo**.

Once the command successfully brings up all the services, you’ll be able to access the specified URLs.

- `mood.local`: The primary URL for accessing the main application.
- `pgadmin.mood.local`: Access the pgAdmin interface for managing your PostgreSQL databases.
- `traefik.mood.local`: The entry point for the Traefik dashboard to monitor and manage routes.
- `rabbitmq.mood.local`: Connect to the RabbitMQ management interface for messaging services.
- `mailcatcher.mood.local`: View and manage captured emails during development via Mailcatcher.
- `elasticsearch.mood.local`: Access the Elasticsearch interface for managing search indexes and queries.

## Setup another project

Now, To start another Docker project with the same setup, you can duplicate the existing directory, like `mood` in this example:

```bash
cp -r mood vibe
```

This command creates an exact copy of `mood` in a new directory called `vibe`, including all contents.

Next, open the `docker-compose.yml` file in the `vibe` directory. Replace all instances of `"mood"` with `"vibe"` throughout the file.

Under the `traefik` service in `docker-compose.yml`, update the `ports` section to avoid conflicts with the first project also make sure they are not in use by any other resources:

```yaml
ports:
  - "8088:80"    # Application access at http://vibe.local:8088
  - "8443:443"   # Secure application access at https://vibe.local:8443
  - "8083:8080"  # Traefik dashboard access at http://traefik.vibe.local:8083
```

Now, in the `traefik > labels` section, set the Traefik dashboard port to `8083` for accessibility:

```yaml
labels:
  - "traefik.http.services.traefik-dashboard-vibe.loadbalancer.server.port=8083"
```

If there are any issues accessing the dashboard, you can comment out this line to troubleshoot:

```yaml
  # - "traefik.http.services.traefik-dashboard-vibe.loadbalancer.server.port=8083"
```

Note that while `nginx.conf` supports secure URLs, SSL is not yet set up here. You can add an SSL certificate if possible; otherwise, SSL support will be added soon.

Add the following URLs to your `/etc/hosts` file:

- 127.0.0.1 vibe.local
- 127.0.0.1 pgadmin.vibe.local
- 127.0.0.1 traefik.vibe.local
- 127.0.0.1 rabbitmq.vibe.local
- 127.0.0.1 mailcatcher.vibe.local
- 127.0.0.1 elasticsearch.vibe.local

After making these changes, start the project by running:

```bash
docker-compose up -d
```

This will set up `vibe` as a separate project with the following URLs:

- `vibe.local:8088`
- `pgadmin.vibe.local:8088`
- `traefik.vibe.local:8088`
- `rabbitmq.vibe.local:8088`
- `mailcatcher.vibe.local:8088`
- `elasticsearch.vibe.local:8088`

