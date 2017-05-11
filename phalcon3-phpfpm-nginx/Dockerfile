# Pull base image
FROM debian:stretch-slim
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -y && \
    apt-get install -yq --no-install-suggests --no-install-recommends apt-utils curl wget apt-transport-https lsb-release ca-certificates debian-archive-keyring && \
    wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list && \
    curl -s https://packagecloud.io/install/repositories/phalcon/stable/script.deb.sh | os=debian dist=stretch bash && \
    apt-get update -y && \
        apt-get install -yq --allow-unauthenticated --no-install-suggests --no-install-recommends \
        php7.1-apcu \
        php-apcu-bc \
        php7.1-common \
        php7.1-curl \
        php7.1-intl \
        php7.1-mongodb \
        php7.1-mbstring \
        php7.1-mcrypt  \
        php7.1-phalcon \
        php7.1-redis \
        php7.1-tidy \
        php7.1-xml \
        php7.1-zip \
        php7.1-fpm \
        nginx-light \
        supervisor \
        libcurl4-openssl-dev \
        libevent-dev \
        libssl-dev \
        libxml2-dev \
        libssh2-1-dev \
        libxml2 \
        libc-dev \
        libmcrypt-dev \
        libtidy-dev \
        libyaml-dev \
        libxpm-dev \
        libvpx-dev && \
    apt-get autoremove -y && \
    apt-get autoclean -y && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /etc/php5 /etc/php/5* /usr/lib/php/20121212 /usr/lib/php/20131226 && \
    phpenmod -s fpm phalcon

WORKDIR /app

ENV PHP_MEMORY_LIMIT=512M \
    MAX_UPLOAD=50M \
    PHP_MAX_FILE_UPLOAD=200 \
    PHP_MAX_POST=100M

# Tune up PHP FPM
RUN sed -i -e "s|;catch_workers_output\s*=.*|catch_workers_output = yes|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e "s|^pm.max_children =.*|pm.max_children = 9|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e "s|^pm.start_servers =.*|pm.start_servers = 3|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e "s|^pm.min_spare_servers =.*|pm.min_spare_servers = 2|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e "s|^pm.max_spare_servers =.*|pm.max_spare_servers = 4|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e "s|;pm.max_requests =.*|pm.max_requests = 200|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e "s|;security.limit_extensions =.*|security.limit_extensions = .php|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    # Tune up PHP
    TIMEZONE=`cat /etc/timezone`; sed -i "s|;date.timezone =.*|date.timezone = ${TIMEZONE}|" /etc/php/7.1/fpm/php.ini && \
    sed -i "s|memory_limit =.*|memory_limit = ${PHP_MEMORY_LIMIT}|" /etc/php/7.1/fpm/php.ini && \
    sed -i "s|upload_max_filesize =.*|upload_max_filesize = ${MAX_UPLOAD}|" /etc/php/7.1/fpm/php.ini && \
    sed -i "s|max_file_uploads =.*|max_file_uploads = ${PHP_MAX_FILE_UPLOAD}|" /etc/php/7.1/fpm/php.ini && \
    sed -i "s|post_max_size =.*|max_file_uploads = ${PHP_MAX_POST}|" /etc/php/7.1/fpm/php.ini && \
    sed -i 's|short_open_tag =.*|short_open_tag = On|' /etc/php/7.1/fpm/php.ini && \
    sed -i 's|error_reporting =.*|error_reporting = -1|' /etc/php/7.1/fpm/php.ini && \
    sed -i 's|display_errors =.*|display_errors = On|' /etc/php/7.1/fpm/php.ini && \
    sed -i 's|display_startup_errors =.*|display_startup_errors = On|' /etc/php/7.1/fpm/php.ini && \
    sed -i -re 's|^(;?)(session.save_path) =.*|\2 = "/tmp"|g' /etc/php/7.1/fpm/php.ini && \
    # Fix ownership of sock file for php-fpm sed -i -e "s|;listen.mode =.*|listen.mode = 0750|g" /etc/php/7.1/fpm/pool.d/www.conf && \
    #sed -i -e 's|^listen =.*|listen = 0.0.0.0:9000|g' /etc/php/7.1/fpm/pool.d/www.conf && \
    sed -i -e 's|^listen.allowed_clients|;listen.allowed_clients|g' /etc/php/7.1/fpm/pool.d/www.conf && \
    chown -R www-data:www-data /etc/php/7.1/fpm/ && \
    usermod -u 1000 www-data

RUN curl -sS https://getcomposer.org/installer | \
    php -- --install-dir=/usr/bin/ --filename=composer

# Apply Nginx configuration
ADD conf/restrict.conf /etc/nginx/global/restrict.conf
ADD conf/nginx.conf /etc/nginx/nginx.conf

RUN chown -R www-data:www-data /var/lib/nginx && \
    rm -rf /etc/nginx/sites-enabled/*

VOLUME ["/app", "/etc/php/7.1/fpm/pool.d/", "/etc/nginx/sites-enabled", "/etc/nginx/conf.d"]

COPY conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Expose ports
EXPOSE 80 443

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]