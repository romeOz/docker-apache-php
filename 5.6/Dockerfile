FROM ubuntu:xenial
MAINTAINER romeOz <serggalka@gmail.com>

ENV OS_LOCALE="en_US.UTF-8"
RUN locale-gen ${OS_LOCALE}
ENV LANG=${OS_LOCALE} \
    LANGUAGE=en_US:en \
    LC_ALL=${OS_LOCALE}

ENV APACHE_CONF=/etc/apache2 \
    PHP_CONF=/etc/php/5.6 \
    PHP_DATA_DIR=/var/lib/php

COPY ./app /var/www/app/
COPY entrypoint.sh /sbin/entrypoint.sh

RUN	\
	buildDeps='software-properties-common python-software-properties' \
	# Install ansible
	&& apt-get update \
	&& apt-get install --no-install-recommends -y $buildDeps \
	&& add-apt-repository -y ppa:ondrej/php \
	&& add-apt-repository -y ppa:ondrej/apache2 \
	&& apt-get update \
    && apt-get install -y curl apache2 libapache2-mod-php5.6 php5.6-cli php5.6-readline php5.6-mbstring php5.6-intl php5.6-json php5.6-curl php5.6-mcrypt php5.6-gd php5.6-pgsql php5.6-mysql \
    # Apache settings
    && cp /dev/null ${APACHE_CONF}/conf-available/other-vhosts-access-log.conf \
    && rm ${APACHE_CONF}/sites-enabled/000-default.conf ${APACHE_CONF}/sites-available/000-default.conf \
    && a2enmod rewrite php5.6 \
    # PHP settings
	&& phpenmod mcrypt \
	# Install composer
	&& curl -sS https://getcomposer.org/installer | php -- --version=1.1.0 --install-dir=/usr/local/bin --filename=composer \	
	# Cleaning
	&& apt-get purge -y --auto-remove $buildDeps \
	&& apt-get autoremove -y \
	&& rm -rf /var/lib/apt/lists/* \
	# Forward request and error logs to docker log collector
	&& ln -sf /dev/stdout /var/log/apache2/access.log \
	&& ln -sf /dev/stderr /var/log/apache2/error.log \
	&& chmod 755 /sbin/entrypoint.sh \
    && chown www-data:www-data ${PHP_DATA_DIR} -Rf

COPY ./configs/apache2.conf ${APACHE_CONF}/apache2.conf
COPY ./configs/app.conf ${APACHE_CONF}/sites-enabled/app.conf
COPY ./configs/php.ini  ${PHP_CONF}/apache2/conf.d/custom.ini

WORKDIR /var/www/app/

EXPOSE 80 443

# By default, simply start apache.
CMD ["/sbin/entrypoint.sh"]