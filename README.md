# Boostrapping stateful webapp with Docker

Docker meetup 28.9.2015

## What

Describing a stateful webapp with compose as easy as possible. Then bootstrapping it. Using Docker container for bbootstrapping tasks and also for data management.

## Why

It's useful to have a state of webapp declaratively described because
 - it's easy to setup development environment of some particular version
 - it makes CI simple
 - it makes deployment easier in general

Loading static files and db with another container makes it appear more compact in my opinion.

## How

The web app - Drupal 8 beta. State of a Drupal site consists of code, db, and static files. We thus need to store/restore/describe all 3 to be able to boostrap a site.


### Store state

Code comes in Docker image t0mk/drupal8demo. I don't show how I build the image in the demo as there is many other resources on it. It's based on this: https://github.com/t0mk/docker-drupal8/tree/master/apache-dev plus the Drupal web root.

Posible approaches: burn the code in image, mount from host, clone on container run.

SQL dump comes from tarball. The tarball contains 1..n files with names corresponding to the partcular database name. I.e. in order to load the tabrall we must:

```
extract tarball to /tmp/tmp.Snt1NAkF2m
cd /tmp/tmp.Snt1NAkF2m
for f in *; do
    create db ${f}
    load dump from ${f} to db ${f}
done
```

The static files are just a directory tree, it can be also loaded from a tarball.


### Storing and restoring logic

In a Docker image: https://github.com/t0mk/docker-s3

![composition](http://i.imgur.com/a09zuuF.png)

S3 credentials in envvar file. Also possible to download and store to a local file, and download from HTTP.

### Usage

You can try this on your own.

```
git clone https://github.com/t0mk/boot_stateful_web.git
cd boot_stateful_web
docker-compose up -d
# check output of containers
# visit http://localhost:6666

```

### Dev setup

Mount a host volume from your webroot (e.g. ./www) to /app of the Drupal container.

```
diff -u docker-compose.yml dev.yml
docker-compose -f dev.yml up -d
# modify sth in www/
```


## Discussion

- What do you think of this scenario - does it capture a real-life use case?
- What would you do differently? Maybe not S3 but ..., or not docker-compose but ... 
- IMO Handling data like this will be obsolete when volumes become truly independent on containers (or vice versa?). Like this but with volumes wherever: https://coreos.com/blog/Flocker-on-CoreOS-Linux/. Then we'll be hopefully able to version and move around whole volumes. Has anybody tried Flocker?

## Links

- How I build the Drupal container: https://github.com/t0mk/d8.git
- 
