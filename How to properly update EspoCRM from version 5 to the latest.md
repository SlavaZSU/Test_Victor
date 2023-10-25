# EspoCRM upgrade from version 5 to the latest

The most convenient way to upgrade EspoCRM from version ***5*** to the current one is to use [Docker Compose](https://docs.espocrm.com/administration/docker/installation/#install-espocrm-with-docker-compose).

## Environment preparation:

To create the correct environment, need to know the following information (can be found in *Administration > System Requirements*):
1. *EspoCRM* version, e.g.`5.5.6`. 
2. *Database* name and version, e.g. `mysql 5.7.43`.
3. *PHP* used version, e.g. `7.4`.
4. *Server type*, e.g. `apache` or `nginx`.

Select the required [Environment](https://github.com/tmachyshyn/espocrm-environment).

### Dockerfile preparation:

**Dockerfile** will look like:

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

### Docker-compose.yml preparation:

The **docker-compose.yml** will have the following form (it's possible to specify another free port for the *espocrm-php* container):

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

Start container using the command:

```
docker-compose up -d
```

or

```
docker-compose up -d --build
```

or

```
docker-compose up -d --build "$@"
```

## Transfer of Instance files and Database dump

- Make [Backup](https://docs.espocrm.com/administration/backup-and-restore/#backup-and-restore) of instance and database files.
  
- Go to `/your_instance_name/data` and delete all data except `/upload`, `config.php`, `config-internal.php` (if you had it), `.data` and `/logs` (if you need them).
  
- In the `config.php` file (or `config-internal.php`, depending the EspoCRM version), change the following data:

```
'dbname' => 'espocrm',
'user' => 'root',
'password' => '1',
```

For *defaultPermissions*, specify the webserver user data and the group you will use (usually it's `www-data` or `root`). 

```
'defaultPermissions' => [
  'user' => 33,
  'group' => 33
],
```

- Return to `/your_instance_name` and copy its data.

- Go to `root_folder_of_your_environment/html` and paste the copied files there.

- While in the `root_folder_of_your_environment/html` folder, give the necessary [Permissions](https://docs.espocrm.com/administration/server-configuration/#permissions).
  
- Go to the *database container* through *Docker*:

```
docker exec -it espocrm-mysql bash
```

- And go to the *database* itself:
  
```
mysql -u root -p
```

- Create an empty ***espocrm*** database:
  
```
CREATE DATABASE espocrm;
```

- Exit from the *database* and *database container* to the folder where our `.sql` dump is located. Run the command:
  
```
docker exec -i espocrm-mysql mysql -uroot -p1 espocrm < name_of_your_dump.sql
```

- Go to the *php container* (using webserver user) the by command:

```
docker exec -u www-data -it espocrm-php bash
```

- Make *Rebuild*:

```
php rebuild.php
```

- To be sure that the instance is working, log in to the UI: `localhost:8080` (or another free port specified in the environment's **docker-composer.yml**).

## Methods and process of upgrading EspoCRM v5.5.6

- Download the following upgrades to `root_folder_of_your_environment/html` folder:

```
wget https://www.espocrm.com/downloads/upgrades/EspoCRM-upgrade-5.5.6-to-5.6.14.zip
wget https://www.espocrm.com/downloads/upgrades/EspoCRM-upgrade-5.6.14-to-5.7.11.zip
wget https://www.espocrm.com/downloads/upgrades/EspoCRM-upgrade-5.7.11-to-5.8.5.zip
```

- Go to the *php container* using the command:

```
docker exec -it espocrm-php bash
```

or better using the command for webserver user:

```
docker exec -u www-data -it espocrm-php bash
```

- Upgrade EspoCRM version from `v5.5.6` to `v5.6.14` using [Legacy way to upgrade](https://docs.espocrm.com/administration/upgrading/#legacy-way-to-upgrade):

```
php upgrade.php EspoCRM-upgrade-5.5.6-to-5.6.14.zip
```

To verify that your instance is running successfully after the *Upgrade*, you can make *Rebuild* from UI (or by CLI):

```
php rebuild.php
```

- To upgrade EspoCRM from `v5.6.14` to `v5.7.11` and from `v5.6.14` to `v5.7.11` also use [Legacy way to upgrade](https://docs.espocrm.com/administration/upgrading/#legacy-way-to-upgrade). 

- You can continue to *Upgrade* in a similar way, but periodically from version to version you can check `root_folder_of_your_environment/html` folder for the presence of the `command.php` file. Once this file appears, you can switch to the most convenient method by CLI:
  
```
php command.php upgrade
```

### How to convert database tables to InnoDB

If at one of the *Upgrade* stages you receive a message that you need to convert the database tables to *InnoDB*, you can do it as follows:  

1. Go to `root_folder_of_your_environment/html` and download the script:

```
wget https://www.espocrm.com/downloads/scripts/convert-myisam-to-innodb.zip
```

2. Unzip it:

```
unzip convert-myisam-to-innodb.zip
```

3. Go to the *php container* (using webserver user) the by command:

```
docker exec -u www-data -it espocrm-php bash
```
