FROM composer:2 AS composer

COPY composer.lock composer.json ./
RUN composer install \
        --no-scripts \
        --no-interaction \
        --ignore-platform-reqs \
        --optimize-autoloader

FROM bitnami/php-fpm:8.0-debian-10

ARG USERNAME=www-data

RUN apt-get update && apt-get install -y \
    libpq-dev \
    apache2 \
    supervisor \
    nano \
    --no-install-recommends

RUN a2enmod \
    proxy \
    proxy_fcgi \
    proxy_http \
    ssl \
    rewrite \
    http2

COPY ./docker/apache_fpm_debian/php.ini-production /opt/bitnami/php/etc/conf.d/php.ini

# VirtualHost
COPY ./docker/apache_fpm_debian/000-default.conf /etc/apache2/sites-available/000-default.conf

COPY ./docker/apache_fpm_debian/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY --from=composer /app/vendor /app/vendor

COPY --chown=daemon:daemon . /app

EXPOSE 80 443

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
