FROM php:8.0.29-apache

ENV ACCEPT_EULA=Y

# primeira atualização
RUN apt-get update && apt-get upgrade -y

# aplicativos básicos para o servidor
RUN apt-get install -y apt-utils \
    libcurl4-gnutls-dev \
    libxml2-dev \
    wget \
    vim \
    unzip \
    libxml2 \
    zlib1g-dev \
    libicu-dev \
    g++ \
    gnupg2 \
    freetype* \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxpm-dev \
    libfreetype6-dev \
    libzip-dev \
    libaio1 \
    libonig-dev \
	git

# limpeza
RUN apt-get clean autoclean && apt-get autoremove -y && rm -rf /var/lib/{apt,dpkg,cache,log}/

# extensoes básicas do php e xdebug
RUN docker-php-ext-install curl \
    xml \
    mbstring \
    bcmath \
    gd \
    pdo \
    pdo_mysql \ 
	mysqli \
    sockets \
    && pecl install xdebug && docker-php-ext-enable xdebug

#intl
RUN docker-php-ext-configure intl && docker-php-ext-install intl

# configs báscias
COPY vhost.conf /etc/apache2/sites-enabled/000-default.conf
COPY php.ini /usr/local/etc/php/php.ini

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
RUN a2enmod rewrite
RUN service apache2 restart
