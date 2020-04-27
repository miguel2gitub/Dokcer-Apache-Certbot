FROM php:7.3.12-apache

COPY php.ini /usr/local/etc/php/

RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libmcrypt-dev \
        libpcre3 \
        libpcre3-dev \
        libicu-dev \
        libzip-dev \
    && docker-php-ext-install -j$(nproc) iconv mysqli pdo pdo_mysql intl\
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install zip \
    && docker-php-ext-install opcache \
    && pecl install mcrypt-1.0.3 \
    && docker-php-ext-enable mcrypt \
    && apt-get -y clean \
    && apt-get -y autoremove

RUN a2enmod rewrite
RUN a2enmod ssl
