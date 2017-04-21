FROM ubuntu:16.04

MAINTAINER toi.xtran <toi.xtran@gmail.com>

RUN DEBIAN_FRONTEND=noninteractive

# Add config file
ADD /root/ /tmp/root/

# Update & upgrade
RUN apt-get update && apt-get -y upgrade

#-------------------------------------------------------------------------------
# Service management by supervisor
#-------------------------------------------------------------------------------
RUN apt-get -y install supervisor
ADD /root/etc/supervisord.conf /etc/supervisord.conf
RUN mkdir -p /etc/supervisord.d/

#-------------------------------------------------------------------------------
# SSH
# Default option `INSTALL_WORKSPACE_SSH=false`
#-------------------------------------------------------------------------------
ADD /root/.ssh/id_rsa /root/.ssh/id_rsa
ADD /root/.ssh/id_rsa.pub /root/.ssh/id_rsa.pub
ARG INSTALL_SSH=false
ENV INSTALL_SSH ${INSTALL_SSH}
RUN if [ ${INSTALL_SSH} = true ]; then \
    apt-get -y install openssh-server && \
    cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys && \
    chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub && \
    chmod 400 /root/.ssh/id_rsa && \
    # SSH login fix. Otherwise user is kicked off after login
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
    # Config start ssh daemon using supervisord
    cp /tmp/root/etc/supervisord.d/sshd.ini /etc/supervisord.d/sshd.ini \
;fi

#-------------------------------------------------------------------------------
# Apache-php
# Default option `INSTALL_APACHE_PHP=false`
#-------------------------------------------------------------------------------
ARG INSTALL_APACHE_PHP=false
ENV INSTALL_APACHE_PHP ${INSTALL_APACHE_PHP}
RUN if [ ${INSTALL_APACHE_PHP} = true ]; then \
    apt-get install -y \
        apache2 \
        php7.0 \
        php7.0-cli \
        php7.0-gd \
        php7.0-json \
        php7.0-mbstring \
        php7.0-xml \
        php7.0-xsl \
        php7.0-zip \
        php7.0-soap \
        php-pear \
        php7.0-mcrypt \
        libapache2-mod-php7.0 \
        php7.0-curl \
        php7.0-mysql \
        php7.0-dev \
        && \
    # Apache2 config
    mkdir -p /var/lock/apache2 /var/run/apache2 && \
    cp /tmp/root/etc/apache2/sites-enabled/000-default.conf /etc/apache2/sites-enabled/000-default.conf && \
    cp /tmp/root/etc/apache2/apache2.conf /etc/apache2/apache2.conf && \
    cp /tmp/root/etc/apache2/envvars /etc/apache2/envvars && \
    # Config start apache daemon using supervisord
    cp /tmp/root/etc/supervisord.d/apache2.ini /etc/supervisord.d/apache2.ini && \

    # PHP config
    # Update php.ini
    cp /tmp/root/etc/php/7.0/apache2/php.ini /etc/php/7.0/apache2/php.ini && \
    # Install composer and add its bin to the PATH.
    curl -s http://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer && \

    # Enable mode
    a2enmod rewrite && \
    phpenmod mcrypt && \

    # Chown own public dir to www-data
    mkdir -p /var/www/html/public/ && \
    cp /tmp/root/var/www/html/public/index.php /var/www/html/public/index.php && \
    chown -R www-data:www-data /var/www/html/public \
;fi

#-------------------------------------------------------------------------------
# xDebug
# Default option `INSTALL_XDEBUG=false`
#-------------------------------------------------------------------------------
ARG INSTALL_XDEBUG=false
ENV INSTALL_XDEBUG ${INSTALL_XDEBUG}
ENV XDEBUGINI_PATH=/etc/php/7.0/mods-available/xdebug.ini
RUN if [ ${INSTALL_XDEBUG} = true ] && [ ${INSTALL_APACHE_PHP} = true ]; then \
    apt-get install -y php-xdebug && \
    echo "xdebug.remote_autostart = 0" >> $XDEBUGINI_PATH && \
    echo "xdebug.remote_enable = 1" >> $XDEBUGINI_PATH && \
    echo "xdebug.remote_connect_back = 0" >> $XDEBUGINI_PATH && \
    echo "xdebug.remote_port=9000" >> $XDEBUGINI_PATH && \
    echo "xdebug.remote_host="`/sbin/ip route|awk '/default/ { print $3 }'` >> $XDEBUGINI_PATH && \
    echo "xdebug.idekey = PHPSTORM" >> $XDEBUGINI_PATH \
;fi

# Clean up
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Start supervisord
CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisord.conf"]
