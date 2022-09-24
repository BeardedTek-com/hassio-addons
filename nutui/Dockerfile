# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}" && \
    \
    apk --no-cache add py3-flask py3-pip py3-wheel py3-requests py3-yaml py3-greenlet git && \
    apk --no-cache add python3-dev build-base linux-headers pcre-dev && \
    git clone https://github.com/beardedtek-com/nutui && \
    cd nutui && \
    python -m pip install -r requirements.txt && \
    apk --no-cache del python3-dev build-base linux-headers pcre-dev
# Copy root filesystem
COPY rootfs /
