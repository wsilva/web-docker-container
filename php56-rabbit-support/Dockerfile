FROM debian:wheezy

RUN dpkg-divert --local --rename --add /sbin/initctl && ln -sf /bin/true /sbin/initctl

ENV DEBIAN_FRONTEND noninteractive

RUN useradd -u 1000 docker && apt-get update && apt-get install -y wget

RUN echo "deb http://packages.dotdeb.org wheezy-php56-zts all" > /etc/apt/sources.list.d/dotdeb.list \
    && echo "deb-src http://packages.dotdeb.org wheezy-php56-zts all" >> /etc/apt/sources.list.d/dotdeb.list \
    && echo "deb http://packages.dotdeb.org wheezy all" >> /etc/apt/sources.list.d/dotdeb.list \
    && echo "deb-src http://packages.dotdeb.org wheezy all" >> /etc/apt/sources.list.d/dotdeb.list \
    && wget -O - http://www.dotdeb.org/dotdeb.gpg | apt-key add - \
    && echo "deb http://apt.newrelic.com/debian/ newrelic non-free" > /etc/apt/sources.list.d/newrelic.list \
    && wget --no-check-certificate -O - https://download.newrelic.com/548C16BF.gpg | apt-key add -


RUN apt-get update && apt-get install -y vim cmake make g++ git whiptail mlocate net-tools curl procps \
    re2c libsnappy1 libsnappy-dev php5-cli php5-fpm mysql-client php5-intl php5-recode \
    php5-snmp php5-mcrypt php5-memcache php5-memcached php5-imagick php5-curl php5-xsl php5-snmp \
    php5-dev php5-tidy php5-xmlrpc php5-gd php5-pspell php-pear nginx pkg-config librabbitmq-dev \
    librabbitmq0 newrelic-php5

RUN sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/cli/php.ini \
    && sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/fpm/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/cli/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/fpm/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/cli/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/www-data/docker/g" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/pm.max_children = 5/pm.max_children = 35/" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/pm.start_servers = 2/pm.start_servers = 10/" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/pm.min_spare_servers = 1/pm.min_spare_servers = 10/" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/pm.max_spare_servers = 3/pm.max_spare_servers = 17/" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/;pm.max_requests = 500/pm.max_requests = 250/" /etc/php5/fpm/pool.d/www.conf \
    && sed -i "s/www-data;/docker;\\ndaemon off;/g" /etc/nginx/nginx.conf \
    && curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/bin/composer

# COPY xdebug.ini /etc/php5/mods-available/xdebug.ini
RUN pecl install xdebug
RUN echo "xdebug.remote_enable=1" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.remote_handler=dbgp" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.remote_mode=req" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.remote_port=9001" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.remote_autostart=1" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.remote_connect_back=1" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.var_display_max_depth = -1 " >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.var_display_max_children = -1" >> /etc/php5/mods-available/xdebug.ini \
    && echo "xdebug.var_display_max_data = -1 " >> /etc/php5/mods-available/xdebug.ini \
RUN echo "\n\nxdebug.remote_host="`/sbin/ip route|awk '/default/ { print $3 }'` >> /etc/php5/mods-available/xdebug.ini

WORKDIR /
RUN git clone https://github.com/alanxz/rabbitmq-c && git clone https://github.com/phadej/igbinary && git clone git://github.com/goatherd/php-snappy.git
WORKDIR /rabbitmq-c
RUN git checkout v0.5.2 \
    && autoreconf -i \
    && ./configure \
    && make \
    && make install \
    && pecl install amqp \
    && echo "extension=amqp.so" > /etc/php5/mods-available/amqp.ini
WORKDIR /igbinary
RUN phpize \
    && ./configure CFLAGS="-O2 -g" --enable-igbinary --with-php-config=/usr/bin/php-config \
    && make \
    && make install \
    && echo -e "\n\n[igbinary]\n;https://github.com/igbinary/igbinary/\nextension=igbinary.so\n\n;Use igbinary as session serializer\nsession.serialize_handler=igbinary\n\n;Enable or disable compacting of duplicate strings\nigbinary.compact_strings=On\n\n;Use igbinary as serializer in APC cache\n;apc.serializer=igbinary" > /etc/php5/mods-available/igbinary.ini
WORKDIR /php-snappy
RUN phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=snappy.so" > /etc/php5/mods-available/snappy.ini
WORKDIR /etc/php5/mods-available
RUN php5enmod amqp && php5enmod igbinary && php5enmod snappy
WORKDIR /
RUN rm -rf rabbitmq-c/ && rm -rf igbinary/ && rm -rf php-snappy/

RUN apt-get clean

COPY run.sh /run.sh
RUN chmod a+x /run.sh

EXPOSE 80

ENTRYPOINT ["/run.sh"]
