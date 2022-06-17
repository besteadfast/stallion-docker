FROM php:8.0-fpm-alpine

# https://github.com/docker-library/php/issues/105#issuecomment-249716758
RUN set -ex \
	&& apk update \
	# Install dependencies
	&& apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		freetype-dev \
		icu-dev \
		imagemagick-dev \
		libjpeg-turbo-dev \
		libpng-dev \
		libtool \
		libxml2-dev \
		libzip-dev \
	&& apk add --no-cache \
		freetype \
		git \
		gnu-libiconv \
		icu \
		imagemagick \
		libgomp \
		libjpeg-turbo \
		libpng \
		libxml2 \
		libzip \
		mysql-client \
		zip \
	# Pecl extensions
	&& pecl install \
		imagick \
	# Configure extensions
	&& docker-php-ext-configure \
		gd --with-freetype --with-jpeg \
	# Install extensions
	&& docker-php-ext-install -j$(nproc) \
		bcmath \
		gd \
		intl \
		pdo_mysql \
		soap \
		zip \
	# Enable extensions
	&& docker-php-ext-enable \
		imagick \
	# Remove dependencies
	&& apk del .build-deps

COPY ./php.ini /usr/local/etc/php/conf.d/zz-craftcms.ini
COPY ./phpfpm.conf /usr/local/etc/php-fpm.d/zz-craftcms.conf

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer

WORKDIR /app

CMD	php-fpm