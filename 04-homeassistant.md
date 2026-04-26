# Home Assistant setup

Ecosystem around HA encompasses 5 containers in total:
1. Home Assistant
2. MQTT broker - for Z2M and Valetudo
3. Zigbee2mqtt - for zigbee
4. Piper - for TTS
5. Media player daemon - to play chimes & TTS

## Prepare

1. Create user: `sudo useradd -m -s /bin/bash hass`
2. Enable linger to keep services alive: `sudo loginctl enable-linger hass`
3. Add to groups: `sudo usermod -aG dialout,audio hass`
4. To allow mapping to the groups, add the following to `/etc/subgid`
```
hass:20:1
hass:29:1
```
5. Setup audio: `sudo apt install alsa-utils sox libsox-fmt-mp3` -> `alsamixer` to unmute the speaker, `play ...` to test
6. Prepare directory for containter files: (under hass user) `mkdir -p .config/containers/systemd`

## Quadlets

All in `.config/containers/systemd` dir. Be sure to also create all folders referenced by `Volumes=...`.

### `hass.pod`
```
[Unit]
Description=Home assistant pod
After=network-online.target
Wants=network-online.target

[Pod]
PodName=hass

PublishPort=1883:1883
PublishPort=8080:8080
PublishPort=10200:10200
#PublishPort=6600:6600

[Install]
WantedBy=default.target
```

### `hass.network`
```
[Unit]
Description=HA Network

[Network]
NetworkName=hanet
Driver=bridge
Subnet=10.0.1.0/24
Gateway=10.0.1.1

[Install]
WantedBy=default.target
```

### `homeassistant.container`

```
[Unit]
Description=Home Assistant
After=network-online.target hass-pod.service
Wants=network-online.target

[Container]
ContainerName=homeassistant
Image=ghcr.io/home-assistant/home-assistant:stable

Pod=hass.pod

Volume=%h/homeassistant/config:/config
Volume=%h/homeassistant/media:/media
Volume=/etc/localtime:/etc/localtime:ro
Volume=/run/dbus:/run/dbus:ro

AddDevice=/dev/snd:/dev/snd  # not sure if needed

Network=host

SecurityLabelDisable=true  # HA guide suggests privilidged mode, but this seems to work as well
AddCapability=NET_ADMIN
AddCapability=SYS_ADMIN

[Service]
Restart=always
```

### `mpd.container`

```
[Unit]
Description=MPD Music Player Daemon
After=network-online.target hass-pod.service
Wants=network-online.target

[Container]
ContainerName=mpd
Image=docker.io/vimagick/mpd:latest

Pod=hass.pod

GroupAdd=keep-groups
UserNS=keep-id
AddDevice=/dev/snd

Volume=%h/mpd/mpd.conf:/etc/mpd.conf:ro
Volume=%h/mpd/mpd-data:/var/lib/mpd

Network=host  # I don't like this, but mpd had trouble connecting back to HA (hairpinning issues)

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=default.target
```

### `mqtt.container`

```
[Unit]
Description=Eclipse Mosquitto MQTT Broker
After=network-online.target hass-pod.service
Wants=network-online.target

[Container]
ContainerName=mqtt
Image=docker.io/eclipse-mosquitto:latest

Pod=hass.pod

Volume=%h/mqtt/config:/mosquitto/config
Volume=%h/mqtt/data:/mosquitto/data
Volume=%h/mqtt/log:/mosquitto/log

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=default.target
```

### `piper.container`

```
[Unit]
Description=Wyoming Piper TTS
After=network-online.target hass-pod.service
Wants=network-online.target

[Container]
ContainerName=piper
Image=docker.io/rhasspy/wyoming-piper:latest

Pod=hass.pod

Environment=PIPER_VOICE=en_US-hfc_female-medium  # I guess only one of these are needed
Exec=--voice en_US-hfc_female-medium

GroupAdd=keep-groups
UserNS=keep-id

AddDevice=/dev/snd

Volume=%h/piper/data:/data

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=default.target
```

### `zigbee2mqtt.container`

```
[Unit]
Description=Zigbee2MQTT
After=network-online.target hass-pod.service mqtt.service
Wants=network-online.target

[Container]
ContainerName=zigbee2mqtt
Image=docker.io/koenkk/zigbee2mqtt:latest

Pod=hass.pod

Environment=TZ=Europe/Prague

AddDevice=/dev/serial/by-id/usb-ITead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_18878e308639ef11956f57f454516304-if00-port0:/dev/ttyUSB0
#AddDevice=/dev/ttyUSB0:/dev/ttyUSB0
GroupAdd=keep-groups
UserNS=keep-id

Volume=/home/hass/zigbee2mqtt/data:/app/data
Volume=/run/udev:/run/udev:ro

[Service]
Restart=always
TimeoutStartSec=900

[Install]
WantedBy=default.target
```

### Run

The final thing is to start the whole pod: `scstrt hass-pod.service`

Use `jc hass-pod` or `jc <service-name>` to check the logs. This took a lot of tweaking so I'd be surprised if I managed to document everything precisely.
