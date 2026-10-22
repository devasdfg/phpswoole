FROM phpswoole/swoole:6.2-php8.3

RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    libicu-dev \
    libzip-dev \
    libfreetype-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libonig-dev \
    libssl-dev \
    zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

COPY configs/php.ini "/usr/local/etc/php/php.ini"

RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
    && docker-php-ext-install -j$(nproc) gd intl zip gettext bcmath pcntl exif

RUN pecl install mongodb \
    && docker-php-ext-enable mongodb \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
