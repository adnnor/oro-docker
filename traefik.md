# Traefik

TODOs

Correct PHP extensions under phpfpm remove mysql add pdo or postgres
Correct path of Credits in TOC
Upload Change log to master
rename test-traefik to test
rename project1 to mood
rename project2 to vibe




Traefik makes running multiple Docker projects easy with a few adjustments. By setting `COMPOSE_PROJECT_NAME` in a `.env` file, you assign unique project names, isolating container and network names to avoid conflicts. Setting unique hostnames in Traefik labels allows each instance to route traffic to its own services, ensuring projects run independently with separate configurations and customized routing.

If you skip setting `COMPOSE_PROJECT_NAME`, Docker Compose defaults to the directory name as the project name. While this can prevent conflicts, explicitly defining project names provides more control and avoids potential confusion in complex directory setups.

Traefik can be used for both single and multi-project setups. Traefik is great for handling automated routing, load balancing, and hostname management, but if you don’t need advanced routing, you can easily manage ports and networks manually. 

For simpler setup, check out the standard Docker guide [here](./standard-setup) if you’re working on a single project.


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

- `mood.local`
- `pgadmin.mood.local`
- `traefik.mood.local`

Run `docker-compose up -d` to start the containers. Ensure you have Docker set up on your machine; if not, please refer to the official documentation and **confirm that Docker is running without requiring sudo**.

Once the command successfully brings up all the services, you’ll be able to access the specified URLs.

## Setup another project

Now, To start another Docker project with the same setup, you can duplicate the existing directory, like `mood` in this example:

```bash
cp -r mood vibe
```

This command creates an exact copy of `mood` in a new directory called `vibe`, including all contents.

Next, open the `docker-compose.yml` file in the `vibe` directory. Replace all instances of `"mood"` with `"vibe"` throughout the file.

Under the `traefik` service in `docker-compose.yml`, update the `ports` section to avoid conflicts with the first project:

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

- `vibe.local`
- `pgadmin.vibe.local`
- `traefik.vibe.local`

After making these changes, start the project by running:

```bash
docker-compose up -d
```

This will set up `vibe` as a separate project with the following URLs:

- `vibe.local:8088`
- `pgadmin.vibe.local:8088`
- `traefik.vibe.local:8088`

