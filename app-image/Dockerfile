FROM nginx

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

COPY crontab /
COPY ./cors /etc/nginx/

# Add runit services
COPY sv /etc/service 
ARG BUILD_INFO
LABEL BUILD_INFO=$BUILD_INFO

COPY start.sh /
COPY nginx.conf /etc/nginx/
COPY nginx-secure.conf /etc/nginx/

COPY dhparams.pem /etc/ssl/private/
CMD /start.sh
