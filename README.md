# Docker PHP

Docker PHP is a collection of optimized and extensible container images for running PHP applications in production.

All images are based on our [Docker Runtime](https://github.com/sitepilot/docker-runtime) image, an optimized and
extensible Ubuntu container image.

## Usage

This repository creates several variations of Docker images, enabling you to select precisely what you require. Just
utilize this image naming pattern in any of your projects:

```bash
ghcr.io/sitepilot/php-{{variation-name}}:{{php-version}}
```

For example, if you wish to run **PHP 8.2** with **FPM** & **NGINX**, use the following image name:

```bash
ghcr.io/sitepilot/php-nginx:8.1
```

## Customize an image

To customize an image and avoid potential breaking changes in your container builds, use the following image naming
pattern in your Dockerfile:

```Dockerfile
FROM ghcr.io/sitepilot/php-{{variation-name}}:{{runtime-version}}-{{php-version}}
```

For example, if you wish to customize the **PHP 8.2** with **FPM** & **NGINX** image, which is built upon
the [Runtime V1](https://github.com/sitepilot/docker-runtime/tree/1.x) image (Ubuntu 22.04 LTS), include the
following `FROM` line in your Dockerfile:

```Dockerfile
FROM ghcr.io/sitepilot/php-nginx:v1-8.2
```

## Variations

The following Docker image variations are available:

| Image                         | Description                            |
|-------------------------------|----------------------------------------|
| `ghcr.io/sitepilot/php-cli`   | PHP command line interface.            |
| `ghcr.io/sitepilot/php-fpm`   | PHP with PHP-FPM.                      |
| `ghcr.io/sitepilot/php-nginx` | PHP-FPM with Nginx web server.         |
| `ghcr.io/sitepilot/php-ols`   | PHP-FPM with OpenLitespeed web server. |

## Versions

The following PHP versions are available:

* PHP 7.4
* PHP 8.0
* PHP 8.1
* PHP 8.2

You can find a list of installed PHP extensions for each PHP version [here](./src/packages).

## Environment

The following environment variables are available to modify the configuration of an image:

| Name                        | Value     | php-nginx | php-ols | php-fpm | php-cli |
|-----------------------------|-----------|-----------|---------|---------|---------|
| `PHP_DATE_TIMEZONE`         | `UTC`     | ✅         | ✅       | ✅       | ✅       |
| `PHP_MEMORY_LIMIT`          | `256M`    | ✅         | ✅       | ✅       | ✅       |
| `PHP_MAX_EXECUTION_TIME`    | `300`     | ✅         | ✅       | ✅       | ✅       |
| `PHP_MAX_INPUT_VARS`        | `10000`   | ✅         | ✅       | ✅       | ✅       |
| `PHP_FPM_CONTROL`           | `dynamic` | ✅         | ✅       | ✅       |         |
| `PHP_FPM_MAX_CHILDREN`      | `20`      | ✅         | ✅       | ✅       |         |
| `PHP_FPM_START_SERVERS`     | `2`       | ✅         | ✅       | ✅       |         |
| `PHP_FPM_MIN_SPARE_SERVERS` | `1`       | ✅         | ✅       | ✅       |         |
| `PHP_FPM_MAX_SPARE_SERVERS` | `3`       | ✅         | ✅       | ✅       |         |
| `PHP_POST_MAX_SIZE`         | `100M`    | ✅         | ✅       |         |         |
| `PHP_UPLOAD_MAX_FILESIZE`   | `100M`    | ✅         | ✅       |         |         |

## Variation Specific Docs

### PHP-NGINX

#### Environment

| Name                         | Value            |
|------------------------------|------------------|
| `NGINX_PUBLIC_DIR`           | -                |
| `NGINX_FASTCGI_PASS`         | `127.0.0.1:9000` |
| `NGINX_CLIENT_MAX_BODY_SIZE` | `100M`           |

#### Add a vhost / site

Custom vhosts are loaded from the `/etc/nginx/sites-enabled` folder. Mount a file with a `.conf` extension to this
folder to add a custom vhost.

#### Add location directive

Custom location directives are loaded from the `/etc/nginx/location.d` folder. Mount a file with a `.conf` extension to
this folder to add a custom location directive.

### PHP-OLS

#### Environment

| Name                 | Value            |
|----------------------|------------------|
| `OLS_PUBLIC_DIR`     | -                |
| `OLS_FASTCGI_PASS` | `127.0.0.1:9000` |
