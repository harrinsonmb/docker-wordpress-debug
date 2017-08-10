FROM php:5.6-apache

# install the PHP extensions we need
RUN set -ex; \
	\
	apt-get update; \
	apt-get install -y \
		libjpeg-dev \
		libpng12-dev \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr; \
	docker-php-ext-install gd mysqli opcache
# TODO consider removing the *-dev deps and only keeping the necessary lib* packages

# XDEBUG install
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
	
# XDEBUG configuration
ENV XDEBUG_PORT 9000
ENV XDEBUGINI_PATH=/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN cat $XDEBUGINI_PATH
RUN echo "zend_extension="`find /usr/local/lib/php/extensions/ -iname 'xdebug.so'` > $XDEBUGINI_PATH
RUN echo "xdebug.remote_enable=1" >> $XDEBUGINI_PATH
RUN echo "xdebug.remote_handler=dbgp" >> $XDEBUGINI_PATH
RUN echo "xdebug.remote_port=$XDEBUG_PORT" >> $XDEBUGINI_PATH
RUN echo "xdebug.remote_autostart=1" >> $XDEBUGINI_PATH
RUN echo "xdebug.remote_connect_back=0" >> $XDEBUGINI_PATH
RUN echo "xdebug.idekey=docker" >> $XDEBUGINI_PATH
RUN echo "xdebug.remote_host=192.168.1.146" >> $XDEBUGINI_PATH

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini
	
RUN a2enmod rewrite expires

VOLUME /var/www/html

ENV WORDPRESS_VERSION 4.7.5
ENV WORDPRESS_SHA1 fbe0ee1d9010265be200fe50b86f341587187302

RUN set -ex; \
	curl -o wordpress.tar.gz -fSL "https://wordpress.org/wordpress-${WORDPRESS_VERSION}.tar.gz"; \
	echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c -; \
# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
	tar -xzf wordpress.tar.gz -C /usr/src/; \
	rm wordpress.tar.gz; \
	chown -R www-data:www-data /usr/src/wordpress
	
COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["apache2-foreground"]
