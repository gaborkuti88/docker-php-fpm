FROM php:7.2-fpm
RUN apt-get update && apt-get -y upgrade \
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
         \
        git \
        && apt-get install --yes ssmtp

RUN pecl install mcrypt-1.0.2 \
		&& docker-php-ext-enable mcrypt \
		&& docker-php-ext-install \
        pdo \
        pdo_mysql \
        bcmath \
        intl \
        xsl \
        zip \
        soap \
        opcache \
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

ARG HOST_UID=1000
RUN usermod -u ${HOST_UID} www-data && \
    groupmod -g ${HOST_UID} www-data


# install composer
RUN curl -s http://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer
RUN mkdir /composer && chown www-data:www-data /composer
RUN chown www-data:www-data /composer

# install libsodium
RUN mkdir /sodiuminstall
WORKDIR /sodiuminstall
RUN curl https://download.libsodium.org/libsodium/releases/libsodium-1.0.18.tar.gz --output libsodium.tar.gz
RUN tar -xzf libsodium.tar.gz 
WORKDIR /sodiuminstall/libsodium-1.0.18
RUN ./configure
RUN make && make install
RUN pecl install -f libsodium
RUN echo "extension=libsodium.so" > "/usr/local/etc/php/conf.d/libsodium.ini"
RUN rm -rf /sodiuminstall

RUN rm -rf /tmp/* /var/lib/apt/lists/* /var/cache/apk/* \
    && apt-get -y autoremove && apt-get -y autoclean


WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
