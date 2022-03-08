# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}"

RUN apk --no-cache add apache2 libxml2-dev apache2-utils py3-pip py3-pillow git bash nano nfs-utils
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
RUN git clone https://github.com/beardedtek-com/chores && mv chores/app/* /var/www/

# Setup CGI Environment & Set Permissions
RUN bash /enable_cgi.sh \
 && chown -R 100:101 /var/www \
 && chmod -R 0770 /var/www

 # Cleanup
 RUN rm -rf /chores