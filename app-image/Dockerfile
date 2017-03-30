FROM debian:jessie

MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"

ENV NGINX_VERSION 1.11.12-1~jessie

RUN set -e; \
	NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
	found=''; \
	for server in \
		ha.pool.sks-keyservers.net \
		hkp://keyserver.ubuntu.com:80 \
		hkp://p80.pool.sks-keyservers.net:80 \
		pgp.mit.edu \
	; do \
		echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
		apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
	done; \
	test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
	exit 0

RUN echo "deb http://nginx.org/packages/mainline/debian/ jessie nginx" >> /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install --no-install-recommends --no-install-suggests -y \
						ca-certificates \
						nginx=${NGINX_VERSION} \
						nginx-module-xslt \
						nginx-module-geoip \
						nginx-module-image-filter \
						nginx-module-perl \
						nginx-module-njs \
						gettext-base \
	&& rm -rf /var/lib/apt/lists/*

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80 443

# Runit
RUN apt-get install -y --no-install-recommends runit
CMD export > /etc/envvars && /usr/sbin/runsvdir-start
RUN echo 'export > /etc/envvars' >> /root/.bashrc
RUN echo "alias tcurrent='tail /var/log/*/current -f'" | tee -a /root/.bashrc /etc/bash.bashrc

# Define Mautic version and expected SHA1 signature
ENV MAUTIC_VERSION 2.7.1

# Utilities
RUN apt-get install -y --no-install-recommends vim less net-tools inetutils-ping wget curl git telnet nmap socat dnsutils netcat tree htop unzip sudo software-properties-common jq psmisc iproute python ssh

#Required
RUN apt-get install -y cron php-fpm php-xml php-mbstring php-mysql php-mcrypt php-intl php-zip php-imap php-curl composer

#Mautic
RUN wget -O - https://github.com/mautic/mautic/archive/${MAUTIC_VERSION}.tar.gz | tar zx -C /var/www/html --strip-components=1 && \
    cd /var/www/html && \
    mkdir -p .git/hooks && \
    composer install && \
    chown -R www-data:www-data /var/www/html

#Put all Mautic instance config files into one directory    
RUN cd /var/www/html/app  && \
    mkdir -p cache/prod local/config local/themes local/idp local/media/files local/media/images
COPY paths_local.php /var/www/html/app/config/
COPY parameters_local.php /var/www/html/app/config/
RUN  chown -R www-data:www-data /var/www/html

RUN mkdir -p /var/log/mautic && \
    chown www-data:www-data /var/log/mautic

COPY default /etc/nginx/sites-enabled/
COPY crontab /
COPY ./cors /etc/nginx/

# Add runit services
COPY sv /etc/service 
ARG BUILD_INFO
LABEL BUILD_INFO=$BUILD_INFO
