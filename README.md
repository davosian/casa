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


InfluxDB:
```
export INFLUX_USERNAME='';export INFLUX_PASSWORD='';
influx -ssl --unsafeSsl -username "$INFLUX_USERNAME" -password "$INFLUX_PASSWORD"
```

### python-nest
```bash
source /opt/homeassistant/.venv/bin/activate
export NEST_CLIENT_ID=$(grep "nest:" -A 2 /opt/homeassistant/configuration.yaml  | awk '/client_id/{print $2}')
export NEST_CLIENT_SECRET=$(grep "nest:" -A 2 /opt/homeassistant/configuration.yaml  | awk '/client_secret/{print $2}')
nest --client-id $NEST_CLIENT_ID --client-secret $NEST_CLIENT_SECRET --token-cache /opt/homeassistant/nest.conf --index 0 camera-show
```

### grafana

Database location: /var/lib/grafana/grafana.db

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

TODO
```
NAT: Error(22) while setting RCV capacity to (65536)
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

### SCP

scp -P 2222 -i ~/repos/casa/.vagrant/machines/home/virtualbox/private_key ubuntu@127.0.0.1:/home/ubuntu/redis.conf .

### Hass APIs
```bash
export HASS_URL="http://0.0.0.0:8123"; export HASS_PASSWORD="$(awk '/api_password: /{print $2}'  /opt/homeassistant/configuration.yaml)"
# Getting  entity picture
curl -s -H "x-ha-access: $HASS_PASSWORD" -H "Content-Type: application/json" $HASS_URL/api/states/camera.hallway | jq -r ".attributes.entity_picture"
```

### Sensu
Location of binary check scripts:
```
/opt/sensu/embedded/bin/check-ping.rb --help
```

Logs:
/var/log/sensu/*

Important note wrt sensu checks:
From https://sensuapp.org/docs/1.1/guides/getting-started/intro-to-checks.html#create-the-check-definition-for-cpu-metrics

By default, Sensu checks with an exit status code of 0 (for OK) do not create events unless they indicate a change in
state from a non-zero status to a zero status (i.e. resulting in a resolve action). Metric collection checks will output
metric data regardless of the check exit status code, however, they usually exit 0. To ensure events are always created
for a metric collection check, the check type of metric is used.


Things I don't like about sensu:
 - For standard checks, not all checks cause events, only if something goes wrong (see above)
 - No remediation included by default - hard to do event with plugins

## TODO
- Groups: https://home-assistant.io/components/group/
- Spotify support:  https://home-assistant.io/components/media_player.spotify/
- Samsung smartTV support: https://home-assistant.io/components/media_player.samsungtv/
- Location tracking via OwnTracks
   -> This requires a DDNS, which means exposing everything to the web, which has security implications.
- https://home-assistant.io/components/media_player.cast/
- Calling "homeassistant/reload_core_config" one config reload instead of doing a HA restart
- Force state update on Nest after changing state through python-nest command
- Metrics dashboard (logstash)
- Improve roofcam accuracy
- sonos-node-http-api: SSL & auth
- Better messaging in slack (also include lights + custom nest sensors)
- Backups: sensu/redis DB
- Backups: auto-copy to Samba share
- sudo askpass program lastpass
- limit speedtest.net CPU cycles using cgroups
- Smarter office lighting behavior (relax, etc -> don't turn off lights when working)

- Upstairs scenes: packing, working

- Security:
    - Let's encrypt support
    - TLS everywhere: home-assistant, HADash, Spotify, Sensu, Uchiwa
    - iptable rules for all (just block all and then selectively allow),
        make sure base role wipes all other ip tables rules -> in case I manually set something, this should be undone

### Automation Ideas
- Cooking: If not watching TV & Music=NoPreset -> play music
- Cooking: If watching TV and cooking active for 1 hour -> disable cooking
- Away/Home Mode Nest => notification to slack (later auto-set)
- Morning: Turn on office lights after 6AM -> mark as home/awake
- Working after 6 AM => Working


### Sensor ideas
- Refactor monit-hass-sensors to custom-hass-sensors so we can do more than just monit sensors
- Roofcam sensor (refactor monit-hass-sensors to be more generic)
- Laptop sensors: e.g. track that Joris started working when laptop is on and connected to monitor
- Car at home detect sensor based on image recognition
- Door/window sensors
- Upload/backups success sensor (based on Monit/other check)
- Custom nest sensors based on python-nest, because current nest sensors in Hass aren't very good
- Nest smoke detector checks integrated with sensu

### HADashboard
- HADashboard: Volume control
- HADashboard: Custom Nest Cam controls (incl enable-disable support + link to livestream)
- HADashboard: Custom weather widget
