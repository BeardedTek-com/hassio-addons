# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN \
    curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}"

# Add apache2, python3 git
RUN apk --no-cache add apache2 libxml2-dev apache2-utils py3-pip py3-pillow git
RUN ln -s /usr/bin/python3 /usr/bin/python

# Add python dependencies
# Install python dependencies
RUN pip install pytz && \
    pip install python-dateutil && \
    pip install requests && \
    pip install sqlite_web

# get rid of symlink /var/www/logs
RUN rm -f /var/www/logs

# get latest version of fEVR
RUN git clone https://github.com/beardedtek-com/fevr && mv fevr/app/* /var/www/

# Copy root filesystem
COPY rootfs /
RUN bash /enable_cgi.sh
RUN chown -R 100:101 /var/www
RUN chmod -R 0770 /var/www


