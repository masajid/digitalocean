# DigitalOcean Deployment Guide

This guide explains how to deploy the Masajid app to DigitalOcean using Docker and Docker Compose, manage SSL certificates with Nginx, handle backups, and configure DNS.

**Prerequisites:**
- Docker and Docker Compose installed locally
- Docker Hub account
- DigitalOcean account with API token
- Access to domain registrar (for DNS records)

## Setup Droplet

Setup droplet using docker-machine:

Note: Nginx configurations and ssl certificates must be added manually for now.

### Setup DB credentials


```shell
cp .env.example .env
```

### Create droplet

```shell
export DO_TOKEN=<do-token>

docker-machine create \
  --driver=digitalocean \
   --digitalocean-access-token=$DO_TOKEN \
   --digitalocean-image ubuntu-25-04-x64 \
   masajid
```

The default droplet size will be the smallest; you can override with --digitalocean-size if needed.

### Connect to droplet

```shell
docker-machine ssh masajid
```

### Set environment variables

```shell
docker-machine env masajid

eval $(docker-machine env masajid)
```

### Check docker

```shell
docker ps
```

## Setup App

### Initiate Database

```shell
docker-compose --project-name=masajid up -d db

docker-compose --project-name=masajid run --rm app rake db:create db:migrate db:seed

docker-compose --project-name=masajid run --rm app rake content_places:import only=countries
```

### Start Services

```shell
docker-compose --project-name=masajid up -d app sidekiq cron_job nginx
```

Build docker app image and push it to docker hub (See deployment section)


### Configure nginx

```shell
docker exec -it masajid-nginx-1 bash
vim /etc/nginx/nginx.conf
nginx -t
service nginx status
service nginx reload # or nginx -s reload
```

### Cleanup droplet

Stop and remove droplet (Please don't do unless you know what you are doing :)

```shell
docker-machine stop masajid

docker-machine rm masajid
```

## Deployment

Github actions will build and push the image to docker hub after pushing changes to production branch in masajid repo.

Then run new deployment

```shell
docker pull masajidworld/masajid:latest

docker-compose --project-name=masajid up --no-deps -d app sidekiq cron_job

docker-compose --project-name=masajid run --rm app rake db:migrate
```

## Backups

### Backup database

```shell
DB_CONTAINER=<db-container-id>
docker exec $DB_CONTAINER pg_dump -a -U deploy masajid > ../masajid/dump_masajid_`date +%d-%m-%Y"_"%H_%M_%S`.sql
```

### Restore to localhost database

```shell
LOCAL_CONTAINER=<local-container-id>
docker exec -i $LOCAL_CONTAINER psql -U postgres -d masajid_development < dump_masajid_dd-mm-yyyy_hh_mm_ss.sql
```

### Backup uploads

```shell
APP_CONTAINER=<app-container-id>
docker cp $APP_CONTAINER:/app/web_container/storage ../masajid/web_container
```

### Backup volumes

```shell
scp -r root@159.203.71.141:/var/lib/docker/volumes/_masajid* ~/workspace/masajid/backups/volumes_`date +%d-%m-%Y"_"%H_%M_%S`
```


### Restore volumes to the server

```shell
scp -r ~/workspace/masajid/backups/volumes_dd-mm-yyyy_hh_mm_ss/masajid_* root@159.203.71.141:/var/lib/docker/volumes
```

## DNS configuration

### masajid.world

| Type  | Host                     | Value                                  | TTL  | Purpose                           |
|-------|--------------------------|----------------------------------------|------|-----------------------------------|
| A     | @                        | 159.203.71.141                         | 3600 | Root domain points to server      |
| CNAME | *                        | masajid.world                          | 60   | Wildcard subdomains               |
| TXT   | _acme-challenge          | <ACME token>                           | 60   | LetsEncrypt DNS verification      |
| NS    | masajid.world            | ns1.digitalocean.com                   | 1800 | DigitalOcean nameservers          |
| NS    | masajid.world            | ns2.digitalocean.com                   | 1800 |                                   |
| NS    | masajid.world            | ns3.digitalocean.com                   | 1800 |                                   |


### ikg-zeven.de

| Type  | Host                     | Value                                  | TTL  |
|-------|--------------------------|----------------------------------------|------|
| A     | @                        | 159.203.71.141                         | 3600 |
| TXT   | _acme-challenge          | <ACME token>                           | 60   |
| TXT   | _acme-challenge.www      | <ACME token>                           | 60   |

### Notes

For troubleshooting DNS or certificate issues

```shell
dig ikg-zeven.masajid.world +short

dig TXT _acme-challenge.masajid.world +short
```
