FROM debian:bullseye

LABEL org.opencontainers.image.source https://github.com/fauzanelka/docker-php-legacy

ADD --chmod=755 https://github.com/shivammathur/php-builder/releases/latest/download/install.sh /usr/local/bin/php-installer

RUN php-installer 7.4 release zts

RUN switch_sapi -v 7.4 -s apache:apache

RUN a2enmod rewrite headers ssl

ENV APACHE_CONFDIR /etc/apache2
ENV APACHE_ENVVARS $APACHE_CONFDIR/envvars

RUN set -ex \
	\
# generically convert lines like
#   export APACHE_RUN_USER=www-data
# into
#   : ${APACHE_RUN_USER:=www-data}
#   export APACHE_RUN_USER
# so that they can be overridden at runtime ("-e APACHE_RUN_USER=...")
	&& sed -ri 's/^export ([^=]+)=(.*)$/: ${\1:=\2}\nexport \1/' "$APACHE_ENVVARS" \
	\
# setup directories and permissions
	&& . "$APACHE_ENVVARS" \
	&& for dir in \
		"$APACHE_LOCK_DIR" \
		"$APACHE_RUN_DIR" \
		"$APACHE_LOG_DIR" \
		/var/www/html \
	; do \
		rm -rvf "$dir" \
		&& mkdir -p "$dir" \
		&& chown -R "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$dir"; \
	done

# logs should go to stdout / stderr
RUN set -ex \
	&& . "$APACHE_ENVVARS" \
	&& ln -sfT /dev/stderr "$APACHE_LOG_DIR/error.log" \
	&& ln -sfT /dev/stdout "$APACHE_LOG_DIR/access.log" \
	&& ln -sfT /dev/stdout "$APACHE_LOG_DIR/other_vhosts_access.log"

ADD --chmod=755 https://raw.githubusercontent.com/docker-library/php/master/apache2-foreground /usr/local/bin/apache2-foreground
WORKDIR /var/www/html

EXPOSE 80
CMD [ "apache2-foreground" ]