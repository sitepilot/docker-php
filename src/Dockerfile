# --------------- [RUNTIME] --------------- #

FROM ubuntu:22.04 AS runtime

ARG TARGETARCH
ARG DOCKERIZE_VERSION=v0.7.0

LABEL org.opencontainers.image.source = "https://github.com/sitepilot/docker-runtime"

ENV DEBIAN_FRONTEND="noninteractive" \
    RUNTIME_UID=1000 \
    RUNTIME_GID=1000 \
    RUNTIME_USER=app \
    RUNTIME_GROUP=app \
    RUNTIME_PASSWORD="" \
    RUNTIME_LOG_LEVEL=1 \
    RUNTIME_DIR=/runtime \
    RUNTIME_WORKDIR=/app \
    RUNTIME_BIN_DIR=/runtime/bin \
    RUNTIME_INC_DIR=/runtime/inc \
    RUNTIME_RUN_DIR=/runtime/run \
    RUNTIME_LOGS_DIR=/runtime/logs \
    RUNTIME_TEMPLATES_DIR=/runtime/templates \
    RUNTIME_ENTRYPOINT_DIR=/runtime/entrypoint.d \
    RUNTIME_BOOTED_FILE=/runtime/booted

ENV PATH="$PATH:$RUNTIME_BIN_DIR"

ADD runtime/runtime/bin/rt-install $RUNTIME_BIN_DIR/rt-install

RUN rt-install curl wget nano ca-certificates \
    && groupadd -r -g ${RUNTIME_GID} ${RUNTIME_GROUP} \
    && useradd --no-log-init -r -s /usr/bin/bash -m -d ${RUNTIME_WORKDIR} -u ${RUNTIME_UID} -g ${RUNTIME_GID} ${RUNTIME_USER} \
    && wget -O - https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-$TARGETARCH-$DOCKERIZE_VERSION.tar.gz | tar xzf - -C /usr/local/bin

COPY --chown=$RUNTIME_UID:$RUNTIME_GID runtime /

USER $RUNTIME_UID

WORKDIR $RUNTIME_WORKDIR

ENTRYPOINT ["runtime"]

# --------------- [PHP-CLI] --------------- #

FROM runtime AS cli

USER root

ARG PHP_VERSION=8.1

ENV RUNTIME_LOG_LEVEL=4 \
    PHP_VERSION=${PHP_VERSION}

LABEL org.opencontainers.image.source = "https://github.com/sitepilot/docker-php"

RUN rt-install gnupg2 software-properties-common less msmtp unzip

ADD packages/php-${PHP_VERSION}.txt /tmp/packages.txt

RUN add-apt-repository ppa:ondrej/php \
    && mkdir -p /etc/php/current \
    && ln -sf /etc/php/current /etc/php/${PHP_VERSION} \
    && rt-install $(cat /tmp/packages.txt) rsync openssh-server mysql-client \
    && rt-rchown /etc/php/current

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY --from=wordpress:cli-2 /usr/local/bin/wp /usr/local/bin/wp

COPY --chown=$RUNTIME_UID:$RUNTIME_GID php-cli /

USER $RUNTIME_UID

# --------------- [PHP-FPM] --------------- #

FROM cli AS fpm

USER root

ENV RUNTIME_LOG_LEVEL=1

RUN rt-install php${PHP_VERSION}-fpm libfcgi-bin \
    && ln -sf /usr/sbin/php-fpm${PHP_VERSION} /usr/sbin/php-fpm \
    && rm /etc/php/current/fpm/pool.d/www.conf \
    && rt-rchown /etc/php/current

COPY --chown=$RUNTIME_UID:$RUNTIME_GID php-fpm /

USER $RUNTIME_UID

EXPOSE 9000

CMD ["php-fpm", "-F"]

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
    && rt-rchown /etc/nginx /var/log/nginx /var/lib/nginx \
    && setcap "cap_net_bind_service=+ep" /usr/sbin/nginx

COPY --chown=$RUNTIME_UID:$RUNTIME_GID php-nginx /

USER $RUNTIME_UID

WORKDIR $NGINX_FILES_DIR

EXPOSE 80

EXPOSE 443

CMD ["/php-nginx"]

# --------------- [PHP-OLS] --------------- #
FROM fpm as ols

USER root

ENV PATH="$PATH:/usr/local/lsws/bin" \
    OLS_LOGS_DIR="$RUNTIME_WORKDIR/logs" \
    OLS_CERTS_DIR="$RUNTIME_WORKDIR/certs" \
    OLS_FILES_DIR="$RUNTIME_WORKDIR/files" \
    OLS_PUBLIC_DIR="/"

RUN wget -O /etc/apt/trusted.gpg.d/lst_debian_repo.gpg http://rpms.litespeedtech.com/debian/lst_debian_repo.gpg \
    && wget -O /etc/apt/trusted.gpg.d/lst_repo.gpg http://rpms.litespeedtech.com/debian/lst_repo.gpg \
    && echo "deb http://rpms.litespeedtech.com/debian/ jammy main" > /etc/apt/sources.list.d/lst_debian_repo.list \
    && rt-install openlitespeed \
    && rt-rchown /usr/local/lsws \
    && setcap "cap_net_bind_service=+ep" /usr/local/lsws/bin/openlitespeed

COPY --chown=$RUNTIME_UID:$RUNTIME_GID php-ols /

USER $RUNTIME_UID

WORKDIR $OLS_FILES_DIR

EXPOSE 80

EXPOSE 443

CMD ["/php-ols"]
