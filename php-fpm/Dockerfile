ARG TARGET_PHP_VERSION
FROM php:${TARGET_PHP_VERSION}-fpm

ARG SERVICE_DIR="./php-fpm"
COPY ./.shared/scripts/ /tmp/scripts/
RUN chmod +x -R /tmp/scripts/

# set timezone
ARG TZ=UTC
RUN /tmp/scripts/set_timezone.sh ${TZ}


# add users
ARG APP_USER=www-data
ARG APP_GROUP=www-data
ARG APP_USER_ID=1000
ARG APP_GROUP_ID=1000

RUN /tmp/scripts/create_user.sh ${APP_USER} ${APP_GROUP} ${APP_USER_ID} ${APP_GROUP_ID}

RUN /tmp/scripts/install_php_extensions.sh

RUN /tmp/scripts/install_software.sh

# php config
COPY ./.shared/config/php/conf.d/*  /usr/local/etc/php/conf.d/

# php-fpm pool config
COPY ${SERVICE_DIR}/php-fpm.d/* /usr/local/etc/php-fpm.d
RUN /tmp/scripts/modify_config.sh /usr/local/etc/php-fpm.d/zz-app.conf \
    "__APP_USER" \
    "${APP_USER}" \
 && /tmp/scripts/modify_config.sh /usr/local/etc/php-fpm.d/zz-app.conf \
    "__APP_GROUP" \
    "${APP_GROUP}" \
;

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Make sure the volumes have the correct ownership
ARG APP_CACHE_PATH
RUN mkdir -p "${APP_CACHE_PATH}" && chown ${APP_USER}: "${APP_CACHE_PATH}"
ARG APP_LOG_PATH
RUN mkdir -p "${APP_LOG_PATH}" && chown ${APP_USER}: "${APP_LOG_PATH}"

# workdir
ARG APP_CODE_PATH
WORKDIR "$APP_CODE_PATH"

# entrypoint
RUN mkdir -p /bin/docker-entrypoint/ \
 && cp /tmp/scripts/docker-entrypoint/* /bin/docker-entrypoint/ \
 && chmod +x -R /bin/docker-entrypoint/ \
;

RUN /tmp/scripts/cleanup.sh
ENTRYPOINT ["/bin/docker-entrypoint/resolve-docker-host-ip.sh","php-fpm"]
EXPOSE 9000

