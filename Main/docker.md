# Index
- [Index](#index)
- [Overview](#overview)
  - [Container \& Layer](#container--layer)
  - [Host Network](#host-network)
- [Docker CLI](#docker-cli)
  - [Use `docker run` to create and run a new container from an image](#use-docker-run-to-create-and-run-a-new-container-from-an-image)
  - [Use `docker exec` to execute a command in a running container](#use-docker-exec-to-execute-a-command-in-a-running-container)
  - [Use `docker ps` to list all containers](#use-docker-ps-to-list-all-containers)
  - [Copy file between a container and the local filesystem](#copy-file-between-a-container-and-the-local-filesystem)
  - [Start / Stop / Restart / Remove a container](#start--stop--restart--remove-a-container)
  - [Check all networks](#check-all-networks)
  - [Check resource usage](#check-resource-usage)
- [Docker compose](#docker-compose)
- [Docker compose CLI](#docker-compose-cli)
  - [Use an alternate compose file / project name](#use-an-alternate-compose-file--project-name)
  - [Stop containers and remove containers, networks, volumes, and images created by `up`](#stop-containers-and-remove-containers-networks-volumes-and-images-created-by-up)
  - [Remove stopped service containers](#remove-stopped-service-containers)
  - [Build or rebuild services](#build-or-rebuild-services)
  - [Build, (re)create, start, and attach to containers for a service](#build-recreate-start-and-attach-to-containers-for-a-service)
  - [Start / stop containers](#start--stop-containers)

# Overview

- https://docs.docker.com/get-started/docker-overview/#the-docker-platform

Docker provides the ability to package and run an application in a loosely isolated environment called a container. The isolation and security lets you run many containers simultaneously on a given host. Containers are lightweight and contain everything needed to run the application, so you don't need to rely on what's installed on the host.

## Container & Layer

- Base layer is linux alpine
- Only different layers are downloaded
- Uses AuFS, a layered file system. Containers can share the common parts of the OS as read only, and mount their own part for writing.
- For example, if you have 1000 containers running the same OS image (1 GB), using full VM would need to have 1000 GB, while using Docker would only need to have a little over 1 GB.

## Host Network

- If container open port `8888:8080`, localhost can connect to it using `localhost:8888`
- If container open port `8888:8080`, other containers can connect to it using `{hostname}:8080`
- If localhost open port `9999`, container can connect to it using `host.docker.internal:9999`

# Docker CLI

All command ref: https://docs.docker.com/reference/cli/docker/container/

## Use `docker run` to create and run a new container from an image

Remember to publish to port in order to connect it from your local machine.

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# MySQL
docker run --name hktv_drdp_db -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:latest

# MongoDB
docker run --name mongo -p 27017:27017 -itd mongo --auth

# MSSQL
docker run --name platform_mssql -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=mF34266223ok' -p 1433:1433 -v D:\Users\peter.chau\work\database\platform_mssql:/var/opt/mssql/data -d mcr.microsoft.com/mssql/server:2022-latest
```

Option | Description
------ | -----------
`-d, --detach` | Run container in background and print container ID
`-e, --env`	|	Set environment variables
`-i, --interactive` | Keep STDIN open even if not attached
`--name` |	Assign a name to the container
`-p, --publish`	|	Publish a container's port(s) to the host
`--rm` | Automatically remove the container when it exits
`-t, --tty` | Allocate a pseudo-TTY (connect container `STDIN` and `STDOUT` to local host bash)
`-v {host_machine}:{container}` | Mount a volume

## Use `docker exec` to execute a command in a running container

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

docker exec -it hktv_drdp_db bash
```

Option | Description
------ | -----------
`-i, --interactive` | Keep STDIN open even if not attached
`-t, --tty` | Allocate a pseudo-TTY (connect container `STDIN` and `STDOUT` to local host bash)


## Use `docker ps` to list all containers
```
docker ps [OPTIONS]

PS C:\work\koc-wowza> docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS              PORTS                                                                                           
                                                                        NAMES
67cfc8958e0e   nginx:latest                  "/docker-entrypoint.…"   13 minutes ago   Up About a minute   0.0.0.0:8080->80/tcp                                                    hktv_wowza_nginx
```
- Hints: You can use `67cfc8958e0e`, `67c`, `hktv_wowza_nginx` to indicate the specific container.

## Copy file between a container and the local filesystem
```
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH

# (/tmp/output.csv)
docker cp .\output.csv hktv_drdp_db:/tmp/

# (/tmp/test.csv)
docker cp .\output.csv hktv_drdp_db:/tmp/test.csv
```

## Start / Stop / Restart / Remove a container
```sh
docker start CONTAINER
docker stop CONTAINER
docker restart CONTAINER

# stronger than stop
docker kill CONTAINER

# -f : (OPTIONAL) Force the removal of a running container (uses SIGKILL)
docker rm CONTAINER -f
```
- When to `restart`? - changing some config files
- What if cannot `stop` / `kill` / `rm`? - try docker compose down, if still cannot then restart docker

## Check all networks
```sh
docker network ls

PS C:\Users\cwchau> docker network ls
NETWORK ID     NAME                          DRIVER    SCOPE
64389daa58fc   bridge                        bridge    local
ced49251f836   hktv-ods_network              bridge    local
1ad23f542480   hktv-pricechart-api_default   bridge    local
7dce5bfdd98f   host                          host      local
8b6764c3845d   none                          null      local

# Display detailed information on the network
docker network inspect NETWORK_NAME
```
- Network Drivers:
  - bridge: The default network driver.
  - host: Containers can use the host's networking directly. Only for Linux host.

## Check resource usage
```sh
docker stats

CONTAINER ID   NAME                   CPU %     MEM USAGE / LIMIT    MEM %     NET I/O           BLOCK I/O     PIDS
41415b8959e9   redis_cluster_node_4   0.38%     13.12MiB / 11.7GiB   0.11%     6.78MB / 3.41MB   1.93MB / 0B   6
432f74d3c044   redis_cluster_node_6   0.38%     4.52MiB / 11.7GiB    0.04%     3.51MB / 3.5MB    2.81MB / 0B   6
76cd900ffa6f   redis_cluster_node_5   0.41%     9.719MiB / 11.7GiB   0.08%     5.71MB / 3.57MB   3.2MB / 0B    6
aef85ce49240   redis_cluster_node_1   0.53%     7.777MiB / 11.7GiB   0.06%     3.57MB / 5.69MB   0B / 0B       5
7cb81f98ff0b   redis_cluster_node_2   0.39%     5.867MiB / 11.7GiB   0.05%     3.53MB / 3.53MB   4.13MB / 0B   5
d723c6959db6   redis_cluster_node_3   0.32%     12.28MiB / 11.7GiB   0.10%     3.55MB / 6.93MB   0B / 0B       5
```


# Docker compose

When using `docker compose [command]`, the default file is `pj_folder/docker-compose.yml`

```yml
# docker-compose.yml
version: "3.5"
services:
  hktv_ods_db:
    container_name: hktv_ods_db
    hostname: hktv.odsdb.com
    build:
      context: ./
      dockerfile: Dockerfile-ods-db
      args:
        - IMAGE_TAG=$IMAGE_TAG  # here $IMAGE_TAG point to .env
    env_file: .env
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
    command: --transaction-isolation=READ-COMMITTED
    ports:
      - "3339:3306"
    networks:
      hktv_ods_network:
        ipv4_address: 172.16.229.11

  other_service_name:
    image: ...
    depends_on:
      - hktv_ods_db
    ...

networks:
  hktv_ods_network:
    ipam:
      driver: default
      config:
        - subnet: 172.16.229.0/24
```
All attribute Ref: https://docs.docker.com/compose/compose-file/
- The `version` top-level attribute can be removed in Compose Specification (most current, and recommended)
  - ref: https://docs.docker.com/compose/compose-file/compose-versioning/
- The `container_name` attribute
  - It specifies a custom container name, rather than a generated default name (eg. `xxx_db_1-1`)
- The `hostname` attribute
  - It just specifies the host name in the network. Can verify the name in container using command `hostname`
- The `env_file` attribute
  - Provide a path to an environment variable file, by default `.env`
    ```yml
    # .env
    MYSQL_ROOT_PASSWORD=HKtv2014
    DB_DATA_PATH=C:\work\db\hktv_ods_db
    DB_URL=hktv_ods_db
    DB_USER=root
    DB_PASSWORD=HKtv2014
    IMAGE_TAG=latest
    ```
  - Set default value of an environment variable if it is unset or empty in the environment `${VARIABLE:-default}` 
- The `build` attribute
  - It will create a new image from source, but will not overwrite the `image` attribute (i.e. still use `image: mysql:5.7.28`). Normally if a `build` subsection is present for a service, no need `image` attribute.
  - The `context` attribute (REQUIRED) defines either a path to a directory containing a Dockerfile, or a url to a git repository.
  - The `dockerfile` attribute allows to set an alternate Dockerfile path (default name is `Dockerfile`)
  - A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image. (also included in official image)
    - ref: https://docs.docker.com/engine/reference/builder
    ```yml
    # Dockerfile
    ARG IMAGE_TAG=1.0.0
    FROM nginx:$IMAGE_TAG

    ENV FOO=/app
    WORKDIR ${FOO}         # create a work directory /app
    ADD settings.xml /root/.m2/settings.xml

    RUN apt-get update \
        && apt-get install -y \
                           curl \
                           wget

    # Dockerfile for DB
    ARG IMAGE_TAG
    FROM mysql:$IMAGE_TAG

    COPY my.cnf /etc/mysql/conf.d/
    COPY sql/schema.sql /docker-entrypoint-initdb.d/schema.sql
    COPY sql/db_data.sql /docker-entrypoint-initdb.d/db_data.sql

    EXPOSE 3306

    # Dockerfile for application
    FROM adoptopenjdk/openjdk11:alpine-jre
    ENV TZ=Asia/Hong_Kong
    ARG JAR_FILE=target/*.jar

    COPY ${JAR_FILE} drdp-backend.jar
    ENTRYPOINT ["java","-jar","/drdp-backend.jar"]

    EXPOSE 8080
    ```
    - `ARG <name>[=<default value>]`
      - The value of an `ARG` variable can be passed from `docker-compose.yml` or `docker build` command. If there is no value passed at build-time, the builder uses the default.
      - Note that `ENV` instruction always override an `ARG` instruction of the same name.
    - `FROM <image>[:<tag>]`
      - Build an image starting from another parent image. A Dockerfile must begin with a `FROM` instruction, which may only be preceded by one or more `ARG` instructions.
    - `RUN <command>`
      - Run command in a shell, which by default is `/bin/sh -c` on Linux
      - If the command is too long, use `\` to continue onto the next line
    - `ADD `
      - Similar to `COPY`, but can unzip file also.
    - `COPY [--chown=<user>:<group>] <src>... <dest>`
      - Copy a file/directory from local host to the container
      - The `<src>` path must be inside the context of the build; you cannot `COPY ../something/something`, because the first step of a `docker build` is to send the context directory (and subdirectories) to the docker daemon.
      - For MySQL container, to initialize the DB, copy the `.sql` files into `/docker-entrypoint-initdb.d` in the container. Any files in this directory ending with `.sql` will be run by `docker-entrypoint.sh` against the MySQL database.
    - `EXPOSE <port> [<port>/<protocol>...]`
      - As a documentation about which ports are intended to be published. It does not actually publish the port.
    - `ENTRYPOINT ["executable", "param1", "param2"]`
      - Configure a container to run as an executable (eg. Java application run as `.jar`/`.war`)
    - `CMD`
      - `CMD ["executable","param1","param2"]` (exec form, this is the preferred form)
      - `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT)
      - Main purpose is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an `ENTRYPOINT` instruction as well.
      - Can only have one `CMD` in a `Dockerfile`. If more than one, only the last `CMD` will take effect.
- The `volumes` attribute
  - `{host path}:{container path}`
  - It mounts the project directory on the host to a directory inside the container **(once change, both host and container update)**
  - It will overwrite the original file/directory in container image
  - Use case: DB container, application container (build JAR/WAR file for tomcat)
- The `command` attribute
  - It runs the command after the container starts, which overrides the default command declared by the container image (i.e. by Dockerfile’s `CMD`)
    - For example, `command: mongod --replSet hktv-pricechart` overrides image's Dockerfile `CMD ["mongod"]`
  - ONLY ONE `command` can be used, if want to execute more than one commands, try concatenate the commands `commandA && commandB`
    ```yml
    # Correct
    command: xxx
  
    # Error
    command:
      - xxx
    ```
- The `ports` attribute
  - `{local machine port}:{container port}`
- The `networks` top level attribute and attribute
  ```yml
  services:
    aaa:
      netowrks:
        - xxx_network
    bbb:
      networks:
        yyy_network:
          ipv4_address: 172.16.213.10

  networks: 
    xxx_network: # default bridge
    yyy_network:
      ipam:
        driver: default
        config:
          - subnet: 172.16.213.0/24
  ```
  - If only specify the name, default driver is a `bridge` with default subnet `192.168.xxx.0/20`, and random IP address for each container
  - If want to specify the static IP address, add `ipam` block in top level, and add `ipv4_address` in attribute level. For example, `172.16.213.1` would be host IP address.

# Docker compose CLI

Ref: https://docs.docker.com/compose/reference/

## Use an alternate compose file / project name
```
docker compose [-f <FILE NAME>] up ...

docker compose [-p <PROJECT NAME>] up ...
```

## Stop containers and remove containers, networks, volumes, and images created by `up`
```
docker compose down [OPTIONS]
```

## Remove stopped service containers
```
docker compose rm [OPTIONS] [SERVICE...]
```

## Build or rebuild services
```
docker compose build [OPTIONS] [SERVICE...]
```
- If you change a service’s Dockerfile or the contents of its build directory, run `docker compose build` to rebuild it.

## Build, (re)create, start, and attach to containers for a service
```
docker compose up [OPTIONS] [SERVICE...]
```
- Note: **You can specify one or more services to build and recreate.**
- `--build` - Build images even it exists **(if dockerfile content changes, add this)**. After image build, the containers will be recreated.
- `--force-recreate` - Recreate containers even if their configuration and image haven't changed.
- `--no-recreate` - If containers already exist, don't recreate them
- `-d / --detach` - Detached mode: Run containers in the background
- `--no-deps` - Don't start linked services

Hints:
- The containers will be recreated if config is changed
- Sometimes you can use a `.bat` file to include `docker compose down` and `docker compose up`, so that you can automate the rebuild process and run some script right after the containers started. Also, you can add specific service in `docker compose up` to create only certain container.

Problem 1:
- After docker compose up, some containers are down with error `no such directory`. To solve it, make sure all `.sh` file is `Unix (LF)` (you can double check it in Notepad++ > Edit > EOL conversion), such that they can be run by `CMD` in a linux container when building image.

## Start / stop containers
```
docker compose start [SERVICE...]
docker compose stop [OPTIONS] [SERVICE...]
```
