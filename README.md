# CASA

## Container as a Service Admin

| [Demo](https://knrdl.github.io/casa/)  | [Docker Hub](https://hub.docker.com/r/knrdl/casa) [![Docker Hub](https://img.shields.io/docker/pulls/knrdl/casa.svg?logo=docker&style=popout-square)](https://hub.docker.com/r/knrdl/casa) | [![CI](https://github.com/knrdl/casa/actions/workflows/docker-image.yml/badge.svg)](https://github.com/knrdl/casa/actions/workflows/docker-image.yml)
| ----------- | ----------- | ----------- |

Outsource the administration of a handful of containers to your co-workers.

CASA provides a simple web-interface to handle basic container admin tasks:

* View resource consumption/runtime behaviour
* Restart, Stop containers
* View logs and process tree
* Execute terminal commands
* Browse filesystem, upload/download files

Restrict permissions per container and user

## Getting started

### 1. Deploy CASA

```yaml
version: '2.4'
services:
  casa:
    image: knrdl/casa
    restart: always
    environment:
      ROLES_casa_admin_basic: info, state, logs, procs, files, files-read
      ROLES_casa_admin_full: info, info-annotations, state, logs, term, procs, files, files-read, files-write
      AUTH_API_URL: https://identity.mycompany.com/login
      AUTH_API_FIELD_USERNAME: username
      AUTH_API_FIELD_PASSWORD: password
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    mem_limit: 150m
    cpu_count: 1
```

> :warning: **For production** is a reverse-proxy with TLS termination in front of CASA highly recommended

Roles are defined via environment variables and might contain these permissions:

* **info**: display basic container metadata
* **info-annotations**: display environment variables and container labels
* **state**: allow start, stop, restart container
* **logs**: display container terminal output
* **term**: spawn (root privileged) terminal inside container
* **procs**: display running processes
* **files**: list files and directories in container
* **files-read**: user can download files from container
* **files-write**: user can upload files to container

### 2. Authentication

To perform logins CASA sends HTTP-POST requests to `AUTH_API_URL` containing a JSON body with
fields `AUTH_API_FIELD_USERNAME` and `AUTH_API_FIELD_PASSWORD`. A 2XX response code (e.g. *200 OK*) represents a
successful login.

> For tests/demos you can point `AUTH_API_URL` to e.g. https://example.org and use any username/password. ***Do not use real credentials then!***

### 3. Annotate containers

If a container should be visible in CASA, it must be annotated with a label defined above as `ROLES_<labelname>` and
list all permitted user IDs

```bash
docker run -it --rm --name casa_demo --label casa.admin.full=user1,user2 nginx:alpine
```

In this example the users `user1` and `user2` are granted the rights of the `casa.admin.full` role for the
container `casa_demo` via CASA web interface.

## Screenshot

![Screenshot](screenshot.png)
