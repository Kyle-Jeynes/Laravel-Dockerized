FROM node:latest AS npm
COPY src /var/www/html

WORKDIR /var/www/html

RUN npm install && \
    npm run prod

FROM php:7.4-fpm

RUN groupadd -g 101 nginx && \
    useradd -u 101 -ms /bin/bash -g nginx nginx

COPY --from=npm --chown=nginx:nginx /var/www/html /var/www/html

WORKDIR /var/www/html

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        curl \
        gifsicle \
        git \
        jpegoptim \
        libfreetype6-dev \
        libicu-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        locales \
        optipng \
        pngquant \
        unzip \
        vim \
        zip && \
    apt-get clean && \
    rm -rf /var/cache/apt/lists/*

RUN docker-php-ext-install pdo_mysql && \
    docker-php-ext-install mysqli && \
    docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install gd && \
    docker-php-ext-install intl; \
    docker-php-ext-install exif; \
    docker-php-ext-enable exif; \

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

LABEL maintainer="kyle.jeynes@iezon.co.uk" \
      version="1.3" \
      description="Builds a PHP 7.4-FPM customised image with Laravel extensions, node npm and vcs files"

USER nginx

EXPOSE 9000

CMD ["php-fpm"]