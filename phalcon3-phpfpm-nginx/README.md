To use this image create a Dockerfile and fetch the image. 
Add your custom Nginx and PHP-config and you'r done.


```
FROM baboons/phalcon3-phpfpm-nginx:latest

COPY docker/nginx/conf.d/default.conf /etc/nginx/sites-enabled/default
COPY docker/php/php.ini /etc/php/7.1/fpm/php.ini
COPY docker/php/www.conf /etc/php/7.1/fpm/pool.d/www.conf```
