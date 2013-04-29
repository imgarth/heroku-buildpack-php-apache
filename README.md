Apache + PHP build pack
========================

[Heroku Buildpack](http://devcenter.heroku.com/articles/buildpacks) para rodar o servidor HTTP Nignx com PHP no Heroku.

Versões
--------

Apache: 2.4.4
ICU: 50.1.2
PHP: 5.4.14
Mencached: 1.4.15

Configuração
-------------

Arquivos de configuração

* conf/httpd.conf
* conf/php.ini

Compilando os binários
------------------

    # [APR](http://apr.apache.org)
    cd /app
    curl -L "http://ftp.unicamp.br/pub/apache//apr/apr-1.4.6.tar.gz" -o - | tar xz
    cd /app/apr-1.4.6/
    ./configure --prefix=/app/apr && make install

    # [APR Tool](http://apr.apache.org)
    cd /app
    curl -L "http://ftp.unicamp.br/pub/apache//apr/apr-util-1.5.2.tar.gz" -o - | tar xz
    cd /app/apr-util-1.5.2/
    ./configure --prefix=/app/apr-util --with-apr=/app/apr && make install

    # [PCRE](http://www.pcre.org)
    cd /app
    curl -L "http://ufpr.dl.sourceforge.net/project/pcre/pcre/8.32/pcre-8.32.tar.gz" -o - | tar xz
    cd /app/pcre-8.32/
    ./configure --prefix=/app/pcre && make install
    
    # [Apache](http://apache.org)
    cd /app
    curl -L "http://ftp.unicamp.br/pub/apache//httpd/httpd-2.4.4.tar.gz" -o - | tar xz
    cd /app/httpd-2.4.4/
    ./configure --prefix=/app/apache --with-apr=/app/apr --with-apr-util=/app/apr-util --with-pcre=/app/pcre --enable-rewrite && make install

    # [ICU](http://site.icu-project.org)
    cd /app  
    curl -L "http://download.icu-project.org/files/icu4c/50.1.2/icu4c-50_1_2-src.tgz" -o - | tar xz
    cd /app/icu
    ./source/configure --prefix=/app/icu4 && make install

    # [PHP 5.3](http://php.net)
    cd /app
    curl -L "http://us.php.net/get/php-5.3.24.tar.gz/from/www.php.net/mirror" -o - | tar xz
    cd /app/php-5.3.24/
    ./configure --prefix=/app/php --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --with-icu-dir=/app/icu4 --enable-intl --enable-fpm --with-zlib --enable-mbstring --disable-debug --enable-inline-optimization --with-bz2 --enable-mbregex --with-mhash --enable-zip --with-pcre-regex && make install
    
    # [PHP 5.4](http://php.net)
    cd /app
    curl -L "http://us.php.net/get/php-5.4.14.tar.gz/from/www.php.net/mirror" -o - | tar xz
    cd /app/php-5.4.14/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --with-icu-dir=/app/icu4 --enable-intl --enable-fpm --with-zlib --enable-mbstring --disable-debug --enable-inline-optimization --with-bz2 --enable-mbregex --with-mhash --enable-zip --with-pcre-regex && make install

    # [Memcached](http://memcached.org)
    cd /app
    curl -L "http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz" -o - | tar xz
    cd /app/memcached-1.4.15/
    ./configure --prefix=/app/memcached && make install

    # Adicionando a biblioteca do PHP no Apache
    cp /app/php-5.4.14/libs/libphp5.so /app/apache/modules/

    # [Driver do MongoDB](http://docs.mongodb.org/ecosystem/drivers/php/)
    cd /app
    git clone https://github.com/mongodb/mongo-php-driver.git
    cd /app/mongo-php-driver/
    ./configure --with-php-config=/app/php/bin/php-config && make install

    # [Extensão do APC](http://pecl.php.net/package/APC)
    cd /app
    curl -L "http://pecl.php.net/get/APC" -o - | tar xz
    cd /app/APC-3.1.13
    /app/php/bin/phpize
    ./configure --enable-apc --enable-apc-mmap --with-php-config=/app/php/bin/php-config && make install

    # [Extensão do Memcache](http://pecl.php.net/package/memcache)
    cd /app
    curl -L "http://pecl.php.net/get/memcache-3.0.8.tgz" -o - | tar xz
    cd /app/memcache-3.0.8
    /app/php/bin/phpize
    ./configure --with-libmemcached-dir=/app/memcached --with-php-config=/app/php/bin/php-config && make install

    # NEWRELIC
    cd /app
    curl -L "http://download.newrelic.com/php_agent/release/newrelic-php5-3.3.5.161-linux.tar.gz" -o - | tar xz
    
    pushd /app/newrelic-php5-3.3.5.161-linux

    mkdir -p /app/newrelic/{bin,etc} /app/newrelic/var/{run,log} /app/newrelic/var/log/newrelic
    cp -f /app/newrelic-php5-3.3.5.161-linux/daemon/newrelic-daemon.x64 /app/newrelic/bin/newrelic-daemon
    cp -f /app/newrelic-php5-3.3.5.161-linux/scripts/newrelic.cfg.template /app/newrelic/etc/newrelic.cfg
    sed -i 's|var|app/newrelic/var|g' /app/newrelic/etc/newrelic.cfg
    sed -i 's|#ssl=false|ssl=true|g' /app/newrelic/etc/newrelic.cfg
    sed -i 's|#pidfile=|pidfile=|g' /app/newrelic/etc/newrelic.cfg
    sed -i 's|#logfile=|logfile=|g' /app/newrelic/etc/newrelic.cfg

    cat > /app/newrelic/bin/newrelic-license <<EOF
    #!/usr/bin/env bash

    if [ -f /app/newrelic/NEWRELIC_VERSION ]; then
        sed -i "s|REPLACE_WITH_REAL_KEY|${NEW_RELIC_LICENSE_KEY}|g" /app/php/etc/*newrelic*.ini
    else
        sed -i "s|REPLACE_WITH_REAL_KEY|${NEW_RELIC_LICENSE_KEY}|g" /app/newrelic/etc/newrelic.cfg
    fi

    chmod +x /app/newrelic/bin/newrelic-license
    echo "3.3.5.161" > /app/newrelic/NEWRELIC_VERSION


    # Empacotando tudo
    cd /app
    
    echo '2.4.4' > apache/VERSION
    tar -zcvf apache-2.4.4.tar.gz apache
    
    echo '5.4.14' > php/VERSION
    tar -zcvf php-5.4.14.tar.gz php

    echo '5.3.24' > php/VERSION
    tar -zcvf php-5.3.24.tar.gz php

    echo '1.4.15' > memcached/VERSION
    tar -zcvf memcached-1.4.15.tar.gz memcached