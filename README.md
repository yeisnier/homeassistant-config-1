
# Home Assistant Configuration by @rrubin0
This is my [Home Assistant](https://home-assistant.io/) configuration.

I run Home Assistant in a Docker Container on a re-purposed [ASUS Chromebox M004U](https://amzn.to/1rdaB85) running Ubuntu Server 18.04

My primary control method is Z-Wave. I chose Z-Wave for its performance and fit, but also have adjusted per use case. For example, I use Z-Wave for instances where I need to control many lights from a switch at once and fine control over dimming or color LED is less important. My kitchen is a good example of this, as there are 6 can lights here and it would be quite expensive to use Philips Hue.

I use Philips Hue for the following use cases:
- Color LED (although there are other options)
- Fine control over dimming is a requirement
- I have a single or just a couple of bulbs
- The area is not one where it is natural for others to hit the wall switch

The switch problem with Hue can be solved by adding one of their wall switches to control, or using automations to control the lights in anticipation of your household users


**My Docker Container Setup**
* [Home Assistant](https://home-assistant.io)
* [Dasher](https://github.com/Nekmo/amazon-dash)
* [Mosquitto](https://hub.docker.com/_/eclipse-mosquitto/)
* [Portainer](https://portainer.io)
* [Influxdb](https://www.influxdb.com)
* [Grafana](https://grafana.com)
* [HA Dockermon] (https://github.com/philhawthorne/ha-dockermon)
```
version: '2.1'

services:
  mqtt:
    image: eclipse-mosquitto:latest
    container_name: "MQTT"
    restart: always
    network_mode: "host"
    environment:
      - TZ=America/Phoenix
    ports:
      - 1883:1883
      - 9001:9001
    volumes:
      - /home/hauser/mqtt/data:/mosquitto/data
      - /home/hauser/mqtt/config:/mosquitto/config
      - /home/hauser/mqtt/logs:/mosquitto/logs


  grafana:
    image: grafana/grafana:latest
    container_name: "grafana"
    depends_on:
      influxdb:
        condition: service_healthy
    network_mode: "host"
    environment:
      - TZ=America/Phoenix
    ports:
      - 3000:3000
    restart: on-failure
    volumes:
      - /home/hauser/grafana:/var/lib/grafana


  influxdb:
    image: influxdb:latest
    container_name: "influxdb"
    healthcheck: 
      test: ["CMD", "curl", "-sI", "http://127.0.0.1:8086/ping"]
      interval: 30s
      timeout: 1s
      retries: 24
    network_mode: host
    environment:
      - TZ=America/Phoenix
    ports:
      - 8083:8083
      - 8086:8086
    restart: on-failure
    volumes:
      - /home/hauser/influxdb:/var/lib/influxdb


  home-assistant:
    #image: homeassistant/home-assistant:0.78.3
    image: homeassistant/home-assistant:latest
    container_name: "home-assistant"
    restart: always
    depends_on:
      influxdb:
        condition: service_healthy
      mqtt:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "https://my-fqdn.com:8123"]
      interval: 30s
      timeout: 10s
      retries: 6
    network_mode: "host"
    environment:
      - TZ=America/Phoenix
    ports:
      - 8123:8123
    volumes:
      - /home/hauser/hass_data/config:/config
      - /usr/share/ssl:/ssl
    devices:
      - /dev/ttyACM0:/dev/ttyACM0


  dasher:
    command: amazon-dash run --ignore-perms --root-allowed --config /config/amazon-dash.yml
    image: nekmo/amazon-dash:latest
    container_name: "dasher"
    restart: unless-stopped
    network_mode: "host"
    volumes:
      -  /home/hauser/dasher/config/amazon-dash.yml:/config/amazon-dash.yml


  ha-dockermon:
    image: philhawthorne/ha-dockermon
    container_name: "ha-dockermon"
    restart: always
    ports:
      - 8126:8126
    volumes:
      -  /var/run/docker.sock:/var/run/docker.sock
      -  /home/hauser/dockermon:/config


  portainer:
    image: portainer/portainer
    container_name: "portainer"
    restart: on-failure
    network_mode: "host"
    ports:
      - 9000:9000
    volumes:
      -  /var/run/docker.sock:/var/run/docker.sock
      -  /home/hauser/portainer:/data


```

* Other 3rd-party component (non-docker): [Haaska](https://github.com/mike-grant/haaska)


**Devices I Use:**
* [ecobee3 Thermostat](http://amzn.to/2iD0v0z) with [ecobee3 Remote Sensors](http://amzn.to/2iCZFRw)
* ecobee3 Lite Thermostat
* [Amazon Echo Dot Gen 2](http://amzn.to/2hvCexj)
* [Amazon Dash Buttons](http://amzn.to/2i6acYv)
* [Amazon Fire TV Stick](http://amzn.to/2iD9uPx)
* Amazon Dash Wand
* [Philips Hue (Gen 2)](http://amzn.to/2hvyzzK)
* [RGBWW LED Strip WiFi Controller](http://amzn.to/2i6mUqn) with [RGB LED Strips](http://amzn.to/2i68N42)
* Fibaro Z-Wave MultiSensor
* [Aeon Labs Z-Wave Minimote](http://amzn.to/2igetsU)
* Somfy Roller Shades with Z-Wave RTS Bridge
* Pentair 5G LED Pool Light
* Etekcity 433MHz RF Switches
* Broadlink RM2/Pro
* HUSBZB-1 ZigBee/Z-Wave Stick (not in use; using Aeotec Z-stick)
* GE Z-Wave Wall Switches & Dimmers
* Flux LED strips and bulb
* Sonoff WiFi DIY power strips
* NodeMCU DIY Multisensor
* Aeon Labs Z-Wave MultiSensor 6
* Aeon Labs Z-Wave Door/Window Sensor
* Aeon Labs Z-Wave Minimote
* Aeon Labs Z-Wave Siren
* GoControl Z-Wave Siren
* Yale Real Living Z-Wave Touchscreen Lever Lock
* Xiaomi Aqara Hub
* Xiaomi Door Sensor
* Xiaomi Smart Button

(coming soon):
* IKEA Tradfri Bridge
* IKEA Tradfri Motion
* IKEA Tradfri Dimmer
* IKEA Tradfri Remote
* IKEA Tradfri Light