FROM php:7.0-fpm
RUN echo "deb [check-valid-until=no] http://cdn-fastly.deb.debian.org/debian jessie main" > /etc/apt/sources.list.d/jessie.list
RUN echo "deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports main" > /etc/apt/sources.list.d/jessie-backports.list
RUN sed -i '/deb http:\/\/deb.debian.org\/debian jessie-updates main/d' /etc/apt/sources.list

RUN apt-get update -o Acquire::Check-Valid-Until=false && apt-get -y upgrade \
    && apt-get install --no-install-recommends -y \
        libpq-dev \
        libmagickwand-dev \
        zlib1g-dev \
        libicu-dev \
        libmcrypt-dev \
        libxslt-dev \
        curl \
        g++ \
        apt-utils \
        cron \
        git \
        && apt-get install --yes ssmtp

RUN docker-php-ext-install \
        pdo \
        pdo_mysql \
        bcmath \
        intl \
        xsl \
        zip \
        soap \
        opcache \
        mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd \
    && pecl install xdebug && \
    docker-php-ext-enable xdebug

# copy configurations
RUN cp /etc/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf.dist
COPY ./ssmtp.conf /etc/ssmtp/ssmtp.conf
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY ./config.ini /usr/local/etc/php/conf.d
COPY ./config.pool.conf /usr/local/etc/php-fpm.d/
COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

ARG HOST_UID=1000
RUN usermod -u ${HOST_UID} www-data && \
    groupmod -g ${HOST_UID} www-data

# install composer
RUN curl -s http://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer
RUN mkdir /composer && chown www-data:www-data /composer
RUN chown www-data:www-data /composer

COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

COPY ./entrypoint.sh /
ENTRYPOINT /entrypoint.sh

RUN rm -rf /tmp/* /var/lib/apt/lists/* /var/cache/apk/* \
    && apt-get -y autoremove && apt-get -y autoclean

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000