FROM phpswoole/swoole:5.1-php8.3

# Install dependencies and compile extensions
RUN apt-get update \
    && apt-get install -y \
        libicu-dev \
        libzip-dev \
        libfreetype6-dev \
        libjpeg-dev \
        libpng-dev \
        libonig-dev \
        libssl-dev \
        zlib1g-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd intl zip gettext bcmath pcntl exif \
    && pecl install mongodb \
    && docker-php-ext-enable mongodb \
    && apt-get clean \
    && apt-get purge -y --auto-remove \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY configs/php.ini "/usr/local/etc/php/php.ini"

WORKDIR /app
