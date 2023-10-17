# --------------- [PHP-CLI] --------------- #
FROM ghcr.io/sitepilot/runtime:1.x AS cli

USER root

ARG PHP_VERSION=8.1

ENV RUNTIME_LOG_LEVEL=4 \
    PHP_VERSION=${PHP_VERSION}

RUN $RUNTIME_BIN_DIR/install gnupg2 software-properties-common less

ADD packages/php-${PHP_VERSION}.txt /tmp/packages.txt

RUN add-apt-repository ppa:ondrej/php; \
    mkdir -p /etc/php/current; \
    ln -sf /etc/php/current /etc/php/${PHP_VERSION}; \
    $RUNTIME_BIN_DIR/install msmtp $(cat /tmp/packages.txt); \
    chown -R $RUNTIME_UID:$RUNTIME_GID /etc/php/current/cli/conf.d

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY --from=wordpress:cli-2 /usr/local/bin/wp /usr/local/bin/wp

COPY cli /

RUN $RUNTIME_BIN_DIR/finalize

USER $RUNTIME_UID

# --------------- [PHP-FPM] --------------- #
FROM cli AS fpm

USER root

ENV RUNTIME_LOG_LEVEL=2

RUN $RUNTIME_BIN_DIR/install php${PHP_VERSION}-fpm libfcgi-bin; \
    ln -sf /usr/sbin/php-fpm${PHP_VERSION} /usr/sbin/php-fpm; \
    chown -R $RUNTIME_UID:$RUNTIME_GID /etc/php/current/fpm; \
    rm /etc/php/current/fpm/pool.d/www.conf

COPY fpm /

RUN $RUNTIME_BIN_DIR/finalize

USER $RUNTIME_UID

EXPOSE 9000

CMD ["/usr/sbin/php-fpm"]

# --------------- [PHP-NGINX] --------------- #
FROM fpm AS nginx

USER root

ENV RUNTIME_LOG_LEVEL=2 \
    NGINX_LOGS_DIR="$RUNTIME_WORKDIR/logs" \
    NGINX_CERTS_DIR="$RUNTIME_WORKDIR/certs" \
    NGINX_FILES_DIR="$RUNTIME_WORKDIR/files" \
    NGINX_PUBLIC_DIR="/"

RUN $RUNTIME_BIN_DIR/install nginx; \
    rm /etc/nginx/sites-enabled/default; \
    chown -R $RUNTIME_UID:$RUNTIME_GID /etc/nginx /var/log/nginx /var/lib/nginx;

COPY nginx /

RUN $RUNTIME_BIN_DIR/finalize

USER $RUNTIME_UID

WORKDIR $NGINX_FILES_DIR

EXPOSE 80

EXPOSE 443

CMD ["/php-nginx"]

# --------------- [PHP-SSH] --------------- #
FROM cli AS ssh

USER root

ENV RUNTIME_LOG_LEVEL=2 \
    SSH_CONFIG_DIR="$RUNTIME_WORKDIR/.ssh"

RUN $RUNTIME_BIN_DIR/install openssh-server; \
    mkdir -p  /run/sshd; \
    rm -rf /etc/update-motd.d/*

COPY ssh /

RUN $RUNTIME_BIN_DIR/finalize

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
