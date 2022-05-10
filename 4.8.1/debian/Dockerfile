FROM registry.drycc.cc/drycc/base:bullseye

ENV DRYCC_UID=1001 \
  DRYCC_GID=1001 \
  ADMINER_VERSION="4.8.1" \
  NGINX_VERSION="1.21.6" \
  VOUCH_PROXY_VERSION="0.37.0" \
  php_version="7.4.29" \
  php_layer_dir="/opt/drycc/php" \
  DRYCC_HOME_DIR=/opt/drycc/adminer

RUN groupadd drycc --gid ${DRYCC_GID} \
  && useradd drycc -u ${DRYCC_UID} -g ${DRYCC_GID} -s /bin/bash -m -d ${DRYCC_HOME_DIR}

WORKDIR ${DRYCC_HOME_DIR}

RUN install-stack php "${php_version}" \
  && install-stack nginx "${NGINX_VERSION}" \
  && install-stack vouch-proxy "${VOUCH_PROXY_VERSION}"

COPY rootfs/lib/ /opt/drycc/php/lib/
COPY rootfs/config/php/ /opt/drycc/php/etc/
COPY rootfs/config/nginx/ /opt/drycc/nginx/conf/

RUN . init-stack \
  && buildDeps='git gcc pkg-config postgresql-server-dev-all sqlite3 libsqlite3-dev unixodbc-dev freetds-dev xz-utils' \
  && install-packages ${buildDeps} libnss-wrapper \
  && ln -s "/usr/lib/`echo $(uname -m)`-linux-gnu/libsybdb.so" /usr/lib/libsybdb.so \
  && echo "export NSS_WRAPPER_LIB=/usr/lib/`echo $(uname -m)`-linux-gnu/libnss_wrapper.so" >> /opt/drycc/php/profile.d/php.sh \
  && php-ext-configure pdo_odbc --with-php-config="${php_layer_dir}"/bin/php-config --with-pdo-odbc=unixODBC,/usr \
  && php-ext-install mysqli pdo_pgsql pdo_sqlite pdo_odbc pdo_dblib --ini-name "${php_layer_dir}"/etc/php.ini \
  && git clone --recurse-submodules=designs --depth 1 --shallow-submodules --branch "v${ADMINER_VERSION}" https://github.com/vrana/adminer.git /tmp/adminer \
  && curl -fsSL "https://github.com/vrana/adminer/releases/download/v${ADMINER_VERSION}/adminer-${ADMINER_VERSION}.php" -o adminer.php \
  && cp -r /tmp/adminer/designs/ /tmp/adminer/plugins/ . \
  && rm -rf /tmp/adminer/ \
  # cleanup
  && scanelp "${php_layer_dir}/lib/php/extensions" > runtime.txt \
  && apt-get purge -y --auto-remove ${buildDeps} \
  && install-packages $(< runtime.txt) \
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
  && mkdir -p /usr/share/man/man{1..8} ${php_layer_dir}/log/ \
  && chown -R ${DRYCC_UID}:${DRYCC_GID} /opt/drycc

COPY entrypoint.sh /opt/drycc/php/bin
COPY *.php /opt/drycc/adminer/
USER 1001
ENTRYPOINT [ "init-stack", "entrypoint.sh", "php-entrypoint" ]

CMD [ "boot" ]
EXPOSE 80
EXPOSE 443
