# Odoo base image

> Odoo base image provides persistent volumes on OpenEBS for addons and filestore and a single postgres container.

## What is Odoo?

> Odoo is a suite of web based open source business apps. Odoo Apps can be used as stand-alone applications, but they also integrate seamlessly so you get a full-featured Open Source ERP when you install several Apps.

https://odoo.com/

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/coobyHQ/coobyHQ-docker-odoo/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Use Cooby Images?

* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.
* Cooby  container images are released frequently with the latest distribution packages available.

[![Anchore Image Overview](tbd)](tbd#security)

> The image overview badge contains a security report with all open CVEs. Click on 'Show only CVEs with fixes' to get the list of actionable security issues.

# How to deploy Odoo in Kubernetes?

Deploying Cooby  applications as Helm Charts is the easiest way to get started with our applications on Kubernetes. Read more about the installation in the [Cooby  Odoo Chart GitHub repository](https://github.com/coobyHQ/charts/tree/master/upstreamed/odoo).

Cooby  containers can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters.

# Supported tags and respective `Dockerfile` links

* [`11-ol-7`, `11.0.20181215-ol-7-r4` (11/ol-7/Dockerfile)](https://github.com/coobyHQ/cooby-docker-odoo-base/blob/11.0.20181215-ol-7-r4/11/ol-7/Dockerfile)
* [`11-debian-9`, `11.0.20181215-debian-9-r2`, `11`, `11.0.20181215`, `11.0.20181215-r2`, `latest` (11/debian-9/Dockerfile)](https://github.com/coobyHQ/cooby-docker-odoo-base/blob/11.0.20181215-debian-9-r2/11/debian-9/Dockerfile)


# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recommended with a version 1.6.0 or later.

# How to use this image

## Run Odoo with a Database Container

Running Odoo with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run Odoo. You can use the following docker compose template:

```yaml
version: '2'

services:
  postgresql:
    image: 'bitnami/postgresql:latest'
    volumes:
      - 'postgresql_data:/bitnami'
  odoo:
    image: 'coobytec/odoo-base:latest'
    ports:
      - '80:8069'
      - '443:8071'
    volumes:
      - 'odoo_data:/cooby'
    depends_on:
      - postgresql
volumes:
  postgresql_data:
    driver: local
  odoo_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create odoo-tier
  ```

2. Start a PostgreSQL database in the network generated:

  ```bash
  $ docker run -d --name postgresql --net odoo-tier bitnami/postgresql:latest
  ```

  *Note:* You need to give the container a name in order to Odoo to resolve the host

3. Run the Odoo container:

  ```bash
  $ docker run -d -p 80:8069 -p 443:8071 --name odoo --net odoo-tier coobytec/odoo-base:latest
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the PostgreSQL data](https://github.com/bitnami/bitnami-docker-postgresql#persisting-your-database).

The above examples define docker volumes namely `postgresql_data` and `odoo_data`. The Odoo application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:

```yaml
version: '2'

services:
  postgresql:
    image: 'bitnami/postgresql:latest'
    volumes:
      - '/path/to/postgresql_persistence:/bitnami'
  odoo:
    image: coobytec/odoo-base:latest
    depends_on:
      - postgresql
    ports:
      - 80:8069
      - 443:8071
    volumes:
      - '/path/to/odoo-persistence:/bitnami'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create odoo-tier
  ```

2. Create a PostgreSQL container with host volume:

  ```bash
  $ docker run -d --name postgresql \
    --net odoo-tier \
    --volume /path/to/postgresql-persistence:/bitnami \
    bitnami/postgresql:latest
  ```

  *Note:* You need to give the container a name in order to Odoo to resolve the host

3. Create the Odoo container with hist volumes:

  ```bash
  $ docker run -d --name odoo -p 80:8069 -p 443:8071 \
    --net odoo-tier \
    --volume /path/to/odoo-persistence:/bitnami \
    coobytec/odoo-base:latest
  ```

# Upgrade this application

Cooby  provides up-to-date versions of PostgreSQL and Odoo, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Odoo container. For the PostgreSQL upgrade see https://github.com/bitnami/bitnami-docker-postgresql/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull coobytec/odoo-base:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop odoo`
 * For manual execution: `$ docker stop odoo`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/odoo-persistence /path/to/odoo-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the PostgreSQL data](https://github.com/bitnami/bitnami-docker-postgresql#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the stopped container

 * For docker-compose: `$ docker-compose rm odoo`
 * For manual execution: `$ docker rm odoo`

5. Run the new image

 * For docker-compose: `$ docker-compose up odoo`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name odoo coobytec/odoo-base:latest`

# Configuration

## Environment variables

When you start the odoo image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
odoo:
  image: coobytec/odoo-base:latest
  ports:
    - 80:8069
    - 443:8071
  environment:
    - ODOO_PASSWORD=my_password
  volumes:
    - 'odoo_data:/bitnami'
  depends_on:
    - postgresql
```

 * For manual execution add a `-e` option with each variable and value:

  ```bash
  $ docker run -d -p 80:8069 -p 443:8071 --name odoo \
    --env ODOO_PASSWORD=my_password  \
    --net odoo-tier \
    --volume /path/to/odoo-persistence:/bitnami \
    coobytec/odoo-base:latest
  ```

Available variables:
 - `ODOO_EMAIL`: Odoo application email. Default: **user@example.com**
 - `ODOO_PASSWORD`: Odoo application password. Default: **bitnami**
 - `POSTGRESQL_USER`: Root user for the PostgreSQL database. Default: **postgres**
 - `POSTGRESQL_PASSWORD`: Root password for the PostgreSQL.
 - `POSTGRESQL_HOST`: Hostname for PostgreSQL server. Default: **postgresql**
 - `POSTGRESQL_PORT_NUMBER`: Port used by PostgreSQL server. Default: **5432**

### SMTP Configuration

To configure Odoo to send email using SMTP you can set the following environment variables:
 - `SMTP_HOST`: SMTP host.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.
 - `SMTP_TLS`: Use TLS encription with SMTP. Default **true**

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:

```yaml
  odoo:
    image: coobytec/odoo-base:latest
    ports:
      - 80:8069
      - 443:8071
    environment:
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
```

 * For manual execution:

  ```bash
  $ docker run -d -p 80:8069 -p 443:8071 --name odoo \
    --env SMTP_HOST=smtp.gmail.com \
    --env SMTP_PORT=587 \
    --env SMTP_USER=your_email@gmail.com \
    --env SMTP_PASSWORD=your_password \
    --net odoo-tier \
    --volume /path/to/odoo-persistence:/bitnami \
    coobytec/odoo-base:latest
  ```

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/coobyHQb/cooby-docker-odoo-base/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-odoo/pulls) with your contribution.


# License

Forked from Bitnami Docker Odoo https://github.com/bitnami/bitnami-docker-odoo, and got heavily reshaped
Copyright 2018 Cooby tec

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

  <http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
