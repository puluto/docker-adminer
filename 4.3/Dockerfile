FROM php:7.0-alpine

STOPSIGNAL SIGINT
ENV GLIBC_VERSION=2.23-r3

RUN     addgroup -S adminer \
&&      adduser -S -G adminer adminer \
&&      mkdir -p /var/www/html \
&&      mkdir -p /opt/oracle \
&&      mkdir -p /var/www/html/plugins-enabled \
&&      chown -R adminer:adminer /var/www/html

RUN     apk add --no-cache libpq unzip wget ca-certificates \
    && wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/sgerrand.rsa.pub \
    && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-${GLIBC_VERSION}.apk \
    && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/${GLIBC_VERSION}/glibc-bin-${GLIBC_VERSION}.apk \
    && apk --no-cache add glibc-${GLIBC_VERSION}.apk \
    && apk --no-cache add glibc-bin-${GLIBC_VERSION}.apk \

WORKDIR /opt/oracle/
RUN     set -x \
    && curl -fsSL http://public-1254427195.coscd.myqcloud.com/instantclient/instantclient-basic-linux.x64-12.2.0.1.0.zip -o instantclient-basic-linux.x64-12.2.0.1.0.zip \
    && curl -fsSL http://public-1254427195.coscd.myqcloud.com/instantclient/instantclient-sdk-linux.x64-12.2.0.1.0.zip -o instantclient-sdk-linux.x64-12.2.0.1.0.zip \
    && unzip instantclient-basic-linux.x64-12.2.0.1.0.zip \
    && unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip \
    && ln -s /opt/oracle/instantclient_12_2/libclntsh.so.12.1 /opt/oracle/instantclient_12_2/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_2/libclntshcore.so.12.1 /opt/oracle/instantclient_12_2/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_2/libocci.so.12.1 /opt/oracle/instantclient_12_2/libocci.so \
    && rm -rf /opt/oracle/*.zip \
    && apk add --no-cache --virtual .build-deps \
        postgresql-dev \
        sqlite-dev \
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_2,12.1 \
    && docker-php-ext-install pdo_mysql pdo_pgsql pdo_sqlite pdo_oci \
    && apk del .build-deps

COPY    *.php /var/www/html/
WORKDIR /var/www/html

ENV     ADMINER_VERSION 4.3.1
ENV     ADMINER_DOWNLOAD_SHA256 71688c5db4d4e504c48c26ec2966ad85e721ef61e0377c5505cfb50b026d5491
ENV     ADMINER_SRC_DOWNLOAD_SHA256 fd016c93bb3bb8e85ed9cf826064584012c4db83f30c4948d701250e29a6d953

RUN     set -x \
&&      curl -fsSL https://github.com/vrana/adminer/releases/download/v$ADMINER_VERSION/adminer-$ADMINER_VERSION-en.php -o adminer.php \
&&      echo "$ADMINER_DOWNLOAD_SHA256  adminer.php" |sha256sum -c - \
&&      curl -fsSL https://github.com/vrana/adminer/archive/v$ADMINER_VERSION.tar.gz -o source.tar.gz \
&&      echo "$ADMINER_SRC_DOWNLOAD_SHA256  source.tar.gz" |sha256sum -c - \
&&      tar xzf source.tar.gz --strip-components=1 "adminer-$ADMINER_VERSION/designs/" "adminer-$ADMINER_VERSION/plugins/" \
&&      rm source.tar.gz

COPY    entrypoint.sh /usr/local/bin/
ENTRYPOINT      [ "entrypoint.sh", "docker-php-entrypoint" ]

USER    adminer
CMD     [ "php", "-S", "[::]:8080", "-t", "/var/www/html" ]

EXPOSE 8080
