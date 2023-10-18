# --------------- [PHP-CLI] --------------- #
FROM ghcr.io/sitepilot/runtime:v1 AS cli

USER root

ARG PHP_VERSION=8.1

ENV RUNTIME_LOG_LEVEL=4 \
    PHP_VERSION=${PHP_VERSION}

RUN rt-install gnupg2 software-properties-common less msmtp unzip

ADD packages/php-${PHP_VERSION}.txt /tmp/packages.txt

RUN add-apt-repository ppa:ondrej/php \
    && mkdir -p /etc/php/current \
    && ln -sf /etc/php/current /etc/php/${PHP_VERSION} \
    && rt-install $(cat /tmp/packages.txt) \
    && rt-rchown /etc/php/current

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY --from=wordpress:cli-2 /usr/local/bin/wp /usr/local/bin/wp

COPY --chown=$RUNTIME_UID:$RUNTIME_GID cli /

USER $RUNTIME_UID

# --------------- [PHP-FPM] --------------- #
FROM cli AS fpm

USER root

ENV RUNTIME_LOG_LEVEL=1

RUN rt-install php${PHP_VERSION}-fpm libfcgi-bin \
    && ln -sf /usr/sbin/php-fpm${PHP_VERSION} /usr/sbin/php-fpm \
    && rm /etc/php/current/fpm/pool.d/www.conf \
    && rt-rchown /etc/php/current

COPY --chown=$RUNTIME_UID:$RUNTIME_GID fpm /

USER $RUNTIME_UID

EXPOSE 9000

CMD ["/usr/sbin/php-fpm"]

# --------------- [PHP-NGINX] --------------- #
FROM fpm AS nginx

USER root

ENV RUNTIME_LOG_LEVEL=1 \
    NGINX_LOGS_DIR="$RUNTIME_WORKDIR/logs" \
    NGINX_CERTS_DIR="$RUNTIME_WORKDIR/certs" \
    NGINX_FILES_DIR="$RUNTIME_WORKDIR/files" \
    NGINX_PUBLIC_DIR="/"

RUN rt-install nginx \
    && rm /etc/nginx/sites-enabled/default \
    && rt-rchown /etc/nginx /var/log/nginx /var/lib/nginx

COPY --chown=$RUNTIME_UID:$RUNTIME_GID nginx /

USER $RUNTIME_UID

WORKDIR $NGINX_FILES_DIR

EXPOSE 80

EXPOSE 443

CMD ["/php-nginx"]

# --------------- [PHP-SSH] --------------- #
FROM cli AS ssh

USER root

ENV RUNTIME_LOG_LEVEL=1 \
    SSH_CONFIG_DIR="$RUNTIME_WORKDIR/.ssh"

RUN rt-install openssh-server \
    && mkdir -p  /run/sshd \
    && rm -rf /etc/update-motd.d/*

COPY --chown=$RUNTIME_UID:$RUNTIME_GID ssh /

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
