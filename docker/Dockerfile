FROM php:8.0-fpm-alpine

ARG APP_HOME=/app
ARG UID=1000
ARG GID=1000
ARG USERNAME=www-data

RUN cd / \
    && apk add postgresql-dev imap-dev krb5-dev supervisor nginx shadow $PHPIZE_DEPS && \
    pecl install redis && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install pdo pdo_pgsql pcntl imap && \
    docker-php-ext-enable imap redis

#COPY ./docker/app/prod/apm-agent-php_1.5_all.apk /
#RUN cd / && apk add --allow-untrusted apm-agent-php_1.5_all.apk && rm apm-agent-php_1.5_all.apk

# create document root, fix permissions for www-data user and change owner to www-data
RUN mkdir -p $APP_HOME/public && \
    usermod -u $UID $USERNAME -d /home/$USERNAME \
    && groupmod -g $GID $USERNAME \
    && chown -R ${USERNAME}:${USERNAME} $APP_HOME

COPY ./docker/site.conf /etc/nginx/http.d/default.conf

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY --chown=root:root ./docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR $APP_HOME

COPY --chown=${USERNAME}:${USERNAME} . $APP_HOME/
#COPY --chown=${USERNAME}:${USERNAME} ./docker/app/prod/index.php $APP_HOME/public/

RUN composer install --optimize-autoloader --no-interaction --no-progress --no-dev

EXPOSE 80/tcp
EXPOSE 8080/tcp
EXPOSE 443/tcp

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
