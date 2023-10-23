# How to properly upgrade EspoCRM from version 5 to the latest

The most convenient way to upgrade EspoCRM from version ***5*** to the current one is to use [Docker Compose](https://docs.espocrm.com/administration/docker/installation/#install-espocrm-with-docker-compose).

## Environment preparation:
Let's say we want to update EspoCRM *v5.5.6*. 
In addition to the version of EspoCRM, we need to know the name and version of our database (for example, `mysql 5.7.43`) and the current version of php. This information can be found in *Administration > System Requirements*.

Now we can select the required [environment](https://github.com/tmachyshyn/espocrm-environment).

In our example, the **Dockerfile** will look like this:
```
FROM php:7.4-apache

# Install php libs
RUN set -ex; \
    \
    aptMarkList="$(apt-mark showmanual)"; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        libjpeg-dev \
        libpng-dev \
        libzip-dev \
        libxml2-dev \
        libc-client-dev \
        libkrb5-dev \
        libldap2-dev \
        libzmq3-dev \
        zlib1g-dev \
        wget \
        git\
    ; \
    \
# Install php-zmq
    cd /home; \
    wget https://github.com/zeromq/libzmq/releases/download/v4.3.2/zeromq-4.3.2.tar.gz; \
    tar -xvzf zeromq-4.3.2.tar.gz; \
    cd /home/zeromq-4.3.2; \
    ./configure; \
    make; \
    make install; \
    ldconfig; \
    ldconfig -p | grep zmq; \
    cd /home; \
    git clone https://github.com/mkoppanen/php-zmq.git; \
    cd /home/php-zmq; \
    phpize && ./configure; \
    make; \
    make install; \
    \
# END Instalation php-zmq
    \
    docker-php-ext-install pdo_mysql; \
    docker-php-ext-install zip; \
    docker-php-ext-configure gd --with-jpeg; \
    docker-php-ext-install gd; \
    PHP_OPENSSL=yes docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
    docker-php-ext-install imap; \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/; \
    docker-php-ext-install ldap; \
    docker-php-ext-install exif; \
    docker-php-ext-enable zmq; \
    docker-php-ext-install mysqli; \
    docker-php-ext-enable mysqli; \
    \
# reset a list of apt-mark
    apt-mark auto '.*' > /dev/null; \
    apt-mark manual $aptMarkList; \
    ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
        | awk '/=>/ { print $3 }' \
        | sort -u \
        | xargs -r dpkg-query -S \
        | cut -d: -f1 \
        | sort -u \
        | xargs -rt apt-mark manual; \
    \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*

# Set timezone
RUN echo "UTC" > /etc/timezone && dpkg-reconfigure --frontend noninteractive tzdata

# Install git, wget
RUN apt-get update; \
    apt-get install -y git; \
    apt-get install -y wget

# Install composer globally
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('composer-setup.php');" \
    && mkdir -p "/var/www/.composer" \
    && chown www-data:www-data "/var/www/.composer"

# Install npm
RUN curl -sL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs

# Install Grunt
RUN npm install -g grunt-cli; \
    mkdir -p /var/www/.npm; \
    mkdir -p /var/www/.config; \
    npm install -g npm@latest

# Install PHPUNIT
RUN wget -P /tmp https://phar.phpunit.de/phpunit.phar \
    && mv /tmp/phpunit.phar /usr/local/bin/phpunit \
    && chmod +x /usr/local/bin/phpunit

# Set available directory list
# ONLY FOR TEST ENVIRONMENT: Options Indexes
RUN { \
    echo '<VirtualHost *:80>'; \
    echo '    ServerName localhost'; \
    echo '    DocumentRoot /var/www/html'; \
    echo '    DirectoryIndex index.php index.html'; \
    echo '    <Directory "/var/www/html">'; \
    echo '        Options Indexes FollowSymLinks'; \
    echo '        AllowOverride All'; \
    echo '        Require all granted'; \
    echo '    </Directory>'; \
    echo ''; \
    echo '    ErrorLog ${APACHE_LOG_DIR}/error.log'; \
    echo '    CustomLog ${APACHE_LOG_DIR}/access.log combined'; \
    echo '</VirtualHost>'; \
} > /etc/apache2/sites-available/000-default.conf

# php.ini
RUN { \
    echo 'expose_php = Off'; \
    echo 'display_errors = On'; \
    echo 'display_startup_errors = On'; \
    echo 'log_errors = On'; \
    echo 'memory_limit=256M'; \
    echo 'max_execution_time=180'; \
    echo 'max_input_time=180'; \
    echo 'post_max_size=30M'; \
    echo 'upload_max_filesize=30M'; \
    echo 'date.timezone=UTC'; \
} > ${PHP_INI_DIR}/conf.d/espocrm.ini

RUN a2enmod rewrite

VOLUME /var/www/html

RUN chown -R www-data:www-data /var/www

CMD ["apache2-foreground"]

```

And **docker-compose.yml** will have the following form (for the *espocrm-php* container, you can specify another unoccupied port):
```
version: '3'

services:

  espocrm-mysql:
    container_name: espocrm-mysql
    image: mysql:5.7.43
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 1
    volumes:
      - ./mysql/data:/var/lib/mysql
    ports:
      - 33306:3306
    
  espocrm-php:
    container_name: espocrm-php
    build:
      context: ./php
      dockerfile: Dockerfile
    volumes:
      - ./html:/var/www/html
      - ./logs:/var/log/apache2
    restart: unless-stopped
    ports:
      - 8080:80
```

Start our container using the command:
```
docker-compose up -d
```
or
```
sudo docker-compose up -d --build
```
or
```
sudo docker-compose up -d --build "$@"
```

## Transfer of instance and database files

- [Backup](https://docs.espocrm.com/administration/backup-and-restore/#backup-and-restore) files of our instance and database.
- Go to the folder `our_instance_name/data` and delete all the contents of the folder except `/upload`, `config.php`, `config-internal.php` (if you had it), `.data` and `/logs` (if you need them).
- In the file `config.php` (or `config-internal.php`, depending on the version of EspoCRM), change the following data:
```
'dbname' => 'espocrm',
'user' => 'root',
'password' => '1',
'defaultPermissions' => [
    'user' => 33,
    'group' => 33
  ],
```
- Return to `our_instance_name` and copy its contents. Go to the folder where you have **docker-compose.yml** and the `html` folder has been created. Paste the copied files here.
- After finishing copying in the `html` folder, give the necessary [Permissions](https://docs.espocrm.com/administration/server-configuration/#permissions).
