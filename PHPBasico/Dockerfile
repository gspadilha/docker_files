FROM php:7.3.26-apache

# primeira atualização
RUN apt-get update && apt-get upgrade -y

# aplicativos básicos para o servidor
RUN apt-get install -y apt-utils libcurl4-gnutls-dev libxml2-dev wget vim unzip redis-server libxml2 zlib1g-dev libicu-dev g++

# instação para o gd
RUN apt-get install -y gnupg2 \
    freetype* \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxpm-dev \
    libfreetype6-dev \
    libzip-dev \
    libaio1 

# limpeza
RUN apt-get clean autoclean && apt-get autoremove -y && rm -rf /var/lib/{apt,dpkg,cache,log}/

# extensoes básicas do php e xdebug
RUN docker-php-ext-install curl xml mbstring bcmath gd pdo pdo_mysql sockets && pecl install xdebug && docker-php-ext-enable xdebug

#intl
RUN docker-php-ext-configure intl && docker-php-ext-install intl

# gd
RUN docker-php-ext-configure gd --with-gd --with-webp-dir \
    --with-png-dir --with-zlib-dir --with-xpm-dir \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2

# cria a pasta publica e já permissão geral
RUN mkdir /var/www/html/public && chmod -R 777 /var/www/html/*

# configs báscias
COPY vhost.conf /etc/apache2/sites-enabled/000-default.conf
COPY php.ini /usr/local/etc/php/php.ini
COPY redis.conf /etc/redis/redis.conf

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
RUN a2enmod rewrite
RUN service apache2 restart
RUN service redis-server restart
