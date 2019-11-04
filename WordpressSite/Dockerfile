FROM wordpress:5.2.4

# Upgrading os 
# memcached installs the needed Linux packages and 
# then enables the PHP extension.
RUN apt-get update && apt-get install -y \
    nano wget
#   apt-utils libz-dev libmemcached-dev && \
#   pecl install memcached && \
#   docker-php-ext-enable memcached
#   docker-php-ext-install mysqli && \
#   docker-php-ext-enable mysqli

COPY etc/apache2/apache2.conf /etc/apache2/apache2.conf

# COPY .htaccess /var/www/html

# TODO INSERT HERE:
# ADD ./plugin /var/ww/html/wp-content/plugins/plugin
# Plugins - https://github.com/docker-library/wordpress/issues/341
# Themes

# NOT sure about this.
# Add admin super user, create install directory.
# RUN useradd -ms /bin/bash -G sudo admin \
    #&& echo "admin ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/admin \
    #&& chmod 0440 /etc/sudoers \
    #&& mkdir -m 0700 /app \
    #&& chown admin:admin /app

#USER admin
#WORKDIR /app
#EXPOSE 80 443

