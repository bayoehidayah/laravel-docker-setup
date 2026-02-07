FROM ubuntu:24.04 AS main

WORKDIR /var/www/html
ENV PHP_VERSION=8.5
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
  ca-certificates \
  curl \
  vim \
  git \
  unzip \
  supervisor \
  htop \
  software-properties-common \
  gnupg \
  gnupg-agent

RUN add-apt-repository -y ppa:ondrej/php \
  && apt-get install -y --no-install-recommends \
  php${PHP_VERSION}-common \
  php${PHP_VERSION}-cli \
  php${PHP_VERSION}-readline \
  php${PHP_VERSION}-mbstring \
  php${PHP_VERSION}-xml \
  php${PHP_VERSION}-yaml \
  php${PHP_VERSION}-xsl \
  php${PHP_VERSION}-zip \
  php${PHP_VERSION}-gd \
  php${PHP_VERSION}-intl \
  php${PHP_VERSION}-curl \
  php${PHP_VERSION}-ds \
  php${PHP_VERSION}-bcmath \
  php${PHP_VERSION}-gmp \
  php${PHP_VERSION}-uuid \
  php${PHP_VERSION}-igbinary \
  php${PHP_VERSION}-ldap \
  php${PHP_VERSION}-pgsql \
  php${PHP_VERSION}-imagick \
  php${PHP_VERSION}-redis \
  php${PHP_VERSION}-decimal \
  php${PHP_VERSION}-gnupg

# install composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# install postgresql client to support artisan schema:dump
RUN curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /etc/apt/keyrings/postgresql.gpg \
  && echo "deb [signed-by=/etc/apt/keyrings/postgresql.gpg] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | tee /etc/apt/sources.list.d/pgdg.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends postgresql-client-18

# install node
# https://github.com/nodesource/distributions#debian-and-ubuntu-based-distributions
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
  && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_22.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
  && apt-get update \
  && apt-get install -y --no-install-recommends nodejs \
  && corepack enable \
  && npm install -g --unsafe-perm puppeteer@22

# puppeteer deps
# https://github.com/puppeteer/puppeteer/blob/puppeteer-v22.0.0/docs/troubleshooting.md#chrome-doesnt-launch-on-linux
RUN apt-get install -y --no-install-recommends \
  ca-certificates \
  fonts-liberation \
  libasound2t64 \
  libatk-bridge2.0-0 \
  libatk1.0-0 \
  libc6 \
  libcairo2 \
  libcups2 \
  libdbus-1-3 \
  libexpat1 \
  libfontconfig1 \
  libgbm1 \
  libgcc1 \
  libglib2.0-0 \
  libgtk-3-0 \
  libnspr4 \
  libnss3 \
  libpango-1.0-0 \
  libpangocairo-1.0-0 \
  libstdc++6 \
  libx11-6 \
  libx11-xcb1 \
  libxcb1 \
  libxcomposite1 \
  libxcursor1 \
  libxdamage1 \
  libxext6 \
  libxfixes3 \
  libxi6 \
  libxrandr2 \
  libxrender1 \
  libxss1 \
  libxtst6 \
  lsb-release \
  wget \
  xdg-utils

# support non latin character for pdf rendering in puppeteer
RUN apt-get install -y --no-install-recommends \
  fontconfig \
  fonts-noto \
  fonts-noto-cjk

RUN apt-get -y upgrade


FROM main AS swoole

RUN apt-get install -y --no-install-recommends \
  php${PHP_VERSION}-xdebug \
  php${PHP_VERSION}-swoole

CMD ["/usr/bin/supervisord", "-n"]


FROM main AS fpm

RUN apt-get install -y --no-install-recommends \
  php${PHP_VERSION}-xdebug \
  php${PHP_VERSION}-fpm

RUN usermod -u 1001 www-data

RUN mkdir /run/php -p
RUN sed -i 's/^listen = .*/listen = 0.0.0.0:9000/' /etc/php/${PHP_VERSION}/fpm/pool.d/www.conf

CMD ["php-fpm8.5", "-F"]
EXPOSE 9000
