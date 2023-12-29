FROM php:8.3.1RC3-apache-bookworm

RUN apt-get update && \
  apt-get install -y git && \
  apt-get -y install unzip libzip-dev libicu-dev libonig-dev libfreetype6-dev libjpeg62-turbo-dev libpng-dev rsync && \
  apt-get clean && \
  docker-php-ext-install intl pdo_mysql zip bcmath opcache && \
  pecl install xdebug && \
  docker-php-ext-enable xdebug && \
  docker-php-ext-configure gd --with-freetype --with-jpeg && \
  docker-php-ext-install -j$(nproc) gd exif && \
  # install composer
  cd /usr/bin && \
  curl -s http://getcomposer.org/installer | php && \
  ln -s /usr/bin/composer.phar /usr/bin/composer
  # install Node.js, npm
  # apt install nodejs npm -y

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV PATH $PATH:/composer/vendor/bin

COPY ./apache2.conf /etc/apache2/apache2.conf
COPY ./000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ./php.ini /usr/local/etc/php/php.ini

WORKDIR /var/www/html/src
