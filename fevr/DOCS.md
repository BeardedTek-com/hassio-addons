# fEVR
## frigate Event Video Recorder - pronounced [fee-ver]
fEVR independently stores frigate events and provides an easy way to manage event notifications, recordings, and more using the power of Frigate and Home Assistant's APIs.

### HASSIO ADDON NOT READY FOR PRODUCTION USE!!!!!
At the moment, all data is lost when the container restarts.  I want to change this in the future, but concentration is on developing the addon at this point.
If you wish to have persistent data, please look into running fEVR as a standalone container (the preferred method anyways).

[See the Project GitHub Page](https://github.com/beardedtek-com/fEVR) for more details on running fEVR as a standalone docker container.

## Requirements:
- Frigate
- Home Assistant (but well, if you're installing this as an addon, I kinda assume you know this).

## Install
- Add hassio-addons repository
  - [![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FBeardedTek-com%2Fhassio-addons)
  - Install fEVR
  - Before starting, click on the "Configuration" tab
  - Click the 3 dots next to Options
  - Click Edit in YAML
    - debug: 'true' or 'false' (including single quotes)
    - title: 'MUST CONTAIN NO SPACES (Need to find a fix to this...)
    - frigate: 'URL of your frigate instance' 

### Add fEVR Automation to Home Assistant
[v2 of the Home Assistant Automation](https://raw.githubusercontent.com/BeardedTek-com/fEVR/main/docs/automation.yml) adds a "break" using an input boolean helper.
```yaml
- id: '1643335976518'
  alias: fEVR Alerts 2.0
  description: fEVR Object Detection Alerts
  trigger:
  - platform: mqtt
    topic: frigate/events
  condition:
  - condition: template
    value_template: '{{ trigger.payload_json["type"] == "end" }}'
  - condition: template
    value_template: "{{ trigger.payload_json[\"after\"][\"label\"] == \"person\" or\n\
      \   trigger.payload_json[\"after\"][\"label\"] == \"car\" or\n   trigger.payload_json[\"\
      after\"][\"label\"] == \"horse\" \n}}"
  - condition: template
    value_template: '{{ trigger.payload_json["after"]["top_score"] > 0.76 }}'
  action:
  - service: rest_command.fevr
    data:
      debug: 'yes'
      event: '{{trigger.payload_json[''after''][''id'']}}'
      camera: '{{trigger.payload_json[''after''][''camera'']}}'
      type: '{{trigger.payload_json[''after''][''label'']}}'
      clip: '{{trigger.payload_json[''after''][''has_clip'']}}'
      snap: '{{trigger.payload_json[''after''][''has_snapshot'']}}'
      score: '{{trigger.payload_json[''after''][''top_score'']}}'
      updated: '{{as_timestamp(now())}}'
  - choose:
    - conditions:
      - condition: state
        entity_id: input_boolean.notification_pause
        state: 'off'
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
              title: fEVR (int)
              uri: http://[YOUR Home Assistant URL]:8000
            image: /api/frigate/notifications/{{trigger.payload_json['after']['id']}}/snapshot.jpg?bbox=1
            tag: '{{trigger.payload_json["after"]["id"]}}'
            alert_once: true
      - service: input_boolean.turn_on
        target:
          entity_id: input_boolean.notification_pause
      - delay:
          hours: 0
          minutes: 2
          seconds: 0
          milliseconds: 0
      - service: input_boolean.turn_off
        target:
          entity_id: input_boolean.notification_pause
    default: []
  mode: single
```
