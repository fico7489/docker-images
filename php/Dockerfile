FROM php:7.4-fpm

ARG PHP_PATH
ARG TIMEZONE

ENV PHP_PATH=${PHP_PATH}
ENV TIMEZONE=${TIMEZONE}

RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        rsync \
        optipng \
        jpegoptim \
        openssl \
        git \
        unzip \
        libzip-dev \
        zip \
        supervisor \
        imagemagick \
        libjpeg-dev \
        libmagickwand-dev \
        xvfb \
        wkhtmltopdf \

    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

#install cron
RUN apt-get update && apt-get -y install cron
RUN touch /var/run/crond.pid
RUN chmod 777 /var/run/crond.pid

#install ps
RUN apt-get update && apt-get install -y procps

RUN docker-php-ext-install pdo pdo_mysql intl bcmath opcache gd pcntl posix exif zip
RUN pecl install imagick
RUN docker-php-ext-enable imagick

RUN pecl install -f igbinary && \
    pecl bundle redis && cd redis && phpize && ./configure --enable-redis-igbinary && make && make install && \
    docker-php-ext-enable igbinary redis

#composer
ENV COMPOSER_ALLOW_SUPERUSER=1
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone \
    && printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini \
    && "date"

RUN echo 'memory_limit = 1024M' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini;

RUN groupadd -g 1000 php && useradd -u 1000  -g php -s /bin/sh php

# PHP ini overrides
COPY ${PHP_PATH}/php-ini-overrides.ini /usr/local/etc/php/conf.d/custom.ini

RUN mkdir /home/php
RUN chown php:php -R /home/php
RUN chown -R php:php /var/www/html/

RUN mkdir /etc/crontabs
COPY ${PHP_PATH}/cron/cronjobs /etc/crontabs/root
COPY ${PHP_PATH}/supervisor/supervisord.conf /etc/supervisord.conf

RUN touch /tmp/supervisor.sock
RUN chmod 777 /tmp/supervisor.sock
RUN chown -R php:php /tmp/supervisor.sock

USER php

WORKDIR /var/www/html

CMD supervisord
