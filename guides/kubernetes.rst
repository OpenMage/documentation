Kubernetes
######

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


.. toctree::
   :glob:
   :maxdepth: 1

   kubernetes/*


