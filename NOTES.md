# TODO
This is a list of things I'm considering to add to casa.

## General

- Grafana: ansible tasks (under top-level tasks/ to restore grafana datasource)
- Prometheus: alert when node_exporter is not adding new textfiles (i.e. when test results aren't being collected)
- HaDash: darker nav buttons
- HaDash: camera on-click expansion
- HaDash: auto-nav back to homescreen
- Synthetic tests for Homeassistant (run pytests continously)
- Envoy/Nginx proxy for prometheus, webhook
- Location tracking via OwnTracks
   -> This requires a DDNS, which means exposing everything to the web, which has security implications.
- https://home-assistant.io/components/media_player.cast/
- Alexa integration
- Improve roofcam accuracy
- sonos-node-http-api: SSL & auth
- Better messaging in slack (also include lights + custom nest sensors)
- sudo askpass program lastpass
- limit speedtest.net CPU cycles using cgroups
- Smarter office lighting behavior (relax, etc -> don't turn off lights when working)
- Use Flux to change light color in hallway depending on time of day: https://home-assistant.io/components/switch.flux/
- Upstairs scenes: packing, working
- Grafana, discrete panel:
  - Patch: https://github.com/NatelEnergy/grafana-discrete-panel/blob/master/src/module.ts#L194
    To make it so that time is shown not in humanize() but in specified timeframe
- Security:
    - Let's encrypt support
    - TLS everywhere: home-assistant, HADash, Spotify, Sensu, Uchiwa
    - iptable rules for all (just block all and then selectively allow),
        make sure base role wipes all other ip tables rules -> in case I manually set something, this should be undone

- Laptop checks:
  - Spotify playing
- Runkeeper API: https://github.com/mko/runkeeper-js
  -> Stats wrt workouts etc

### Old TODO (no longer relevant)
- Force state update on Nest after changing state through python-nest command


## Automation Ideas
- When pausing music -> set music preset to NoMusic
- Alerting on e.g. open window: https://home-assistant.io/components/alert/
## Sensor ideas
- Car at home detect sensor based on image recognition
- Door/window sensors
- Custom nest sensors based on python-nest, because current nest sensors in Hass aren't very good
- Nest smoke detector checks integrated with sensu
- Sensu vmstat: https://github.com/sensu-plugins/sensu-plugins-vmstats
- Alarm:
  https://home-assistant.io/components/alarm_control_panel.manual/
  http://appdaemon.readthedocs.io/en/stable/DASHBOARD_CREATION.html#alarm

- Smart meter:
  Use this cable: https://www.sossolutions.nl/slimme-meter-kabel
  https://home-assistant.io/components/sensor.dsmr/

- Weekend Hobby sensor
- Raspberry PI online sensor
- Fix aggregate sensor for homeassistant (google not working)

## Actuator ideas
- PS4Waker
- Lights in back kitchen

### Homematic Radiotor thermostat
https://www.conrad.nl/nl/homematic-ip-draadloze-radiatorthermostaat-hmip-etrv-2-1406552.html?WT.mc_id=gshop&WT.srch=1&gclid=CjwKCAiArOnUBRBJEiwAX0rG_fbAavfdl8fReKPGIuYmW6GDnaOdXExPkVhENMpaS9t9W8L_VXlm6BoCL-oQAvD_BwE&insert=8J&tid=933477491_46789847735_pla-415594332407_pla-1406552

Seems to work with homeassistant:
https://community.home-assistant.io/t/ccu2-w-homematic-and-homematic-ip-devices/4045/32

Installation instructions
http://www.eq-3.com/Downloads/eq3/downloads_produktkatalog/homematic_ip/bda/HmIP-eTRV-2-UK_UM_web.pdf

Not clear if this will work on our radiators.

## HADashboard
- HADashboard: Volume control
- HADashboard: Custom Nest Cam controls (incl enable-disable support + link to livestream)
- HADashboard: enlarge camera view on click (not so hard using custom JS)
- HADashboard: Custom weather widget

# Technical notes
Keeping these here mostly for personal reference.

## Samsung TV

TV Model: UE48H6200AW

- Samsung smartTV support: https://home-assistant.io/components/media_player.samsungtv/
- https://github.com/Ape/samsungctl
- Open ports: 7676, 8000, 8001, 8080, 8443

Learned about v2 from here: https://github.com/Ape/samsungctl/issues/22

```bash
# TV off
$ curl -I -m 2 http://$SAMSUNGTV_IP:8001/api/v2/
curl: (28) Connection timed out after 2001 milliseconds

# TV on
$ curl -I -m 2 http://$SAMSUNGTV_IP:8001/api/v2/
curl: (7) Failed to connect to $SAMSUNGTV_IP port 8001: Connection refused
```


## InfluxDB:
```
export INFLUX_USERNAME="$(vault-get influxdb_admin_user)";export INFLUX_PASSWORD="$(vault-get influxdb_admin_password)";
echo "export INFLUX_USERNAME=\"$INFLUX_USERNAME\"; export INFLUX_PASSWORD=\"$INFLUX_PASSWORD\";"
influx -ssl --unsafeSsl -username "$INFLUX_USERNAME" -password "$INFLUX_PASSWORD"
# Examples
show users;
show grants for "<example user>";
use mydatabase;
show series; # recall, influx doesn't have tables, it has timeseries

influx -ssl --unsafeSsl -username "$INFLUX_USERNAME" -password "$INFLUX_PASSWORD" -execute "show databases"
```

## python-nest
```bash
source /opt/homeassistant/.venv/bin/activate
export NEST_CLIENT_ID=$(grep "nest:" -A 2 /opt/homeassistant/configuration.yaml  | awk '/client_id/{print $2}')
export NEST_CLIENT_SECRET=$(grep "nest:" -A 2 /opt/homeassistant/configuration.yaml  | awk '/client_secret/{print $2}')
nest --client-id $NEST_CLIENT_ID --client-secret $NEST_CLIENT_SECRET --token-cache /opt/homeassistant/nest.conf --index 0 camera-show
```

## grafana

Using this tool for backups: https://github.com/ysde/grafana-backup-tool

Note that the exported json files (with extension .dashboard) can not just be imported through the grafana UI, you need
to use the same grafana-backup-tool to do the restore, like so:

```bash
cd /tmp; git clone https://github.com/ysde/grafana-backup-tool.git
# Get token from http://casa:3001/org/apikeys
export GRAFANA_URL="http://casa:3001"; export GRAFANA_TOKEN="<token>";
python grafana-backup-tool/createDashboard.py ~/Downloads/dashboards/Overview.dashboard
```

# Miscellaneous notes

## Upgrading homeassistant

When upgrading homeassistant, you need to manually start it because on first run hass will install a bunch of additional packages.
Especially installing pyatv tends to take 10+ mins.

Try running
```bash
ps -ef | grep pip
```
# Virtualbox performance issues
I keep running into vagrant box lock ups. Some details of research I've done around this:

https://joeshaw.org/terrible-vagrant-virtualbox-performance-on-mac-os-x/

```
export CASA_VM=$(VBoxManage list vms | grep casa_home | cut -f1 -d" " | tr -d "\"")
VBoxManage storagectl $CASA_VM --name "IDE" --hostiocache on
```

TODO
```
NAT: Error(22) while setting RCV capacity to (65536)
```
## SCP

scp -P 2222 -i ~/repos/casa/.vagrant/machines/home/virtualbox/private_key ubuntu@127.0.0.1:/home/ubuntu/redis.conf .

## Hass APIs
```bash
export HASS_URL="http://0.0.0.0:8123"; export HASS_PASSWORD="$(awk '/api_password: /{print $2}'  /opt/homeassistant/configuration.yaml)"
export HASS_URL="http://$HASS_IP:8123"; export HASS_PASSWORD="$(vault-get ' api_password')";

# Getting  entity picture
curl -s -H "x-ha-access: $HASS_PASSWORD" -H "Content-Type: application/json" $HASS_URL/api/states/camera.hallway | jq -r ".attributes.entity_picture"
# Error log:
curl -s -H "x-ha-access: $HASS_PASSWORD" -H "Content-Type: application/json" $HASS_URL/api/error_log
# Specific entity state:
curl -s -H "x-ha-access: $HASS_PASSWORD" -H "Content-Type: application/json" $HASS_URL/api/states/light.office

# Turn on light
curl -s -H "x-ha-access: $HASS_PASSWORD" -H "Content-Type: application/json" -d '{"entity_id": "light.office"}' $HASS_URL/api/services/light/turn_on

```

## seshat

```sh
export INFLUXDB_BIND_IP="0.0.0.0"
export INFLUXDB_DB=""
export INFLUXDB_USERNAME=""
export INFLUXDB_PASSWORD=""


/usr/bin/node /opt/sensu/checks/seshat/dist/duration.js -k --measurement sensor.desk_state --state up --write-duration --host $INFLUXDB_BIND_IP --database $INFLUXDB_DB  --username $INFLUXDB_USERNAME  --password $INFLUXDB_PASSWORD
```
## Sensu
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

## Lights
Color:

Tradfri white spectrum lamps support between 2200 (warm)  and 4000 (cold) Kelvin.
That *should* translate to 0-250 mireds (color_temp in hass), but that doesn't seem to be working
http://www.leefilters.com/lighting/mired-shift-calculator.html

Home-assistant does not allow you to change attributes (like kelvin/brightness) on a group of lights at a time yet
Even if they are exposed by Tradfri/Hue as a single light.
https://community.home-assistant.io/t/grouped-light-control/1034/49

## Extra open ports

6379 -> Redis

1400 -> Homeassistant SoCo API (Sonos)
https://github.com/home-assistant/home-assistant/blob/44e4f8d1bad624f8d27dfda7230f0a0a2409a404/homeassistant/components/media_player/sonos.py#L557

631 -> IPP Port (Internet Printing Protocol)
TODO: Disable/remove CUPS

8088 -> InfluxDB

3030, 3031 -> Sensu-client (Ruby)

5355 -> systemd resolve:
https://askubuntu.com/questions/907246/how-to-disable-systemd-resolved-in-ubuntu


IPv6 traffic?

## Sonos-http-api
When Sonos playbar is streaming from tv the /TV Room/state call returns the following.
When turning off the TV, this state stays the same, except for the elapsedTime being reset to 0. This does
also happen at different occasions (e.g. when switching HDMI inputs, etc), so this doesn't seem to be a reliable
way of determining whether the TV is on or off.

```
curl -s "http://192.168.1.121:5005/TV%20Room/state" | jq
```

```json
{
  "volume": 24,
  "mute": false,
  "equalizer": {
    "bass": 0,
    "treble": 0,
    "loudness": true,
    "speechEnhancement": false,
    "nightMode": false
  },
  "currentTrack": {
    "duration": 0,
    "uri": "x-sonos-htastream:RINCON_B8E93742407C01400:spdif",
    "type": "line_in",
    "stationName": ""
  },
  "nextTrack": {
    "artist": "",
    "title": "",
    "album": "",
    "albumArtUri": "",
    "duration": 0,
    "uri": ""
  },
  "trackNo": 1,
  "elapsedTime": 74,
  "elapsedTimeFormatted": "00:01:14",
  "playbackState": "PLAYING",
  "playMode": {
    "repeat": "none",
    "shuffle": false,
    "crossfade": false
  }
}
```

# Tests

```bash
source activate .venv36 # conda env
cd tests  # important to be inside the tests directory!
pytest -rw -s --hadashboard-url http://$HASS_IP:5050 --homeassistant-url http://$HASS_IP:8123 --homeassistant-password "$(vault-get ' api_password')" tests/
```

## Tradfri issue


```
2018-05-23 04:02:21 WARNING (MainThread) [homeassistant.components.sensor.tradfri] Observation failed for TRADFRI motion sensor
Traceback (most recent call last):
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/pytradfri/api/aiocoap_api.py", line 92, in _get_response
    r = yield from pr.response
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/aiocoap/protocol.py", line 816, in _run_outer
    yield from cls._run(app_request, response, weak_observation, protocol, log, exchange_monitor_factory)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/aiocoap/protocol.py", line 865, in _run
    blockresponse = yield from blockrequest.response
aiocoap.error.RequestTimedOut
```

```
Traceback (most recent call last):
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/helpers/entity.py", line 196, in async_update_ha_state
    yield from self.async_device_update()
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/helpers/entity.py", line 317, in async_device_update
    yield from self.async_update()
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/components/light/tradfri.py", line 153, in async_update
    await self._api(self._group.update())
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/pytradfri/api/aiocoap_api.py", line 152, in request
    result = yield from self._execute(api_commands)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/pytradfri/api/aiocoap_api.py", line 142, in _execute
    _, res = yield from self._get_response(msg)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/pytradfri/api/aiocoap_api.py", line 97, in _get_response
    yield from self._reset_protocol(e)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/pytradfri/api/aiocoap_api.py", line 79, in _reset_protocol
    yield from protocol.shutdown()
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/aiocoap/protocol.py", line 133, in shutdown
    for exchange_monitor, cancellable in self._active_exchanges.values():
AttributeError: 'NoneType' object has no attribute 'values'
```

https://github.com/ggravlingen/pytradfri/blob/master/pytradfri/api/aiocoap_api.py#L79


Probably should keep eye on https://aiocoap.readthedocs.io/en/latest/news.html for newer versions.



## AppDaemon issue
Ran into https://github.com/home-assistant/home-assistant/issues/13644,
fixed by running.

TypeError: __init__() got an unexpected keyword argument 'ssl'
-> this has to do with the fac that aiohttp is outdated

Fix with:

```bash
sudo /opt/homeassistant/.venv/bin/pip install aiohttp==3.1.3
```

Or by reinstalling homeassistant

# Hue issues

```
Traceback (most recent call last):
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/components/hue/bridge.py", line 47, in async_setup
    hass, host, self.config_entry.data['username'])
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/components/hue/bridge.py", line 159, in get_bridge
    websession=aiohttp_client.async_get_clientsession(hass)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/helpers/aiohttp_client.py", line 38, in async_get_clientsession
    hass.data[key] = async_create_clientsession(hass, verify_ssl)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/helpers/aiohttp_client.py", line 55, in async_create_clientsession
    connector = _async_get_connector(hass, verify_ssl)
  File "/opt/homeassistant/.venv/lib/python3.6/site-packages/homeassistant/helpers/aiohttp_client.py", line 163, in _async_get_connector
    connector = aiohttp.TCPConnector(loop=hass.loop, ssl=ssl_context)
TypeError: __init__() got an unexpected keyword argument 'ssl'
```

This has to do with an incompatible version of aiohttp

--------
Before"

aiocoap==0.4a1
aiohttp==2.3.10
aiohttp-cors==0.7.0
aiohue==1.5.0

------------
after:
aiocoap==0.4a1
aiohttp==3.3.2
aiohttp-cors==0.7.0
aiohue==1.5.0

# Prometheus
Add basic auth through nginx (or envoy?): https://prometheus.io/docs/guides/basic-auth/

Checks to convert:
- PS4 On/off -> homeassistant device
- Roofcam alive
- Roofcam water detector
- Sonos Error
- InfluxDB systemd service
- 