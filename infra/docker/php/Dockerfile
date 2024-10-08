FROM php:8.3-fpm-bullseye AS base

WORKDIR /workspace

# timezone environment
ENV TZ=UTC \
  # locale
  LANG=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LC_ALL=en_US.UTF-8 \
  # composer environment
  COMPOSER_HOME=/composer

ARG UID=1000
ARG GID=1000

COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer
# install Node.js
COPY --from=node:18.14.2 /usr/local/bin /usr/local/bin
COPY --from=node:18.14.2 /usr/local/lib /usr/local/lib
RUN <<EOF
  apt-get update
  apt-get -y install --no-install-recommends \
    locales=2.31-13+deb11u10 \
    git=1:2.30.2-1+deb11u2 \
    unzip=6.0-26+deb11u1 \
    libzip-dev=1.7.3-1 \
    libicu-dev=67.1-7 \
    libonig-dev=6.9.6-1.1 \
    default-mysql-client=1.0.7
  locale-gen en_US.UTF-8
  localedef -f UTF-8 -i en_US en_US.UTF-8
  docker-php-ext-install \
    intl \
    pdo_mysql \
    zip \
    bcmath
  # permission denied bind mount in Linux environment
  groupadd --gid $GID phper
  useradd --uid $UID --gid $GID phper
  mkdir /composer
  mkdir -p /home/phper/.config/psysh
  chown phper:phper /composer
  chown phper:phper /workspace
  chown phper:phper /home/phper/.config/psysh
  apt-get clean
  rm -rf /var/lib/apt/lists/*
EOF

FROM base AS development

COPY ./infra/docker/php/php.development.ini /usr/local/etc/php/php.ini

USER phper

FROM base AS development-xdebug

RUN <<EOF
  pecl install xdebug
  docker-php-ext-enable xdebug
EOF

COPY ./infra/docker/php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

USER phper

FROM base AS deploy

COPY ./infra/docker/php/php.deploy.ini /usr/local/etc/php/php.ini
COPY --chown=phper:phper ./src /workspace

USER phper

RUN <<EOF
  composer install --quiet --no-interaction --no-ansi --no-dev --no-scripts --no-progress --prefer-dist
  composer dump-autoload --optimize
  chmod -R 777 storage bootstrap/cache
  php artisan optimize:clear
  php artisan optimize
  php artisan serve
EOF
