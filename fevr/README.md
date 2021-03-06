[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports armhf Architecture][armhf-shield]
![Supports armv7 Architecture][armv7-shield]
![Supports i386 Architecture][i386-shield]
<p align="right" style="vertical-align:middle;"><a target="_blank" href="https://www.facebook.com/sharer/sharer.php?u=http%3A%2F%2Ffevr.video"><img src='https://fevr.video/img/share-fb.svg' style="height: 2em;"></a><a target="_blank" href="https://twitter.com/intent/tweet?url=http%3A%2F%2Ffevr.video&text=AI%20Object%20Detection%20with%20fEVR%20-%20frigate%20Event%20Video%20Recorder"><img src='https://fevr.video/img/share-twitter.svg' style="height: 2em;"></a><a target="_blank" href="http://pinterest.com/pin/create/button/?url=http%3A%2F%2Ffevr.video&media=&description=AI%20Object%20Detection%20with%20fEVR%20-%20frigate%20Event%20Video%20Recorder"><img src='https://fevr.video/img/share-pin.svg' style="height: 2em;"></a><a target="_blank" href="https://reddit.com/submit?url=https://fevr.video&title=AI%20Object%20Detection%20with%20fEVR%20-%20frigate%20Event%20Video%20Recorder"><img src='https://fevr.video/img/share-reddit.svg' style="height: 2em;"></a><a target="_blank" href="http://www.linkedin.com/shareArticle?mini=true&url=http%3A%2F%2Ffevr.video&title=AI%20Object%20Detection%20with%20fEVR%20-%20frigate%20Event%20Video%20Recorder"><img src='https://fevr.video/img/share-linkedin.svg' style="margin-right: 2em;height: 2em;"></a><a href="https://www.paypal.com/donate/?hosted_button_id=ZAHLQF24WAKES"><img src='https://fevr.video/img/paypal-donate.svg' style="height: 2em;"></a><a href="https://github.com/sponsors/BeardedTek-com"><img src='https://fevr.video/img/github-sponsor.svg' style="height: 2em;"></a><a href="https://tallyco.in/s/waqwip/"><img src='https://fevr.video/img/tallycoin-donate.png' style="height: 2em;"></a></p>

# fEVR - frigate Event Video Recorder
fEVR works along side of [frigate](https://frigate.video) and [home assistant](https://www.home-assistant.io/) to collect video and snapshots of objects detected using your existing camera systems.
![fEVR v0.6 Screenshot](https://user-images.githubusercontent.com/93575915/165704583-fec8e202-88b8-4ca2-9ff2-345c04da3722.png)

## Requirements:
- Frigate fully setup and working
- MQTT server (if you have frigate running, you have this)

## Optional but nice:
- Tailscale Account (for secure remote access)
  - A docker-compose file will be included below for users not wanting tailscale
- Home Assistant (for notifications)
- Proxy Server
  - Python's built in flask server can be flaky.  Sometimes images won't load properly, but I've found with ngnix in front of it these problems do not exist.  If someone could explain why, that would be nice...
    - NOTE: I do plan on transitioning to possibly gunicorn or another wsgi server, but for now, I'm sticking with this.


## .env Setup
Copy template.env to .env and adjust as necessary:
NOTE: The IP addresses in the .env file are for internal bridge networking and SHOULD NOT be on the same subnet as your home network.
The default values should serve you well.
```
#fEVR Setup
#####################################################################
# Changes the port fEVR runs on DEFAULT: 5090
FEVR_PORT=5090
# Uncomment FLASK_ENV=development to put fEVR into Debug Mode
#FLASK_ENV=development
#####################################################################

#Tailscale
#####################################################################
# Obtain Auth Key from https://login.tailscale.com/admin/authkeys
AUTH_KEY=
TAILSCALE_IP=192.168.101.253
#####################################################################

# Bridge Network Variables
BRIDGE_SUBNET=192.168.101.0/24
BRIDGE_GATEWAY=192.168.101.254

# fEVR container Network Address
FEVR_IP=192.168.101.1

# mqtt_client container Network Address
MQTT_CLIENT_IP=192.168.101.2
```

## Docker Compose (Tailscale):
Example docker-compose.yml:
```
version: '2.4'
services:
  tailscale:
    image: tailscale/tailscale
    container_name: fevr_tailscale
    restart: unless-stopped
    privileged: true
    volumes:
      - ./tailscale/varlib:/var/lib
      - /dev/net/tun:/dev/net/tun
      - ./run_tailscale.sh:/run.sh
    cap_add:
      - net_admin
      - sys_module
    environment:
      AUTH_KEY: ${AUTH_KEY}
      BRIDGE_SUBNET: ${BRIDGE_SUBNET:-192.168.101.0/24}
    command: /run.sh
    networks:
      fevrnet:
        ipv4_address: ${TAILSCALE_IP:-192.168.101.253}
    
  fevr_flask:
    image: ghcr.io/beardedtek-com/fevr-flask:main
    container_name: fevr_flask
    restart: unless-stopped
    privileged: true
    networks:
      fevrnet:
        ipv4_address: ${FEVR_IP:-192.168.101.1}
    volumes:
      - ./:/fevr
    environment:
      FLASK_ENV: ${FLASK_ENV:-}
      FEVR_PORT: ${FEVR_PORT:-5090}

networks:
  fevrnet:
    driver: bridge
    ipam:
      config:
        - subnet: ${BRIDGE_SUBNET:-192.168.101.0/24}
          gateway: ${BRIDGE_GATEWAY:-192.168.101.254}
```

## Docker Compose (No Tailscale):
Example docker-compose.yml:
```
version: '2.4'
services:
  fevr_flask:
    image: ghcr.io/beardedtek-com/fevr-flask:main
    container_name: fevr_flask
    restart: unless-stopped
    privileged: true
    ports:
      - 5090:${FEVR_PORT:-5090}
    volumes:
      - ./:/fevr
    environment:
      FLASK_ENV: ${FLASK_ENV:-}
      FEVR_PORT: ${FEVR_PORT:-5090}
```

Bring the system up:
```
docker-compose up -d
```

View the logs:
```
docker-compose logs -f fevr_flask fevr_mqtt
```
You will notice right away that fevr_mqtt will be saying:
```
bash: /fevr/run_mqtt_client.sh: No such file or directory
```
This is 100% NORMAL BEHAVIOR.  You must go through the Web UI setup to enable the mqtt_client.  Until then, It's just a pretty interface that does nothing.

# Setup
Procedure:

- Visit http(s)://<fevr_url>/setup
- Create admin account
- Login to new admin account
- Visit http(s)://fevr_url>/setup AGAIN.
- Add all of your cameras.
  - It asks for both HLS and RTSP feeds.  Technically you don't need to enter anything but the camera name, but in a future release live view and frigate config will be enabled and will require these values
  - Click Next
- Configure Frigate
  - make one entry called 'frigate' (without the quotes) with your internally accessible frigate URL
  - make another entry called 'external' (without the quotes) with your externally accessible frigate URL
    - This is 100% Optional.  It does, however, enable live view outside your network.
    - If you don't have an externally accessible frigate URL, you can skip this step.
  - Click Next
- Configure MQTT Client
  - You need to generate an API Auth Key to configure this step.
    - This can be done on your profile page.  Click the Bearded Tek logo to drop down the menu and click on profile.
    - Scroll down and fill in the fields:
      - Name: Enter a name to remember this is for the mqtt client (mqtt_client)
      - ipv4 Address: OPTIONAL
      - Limit: Enter 0
        - If anything above 0 is entered, it is a limited use key.  It can only be used that many times to authenticate with the system.  Once it has been used x amount of times, it will be disabled.
  - Click Add and then Next
- Other is not populated yet, There are future plans for this page, just click Next again and you'll be brought to the main interface.


# Home Assistant Notifications
As of right now it's a bit complicated. For each notification type you want for each camera, a helper entity must be added.
For example, I have notifications setup for my driveway camera for person, animal, and vehicle, so I have the following helpers:
- fevrDrivewayAnimal
- fevrDrivewayCar
- fevrDrivewayPerson

The automation uses this helper entity for 2 purposes.
- As a motion sensor
  - If the helper is on, that means a notification is active
- As a pause for notifications
  - If the helper is on, it does not allow further notifications until it is turned off.
  - In the automation, this time can be adjusted to your liking

Here is the automation I'm currently using:
As displayed when:
- editing the automation via the UI
- click on overflow menu (3 dots)
- click Edit in YAML

NOTES:
- ***CAMERA*** is the camera name
- ***HELPER ENTITY*** is the entity you created for this notification
- ***YOUR FEVR URL*** is the url to your fevr instance

```
alias: fEVR <<CAMERA>> Person Alert
description: fEVR Object Detection Alerts
trigger:
  - platform: mqtt
    topic: frigate/events
condition:
  - condition: template
    value_template: '{{ trigger.payload_json["type"] == "end" }}'
  - condition: template
    value_template: |-
      {{
      trigger.payload_json["after"]["label"] == "person"
      }}
  - condition: template
    value_template: |-
      {{
      trigger.payload_json["after"]["top_score"] > 0.76
      }}
  - condition: template
    value_template: |-
      {{
      trigger.payload_json["after"]["camera"] == "<<CAMERA>>"
      }}
action:
  - choose:
      - conditions:
          - condition: state
            state: 'off'
            entity_id: input_boolean.fevrbackyardperson
        sequence:
          - service: notify.mobile_app_sg20plus
            data:
              message: '{{ trigger.payload_json["after"]["label"] | title }} Detected'
              data:
                notification_icon: mdi:cctv
                ttl: 0
                priority: high
                sticky: true
                actions:
                  - action: URI
                    title: Clip
                    uri: >-
                      <<YOUR FEVR URL>>/event/{{trigger.payload_json['after']['id']}}/snap
                  - action: URI
                    title: Snapshot
                    uri: >-
                      <<YOUR FEVR URL>>/event/{{trigger.payload_json['after']['id']}}/snap
                image: >-
                  <<YOUR FEVR URL>>/static/events/{{trigger.payload_json['after']['id']}}/snapshot.jpg
                tag: '{{trigger.payload_json["after"]["id"]}}'
                alert_once: true
          - service: input_boolean.turn_on
            data: {}
            target:
              entity_id: input_boolean.<<HELPER ENTITY>>
          - delay:
              hours: 0
              minutes: 1
              seconds: 0
              milliseconds: 0
          - service: input_boolean.turn_off
            data: {}
            target:
              entity_id: input_boolean.<<HELPER ENTITY>>
    default: []
mode: single
```