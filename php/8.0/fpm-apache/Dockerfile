####################################################
# Server Side Up -  PHP 8.0 / FPM-Apache image 
#####################################################

FROM serversideup/php:8.0-fpm

LABEL maintainer="Jay Rogers (@jaydrogers)"

ENV MSMTP_RELAY_SERVER_HOSTNAME="mailhog" \
    MSMTP_RELAY_SERVER_PORT="1025" \
    PHP_POOL_NAME="www" \
    PHP_PM_CONTROL=ondemand \
    PHP_PM_MAX_CHILDREN="20" \
    PHP_PM_START_SERVERS="2" \
    PHP_PM_MIN_SPARE_SERVERS="1" \
    PHP_PM_MAX_SPARE_SERVERS="3"\
    APACHE_START_SERVERS="2" \
    APACHE_MIN_SPARE_THREADS="10" \
    APACHE_MAX_SPARE_THREADS="75" \
    APACHE_THREAD_LIMIT="64" \
    APACHE_THREADS_PER_CHILD="25" \
    APACHE_MAX_REQUEST_WORKERS="150" \
    APACHE_MAX_CONNECTIONS_PER_CHILD="0" \
    APACHE_RUN_USER="webuser" \
    APACHE_RUN_GROUP="webgroup" \
    APACHE_DOCUMENT_ROOT="/var/www/html/public/" \
    DEBIAN_FRONTEND=noninteractive
    

# Install Apache (web), MSMTP (email client), and Supervisor (process management)
RUN apt-get update \
    && echo "Install requirements..." \
    && apt-get -y --no-install-recommends install \
        msmtp \
        msmtp-mta \
        apache2 \
    && echo "Enabling Apache mods..." \
    && a2enmod actions autoindex deflate headers http2 proxy proxy_fcgi remoteip rewrite setenvif ssl unique_id \
    && echo "Clean up after ourselves..." \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* /var/www/html/* \
    && echo "Set apache logs to redirect to STDOUT and STDERR and configure Apache..." \
    && ln -sf /dev/stdout /var/log/apache2/access.log \
    && ln -sf /dev/stdout /var/log/apache2/other_vhosts_access.log \
    && ln -sf /dev/stderr /var/log/apache2/error.log \
    && echo "ServerName localhost" >> /etc/apache2/apache2.conf \
    && echo "Set log format for CloudFlare and enable Remote IP..." \
    && sed -i 's/LogFormat "%h %l %u %t \\\"%r\\\" %>s %O \\\"/LogFormat "%a %l %u %t \\\"%r\\\" %>s %O \\\"/' /etc/apache2/apache2.conf \
    && ln -sf /etc/apache2/conf-available/remoteip.conf /etc/apache2/conf-enabled/remoteip.conf \
    && echo "Force Docker ENVs to handle the apache variables for the run user and group..." \
    && echo "export APACHE_RUN_USER=${APACHE_RUN_USER}" >> /etc/apache2/envvars \
    && echo "export APACHE_RUN_GROUP=${APACHE_RUN_GROUP}" >> /etc/apache2/envvars \
    && echo "Fixing permissions issues on the webroot..." \
    && chown -R webuser:webgroup $WEBUSER_HOME \
    && echo "Making sure Apache is stopped and disabled at start up..." \
    && service apache2 stop \
    && update-rc.d apache2 disable

#APACHE: Copy over configurations
COPY etc/apache2/ /etc/apache2/

# Copy over S6 configurations
COPY etc/services.d/ /etc/services.d/
COPY etc/cont-init.d/ /etc/cont-init.d/

#PHP: Copy over PHP overrides
COPY etc/php/fpm/pool.d/ /etc/php/8.0/fpm/pool.d/

EXPOSE 80
EXPOSE 443

################################################################
# Commenting these out for now because Traefik and Swarm
# we not playing well together with this for rolling updates

# HEALTHCHECK --start-period=5s \
#   CMD curl -k -f https://127.0.0.1/ping/ || exit 1


ENTRYPOINT ["/init"]