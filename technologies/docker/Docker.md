# <center> Docker Quick Guide </center>

Docker is a containerization platform that uses containers to package and run applications. Containers are lightweight, portable, and consistent environments that run on any host with Docker installed. They allow for a high level of isolation and efficiency compared to virtual machines (VMs).

<p align="center">
    <figure class="figure">
        <p><img src="images/docker&VM.png" alt="" /></p>
    </figure>
</p>

## Table of content

-   **[Basics](#basics)**
-   **[Images](#images)**
-   **[Run](#run)**
-   **[Compose](#compose)**
-   **[Networking](#networking)**
-   **[Swarm](#swarm)**
-   **[Registry](#registry)**

## Basics

Here are some basics commands :

| Name                           | Command                                                          |
| ------------------------------ | ---------------------------------------------------------------- |
| List running containers        | `docker ps`                                                      |
| List all containers            | `docker ps -a`                                                   |
| Inspect a container            | `docker inspect <container_name>`                                |
| Retrieve the logs              | `docker logs <container_name>`                                   |
| Start a stopped container      | `docker start <container_name>`                                  |
| Stop a running container       | `docker stop <container_name>`                                   |
| Connect to a running container | `docker exec -it <container_name> /bin/bash`                     |
| Copy from container to host    | `docker cp <container_name>:/path/to/file /path/to/file/on/host` |
| Copy from host to container    | `docker cp /path/to/file/on/host <container_name>:/path/to/file` |
| Remove a container             | `docker rm <container_name>`                                     |

## Images

| Name                                      | Command                                                 |
| ----------------------------------------- | ------------------------------------------------------- |
| List images                               | `docker images`                                         |
| Pull an image                             | `docker pull <image_name>`                              |
| Remove an image                           | `docker rmi <image_name>`                               |
| Save a container to a tarball             | `docker export <container_name> > <container_name>.tar` |
| Load a container from an exported tarball | `docker import <container_name>.tar`                    |
| Save an image to a tarball                | `docker save -o <image_name>.tar <image_name>:<tag>`    |
| Load an image from a tarball              | `docker load -i <image_name>.tar`                       |
| Build an image                            | `docker build Dockerfile -t <image_name>:<tag>`         |

## Run

| Name                         | Command                                  |
| ---------------------------- | ---------------------------------------- |
| Run a container              | `docker run <image_name>`                |
| Run and override the command | `docker run <image_name> <command>`      |
| Run an interractive sell     | `docker run -it <image_name>`            |
| Run in the background        | `docker run -d <image_name>`             |
| Run and set an env variable  | `docker run -e COLOR=green <image_name>` |

## Compose

```yaml
services:
    proxy:
        image: nginx:1.27
        container_name: proxy
        volumes:
            - ./proxy/nginx.conf:/etc/nginx/nginx.conf
            - ./proxy/reload.sh:/reload.sh:ro
        command: /bin/bash -c "apt update && apt install -y inotify-tools && nginx && bash /reload.sh"
        ports:
            - '80:80'
        networks:
            - default
        depends_on:
            mysql:
                condition: service_healthy
                restart: always

    mysql:
        image: mysql:8.0
        container_name: mysql
        environment:
            MYSQL_ROOT_PASSWORD: ${DB_ADMIN_PASSWORD}
            MYSQL_USER: ${DB_USER}
            MYSQL_PASSWORD: ${DB_PASSWORD}
        volumes:
            - db_data:/var/lib/mysql
            - ./database:/docker-entrypoint-initdb.d
        networks:
            - default
        healthcheck:
            test:
                [
                    'CMD-SHELL',
                    'mysqladmin ping -h localhost -u ${DB_USER} -p${DB_PASSWORD} --silent',
                ]
            interval: 10s
            retries: 5
            start_period: 30s
            timeout: 10s
volumes:
    db_data:

networks:
    default: {}
```

Other usefull arguments :

-   Use when you want to keep a container open interactively — helpful for debugging or when using interactive tools like a shell.

    ```yaml
    tty: true
    stdin_open: true
    ```

-   Use when you want to build a custom image from a local Dockerfile and you want to specify args, target, and more… :

    ```yaml
    build:
    context: .
    dockerfile: Dockerfile
    args:
        MY_BUILD_ARG: value
    target: dev
    ```

-   Overrides the container’s default startup command.

    ```yaml
    command: ['npm', 'start']
    ```

-   Override the container's entrypoint script.
    ```yaml
    entrypoint: ['bash', 'custom-entrypoint.sh']
    ```

| Name                                    | Command                                  |
| --------------------------------------- | ---------------------------------------- |
| Create / Launch all relative containers | `docker compose up`                      |
| Create / Launch a specific container    | `docker compose up \<container_name\>`   |
| Remove all relative containers          | `docker compose down`                    |
| Remove a specific container             | `docker compose down \<container_name\>` |
| Remove a volume                         | `docker volume rm \<volume_name\>`       |

```Dockerfile
FROM ubuntu:22.04

USER root

WORKDIR /home/app

COPY . /home/app

RUN apt update && apt upgrade -y
RUN install -y python3 python3-pip
RUN python3 -m env env
RUN source env/bin/activate
RUN pip install --no-cache-dir -r requirements.txt

ENTRYPOINT python3 app.py
CMD echo "hello world"
```

Particularities :

-   Docker uses layer caching, which allows faster rebuilds by reusing unchanged layers — even when a rebuild is triggered due to a failure.
-   An ENTRYPOINT in a Dockerfile defines the default command that runs every time a container is started from the image.
-   A command defines the default command that runs if it is not overwritten.

If neither the command or the entrypoint are defined, the container will stop.

## Networking

## Swarm

## Re
