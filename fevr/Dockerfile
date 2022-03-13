# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}" && \
    apk --no-cache add apache2 libxml2-dev apache2-utils py3-pip py3-pillow git bash nano &&\
    ln -s /usr/bin/python3 /usr/bin/python && \
    pip install pytz && \
    pip install python-dateutil && \
    pip install requests && \
    pip install sqlite_web && \
    pip install css-html-js-minify && \
    rm -rf /var/www/logs && \
    git clone https://github.com/beardedtek-com/fevr && \
    mv fevr/docker/fEVR/rootfs / && \
    chmod +x /opt/fevr/setup/enable_cgi && /opt/fevr/setup/enable_cgi && \
    chown -R 100:1000 /var/www && \
    chmod -R 0770 /var/www && \
    chmod -R 0770 /opt/fevr && \
    chown root:101 /opt/fevr/setup/dbsetperms && \
    chmod u+s /opt/fevr/setup/dbsetperms
