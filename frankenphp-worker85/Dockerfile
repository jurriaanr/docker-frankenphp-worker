FROM dunglas/frankenphp:1-php8.5
LABEL maintainer="jurriaanr@gmail.com"

ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm \
    PHP_MAX_EXECUTION_TIME=120 \
    PHP_MEMORY_LIMIT=256M \
    PHP_UPLOAD_LIMIT=8M \
    PHP_OPCACHE_LIMIT=64M \
    PHP_OPCACHE_REVALIDATE=2 \
    PHP_SESSION_SAVE_HANDLER=files \
    PHP_SESSION_SAVE_PATH="" \
    PHP_SMTP_HOST=localhost \
    PHP_MAX_INPUT_VARS=1000 \
    FRANKENPHP_DOCUMENTROOT=/app/public \
    FRANKENPHP_WORKER_FILE=/app/public/index.php \
    FRANKENPHP_NUM_WORKERS=2 \
    SERVER_NAME=":80"

# Locales
RUN echo "LANG=\"en_US.UTF-8\"" > /etc/default/locale && \
    echo "de_DE.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "es_ES.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "en_GB.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "nl_NL.UTF-8 UTF-8" >> /etc/locale.gen

# Runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    cron nano git locales zip openssh-client ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Regenerate locales after apt-get install
RUN locale-gen

# PHP extensions (matching jurriaanr/php)
RUN install-php-extensions \
    bcmath curl exif gd intl imagick ldap mbstring memcached \
    mysqli opcache pdo_mysql pdo_pgsql simplexml soap sockets \
    redis xmlrpc xsl zip

# Composer
RUN curl https://getcomposer.org/composer-2.phar --output /usr/local/bin/composer && \
    chmod +x /usr/local/bin/composer

# PHP config
COPY settings/production.ini /usr/local/etc/php/conf.d/00-production.ini
COPY settings/docker-settings.ini /usr/local/etc/php/conf.d/01-docker-settings.ini

# Caddyfile (see below)
COPY Caddyfile /etc/frankenphp/Caddyfile

# Timezone
RUN echo "Europe/Amsterdam" > /etc/timezone && \
    dpkg-reconfigure --frontend noninteractive tzdata

# Git defaults
RUN git config --global user.email "you@example.com" && \
    git config --global user.name "Your Name"

# Cron log target
RUN touch /var/log/cron.log

# Healthcheck: hit the Caddy server's root
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
    CMD curl -f http://localhost${SERVER_NAME#":"}/ || exit 1

RUN mkdir -p /app
VOLUME /app
WORKDIR /app