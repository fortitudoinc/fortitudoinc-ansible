# Fortitudo OpenMRS Infrastructure

[![Build Status](https://travis-ci.org/fortitudoinc/fortitudoinc-infra.svg?branch=master)](https://travis-ci.org/fortitudoinc/fortitudoinc-infra)

This repo contains the Fortitudo OpenMRS docker deployment

## Overview

The software stack consists of three docker containers: an nginx TLS proxy (with TLS certificates managed by letsencypt), a tomcat server with OpenMRS reference application installed, and a MySQL 5.6 database backend.

![software stack](stack.png)

## Requirements
- docker
- docker-compose
- python3 (optional)

## Usage

The first-time run of this infrastructure is slightly different in that the `DB_CREATE_TABLES` environment variable must be set to true (as in `environments/example-new-db.env`). This lets the refapp container know that it has to do a little extra installation work because it is using a completely blank database.

Thereafter, the `DB_CREATE_TABLES` environment must be set to false (as in `environments/example.env`) to let the refapp know that those tables already exist.

### First-time setup

```bash
source environments/example-new-db.env
docker-compose up --build -d
```

### Subsequent updates/operations

```bash
source environments/example.env
docker-compose down
# Make some changes..
docker-compose up --build -d
```

## Adding custom modules (omod files)

To launch the reference application with custom modules, drop your .omod files into `./refapp/omods/`. In addition, append the `--build` flag to the docker-compose command, i.e. `docker-compose up -d --build`

## Backups

If the backup environment variables are pointing to a valid directory (see below), the backup script under `util/backup.py` can be used to automatically take a database snapshot. This can be setup in a cronjob, for example, to do automated database backups.

## Mobile Gateway

The mobile gateway is a small flask app that accepts traffic from the fortitudoinc mobile app, which prospective patients can use to self-register.

## Environment variables

See `environments/` directory for examples of how to set up in each situation.

### MySQL Options
- MYSQL_ROOT_PASSWORD: password for the root user of the mysql 5.6 database
- MYSQL_DATABASE: A database to setup when creating the mysql container

### OpenMRS Options
- DB_HOST: Address of database server (must specify if using existing-db.yml, ignored otherwise)
- DB_DATABASE: The database used
- DB_USERNAME: The username used to connect to the database container
- DB_PASSWORD: The password used  to connect to the database container
- DB_CREATE_TABLES: (`true`/`false`) Whether new tables should be created or existing ones used
- MODULE_WEB_ADMIN: (`true`/`false`) Whether to allow modules to be uploaded via the web interface
- OPENMRS_ADMIN_PASSWORD: Administrative password (must be a combination of upper case, lower case, and digits)

### Nginx Options
- URL: The FQDN to use for the TLS certificate
- STAGING: (`true`/`false`)
    - If true: will use a non-valid certificate, but will not be rate-limited by letsencrypt service (use this option for testing)
    - If false: will use a valid certificate, but will be rate-limited by letsencrypt (use this option for production deployments)
- TZ: optional time zone
- EMAIL: an email address for the server administrator (for letsencrypt security notices, etc.)

### Backup Options (optional, for use with backup.py as cron job)
- BACKUP_PATH: Path to a directory used to store db backups 
- BACKUP_RETENTION: Number of backups to hold on the file system before deleting the oldest