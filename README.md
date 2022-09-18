# digitalocean

App deployment to digitalocean

## Setup

Setup droplet using docker-machine:

Note: Nginx configurations and ssl certificates must be added manually for now.

Initial steps:

- copy `.env` file to root folder and change the database environment variables

```
$ export DO_TOKEN=<do-token>

$ docker-machine create \
  --driver=digitalocean \
   --digitalocean-access-token=$DO_TOKEN \
   --digitalocean-image ubuntu-20-04-x64 \
   masajid

# --digitalocean-size not needed, by default will be the smallest Droplet

$ docker-machine ssh masajid

$ docker-machine env masajid
$ eval $(docker-machine env masajid)

$ docker ps

$ docker-compose --project-name=masajid up -d db
# build docker image and push it to docker hub (See deployment section)
$ docker-compose --project-name=masajid run --rm app rake db:create db:migrate db:seed
$ docker-compose --project-name=masajid run --rm app rake content_places:import only=countries
$ docker-compose --project-name=masajid up -d app sidekiq cron_job nginx
```

Setup nginx

```
$ docker exec -it masajid-nginx-1 bash
$ vim /etc/nginx/nginx.conf
$ nginx -t
$ service nginx status
$ service nginx reload # or nginx -s reload
```

Stop and remove droplet (Please don't do unless you know what you are doing :)

```
$ docker-machine stop masajid
$ docker-machine rm masajid
```

## Deployment

Github actions will build and push the image to docker hub after pushing changes to production branch in masajid repo.

Then run new deployment

```
$ docker pull masajidworld/masajid:latest
$ docker-compose --project-name=masajid up --no-deps -d app sidekiq cron_job
$ docker-compose --project-name=masajid run --rm app rake db:migrate
```

## Backups

Backup database:

```
$ docker exec <server-container-id> pg_dump -a -U deploy masajid > ../masajid/dump_masajid_`date +%d-%m-%Y"_"%H_%M_%S`.sql
```

Restore to localhost database:

```
$ docker exec -i <local-container-id> psql -U postgres -d masajid_development < dump_masajid_dd-mm-yyyy_hh_mm_ss.sql
```

Backup uploads:

```
$ docker cp <app-container-id>:/app/web_container/storage ../masajid/web_container
```
