# casa

Home automation for the house :)

## Installing Raspbian

Use [etcher.io](https://etcher.io) to flash raspbian lite to SD card.
Then add an empty 'ssh' file to the SD card to enable SSH on boot on Raspbian.

## Running ansible

### Prerequisites

Install dependencies (best globally):
```bash
pip install -r requirements.txt
```

### DEV
Development should be done locally in Vagrant.
```bash
vagrant up
ansible-playbook home.yml -i inventory/vagrant 
# roofcam only
ansible-playbook home.yml -i inventory/vagrant --tags roofcam
# using production data
ansible-playbook home.yml -i  ~/repos/casa-data/inventory/vagrant
```
### PROD

This won't be useful to anyone else but me :-)

```bash
ansible-playbook --ask-pass --ask-sudo-pass home.yml -i ~/repos/casa-data/inventory/mbp-server
# roofcam only
ansible-playbook --ask-pass --ask-sudo-pass home.yml -i ~/repos/casa-data/inventory/mbp-server --tags roofcam
```

## Convenient commands

```bash
# Check logs
sudo journalctl -fu homeassistant@pi.service
# Restart server
sudo systemctl restart homeassistant@pi.service
# Get monit status
sudo monit status
sudo monit summary
```

Getting status from monit using API
```bash
# Status
curl -u "$CASA_MONIT_USERNAME:$CASA_MONIT_PASSWORD" http://localhost:2812/_status
curl -u "$CASA_MONIT_USERNAME:$CASA_MONIT_PASSWORD" http://localhost:2812/_status?format=xml
# Summary
curl -u "$CASA_MONIT_USERNAME:$CASA_MONIT_PASSWORD" http://localhost:2812/_summary
```

### python-nest
```bash
source /opt/homeassistant/.venv/bin/activate
export NEST_CLIENT_ID=$(grep "nest:" -A 2 /opt/homeassistant/configuration.yaml  | awk '/client_id/{print $2}')
export NEST_CLIENT_SECRET=$(grep "nest:" -A 2 /opt/homeassistant/configuration.yaml  | awk '/client_secret/{print $2}')
nest --client-id $NEST_CLIENT_ID --client-secret $NEST_CLIENT_SECRET --token-cache /opt/homeassistant/nest.conf --index 0 camera-show
```

## Upgrading homeassistant

When upgrading homeassistant, you need to manually start it because on first run hass will install a bunch of additional packages.
Especially installing pyatv tends to take 10+ mins.

Try running 
```bash
ps -ef | grep pip
```

## Other convenient info

# Virtualbox performance issues

https://joeshaw.org/terrible-vagrant-virtualbox-performance-on-mac-os-x/

```
export CASA_VM=$(VBoxManage list vms | grep casa_home | cut -f1 -d" " | tr -d "\"")
VBoxManage storagectl $CASA_VM --name "IDE" --hostiocache on
```

### Spotify/Sonos



Playing music on Sonos:
http://hasshostname:8123/dev-service
Domain: media_player
Service: select_source
Service Data: {"entity_id": "media_player.tv_room", "source": "'t Koffiehuis"}

Apparently, it's not that easy to play a playlist during a scene:
Some pointers on how to play spotify playlist here: https://community.home-assistant.io/t/sonos-automation-scenes-and-specific-playlists/3002/16

Also, sonos  has a sonos_join service that should allow speakers to be paired up, need to look into that:
https://home-assistant.io/components/media_player.sonos/


### Hass APIs
```bash
export HASS_URL="http://0.0.0.0:8123"; export HASS_PASSWORD="$(awk '/api_password: /{print $2}'  /opt/homeassistant/configuration.yaml)"
# Getting  entity picture
curl -s -H "x-ha-access: $HASS_PASSWORD" -H "Content-Type: application/json" $HASS_URL/api/states/camera.hallway | jq -r ".attributes.entity_picture"
```

## TODO
- Groups: https://home-assistant.io/components/group/
- Spotify support:  https://home-assistant.io/components/media_player.spotify/
- Samsung smartTV support: https://home-assistant.io/components/media_player.samsungtv/
- Location tracking via OwnTracks
   -> This requires a DDNS, which means exposing everything to the web, which has security implications.
- https://home-assistant.io/components/media_player.cast/
- Let's encrypt support
- Calling "homeassistant/reload_core_config" one config reload instead of doing a HA restart
- Force state update on Nest after changing state through python-nest command
- Metrics dashboard
- Improve roofcam accuracy
- sonos-node-http-api: SSL & auth

### Sensor ideas
- Roofcam sensor (refactor monit-hass-sensors to be more generic)
- Laptop sensors: e.g. track that Joris started working when laptop is on and connected to monitor
- Car at home detect sensor based on image recognition
- Door/window sensors
- Upload/backups success sensor (based on Monit/other check)
- Custom nest sensors based on python-nest, because current nest sensors in Hass aren't very good


### HADashboard
- HADashboard: Volume control
- HADashboard: Custom Nest Cam controls (incl enable-disable support + link to livestream)
- HADashboard: Custom weather widget
