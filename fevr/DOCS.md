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
