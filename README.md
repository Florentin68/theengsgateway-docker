# Theengs Gateway - Docker Version

|        Arch       |               Docker Image               |
| ----------------- | ---------------------------------------- |
| ![aarch64-shield] | ![aarch64-docker]  ![aarch64-ghcr]       |
|  ![amd64-shield]  |  ![amd64-docker]    ![amd64-ghcr]        |
|  ![armv6-shield]  |  ![armv6-docker]    ![armv6-ghcr]        |
|  ![armv7-shield]  |  ![armv7-docker]    ![armv7-ghcr]        |
|  ![i386-shield]   |  ![i386-docker]     ![i386-ghcr]         |

## So why the Docker Version?
It happened just so that in my home, somehow Raspberry Pi 3 (where my Home Assistance Instance is hosted) Integrated Bluetooth adapter died and all my BLE sensors are now unavailable. Theengs Gateway seemed like a perfect thing to install on my Raspberry Pi 4 which is used as a media server.
However, this is where the problem happened.
I have LibreElec installed on RPi 4, therefore i could not use any of methods above to install Theengs Gateway, because:
- pip was not installed, installing it on LibreElec would not keep it there since it uses overlay image. On next boot it would be overwritten
- snap is not part of LibreElec and could not be installed
- I want to keep RPi 4 as a media server, so overwriting it with Home Assistant installation is not an option

## Birth of Docker Version
LibreElec natively comes with Docker. So building a docker image and running it inside LibreElec is a great way to run Theengs Gateway there.
Also, this may help other users who prefer to use Docker to install Theengs Gateway.

## Usage
Here are two ways to run Theengs Gateway in Docker.

### docker-compose.yml
You can write a docker-compose.yml and use docker compose plugin to run. This is a preferred method as it's more humanly readable. Following docker-compose.yml can be used:
```
# docker-compose.yml
version: '3.1'

services:
  theengsgateway:
    image: ghcr.io/florentin68/theengsgateway-docker:latest
    network_mode: host
    environment:
      MQTT_HOST: <host_ip>
      MQTT_USERNAME: <username>
      MQTT_PASSWORD: <password>
      MQTT_PUB_TOPIC: home/TheengsGateway/BTtoMQTT
      MQTT_SUB_TOPIC: home/+/BTtoMQTT/undecoded
      MQTT_PRE_TOPIC: home/presence/TheengsGateway
      PRESENCE: false
      PUBLISH_ALL: true
      PUBLISH_ADVDATA: false
      TIME_BETWEEN: 60
      SCAN_TIME: 60
      LOG_LEVEL: DEBUG
      DISCOVERY: true
      HASS_DISCOVERY: true
      DISCOVERY_TOPIC: homeassistant
      DISCOVERY_DEVICE_NAME: TheengsGateway
      DISCOVERY_FILTER: "[IBEACON,GAEN,MS-CDP]"
      SCANNING_MODE: active
      ADAPTER: hci0
      TIME_SYNC: "[]"
      TIME_FORMAT: 0
      BINDKEYS: "[]"
      ENABLE_TLS: false
      ENABLE_WEBSOCKET: false
      IDENTITIES: "[]"

    volumes:
      - /var/run/dbus:/var/run/dbus
```

*MQTT_HOST* is mandatory field, ofcourse, unless you provide a config file as a volume.
*MQTT_USERNAME* and *MQTT_PASSWORD* you use if you require authentication with MQTT Host.

Other variables are not mandatory and you can even leave them out, default values will be used, so you can shorten your docker-compose.yml to this one (if username/password is not used):

```
# docker-compose.yml
version: '3.1'

services:
  theengsgateway:
    image: ghcr.io/florentin68/theengsgateway-docker:latest
    network_mode: host
    environment:
      MQTT_HOST: <host_ip>
    volumes:
      - /var/run/dbus:/var/run/dbus
```

Another way to go is to provide a config file as a volume. No environment variable is then taken into account:

```
# docker-compose.yml
version: '3.1'

services:
  theengsgateway:
    image: ghcr.io/florentin68/theengsgateway-docker:latest
    network_mode: host
    volumes:
      - /var/run/dbus:/var/run/dbus
      - theengsgw.conf:/root/theengsgw.conf
```

The configfile is a json file.

After you have the file created, run `docker-compose up` or `docker compose up`. It will start the process where you can monitor the progress.
This is not ideal way to run but great for first run and for testing to see what's going on and for checking messages.
If you wish to run in background, append --detach after "up" command.

To bring it down, use `docker-compose down; docker-compose rm -f` or `docker compose down; docker compose rm -f`.

All of these docker compose commands need to be ran from same directory as where docker-compose.yml file is created.

### Using docker run command
You can use following command to run if you don't have docker compose plugin installed:
```
docker run --rm \
    --network host \
    --privileged \
    --restart always \
    -e MQTT_HOST=<host_ip> \
    -e MQTT_USERNAME=<username> \
    -e MQTT_PASSWORD=<password> \
    -e MQTT_PUB_TOPIC=home/TheengsGateway/BTtoMQTT \
    -e MQTT_SUB_TOPIC=home/+/BTtoMQTT/undecoded \
    -e MQTT_PRE_TOPIC=home/presence/TheengsGateway \
    -e PRESENCE=false \
    -e GENERAL_PRESENCE=false \
    -e PUBLISH_ALL=true \
    -e PUBLISH_ADVDATA=false \
    -e TIME_BETWEEN=60 \
    -e TRACKER_TIMEOUT=120 \
    -e SCAN_TIME=60 \
    -e LOG_LEVEL=DEBUG \
    -e DISCOVERY=true \
    -e HASS_DISCOVERY=true \
    -e DISCOVERY_TOPIC=homeassistant \
    -e DISCOVERY_DEVICE_NAME=TheengsGateway \
    -e DISCOVERY_FILTER="[IBEACON]" \
    -e SCANNING_MODE=active
    -e ADAPTER=hci0 \
    -e TIME_SYNC="[]" \
    -e TIME_FORMAT=0 \
    -e BLE=true \
    -e IDENTITIES="{\"00:11:22:33:44:55:66\":\"0dc540f3025b474b9ef1085e051b1add\",\"AA:BB:CC:DD:EE:FF\":\"6385424e1b0341109942ad2a6bb42e58\"}" \
    -e BINDKEYS="{\"00:11:22:33:44:55:66\":\"0dc540f3025b474b9ef1085e051b1add\",\"AA:BB:CC:DD:EE:FF\":\"6385424e1b0341109942ad2a6bb42e58\"}" \
    -v /var/run/dbus:/var/run/dbus \
    --name theengsgateway \
    ghcr.io/florentin68/theengsgateway-docker:latest
```

And again, shorten it like this if you wish to use recommended defaults (without user/pass):

```
docker run --rm \
    --network host \
    --privileged \
    -e MQTT_HOST=<host_ip> \
    -v /var/run/dbus:/var/run/dbus \
    --name gateway \
    ghcr.io/florentin68/theengsgateway-docker:latest
```

Or with config file mounted as a volume:

```
docker run --rm \
    --network host \
    --privileged \
    -v /var/run/dbus:/var/run/dbus \
    -v theengsgw.conf:/root/theengsgw.conf \
    --name gateway \
    ghcr.io/florentin68/theengsgateway-docker:latest
```

These commands will run the container in foreground, it's not ideal, but if you wish to see what's happening it's great - for testing.
If you want to run it in background, add "--detach" between *docker run* and *--rm*

## Configuration

Once again, MQTT_HOST is the *ONLY* required option if no config file mounted as a volume, all others can be ommited and defaults will be used.
If used without MQTT_USERNAME and MQTT_PASSWORD, it will try to login annonymously, otherwise, please provide MQTT_USERNAME and MQTT_PASSWORD too.

As for other configuration options for Theengs Gateway, please review any and all on [Details options](https://gateway.theengs.io/use/use.html#details-options) page on Theengs Gateway official site.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armv6-shield]: https://img.shields.io/badge/armv6-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
[current-version]: https://img.shields.io/badge/Current%20Version-1.5.0.1-blue
[aarch64-docker]: https://img.shields.io/badge/DockerHub-aarch64:latest-blue?logo=docker&logoColor=#2496ED&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[amd64-docker]: https://img.shields.io/badge/DockerHub-amd64:latest-blue?logo=docker&logoColor=#2496ED&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[armv6-docker]: https://img.shields.io/badge/DockerHub-armv6:latest-blue?logo=docker&logoColor=#2496ED&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[armv7-docker]: https://img.shields.io/badge/DockerHub-armv7:latest-blue?logo=docker&logoColor=#2496ED&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[i386-docker]: https://img.shields.io/badge/DockerHub-i386:latest-blue?logo=docker&logoColor=#2496ED&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[aarch64-ghcr]: https://img.shields.io/badge/Github_Container_Registry-aarch64:latest-blue?logo=github&logoColor=#181717&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[amd64-ghcr]: https://img.shields.io/badge/Github_Container_Registry-amd64:latest-blue?logo=github&logoColor=#181717&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[armv6-ghcr]: https://img.shields.io/badge/Github_Container_Registry-armv6:latest-blue?logo=github&logoColor=#181717&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[armv7-ghcr]: https://img.shields.io/badge/Github_Container_Registry-armv7:latest-blue?logo=github&logoColor=#181717&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags
[i386-ghcr]: https://img.shields.io/badge/Github_Container_Registry-i386:latest-blue?logo=github&logoColor=#181717&link=https://hub.docker.com/r/fmunsch/theengsgateway-docker/tags