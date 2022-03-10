# fEVR
## frigate Event Video Recorder - pronounced [fee-ver]

[![BeardedTekSponsorCard][sponsor-card]](https://github.com/sponsors/BeardedTek-com)

[sponsor-card]: https://user-images.githubusercontent.com/93575915/157555140-2890590f-62de-4f9a-a384-22a88f120daa.png

### Cloud Instances of fEVR
- I will be offering cloud instances of fEVR starting on March 31st.
- See [My Sponsorship Page](https://github.com/sponsors/BeardedTek-com) for more details.
#### Cloud BETA Testing
- If you would like to beta test this feature, please let me know by submitting an issue.

![fEVR-0 3 3 Main](https://user-images.githubusercontent.com/93575915/155628108-99e39877-b57b-4c13-ba62-fcf1a04941ee.png)

## Known Bugs
- Modals do not work properly in Firefox < 98
  - Either upgrade to the Firefox Beta, wait for Firefox 98 to come out, or use a chromium derivative.
  - For now Chromium derivatives work
- [See Issues for other known bugs.](https://github.com/BeardedTek-com/fEVR/issues)

## Requirements:
- Home Assistant
- Frigate

- Configure Home Assistant Automation to feed the fevr with data:

### Home Assistant Automation v2
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
              title: Event Viewer
              uri: https://hpf.jeandr.net/cgi-bin/hasspyfrigate.py?id={{trigger.payload_json['after']['id']}}&camera={{trigger.payload_json['after']['camera']}}&bbox=true&url=https://hass.jeandr.net/api/frigate/notifications/&time={{trigger.payload_json['after']['start_time']}}&css=../css/hasspyfrigate.css#
            - action: URI
              title: fEVR (int)
              uri: http://192.168.2.240/
            - action: URI
              title: fEVR (ext)
              uri: https://fEVR.jeandr.net/
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

## Pull Requests welcome!
Feel free to fork the project and submit pull requests.

Hope you find this useful.
