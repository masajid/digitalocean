# digitalocean

App deployment to digitalocean

## Setup

Setup droplet using docker-machine:

Initial steps:

- copy `.env` file to root folder and change the environment variables

```
$ export DO_TOKEN=<do-token>

$ docker-machine create \
  --driver=digitalocean \
   --digitalocean-access-token=$DO_TOKEN \
   --digitalocean-size=1gb \
   masajid

$ docker-machine ssh masajid

$ docker-machine env masajid
$ eval $(docker-machine env masajid)

$ docker ps

$ export RAILS_MASTER_KEY=$(cat ../masajid/web_container/config/master.key)

$ docker-compose --project-name=masajid up -d db
$ docker-compose --project-name=masajid build --build-arg="RAILS_MASTER_KEY=${RAILS_MASTER_KEY}" app
$ docker-compose --project-name=masajid run --rm app rake db:create db:migrate db:seed
$ docker-compose --project-name=masajid run --rm app rake content_places:import only=countries
$ docker-compose --project-name=masajid up -d app sidekiq cron_job nginx
```

Stop and remove droplet:

```
$ docker-machine stop masajid
$ docker-machine rm masajid
```

## Deployment

```
$ docker-compose --project-name=masajid build --build-arg="RAILS_MASTER_KEY=${RAILS_MASTER_KEY}" app sidekiq cron_job
$ docker-compose --project-name=masajid up --no-deps -d app sidekiq cron_job
$ docker-compose --project-name=masajid run --rm app rake db:migrate
```


## Backups

Backup database:

```
$ docker exec <serverContainerId> pg_dump -a -U deploy masajid > ../masajid/dump_masajid_`date +%d-%m-%Y"_"%H_%M_%S`.sql
```

Restore to localhost database:

```
$ docker exec -i <localContainerId> psql -U postgres -d masajid_development < dump_masajid_dd-mm-yyyy_hh_mm_ss.sql
```

Backup uploads:

```
$ docker cp <containerId>:/app/web_container/storage ../masajid/web_container
```
