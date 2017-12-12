---
title:        Making a Hass.io add-ons for Life360
author:       Eric Ditter
date:         2017-12-12
tags: [Hass.io]
excerpt_separator: <!--more-->
---

When I moved from SmartThings to Home Assistant one of the features I lost was the [Life360](https://www.life360.com/){:target="_blank"} integration so I decided to try and create an addon instead of moving to something like [OwnTracks](http://owntracks.org/){:target="_blank"}. I have been using Life360 for years and I have had a great experience with them in the past so I wanted to at least try to integrate it into Home Assistant.  Since then I have heard that a few of the location tracking platforms in Home Assistant aren't 100% reliable so I am glad I went this route. Again, I have no experience with other location tracking platforms so it is just word of mouth.

<!--more-->

Writing the addon was pretty easy and there are a lot of great examples out there to get started with.  I wasn't familiar with Docker so that took a bit to get started and I am still not completely sure where the line is between Hass.io and Docker. ~~The only thing to note if you make a Hass.io addon is that Home Assistant doesn't always update the config information correctly especially with local development and you just need to trust that it is updated after a re-install~~. (In recent versions this seems to have been fixed!)

Next came the easy part which was actually writing the code. There are a lot of examples of people using the Life360 API out there but as far as I can tell there is no official documentation for external use which seemed weird.  I played around with getting the information from the API and it was pretty straightforward.  Finally, it came to building it and deployment which again, I looked at a lot of examples. Setting up Travis and getting the auto deploy to the Docker Hub took a bit and required some patience but after some trial and error there it seems to be working great.

When the addon starts up it pulls the latest information from the API and resets the webhook URL that is used by Life360 when someone enters or leaves a Place.  It is important to note it will remove previous webhooks you may have had set up so for me my SmartThings suddenly stopped triggering.  **You will also need to port forward your router to the port that the addon uses if you want to use this feature**.  Then every X minutes it will poll the information and push it out via MQTT.

The webhooks part took a bit of trial and error because they only fire when you enter and leave a Place and I didn't want to run up and down the block every time I made a change so it took a few days. GPS spoofing didn't seem to fire them either so I don't really know how those work behind the scenes. After everything was said and done it seemed to be working pretty well and there have already been a few downloads which is great!

Life360 Hassio Addon options

```json
{
    "mqtt_host": "127.0.0.1",
    "mqtt_port": 1883,
    "mqtt_username": "MyUser",
    "mqtt_password": "MyPassword",
    "preface": "life360",
    "cert_file": "fullchain.pem",
    "key_file": "privkey.pem",
    "host_url": "https://mySite.com:8081",
    "life360_user": "MyLife360User",
    "life360_password": "MyLife360Password",
    "refresh_minutes": 5
  }
```

known_devices.yaml

```yaml
eric_phone:
  track: yes
  hide_if_away: no
  name: Eric Location
```

configuration.yaml

```yaml
sensor:
  - platform: mqtt
    name: Eric Battery
    state_topic: life360/location/Eric
    unit_of_measurement: '%'
    retain: true
    value_template: '{{ value_json.battery_level }}'
  - platform: mqtt
    name: Eric Address
    state_topic: life360/location/Eric
    retain: true
    value_template: '{{ value_json.address }}'

# Device tracking / Locations
device_tracker:
  - platform: mqtt_json
    devices:
      my_phone: life360/location/Eric

zone:
  # mirrored from Life360 Places
  - name: Home
    latitude: 30.268556
    longitude: -30.410156
    radius: 152
    icon: mdi:account-multiple
```

automations.yaml

```yaml
# Turn NEST to away if BOTH people are gone
- id: nest_location_away
  alias: Nest Location Away
  trigger:
    - entity_id: device_tracker.eric_phone
      platform: state
    - entity_id: device_tracker.jas_phone
      platform: state
  condition:
    - condition: template
      value_template: "{% if states('device_tracker.eric_phone') == 'Away' and states('device_tracker.jas_phone') == 'Away' %}true{% else %}false{% endif %}"
  action:
    - service: nest.set_mode
      data:
        home_mode: away
# Turn NEST to home if anyone comes home
- id: nest_location_home
  alias: Nest Location Home
  trigger:
    - entity_id: device_tracker.eric_phone, device_tracker.jas_phone
      platform: state
      from: 'not_home'
      to: 'home'
  action:
    - service: nest.set_mode
      data:
        home_mode: home
    - service: notify.html_notification
      data:
        message: Nest changed to home
```

Here is the "I skipped over everything except the code blocks" checklist

1. Open your router port for the add-on (default is 8081)
1. Setup your Home Assistant Zone section to mirror your Life360 Place information so the webhooks trigger at the right time.
1. Add your device to the known_devices.yaml
1. Add the device_tracker information to the configuration.yaml

If you have any requests please reach out and I will do what I can to accommodate them.  From now on I will be adding release information to the readme so check out the [releases](https://github.com/editter/hassio-addons/tree/master/life360#releases){:target="_blank"} section on Github.  If you notice an issue or have a question please create an [issue](https://github.com/editter/hassio-addons/issues){:target="_blank"} on Github
