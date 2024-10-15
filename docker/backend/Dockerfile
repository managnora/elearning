FROM php:7.4-fpm-bullseye AS elearning_php_fpm_base

ENV PHP_MEMORY_LIMIT 4G

#Install Locales
RUN apt-get update
RUN apt-get install -y --no-install-recommends --no-install-suggests locales
RUN sed -i '/fr_FR.UTF-8/s/^# //g' /etc/locale.gen && \
    locale-gen
ENV LANG fr_FR.UTF-8
ENV LANGUAGE fr_FR:fr
ENV LC_ALL fr_FR.UTF-8
ENV LC_CTYPE fr_FR.UTF-8

# Install System dependencies
RUN apt-get install -y --no-install-recommends --no-install-suggests\
    libpq-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libicu-dev \
    libxml2-dev \
    libmagickwand-dev \
    libmagickcore-dev \
    libldap2-dev \
    libzip-dev


# Install required PHP extensions
RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql
RUN docker-php-ext-configure gd --with-jpeg --with-freetype
RUN docker-php-ext-configure intl

RUN docker-php-ext-install \
    pdo pdo_pgsql pdo_mysql\
    gd \
    intl \
    soap \
    zip \
    ldap

# Configure PHP INI
COPY docker/backend/php-fpm.conf /usr/local/etc/
COPY docker/backend/conf.d/custom.ini /usr/local/etc/php/conf.d/zz-local.ini
COPY docker/backend/conf.d/opcache.ini /usr/local/etc/php/conf.d/opcache.ini

RUN mkdir -p "/var/www/elearning/public"

WORKDIR /var/www/elearning

EXPOSE 9000

# Install apache
RUN apt-get install -y --no-install-recommends --no-install-suggests \
    apache2 \
    supervisor \
    # && service apache2 stop 2>&1 \
    # && update-rc.d apache2 disable 2>&1 \
    && sed -ri ' \
        s!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g; \
        s!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g; \
        ' /etc/apache2/apache2.conf \
    && rm -f /etc/apache2/sites-enabled/* \
    # modifies Apache modules
    && a2dismod \
        mpm_prefork mpm_worker \
    && a2enmod \
        actions autoindex brotli cache deflate expires headers http2 \
        mpm_event proxy proxy_fcgi remoteip rewrite setenvif ssl unique_id

RUN mkdir -p /var/log/supervisor /var/lock/apache2 /var/run/apache2
COPY docker/backend/vhosts/* /etc/apache2/sites-enabled/
COPY docker/backend/supervisor.d/supervisor.conf /etc/supervisor/supervisor.conf
COPY docker/backend/supervisor.d/* /etc/supervisor/conf.d/

CMD ["supervisord"]

EXPOSE 80

######### Dev Dependencies ############
FROM elearning_php_fpm_base AS elearning_php_fpm_dev

# Install System dependencies
RUN apt-get install -y --no-install-recommends --no-install-suggests \
    git

# Install and configure Xdebug
RUN pecl channel-update pecl.php.net \
  && pecl install -o -f \
  xdebug-3.1.6

RUN docker-php-ext-enable xdebug

COPY docker/backend/conf.d/xdebug.ini /usr/local/etc/php/conf.d/zz-xdebug-settings.ini

# Install Composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
	&& php composer-setup.php --version=2.5.8\
	&& php -r "unlink('composer-setup.php');" \
	&& mv composer.phar /usr/bin/composer

# File mode/permissions trick
RUN groupadd --gid 1001 elearningdev \
    && useradd --uid 1001 --gid elearningdev --shell /bin/bash --create-home --home /var/www/elearning elearningdev \
    && chown -R elearningdev /var/www/elearning \
    && usermod -a -G www-data elearningdev

# RUN chown -R ${DOCKER_USER_NAME}:${DOCKER_GROUP_NAME} /var/log/supervisor /var/log/apache2 /var/lock/apache2 /var/run/apache2

# USER ${DOCKER_USER_NAME}:${DOCKER_GROUP_NAME}

FROM elearning_php_fpm_dev AS elearning_php_fpm_builder

COPY --chown=elearningdev . "/var/www/elearning"

RUN composer install --no-dev

RUN php artisan storage:link
RUN php artisan view:cache
RUN php artisan config:cache
RUN php artisan route:cache
RUN php artisan optimize

FROM elearning_php_fpm_base AS elearning_php_fpm_run

COPY --from=elearning_php_fpm_builder --chown=www-data "/var/www/elearning" "/var/www/elearning"
