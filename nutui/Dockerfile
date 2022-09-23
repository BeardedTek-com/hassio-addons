# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN \
    curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}" && \
    \
    apk --no-cache add py3-flask py3-flask-sqlalchemy py3-requests py3-yaml && \
    python3 -m pip install -r requests.txt && \
    \
    git clone https://github.com/beardedtek-com/nutui
# Copy root filesystem
COPY rootfs /
