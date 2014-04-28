Heroku Buildpack (Nginx + PHP + Phalcon)
========================
Buildpack Stack:
* Nginx 1.5.13
* PHP-FPM 5.5.11
* MCrypt 2.5.8
* Phalcon (Latest)
* MongoDB PHP Driver (latest)

Compile and Package PHP and MCrypt
----------------------------------
This script should be excuted via the terminal ```heroku run bash --app=app-name```.
```
# set starting point
cd ~

# vendor mcrypt
curl -L http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz/download | tar xzv
cd libmcrypt-*
./configure \
	--prefix=/app/vendor/mcrypt \
	--disable-posix-threads \
	--enable-dynamic-loading
make && make install

# vendor php
cd ~
curl -L http://us1.php.net/get/php-5.5.11.tar.gz/from/this/mirror | tar xzv
cd php-*
./configure \
	--prefix=/app/vendor/php \
	--with-mysql \
	--with-pdo-mysql \
	--with-pgsql \
	--with-pdo-pgsql \
	--with-iconv \
	--with-gd \
	--with-openssl \
	--with-config-file-path=/app/vendor/php \
	--with-mcrypt=/app/vendor/mcrypt \
	--enable-mbstring \
	--enable-opcache \
	--enable-fpm \
	--enable-sockets \
	--enable-soap=shared
make && make install

# copy mysql into php directory
mkdir -p /app/vendor/php/lib/php
cp /usr/lib/libmysqlclient.so.16 /app/vendor/php/lib/php

# set php path
export PATH=/app/vendor/php/bin:$PATH

# update pecl
/app/vendor/php/bin/pecl channel-update pecl.php.net

# install mongo pecl extension
/app/vendor/php/bin/pecl install mongo

# compile and install phalcon
mkdir -p /app/tmp && cd /app/tmp
git clone --depth=1 git://github.com/phalcon/cphalcon.git
cd cphalcon/build
./install

# package php
cd /app/vendor/php && tar cvzf /tmp/php-5.5.11.tar.gz .

# package mcrypt
cd /app/vendor/mcrypt && tar cvzf /tmp/mcrypt-2.5.8.tar.gz .
```
Once the packages are created, use the ```scp``` command to get them off the dyno.
```
scp /tmp/php-5.5.11.tar.gz username@server:
scp /tmp/mcrypt-2.5.8.tar.gz username@server:
```
