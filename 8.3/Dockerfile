# Built directly on the official PHP image, so a rebuild picks up a new PHP release as soon as the PHP team
# publishes it. Everything the phpswoole/swoole image used to provide (Swoole, Redis, pdo_mysql, Composer,
# Supervisor, the entrypoint and its boot scripts) is installed here instead.
#
# To move to a different release, override the build arguments below, e.g.
#   docker build --pull --build-arg SWOOLE_VERSION=6.2.7 -f Dockerfile .

ARG PHP_VERSION=8.3
ARG DEBIAN_RELEASE=trixie
ARG COMPOSER_VERSION=2

FROM composer:${COMPOSER_VERSION} AS composer

FROM php:${PHP_VERSION}-cli-${DEBIAN_RELEASE}

# Swoole is built from https://github.com/swoole/swoole-src at this tag.
ARG SWOOLE_VERSION=6.2.6
ARG REDIS_VERSION=6.3.0
# Latest stable release by default; set to e.g. "mongodb-2.5.8" to pin.
ARG MONGODB_PACKAGE=mongodb

ENV DEBIAN_FRONTEND=noninteractive
ENV TERM=xterm-color

# DEV_MODE=true keeps the Swoole sources and adds debugging tools (see /usr/local/bin/install-swoole.sh).
ARG DEV_MODE
ENV DEV_MODE=$DEV_MODE

COPY --from=composer /usr/bin/composer /usr/bin/composer

# Entrypoint, boot scripts, Supervisor layout and helper scripts, taken unchanged from the
# phpswoole/swoole:6.2 image (https://github.com/swoole/docker-swoole, Apache License 2.0).
COPY rootfs/ /

RUN set -eux; \
    chmod +x /entrypoint.sh /usr/local/bin/*.sh /usr/local/boot/*.sh /var/www/server.php; \
    apt-get update; \
    # Needed to build and run Swoole and Redis, plus Supervisor.
    apt-get install -y --no-install-recommends \
        libbrotli-dev \
        libcurl4-openssl-dev \
        libpq-dev \
        libsqlite3-dev \
        libssl-dev \
        supervisor \
        unixodbc-dev \
        unzip \
        zlib1g-dev; \
    # Needed by the other extensions.
    apt-get install -y \
        ca-certificates \
        curl \
        gnupg \
        libicu-dev \
        libzip-dev \
        libfreetype-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libwebp-dev \
        libonig-dev; \
    mkdir -p /var/log/supervisor; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp; \
    docker-php-ext-install -j"$(nproc)" pdo_mysql sockets gd intl zip gettext bcmath pcntl exif; \
    pecl channel-update pecl.php.net; \
    pecl install --configureoptions 'enable-redis-igbinary="no" enable-redis-lzf="yes" enable-redis-zstd="yes" enable-redis-msgpack="no" enable-redis-lz4="no" with-liblz4="yes"' "redis-${REDIS_VERSION}"; \
    pecl install "${MONGODB_PACKAGE}"; \
    docker-php-ext-enable redis mongodb; \
    install-swoole.sh "${SWOOLE_VERSION}" \
        --enable-brotli \
        --enable-mysqlnd \
        --enable-sockets \
        --enable-swoole-curl \
        --enable-swoole-ftp \
        --enable-swoole-pgsql \
        --enable-swoole-sqlite \
        --enable-swoole-stdext \
        --enable-zstd \
        --with-swoole-odbc=unixODBC,/usr \
        --with-swoole-ssh2; \
    rm -rf /tmp/pear

COPY configs/php.ini "/usr/local/etc/php/php.ini"

# Fail the build rather than publish an image with a missing or unloadable extension: PHP only prints a
# warning for an extension it cannot load and still exits 0.
RUN set -eux; \
    php -r 'foreach (["swoole", "mongodb", "redis", "pdo_mysql", "sockets", "gd", "intl", "zip", "gettext", "bcmath", "pcntl", "exif", "Zend OPcache", "sodium"] as $e) { if (!extension_loaded($e)) { fwrite(STDERR, "Missing PHP extension: $e\n"); exit(1); } }'; \
    if php -v 2>&1 | grep -qi warning; then php -v; exit 1; fi; \
    composer --version; \
    supervisord --version

# Legacy helper scripts that the phpswoole/swoole:6.2 image still ships (branch 6.2.x of docker-swoole), even
# though upstream has since removed them on master. Kept so the image provides everything the old base did.
COPY rootfs-legacy/ /

ENTRYPOINT ["/entrypoint.sh"]
CMD []
WORKDIR /app
