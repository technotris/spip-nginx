#syntax=docker/dockerfile:1.4
# Versions
# For spip 3.1 not supporting 7.3 without errors
# 7.4-fpm-alpine3.15 latest 7.4
FROM php:7.4-fpm-alpine3.15 AS spip_php

ENV APP_ENV=dev
WORKDIR /app
# persistent / runtime deps
# hadolint ignore=DL3018
RUN apk add --no-cache \
		acl file gettext git zip wget composer msmtp \
		php7-sodium php7-simplexml php7-tokenizer php7-xmlwriter php7-xml php7-dom \
		libzip-dev curl-dev libxml2-dev libpng-dev libsodium-dev icu-dev \
		libwebp-dev libpng-dev libjpeg-turbo-dev  freetype-dev \
	;

#https://github.com/docker-library/php/issues/1532
RUN set -eux; \
	apk add --no-cache imagemagick-dev; \
# WARNING: imagick is likely not supported on Alpine: https://github.com/Imagick/imagick/issues/328
# https://pecl.php.net/package/imagick
# https://github.com/Imagick/imagick/commit/5ae2ecf20a1157073bad0170106ad0cf74e01cb6 (causes a lot of build failures, but strangely only intermittent ones 🤔)
# see also https://github.com/Imagick/imagick/pull/641
# this is "pecl install imagick-3.7.0", but by hand so we can apply a small hack / part of the above commit
	curl -fL -o imagick.tgz 'https://pecl.php.net/get/imagick-3.7.0.tgz'; \
	echo '5a364354109029d224bcbb2e82e15b248be9b641227f45e63425c06531792d3e *imagick.tgz' | sha256sum -c -; \
	tar --extract --directory /tmp --file imagick.tgz imagick-3.7.0; \
	grep '^//#endif$' /tmp/imagick-3.7.0/Imagick.stub.php; \
	test "$(grep -c '^//#endif$' /tmp/imagick-3.7.0/Imagick.stub.php)" = '1'; \
	sed -i -e 's!^//#endif$!#endif!' /tmp/imagick-3.7.0/Imagick.stub.php; \
	grep '^//#endif$' /tmp/imagick-3.7.0/Imagick.stub.php && exit 1 || :; \
	docker-php-ext-install /tmp/imagick-3.7.0; \
	rm -rf imagick.tgz /tmp/imagick-3.7.0;
	
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp;
# dom for phpunit
RUN docker-php-ext-install intl opcache zip curl xml gd mysqli simplexml phar dom;
RUN docker-php-ext-enable xml simplexml sodium;
RUN echo 'chdir = /var/www/html' >> /usr/local/etc/php-fpm.d/www.conf
RUN sed -i "s/user = www-data/user = 1000/g" /usr/local/etc/php-fpm.d/www.conf
RUN sed -i "s/group = www-data/group = 1000/g" /usr/local/etc/php-fpm.d/www.conf

#change SENDMAIL FOR DEV purpose msmtp
FROM spip_php AS spip_php_dev
ENV APP_ENV=dev
RUN apk add --no-cache msmtp;
# option t to read the "to" and read-envelope-from for the "from" in headers
RUN echo "sendmail_path = '/usr/bin/msmtp -t --host=mailer --port=1025 --read-envelope-from'" >> /usr/local/etc/php/conf.d/php-sendmail.ini
RUN echo "sendmail_from = test@spip.net" >> /usr/local/etc/php/conf.d/php-sendmail.ini