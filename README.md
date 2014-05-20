Heroku Buildpack (Nginx + PHP + Phalcon)
----------------------------------
Buildpack Stack:
* Nginx 1.5.13
* PHP-FPM 5.5.11
* MCrypt 2.5.8
* PCRE 8.35
* Phalcon (Latest)
* MongoDB PHP Driver (Latest)

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
    --with-iconv \
    --with-gd \
    --with-openssl \
    --with-curl \
    --with-config-file-path=/app/vendor/php \
    --with-mcrypt=/app/vendor/mcrypt \
    --enable-mbstring \
    --enable-opcache \
    --enable-fpm \
    --enable-sockets
make && make install

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

Compile and Package Nginx
----------------------------------
This script should be excuted via the terminal ```heroku run bash --app=app-name```.
```
# set starting point
cd ~

# vendor nginx
curl -L http://nginx.org/download/nginx-1.5.13.tar.gz | tar xzv
cd nginx-*
curl -L http://garr.dl.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.bz2 | tar xvj
./configure \
	--prefix=/app/vendor/nginx \
	--with-pcre=pcre-8.35 \
	--with-http_ssl_module \
	--with-http_gzip_static_module \
	--with-http_stub_status_module \
	--with-http_realip_module
make install

# package php
cd /app/vendor/php && tar cvzf /tmp/nginx-1.5.13.tar.gz .
```
Once the package is created, use the ```scp``` command to get it off the dyno.
```
scp /tmp/nginx-1.5.13.tar.gz username@server:
```

Note
----
To make our lives easier when we built this package, we created a fresh dyno ```heroku create``` and then terminal into it via ```heroku run bash --app=app-name```.

Sample Project
--------------
Check out our sample project for this buildpack. It includes everything you need to test your installation and can be used as a reference for your project. Enjoy!

https://github.com/droc-ventures/heroku-nginx-php-phalcon-sample

Inspiration and Credit
----------------------
The wordpress buildpack [@mchung](https://github.com/mchung) created provided some valuable insight for this buildpack.
