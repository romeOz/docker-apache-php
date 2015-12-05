FROM ubuntu:trusty
MAINTAINER romeOz <serggalka@gmail.com>

ENV PHP_VERSION=5.6 \
	OS_LOCALE="en_US.UTF-8"
RUN locale-gen ${OS_LOCALE}
ENV LANG ${OS_LOCALE}
ENV LANGUAGE en_US:en
ENV LC_ALL ${OS_LOCALE}

# Add playbooks to the Docker image
ADD ./app /var/www/app/
ADD ./provisioning /tmp/provisioning/

COPY entrypoint.sh /sbin/entrypoint.sh

WORKDIR /tmp/provisioning/

RUN	\
	buildDeps='software-properties-common python-software-properties' \
	# Install ansible
	&& apt-get update \
	&& apt-get install --no-install-recommends -y $buildDeps \
	&& add-apt-repository -y ppa:ansible/ansible \
	&& apt-get update \
	&& apt-get install -y ansible \
	# Apply playbooks
	&& ansible-playbook -v docker.yml -i 'docker,' -c local --extra-vars "php_version=${PHP_VERSION}" \
	# Cleaning
	&& apt-get purge -y --auto-remove ansible $buildDeps \
	&& apt-get autoremove -y \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf /tmp/provisioning* \
	# Forward request and error logs to docker log collector
	&& ln -sf /dev/stdout /var/log/apache2/access.log \
	&& ln -sf /dev/stderr /var/log/apache2/error.log \
	&& chmod 755 /sbin/entrypoint.sh

WORKDIR /var/www/app/

EXPOSE 80 433

# By default, simply start apache.
CMD ["/sbin/entrypoint.sh"]