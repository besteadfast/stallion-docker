FROM php:7.4-fpm-alpine

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

# Fix for iconv: https://github.com/docker-library/php/issues/1121
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so

COPY ./php.ini /usr/local/etc/php/conf.d/zz-craftcms.ini
COPY ./phpfpm.conf /usr/local/etc/php-fpm.d/zz-craftcms.conf

RUN set -ex \
	&& curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer

# Create the storage directory and make it writeable by PHP
RUN mkdir -p /app/storage && \
	mkdir -p /app/storage/runtime && \
	chown -R www-data:www-data /app/storage

# Create the cpresources directory and make it writeable by PHP
RUN mkdir -p /app/web/cpresources && \
	chown -R www-data:www-data /app/web/cpresources

WORKDIR /app

# Force permissions, update Craft, and start php-fpm

# Do a `composer install` without running any Composer scripts
# - If `composer.lock` is present, it will install what is in the lock file
# - If `composer.lock` is missing, it will update to the latest dependencies
#   and create the `composer.lock` file
# This automatic running adds to the startup overhead of `docker-compose up`
# but saves far more time in not having to deal with out of sync versions
# when working with teams or multiple environments
CMD composer install --no-scripts --optimize-autoloader --no-interaction \
	&& \
	chown -R www-data:www-data /app/vendor \
	&& \
	chown -R www-data:www-data /app/storage \
	&& \
	php-fpm