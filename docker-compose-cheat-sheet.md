# Docker and Docker Compose Cheat Sheet

> A practical reference for Docker CLI, Dockerfiles, and Docker Compose.
>
> **Updated:** October 4, 2026
>
> **Compose format:** This cheat sheet uses the current [Compose Specification](https://docs.docker.com/reference/compose-file/). Do not add a top-level `version` field to new files; the legacy 2.x and 3.x formats have been merged into the specification.

## Installation and verification

| Platform | Installation |
| --- | --- |
| Linux | [Install Docker Engine](https://docs.docker.com/engine/install/) |
| Windows or macOS | [Install Docker Desktop](https://docs.docker.com/desktop/) |

```console
docker run hello-world
```

## Docker CLI

### Containers

| Command | Description |
| --- | --- |
| `docker run <image>` | Create and start a new container |
| `docker run -p 8080:80 <image>` | Publish container port `80` on host port `8080` |
| `docker run -d <image>` | Start a container in the background (detached) |
| `docker run -v <host>:<path> <image>` | Mount a host path at a path in the container |
| `docker run -v <host>:<path>:ro <image>` | Mount a host path as read-only |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers, including stopped containers |
| `docker stats` | Display a live stream of resource usage statistics |
| `docker logs -f <container>` | Follow a container's logs |
| `docker stop <container>` | Stop a running container |
| `docker start <container>` | Start a stopped container |
| `docker rm <container>` | Remove a stopped container |

### Execute commands in a container

| Command | Description |
| --- | --- |
| `docker exec <container> <command>` | Execute a command in a running container |
| `docker exec -it <container> sh` | Open an interactive shell |
| `docker exec -it <container> bash` | Open an interactive Bash shell when Bash is installed |

### Images

| Command | Description |
| --- | --- |
| `docker build -t <image> .` | Build an image from a `Dockerfile` and tag it |
| `docker images` | List local images |
| `docker rmi <image>` | Remove an image |
| `docker image prune` | Remove dangling unused images |

### Container registries

| Command | Description |
| --- | --- |
| `docker login` | Log in to Docker Hub |
| `docker login <server>` | Log in to another container registry |
| `docker logout [<server>]` | Log out of a registry |
| `docker push <image>` | Upload an image to a registry |
| `docker pull <image>` | Download an image from a registry |
| `docker search <image>` | Search Docker Hub for images |

### System

| Command | Description |
| --- | --- |
| `docker system df` | Show Docker disk usage |
| `docker system prune` | Remove unused containers, networks, images, and build cache after confirmation |
| `docker system prune -a` | Also remove all unused images, not only dangling images |
| `docker info` | Display system-wide information |

> `docker system prune` does not remove volumes by default. Add `--volumes` only when you intend to remove unused volumes.

## Docker Compose commands

Run these commands from the directory containing `compose.yaml`, or specify a file with `-f <file>`. Compose automatically loads `compose.yaml` (preferred) and `compose.yml`; `docker-compose.yml` is also supported for compatibility.

| Command | Description |
| --- | --- |
| `docker compose up` | Create and start the application's services |
| `docker compose up -d` | Create and start services in the background |
| `docker compose up -d --build` | Rebuild images, then start services in the background |
| `docker compose up --scale <service>=<n>` | Run a service with a specified number of containers |
| `docker compose stop` | Stop services without removing containers |
| `docker compose start` | Start previously stopped services |
| `docker compose restart` | Restart services |
| `docker compose down` | Stop and remove containers and networks created by `up` |
| `docker compose down -v` | Also remove named and anonymous volumes declared by the Compose project |
| `docker compose ps` | List the project's containers and their status |
| `docker compose logs` | Show logs from all services |
| `docker compose logs <service>` | Show logs for one service |
| `docker compose logs -f` | Follow logs from all services |
| `docker compose exec <service> <command>` | Execute a command in a running service container |
| `docker compose run --rm <service> <command>` | Run a one-off command and remove its container afterward |
| `docker compose pull` | Pull service images |
| `docker compose build` | Build or rebuild services |
| `docker compose build --pull` | Pull newer base images before building |
| `docker compose config` | Render and validate the fully resolved Compose configuration |
| `docker compose config -q` | Validate the configuration without printing it |

Useful global options include `-f <file>` to select a Compose file, `-p <project>` to select a project name, and `--profile <profile>` to enable a profile.

## Dockerfile instructions

| Instruction | Description |
| --- | --- |
| `FROM <image>` | Set the base image |
| `FROM <image> AS <name>` | Set the base image and name a build stage |
| `RUN <command>` | Execute a command while building the image |
| `CMD ["exec", "param1", "param2"]` | Set the default command when a container starts |
| `ENTRYPOINT ["exec", "param1"]` | Configure the container's executable |
| `ENV <key>=<value>` | Set an environment variable in the image |
| `EXPOSE <port>` | Document a port the application listens on; it does not publish the port |
| `COPY <src> <dest>` | Copy files from the build context into the image |
| `COPY --from=<name> <src> <dest>` | Copy files from another build stage |
| `WORKDIR <path>` | Set the working directory for subsequent instructions |
| `VOLUME <path>` | Declare a mount point |
| `USER <user>` | Set the user for subsequent instructions and the default runtime user |
| `ARG <name>` | Define a build argument |
| `ARG <name>=<default>` | Define a build argument with a default value |
| `LABEL <key>=<value>` | Add image metadata |
| `HEALTHCHECK <command>` | Configure a container health check |

See the [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) for syntax, shell versus exec form, and all available instructions.

## Compose file

### Minimal example

Save as `compose.yaml`:

```yaml
services:
  service1:
    image: <image>
    build: .
    volumes:
      - .:/code:ro
    ports:
      - "8000:80"
    environment:
      KEY: value
```

`image` selects the image to run. `build` defines how to build an image when one is not available or when `--build` is requested. They are separate properties and may be used together.

### Top-level keys

| Key | Description |
| --- | --- |
| `name` | Set the Compose project name |
| `services` | Define the application's containers |
| `networks` | Define networks used by services |
| `volumes` | Define named volumes used by services |
| `configs` | Define non-sensitive configuration data |
| `secrets` | Define sensitive data made available to services |
| `include` | Include additional Compose files |

Most applications need only `services`; the other keys are optional. Profiles are assigned to individual services with `services.<name>.profiles`.

### Service keys

| Key | Description |
| --- | --- |
| `services.<name>.image` | Image to run |
| `services.<name>.build` | Build context and image build options |
| `services.<name>.build.context` | Build context path |
| `services.<name>.build.dockerfile` | Dockerfile path, relative to the build context by default |
| `services.<name>.build.target` | Multi-stage build target |
| `services.<name>.build.args` | Build-time arguments |
| `services.<name>.command` | Override the image's default `CMD` |
| `services.<name>.entrypoint` | Override the image's `ENTRYPOINT` |
| `services.<name>.volumes` | Mount bind mounts or named volumes |
| `services.<name>.ports` | Publish container ports on the host |
| `services.<name>.environment` | Set container environment variables |
| `services.<name>.env_file` | Read environment variables from a file |
| `services.<name>.restart` | Set a restart policy: `no`, `always`, `on-failure`, or `unless-stopped` |
| `services.<name>.scale` | Set a default replica count; prefer `docker compose up --scale` for one-off scaling |
| `services.<name>.networks` | Connect the service to networks |
| `services.<name>.depends_on` | Express service startup and shutdown dependencies |
| `services.<name>.healthcheck` | Define how container health is tested |
| `services.<name>.labels` | Add container metadata |
| `services.<name>.profiles` | Enable the service only for selected profiles |
| `services.<name>.secrets` | Mount secrets into the container |
| `services.<name>.configs` | Mount configuration files into the container |

`depends_on` controls dependency order, but short syntax does not wait for an application to become ready. Use a `healthcheck` and long syntax with `condition: service_healthy` when readiness matters.

Example:

```yaml
services:
  app:
    image: example/app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:18
    environment:
      POSTGRES_PASSWORD: example
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### Networks

```yaml
networks:
  frontend:
    driver: bridge
  existing:
    external: true
    name: my-existing-network
```

| Key | Description |
| --- | --- |
| `networks.<name>.driver` | Select the network driver, commonly `bridge` |
| `networks.<name>.external` | Use an existing network instead of creating one |
| `networks.<name>.name` | Set the actual Docker network name |

### Volumes

```yaml
volumes:
  app-data:
    driver: local
    name: my-app-data
```

| Key | Description |
| --- | --- |
| `volumes.<name>.driver` | Select the volume driver |
| `volumes.<name>.name` | Set the actual Docker volume name |
| `volumes.<name>.external` | Use an existing volume instead of creating one |

Use a named volume in a service with `- app-data:/var/lib/app`.

### Secrets and configs

```yaml
services:
  app:
    image: example/app
    secrets:
      - db_password
    configs:
      - source: app_config
        target: /etc/example/app.conf

secrets:
  db_password:
    file: ./secrets/db_password.txt

configs:
  app_config:
    file: ./config/app.conf
```

Secrets are intended for sensitive values and configs for non-sensitive configuration. Do not commit secret files to source control. Support and behavior can vary by Compose implementation; check the target platform documentation.

## Common patterns

### Environment variable interpolation

```yaml
services:
  web:
    image: "nginx:${NGINX_TAG:-latest}"
    ports:
      - "${HTTP_PORT:-8080}:80"
```

Compose can read values from the shell and from a project `.env` file. Use `docker compose config` to inspect the resolved result, and quote published ports to prevent YAML from interpreting values unexpectedly.

### Development bind mount

```yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - .:/app
    ports:
      - "8000:8000"
```

### Read-only bind mount

```yaml
volumes:
  - ./config:/etc/app:ro
```

## References

- [Docker Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Compose Specification](https://compose-spec.io/)
- [Docker Compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Engine installation](https://docs.docker.com/engine/install/)
- [Docker Desktop](https://docs.docker.com/desktop/)

This Markdown version preserves and reorganizes its command and reference material while adding clarifications and current Compose guidance.
