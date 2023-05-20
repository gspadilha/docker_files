FROM php:8.1.12-apache

ENV ACCEPT_EULA=Y
ENV NODE_VERSION=18.12.1
ENV NVM_DIR=/root/.nvm

# atualização do 'linux'
RUN apt-get update && apt-get upgrade -y

# aplicativos básicos necessários para o servidor PHP/Apache
RUN apt-get install -y apt-utils \
    libcurl4-gnutls-dev \
    libxml2-dev \
    curl \
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
    libonig-dev

# limpeza
RUN apt-get clean autoclean && apt-get autoremove -y && rm -rf /var/lib/{apt,dpkg,cache,log}/

# ORACLE oci 
RUN mkdir /opt/oracle && cd /opt/oracle     

# move o instant client pra o docker
ADD instantclient-basic-linux.x64-12.1.0.2.0.zip /opt/oracle
ADD instantclient-sdk-linux.x64-12.1.0.2.0.zip /opt/oracle

# Instala o Instant Client do Oracle
RUN  unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
    && rm -rf /opt/oracle/*.zip

ENV LD_LIBRARY_PATH  /opt/oracle/instantclient_12_1:${LD_LIBRARY_PATH}

# ativar extensões básicas do php e xdebug
RUN docker-php-ext-install curl \
    xml \
    mbstring \
    bcmath \
    gd \
    pdo \
    pdo_mysql \ 
    sockets \
    && pecl install xdebug && docker-php-ext-enable xdebug

# intl
RUN docker-php-ext-configure intl && docker-php-ext-install intl

# gd ? erro no 8
#RUN docker-php-ext-configure gd --with-gd --with-webp-dir \
#    --with-png-dir --with-zlib-dir --with-xpm-dir \
#    --with-jpeg-dir=/usr/lib \
#    --with-freetype-dir=/usr/include/freetype2

# Install Oracle extensions
RUN echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-3.2.1 && docker-php-ext-enable oci8 \ 
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 && docker-php-ext-install pdo_oci 

# instalar NodeJS LTS
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash

RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}

ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"

# instalar gerenciador YARN
RUN npm -g install yarn

# instalar o composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# cria a pasta publica e permissão geral
RUN mkdir -p /var/www/html && mkdir -p /var/www/html/public && chmod -R 777 /var/www/html/* && mkdir -p /var/www/certificados

# configs existentes em arquivos
COPY vhost.conf /etc/apache2/sites-enabled/000-default.conf
COPY php.ini /usr/local/etc/php/php.ini

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
RUN a2enmod rewrite
RUN service apache2 restart
