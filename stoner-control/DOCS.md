# Stoner Control RV Controller Bridge

Bridges a compatible BMPro control node to Home Assistant via MQTT
discovery. Exposes lights (including dimmable lights), climate/HVAC zones,
slide/awning motors, water tanks, voltage sources, temperature sensors, water
pump, line heaters, generator status, inverter energy, shore power load
shedding, vehicle configuration, and diagnostic entities as native Home
Assistant entities. Optional Firebase cloud mirroring keeps the BMPro mobile
app in sync while the local BLE bridge is active.

## Requirements

- Home Assistant OS, Supervised, or any installation that can run apps.
- A working **Mosquitto** (or other) MQTT broker app, configured and running.
- A Bluetooth adapter on the Home Assistant host with BLE support. Any adapter
  name works (`hci0`, `hci1`, …). If the host has more than one adapter, the
  app uses one of them, and it is not necessarily `hci0`. Once the app has
  found the node, it logs `using Bluetooth adapter /org/bluez/hciN` to say
  which adapter it is connecting through, even if that connect then fails.
  There is no option yet to choose a different adapter.
- The BMPro node powered on and within Bluetooth range.

## First-time pairing

The BMPro node only accepts new BLE bonds while it is in **pairing mode**.

1. On the BMPro wall display: *Settings → Bluetooth → Pair new device*
   (the exact menu varies by coach builder; consult your owner's manual).
2. Within ~60 seconds, start this app. It will scan for a compatible BMPro
   BLE advertisement, perform a BlueZ "Just Works" pairing, and authenticate.
3. Subsequent restarts re-use the bond and reconnect silently. The app
   automatically caches the last-connected BLE device ID so subsequent
   startups connect instantly without re-scanning.

If pairing fails, the app log will explain why. The most common cause is
that the node was not in pairing mode when the app tried to connect.

## Configuration

| Option                     | Default            | Description |
| -------------------------- | ------------------ | ----------- |
| `device_identifier`        | `1`                | Device identifier byte sent during the BMPro auth handshake. Leave at `1` unless you know you need a different value. |
| `ble_device_id`            | *(empty)*          | Optional Bluetooth device ID of a previously-paired BMPro node. When set the app resolves the device instantly instead of performing a 30-second BLE scan. Delete this value to force a new device scan and pairing on the next restart. Automatically populated after a successful connection. |
| `mqtt_discovery_prefix`    | `homeassistant`    | Override the MQTT discovery prefix if your HA install uses a non-default value. |
| `mqtt_base_topic`          | `stoner-control`   | Base topic used for state and command messages (e.g. `stoner-control/light/awning/state`). |
| `mqtt_host`                | *(auto-detected)*  | Optional explicit MQTT broker host. When omitted the app uses the broker advertised by the Supervisor service-discovery API. |
| `mqtt_port`                | *(auto-detected)*  | Optional explicit MQTT broker port (1–65535). Defaults to the broker's advertised port, typically `1883`. With `mqtt_host` and `mqtt_ssl` set, defaults to `8883`. |
| `mqtt_username`            | *(auto-detected)*  | Optional explicit MQTT username override. |
| `mqtt_password`            | *(auto-detected)*  | Optional explicit MQTT password override. |
| `mqtt_ssl`                 | `false`            | For an external broker set with `mqtt_host`: when `true` the app connects to it over TLS, on port `8883` unless `mqtt_port` is set. The broker's certificate must be trusted and must match `mqtt_host`. Without `mqtt_host` the app uses the Mosquitto app's internal broker, which it reaches without TLS, so `mqtt_ssl` is ignored and the log says so. |
| `firebase_username`        | *(empty)*          | Optional BMPro Firebase account e-mail. When set together with `firebase_password`, the app mirrors live BLE state back into the BMPro cloud document and relays fresh Firebase commands to the RV over Bluetooth. |
| `firebase_password`        | *(empty)*          | Password for the BMPro Firebase account used for cloud mirroring and command relay. |
| `firebase_project_id`      | `setec-bmpro-prod` | Firebase project ID for the BMPro cloud environment. Override only if BMPro changes environments. |
| `firebase_api_key`         | *(set by default)*  | Firebase web API key used for email/password sign-in. |
| `firebase_app_cloud_id`    | `rvm_bmpro_connect` | Firestore app partition used under `apps/{appCloudId}/users/{uid}`. |
| `log_level`                | `info`             | One of `trace`, `debug`, `info`, `warn`, `error`. |

Only `device_identifier`, `mqtt_discovery_prefix`, `mqtt_base_topic` and
`log_level` appear on the app's **Configuration** tab by default. To set any
other option, turn on **Show unused optional configuration options**. An
option left unset uses the default shown in the table.

MQTT broker credentials are obtained automatically from the Supervisor when
the official Mosquitto app is installed; no manual broker config is needed
in the typical case.

If Firebase credentials are configured, the app signs in to the BMPro cloud
and behaves as a local gateway. It publishes live BLE light/climate state back to
Firestore while the RV is connected over Bluetooth, polls the vehicle command
subcollection, and relays fresh supported command documents to the RV over BLE.
Commands more than 10 seconds old are discarded, as the BMPro gateway does.
Their age, and the `serverTimestamp` the app writes with the state, come from
the cloud's clock, so the Home Assistant host's clock doesn't need to be right.
Firebase is never used as the source of truth for current state.

If the BMPro cloud refuses the e-mail address or password, the app logs an
error saying why and turns cloud mirroring off until it restarts, rather than
trying again, which could get the account locked. Correct `firebase_username`
or `firebase_password`, save the options, and restart the app when Home
Assistant offers to. Bluetooth control and Home Assistant carry on either way.

If the BMPro cloud is limiting sign-ins instead (after too many attempts, or
when a quota runs out), the app logs a warning and makes no further sign-in
attempt for 5 minutes, since each one could extend the lockout. The wait
doubles each time the cloud refuses again, up to 30 minutes, or longer if the
cloud asks for it, and it starts again from 5 minutes after a sign-in
succeeds. Cloud mirroring and remote commands resume on their own afterwards;
nothing needs changing.

## What gets exposed

### Lighting

| BMPro concept                    | HA entity                    |
| -------------------------------- | ---------------------------- |
| On/off lights                    | `light` (no brightness)      |
| Dimmable lights                  | `light` (with `brightness`)  |
| All-lights special command       | `switch` (`All Lights`)      |

### Climate

| BMPro concept                    | HA entity                                              |
| -------------------------------- | ------------------------------------------------------ |
| Climate zones (no compressor)    | `climate` (off, fan_only, heat)                        |
| Climate zones (with compressor)  | `climate` (off, fan_only, cool, heat, auto)            |
| Zone temperature setpoint        | `climate` temperature control (50–90 °F)               |
| Zone current temperature         | `climate` current temperature sensor                   |
| Zone fan speed                   | `climate` fan mode (low, high)                         |
| Zone heat source (heat pump and furnace) | `select` (Furnace, Heat Pump, Both)          |

A zone with both a heat pump and a furnace gets a **Heat Source** select, named
after the zone (for example "Main Heat Source"), that chooses what Heat mode
runs. Home Assistant's climate entity has no heat-source setting, so it is a
separate entity. As in the BMPro app, a choice only takes effect while the zone
is in Heat mode; in any other mode the add-on logs and ignores it. A zone with
only one heat source gets no select.

High is the default fan speed. Zones report their fan speed as low or high. If a
zone ever reports no valid fan speed, it shows as `high`, and the add-on sends it
High automatically, in any mode. It sends once each time a zone starts reporting
no valid speed, retries after 30 seconds if the command fails, and logs each
correction in the add-on log.

### Motors

| BMPro concept                    | HA entity                    |
| -------------------------------- | ---------------------------- |
| Slides / awnings (motors)        | `cover` (open/close/stop)    |

### Water & sensors

| BMPro concept                    | HA entity                    |
| -------------------------------- | ---------------------------- |
| Water tanks                      | `sensor` (percentage, %)     |
| Voltage sources (battery banks)  | `sensor` (voltage, V)        |
| Temperature sensors              | `sensor` (temperature, °C)   |
| Water pump                       | `switch`                     |
| Line water heaters               | `switch`                     |

### Energy (dynamic discovery — entities appear on first notification)

| BMPro concept                    | HA entity                    |
| -------------------------------- | ---------------------------- |
| Generator running status         | `binary_sensor`              |
| Generator hours                  | `sensor` (hours, h)          |
| Generator state                  | `sensor`                     |
| Generator start output fault     | `binary_sensor` (problem)    |
| Generator stop output fault      | `binary_sensor` (problem)    |
| Inverter AC voltage              | `sensor` (voltage, V)        |
| Inverter AC current              | `sensor` (current, A)        |
| Inverter power                   | `sensor` (power, W)          |
| Shore power line 1 connected     | `binary_sensor` (plug)       |
| Shore power line 2 connected     | `binary_sensor` (plug)       |
| 50A shore power detected         | `binary_sensor` (plug)       |
| Shore line 1 current             | `sensor` (current, A)        |
| Shore line 2 current             | `sensor` (current, A)        |

Generator state is Running or Stopped, or Starting or Stopping while the
auto-gen smart start is starting or stopping the generator. The start and stop
output fault sensors exist only for a gas generator; they report a fault on the
line that drives the generator's start or stop input.

### System status

| BMPro concept                    | HA entity                    |
| -------------------------------- | ---------------------------- |
| System overvoltage               | `binary_sensor` (problem)    |
| Motor controls disabled          | `binary_sensor` (problem)    |
| Slideout lockout                 | `binary_sensor` (on/off)     |

*Slideout Lockout* is on while the node locks out the slide-outs, which on a
trailer is while it's hitched for towing. It has no device class, so Home
Assistant shows it as On or Off.

### Vehicle configuration

| Concept                          | HA entity                    |
| -------------------------------- | ---------------------------- |
| Vehicle config availability      | `sensor` (available/unavailable) |
| Refresh vehicle config           | `button`                     |
| Raw config JSON                  | Retained MQTT topic `{base}/rvmconfig` |

### Diagnostics & control

| Concept                          | HA entity                    |
| -------------------------------- | ---------------------------- |
| BLE connection status            | `binary_sensor` (connectivity) |
| Last Firebase publish attempt    | `sensor` (timestamp)         |
| Last successful Firebase publish | `sensor` (timestamp)         |
| Force BLE reconnect              | `button`                     |
| Force status refresh             | `button`                     |

Everything is auto-discovered: as soon as the app connects and downloads
the vehicle's `app.json`, HA will populate the entities.

### Availability

Home Assistant shows an entity as unavailable, rather than its last value,
when the app can't reach what it reports on:

- **The vehicle's entities** (every table above except *Vehicle
  configuration* and *Diagnostics & control*) are unavailable while the app is stopped or has lost
  the MQTT broker, and while the Bluetooth link to the RV is down or
  reconnecting. After a reconnect they come back once the app has read the
  node again and can send it commands, a few seconds after *BLE Connected*
  turns on.
- **The vehicle configuration and diagnostics entities** are unavailable only
  while the app is stopped or has lost the broker. They work without the
  Bluetooth link, so *BLE Connected* shows off and **Force BLE Reconnect**
  can still be pressed while the RV is out of reach.

The app publishes `online` or `offline`, retained, to two topics:
`<base topic>/availability` for itself (the broker publishes `offline` there
if the app dies without disconnecting), and `<base topic>/ble/availability`
for the vehicle, which is `online` once the Bluetooth link is up and the app
is ready to send commands over it.

## Connection recovery

The bridge keeps itself connected without an add-on restart:

- **Starting without the vehicle.** If the node is out of range or powered
  off when the add-on starts, the add-on keeps trying to connect, and creates
  no Home Assistant entities until it has connected and read the node's serial
  number. Earlier versions created a placeholder device named "Disconnected"
  instead, which duplicated every entity once the real device appeared. The
  add-on deletes that placeholder device and its entities the first time it
  connects. Earlier versions could also create a second device named after
  the vehicle when they failed to read its serial number. That one worked, and
  automations may use its entities, so the add-on leaves it alone. If you have
  two devices named after the vehicle and want only one, delete the one you
  don't use from its device page in Home Assistant.
- **Vehicle link.** When the Bluetooth link drops, or you press **Force BLE
  Reconnect**, the add-on reconnects to the node, and the existing HA entities
  start updating again once it is back. Until then the vehicle's entities are
  unavailable, and the *BLE Connected* sensor shows the gap.
- **MQTT broker.** The add-on connects to the broker before the vehicle, so
  once Home Assistant has the add-on's entities (after its first connection
  to the RV), *BLE Connected* shows off while it waits for the RV. If it can't find a broker yet (for example while the Mosquitto app is
  still starting), it logs why and keeps looking instead of stopping. If the
  broker goes away (for example while the Mosquitto add-on updates), the bridge
  retries every few seconds, backing off to once a minute. When the broker returns, the bridge restores its subscriptions,
  finishes any entity discovery that was cut off, and re-reads the vehicle's
  state so Home Assistant catches up on anything that changed in the meantime.
- **Shutdown.** When the add-on stops it publishes `offline` to
  `<base topic>/availability`, so all of its entities show as unavailable.

## The app's image

Home Assistant downloads the app as a prebuilt image from the GitHub Container
Registry, `ghcr.io/ashleygrant/stoner-control/ha-stoner-control-amd64` or
`ha-stoner-control-aarch64`, tagged with the app version. The images are
public, so no registry login is needed.

To build the image yourself, use a clone of the whole repository. The
Dockerfile copies the .NET projects from `src/`, so its build context must be
the repository root, and it needs BuildKit, which is Docker's default builder:

```sh
docker build -f home-assistant-addon/stoner-control/Dockerfile -t stoner-control-bridge .
```

Home Assistant can't build this app on the device itself. Its local build uses
the app's own folder as the build context, which doesn't contain the source.
