FROM php:8.0.28-apache

ENV ACCEPT_EULA=Y
ENV NODE_VERSION=18.16.0
ENV NVM_DIR=/root/.nvm

ENV PATH "$PATH"
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"
ENV PATH="/root/.composer/vendor/bin/:${PATH}"

# primeira atualização
RUN apt update && apt upgrade -y && apt dist-upgrade -y && apt clean -y && apt autoremove -y && rm -rf /var/lib/{apt,dpkg,cache,log}

# aplicativos básicos para o servidor
RUN apt update && \
	apt install -y apt-utils \
	build-essential \
	shared-mime-info \
    libcurl4-gnutls-dev \
    curl \
    wget \
    vim \
    unzip \
    libxml2 \
	libxml2-dev \
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
    git \
	libpq-dev \
	libaio-dev \
	&& apt clean -y && apt autoremove -y && rm -rf /var/lib/{apt,dpkg,cache,log}

# Ativar extensões básicas do PHP
RUN docker-php-ext-install curl \
    xml \
    mbstring \
    bcmath \
    gd \
    pdo \
    pdo_mysql \ 
    sockets \
    zip

# Ativar extensões xdebug	
#RUN pecl install xdebug && docker-php-ext-enable xdebug

# Ativar extensões intl
RUN docker-php-ext-configure intl && docker-php-ext-install intl

# Instalar o ODBC do Microsoft SQL Server
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/debian/11/prod.list \
    > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get install -y --no-install-recommends \
    locales \
    apt-transport-https \
	unixodbc-dev \
	unixodbc \
    && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
    && locale-gen    

RUN TEMP_DEB="$(mktemp)" \
 && wget -O "$TEMP_DEB" 'https://packages.microsoft.com/debian/11/prod/pool/main/m/msodbcsql17/msodbcsql17_17.10.2.1-1_amd64.deb' \
 && yes | dpkg --skip-same-version -i "$TEMP_DEB" \
 && rm -f "$TEMP_DEB"

# Ativar extensões sqlsrv
RUN pecl install sqlsrv-5.11.0 pdo_sqlsrv-5.11.0 && docker-php-ext-enable sqlsrv pdo_sqlsrv

# Ativar extensões xdebug
RUN pecl install xdebug-3.2.1 && docker-php-ext-enable xdebug

# Instalar NVM
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

# Instalar o NODE LTS
RUN . "$NVM_DIR/nvm.sh" \
	&& nvm install ${NODE_VERSION} \
	&& nvm use v${NODE_VERSION} \
	&& nvm alias default v${NODE_VERSION}

# Instalar gerenciador YARN e NPM
RUN npm -g install yarn npm

# Instalar o composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Arguments defined in docker-compose.yml
ARG user=user
ARG uid=123

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user \
	&& mkdir -p /home/$user/.composer \
	&& chown -R $user:$user /home/$user

# cria a pasta publica e permissão geral
RUN mkdir /var/www/html/laravel_vue && mkdir /var/www/html/laravel_vue/public

# configs básicas
COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf
COPY php.ini /usr/local/etc/php/php.ini

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf && a2enmod rewrite && service apache2 restart
