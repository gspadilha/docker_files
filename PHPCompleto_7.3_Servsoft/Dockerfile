FROM php:7.3.33-apache

ENV ACCEPT_EULA=Y
ENV NODE_VERSION=18.16.0
ENV NVM_DIR=/root/.nvm
ENV LD_LIBRARY_PATH /opt/oracle/instantclient_12_1:${LD_LIBRARY_PATH}

ENV PATH "$PATH"
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"
ENV PATH="/root/.composer/vendor/bin/:${PATH}"

# Instalar atualizações do linux
RUN apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y

# Instalar aplicativos básicos necessários para o servidor PHP/Apache
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
    libonig-dev \
    git

# Executar limpeza do apt
RUN apt-get clean autoclean && apt-get autoremove -y && rm -rf /var/lib/{apt,dpkg,cache,log}/

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

# Ativar extensões gd ? erro no 8
#RUN docker-php-ext-configure gd --with-gd --with-webp-dir \
#    --with-png-dir --with-zlib-dir --with-xpm-dir \
#    --with-jpeg-dir=/usr/lib \
#    --with-freetype-dir=/usr/include/freetype2

# Criar diretórios para o instant client da oracle
RUN mkdir /opt/oracle && cd /opt/oracle     

# Mover o instalador do instant client da oracle pra o docker
ADD instantclient-basic-linux.x64-12.1.0.2.0.zip /opt/oracle
ADD instantclient-sdk-linux.x64-12.1.0.2.0.zip /opt/oracle

# Instalar o instant client da oracle
RUN  unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
    && rm -rf /opt/oracle/*.zip

# Ativar extensões oci
RUN echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.2.0 \
	&& docker-php-ext-enable oci8 \ 
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 \
	&& docker-php-ext-install pdo_oci 

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
RUN pecl install sqlsrv-5.9.0 pdo_sqlsrv-5.9.0 && docker-php-ext-enable sqlsrv pdo_sqlsrv

# Ativar extensões xdebug
RUN pecl install xdebug-3.1.6 && docker-php-ext-enable xdebug

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

# cria a pasta publica e permissão geral
RUN mkdir -p /var/www/html \
	&& mkdir -p /var/www/html/marca_php_servsoft \
	&& mkdir -p /var/www/html/marca_php_servsoft/zf \
	&& mkdir -p /var/www/html/marca_php_servsoft/zf/public \
	&& chmod -R 777 /var/www/html/* \
	&& mkdir -p /var/www/certificados

# Copiar as configurações existentes em arquivos
COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf
COPY php.ini /usr/local/etc/php/php.ini

# Arguments defined in docker-compose.yml
ARG user=user
ARG uid=123

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user \
	&& mkdir -p /home/$user/.composer \
	&& chown -R $user:$user /home/$user

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf \
	&& a2enmod rewrite \
	&& service apache2 restart
