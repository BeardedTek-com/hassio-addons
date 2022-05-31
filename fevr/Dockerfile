# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}" && \
    apk --no-cache add py3-pip py3-pillow py3-paho-mqtt py3-requests py3-dotenv git && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    git clone https://github.com/beardedtek-com/fevr && \
    cp -r /fevr/docker/rootfs/etc/services.d/* /etc/services.d/
WORKDIR /fevr
