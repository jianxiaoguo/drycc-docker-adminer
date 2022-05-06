FROM registry.drycc.cc/drycc/base:bullseye

ENV  ADMINER_VERSION="4.8.1" \
    php_version="7.4.29" \
    php_layer_dir=/opt/drycc/php

RUN buildDeps='git postgresql-server-dev-all sqlite3 libsqlite3-dev unixodbc-dev freetds-dev'; \
    install-packages ${buildDeps} libnss-wrapper xz-utils \
    && install-stack php $php_version \
    && echo "export NSS_WRAPPER_LIB=/usr/lib/`echo $(uname -m)`-linux-gnu/libnss_wrapper.so" >> /opt/drycc/php/profile.d/php_exporter.sh

COPY rootfs/* /opt/drycc/php/

RUN . init-stack \
    && php-ext-configure pdo_odbc --with-php-config="${php_layer_dir}"/bin/php-config --with-pdo-odbc=unixODBC,/usr \
    && php-ext-install mysqli pdo_pgsql pdo_sqlite pdo_odbc pdo_dblib --ini-name "${php_layer_dir}"/config/php/php.ini \  
    && git clone --recurse-submodules=designs --depth 1 --shallow-submodules --branch "v$ADMINER_VERSION" https://github.com/vrana/adminer.git /opt/drycc/adminer \
    && cp -r /opt/drycc/adminer/designs/ /opt/drycc/adminer/plugins/ . \
    # cleanup
    && apt-get purge -y --auto-remove ${buildDeps} \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf \
    /usr/share/doc \
    /usr/share/man \
    /usr/share/info \
    /usr/share/locale \
    /var/lib/apt/lists/* \
    /var/log/* \
    /var/cache/debconf/* \
    /etc/systemd \
    /lib/lsb \
    /lib/udev \
    /usr/lib/`echo $(uname -m)`-linux-gnu/gconv/IBM* \
    /usr/lib/`echo $(uname -m)`-linux-gnu/gconv/EBC* \
    && mkdir -p /usr/share/man/man{1..8}

COPY entrypoint.sh /opt/drycc/php/bin
COPY *.php /var/www/html/
USER 1001

ENTRYPOINT [ "init-stack", "entrypoint.sh", "docker-php-entrypoint" ]
# CMD	[ "php", "-S", "[::]:8080", "-t", "/opt/drycc/adminer" ]
# CMD	[ "php-fpm" ]
CMD [ "php-fpm", "-c", "${php_layer_dir}/config/php.ini", "-y", "${php_layer_dir}/config/php-fpm.conf", "-p", "${php_layer_dir}" ]

EXPOSE 8080
