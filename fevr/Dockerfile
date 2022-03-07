# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}"

# Add apache2, python3 git
RUN apk --no-cache add apache2 libxml2-dev apache2-utils py3-pip py3-pillow git
RUN ln -s /usr/bin/python3 /usr/bin/python

# Install python dependencies
RUN pip install pytz && \
    pip install python-dateutil && \
    pip install requests && \
    pip install sqlite_web

# get rid of symlink /var/www/logs
RUN rm -f /var/www/logs

# Copy root filesystem
COPY rootfs /

# get latest version of fEVR
RUN git clone https://github.com/beardedtek-com/fevr && mv fevr/app/* /var/www/
RUN mkdir /var/www/logs && touch /var/www/logs/debug.log
RUN mv /var/www/db/fEVR.blank.sqlite /var/www/db/fEVR.sqlite
RUN mv /var/www/config/config-example.json /var/www/config/config.json

# Setup CGI Environment
RUN bash /enable_cgi.sh

# Set permissions on /var/www
RUN chown -R 100:101 /var/www
RUN chmod -R 0770 /var/www


