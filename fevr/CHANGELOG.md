<!-- https://developers.home-assistant.io/docs/add-ons/presentation#keeping-a-changelog -->
## 0.3.17
- modify /etc/services.d
  - modify startup script for apache2

## 0.3.15 Apache Failed to Start
- modify /etc/services.d
  - add basic startup script for apache2

## 0.3.14
- modify /etc/services.d

## 0.3.13 httpd (pid 263) already running flooding the logs
- Seperate out fEVR, apache2, and sqlite_web into their own services

## 0.3.12: APACHE2 and SQLITE_WEB Fail to start
- Update run command

## 0.3.11:  APACHE2 FAILS TO START
- Update writeConfig entry in run as entering a URL for frigate variable caused it to fail.

## 0.3.10: FAILED TO BUILD PROPERLY

- User Configuration via configuration panel update.
  - must edit as yaml
  - writeConfig updated
    - python script that generates config.json file for fEVR