Kubernetes
##########

preamble
========

Kubernetes is a highly complex approach to hosting with a limited number of cases where it offers an actual benefit.

As Kubernetes is still evolving and changing a lot, we will only give hints about areas.
If you search for a way to setup a cluster or application, you should start with Kubernetes own Documentation.

Usual cases where Kubernetes is used for OpenMage are:
- hosting companies which primarily offer Kubernetes
- Agencies which have a lot of customers with an offered capability to provide scaling in number of instances.
- big companies with existing kubernetes based hosting infrastructure
- companies with a big number of additional application services and/or a high number of developers

When you have to maintain and update the kubernetes setup yourself, make sure you have people with deep knowledge in
- networking
- docker internals
- docker in general
- go language
- Filesystems and IO
- build and deployment pipelines

If you are just using Kubernetes provided as a managed service/platform, the needed knowledge is reduced, but still
needed to build the docker containers.




General Informations
====================


**Kubernetes** :doc:`https://kubernetes.io/docs/home/`.

**Image build:**

- Custom built images based on `nginx:alpine <https://https://hub.docker.com/_/nginx>`_ and `php:fpm-alpine <https://hub.docker.com/_/php>`_ (for reverse proxy)
	- stage step with `composer` image to git pull or dockerfile COPY app and `composer install` images locally (for both nginx and php dockerfiles)
	- on PHP image dockerfile
		- `COPY --chown="www-data"` app from `composer` to this image
		- install any required php libs etc
		- (optional) `COPY` here custom php.ini files inside image, from your host
	- on NGINX image dockerfile
		- `COPY --chown="nginx"` app from `composer` to this image
		- (optional) `COPY` here custom server.conf or *.nginxtemplate files inside image, from your host
- push images to preferred repository


I guess the above is app agnostic for php apps. So we have 2 images. 1 for php-fpm and 1 for nginx.
In this stage with an added mysql service to docker-composer.yml we can quickly test locally for compatibilities.
Best practice because the whole app is bundled in image (then just do a cluster apply ____?)

**The cluster** `should` **have:**

- an admin pod with 2 containers (phpfpm + nginx) and a cluster ip service which maybe accessed by `admin123.example.com`
- an frontend horizontally auto-scaled pod with 2 containers (phpfpm + nginx) accessed by `example.com`
	- admin & frontend pods could have an volumeMount of a configMap containing php .ini and nginx .conf files (even nginx templates) mounted on php and nginx containers respectively
	- local.xml as stringData opaque secret mounted on the php containers
- (opt) phpmyadmin container on the admin pod or as separate pod (?)
- database (mysql or mariadb) as separate pod, with persistentVolumeClaim OR remote sql config on local.xml (remote sql adds latencies?)
- (opt) auto-generation of lets-encrypt certs, otherwise storage mounts for custom certs in nginx container
- (opt) redis server pod (probably required instead of having persistentVolume for sharing sessions between pods)
- log collection + monitoring (ex. graylog, elastic?) + private endpoints
- php-fpm pod for cronjobs (? per minute run but monitor if another cronjob already runs)


Common Approaches for Pods
==========================

There are 3 core services for a Magento instance, which are needed and relevant for a setup

Database
--------

The Database is the hardest part in such a setup for OpenMage, as the architecture is strongly focused
on a single database instance, which also in a source/replica setup means to have only a single active source usually.

In a managed Kubernetes this is usually something which is somehow provided or preconfigured. In other cases you will
need to think about how much fail safety you need and want.

There are also MySql variants which are better suited for such environments like Galera Cluster, which make it easier
to build multi Node MySql setups, even for such cases like OpenMage.

Of course using kubernetes does also allow to use a database hosted/managed outside of it.

single Pod/Container sharing Node with Volume
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For simplicity this is the most common approach to databases with OpenMage and Kubernetes.
You only deploy a single Pod on the same Node, where the Volume is persisted.

You need to take extra care with Backups here, as a failure of the node, als loses the Volume.
It is the equivalent to hosting a database directly on a server (or also an AWS instance).

single Pod/Container with separate persistent Volume
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Same as the previous one, but with a volume which is independent of the node.
It makes it easier to move Pods to different Nodes, and the persisting is reliable.

Depending on the used type of Storage you will have different performance regarding bandwith, IOPS and latency.
Also the bootstrap time of the database can vary a lot.


single source Container, multiple replica Containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you need the ability for a fast failover or improved scalability by read-only Database instances, this one is for you.


Web
---

This part covers the php and webserver part of the application.

Besides some exceptions, the directories are part of a common build docker image.
Exceptions are: etc, media, var

The media directory is usually a mounted persisted volume.

shared Pod
~~~~~~~~~~

One of the possible approaches is to have a shared Pod which runs both, the webserver and the php part.
This can be nginx and php-fpm, or apache with mod-php, or any other combination.

This is a common approach, as OpenMage shares static assets with php code in a common directory structure.
In most case no big downsides, as nginx doesnt require many resources compared to php.

separate webserver and php
~~~~~~~~~~~~~~~~~~~~~~~~~~

same as above but in different pods.
Useful when both need to be scalable independently.


Others
------

Some other common Pods

- Frontend Cache: Varnish
- ssl Termination: additional nginx (in front of the Frontend Cache)
- redis




.. toctree::
   :glob:
   :maxdepth: 1

   kubernetes/*


