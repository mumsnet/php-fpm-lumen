# PHP image with Lumen

This repo is used to generate a base PHP image that includes Lumen, the lean and fast subset of Laravel, mainly
used for APIs.  All the usual extensions are already built into the image, including support for mysql.  
`composer` is also installed in the image to allow you to manage your packages within Docker.  Currently 
PHP version 7.3 is supported, but new versions will be added as they become stable.

## How to create a new Lumen microservice based on this image

Let's say you want to create a new service called `testlumen1` to work alongside our other microservices.

First create your top level dev folder which will be your repo in github:

```bash
mkdir ~/dev/testlumen1_service
cd ~/dev/testlumen1_service
```

Next create a `Dockerfile` like this:

```Dockerfile
FROM mumsnet/php-fpm-lumen:7.3

# Install whatever extra software you need
# RUN apt-get install -y ...

# No need for a final CMD.  It is already part of the base image.
```

Build your docker image and run Lumen to create your new application skeleton:

```bash
docker build -t testlumen1 .
docker run -it -v `pwd`:/var/www/html testlumen1 lumen new testlumen1
```

Configure Lumen:
```bash
cd testlumen1
cp .env.example .env
```

Edit the `.env` file and set `APP_NAME=Testlumen1`, `APP_KEY=somerandomstring`,
and `APP_URL=https://lhYOURNAME.devmn.net/service/testlumen1`

Then edit `routes/web.php` and wrap all the routes in a group like this:

```php
$router->group(['prefix' => 'service/testlumen1'], function () use ($router) {
    // all existing routes eg:
    $router->get('/', function () use ($router) {
        return $router->app->version();
    });
});

```

Next create your `docker-compose.yml` like this:

```yaml
version: '3'

services:
  web:
    container_name: testlumen1
    build: .
    volumes:
      - ./testlumen1:/var/www/html
networks:
  default:
    external:
      name: mn_network
```

Finally add the relevant nginx entry to your router service `dev_server.conf` file:

```nginx
location ~ ^/(service/testlumen1) {
    proxy_set_header Host ${MY_HOSTNAME};
    proxy_pass http://testlumen1:8080;
    include proxy_directives.conf;
}
```

Start your new testlumen1 service and restart the router service and you should be able to access it:

https://lhYOURNAME.devmn.net/service/testlumen1
