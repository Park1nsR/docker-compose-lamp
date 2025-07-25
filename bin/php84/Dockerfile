FROM php:8.4-apache-bookworm as builder

# Surpresses debconf complaints of trying to install apt packages interactively
# https://github.com/moby/moby/issues/4032#issuecomment-192327844
 
ARG DEBIAN_FRONTEND=noninteractive

# Update
RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get --no-install-recommends install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*

# Install useful tools and install important libaries
RUN apt-get -y update && \
    apt-get -y --no-install-recommends install \
        nano wget dialog libsqlite3-dev libsqlite3-0 \
        default-mysql-client zlib1g-dev libzip-dev libicu-dev && \
    apt-get install -y iputils-ping && \
    apt-get -y --no-install-recommends install --fix-missing \
        apt-utils build-essential git curl libonig-dev \
        libcurl4 libcurl4-openssl-dev zip openssl && \
    rm -rf /var/lib/apt/lists/*

# Install xdebug
RUN pecl install xdebug-3.4.4 && \
    docker-php-ext-enable xdebug && \
    mkdir /var/log/xdebugi && \
    echo "xdebug.mode=coverage" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install redis
RUN pecl install redis-6.2.0 && \
    docker-php-ext-enable redis

# Install imagick
RUN apt-get update && \
    apt-get -y --no-install-recommends install --fix-missing libmagickwand-dev && \
    rm -rf /var/lib/apt/lists/*

# Imagick Commit to install
# https://github.com/Imagick/imagick
ARG IMAGICK_COMMIT="b47e91c83b0eb8db951794d0ed363ae20748b795"

RUN cd /usr/local/src && \
    git clone https://github.com/Imagick/imagick && \
    cd imagick && \
    git checkout ${IMAGICK_COMMIT} && \
    phpize && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf imagick && \
    docker-php-ext-enable imagick

# Other PHP8 Extensions
RUN docker-php-ext-install pdo_mysql \
    pdo_sqlite \
    bcmath \
    mysqli \
    curl \
    zip \
    -j$(nproc) intl \
    mbstring \
    gettext \
    calendar \
    exif

# Install Freetype 
RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
    docker-php-ext-install gd

# Ensure an SSL directory exists
RUN mkdir -p /etc/apache2/ssl

# Enable SSL support
RUN a2enmod ssl && a2enmod rewrite

# Enable apache modules
RUN a2enmod rewrite headers

# Install Composer
curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Cleanup
RUN rm -rf /usr/src/*
