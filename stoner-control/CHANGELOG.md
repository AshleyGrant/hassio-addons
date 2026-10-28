# Changelog

## 1.0.31

- The app's image now carries the standard source, licence (MIT) and title labels. The source label links its GitHub Container Registry packages to this repository however they are pushed, and the package page shows the licence. The images are public, so pulling them needs no login (#115).
- The Documentation tab explains where the image comes from and how to build it yourself from a clone of the repository (#115).

## 1.0.30

- A fresh install now starts with its default options. Home Assistant refused to start it with "Missing required option 'firebase_username'", because the Firebase credentials were listed as defaults without a value. They are now optional fields with no default, shown on the Configuration tab once you turn on **Show unused optional configuration options**. Existing installs keep their settings (#114).

## 1.0.29

- On newer BMPro node families, downloading the vehicle configuration (`app.json`) no longer lets a stray Bluetooth fragment drop or garble a chunk. It now applies the same response check the RVMN101C download already had. The RVMN101C is unaffected (#135).

## 1.0.28

- No change in the add-on's behaviour. The shared Bluetooth library gains a stricter Bluetooth address parser for the Windows platform code, which the add-on doesn't use (#19).

## 1.0.27

- New **Heat Source** select for each climate zone that has both a heat pump and a furnace: choose Furnace, Heat Pump or Both for Heat mode. As in the BMPro app, a choice only takes effect while the zone is in Heat. Zones with one heat source get none (#112).

## 1.0.26

- A climate zone whose node reports heat source 0 is now treated as a heat pump, as the BMPro app does, instead of an unknown heat source that the cloud mirror published as a furnace (#101).
- The cloud mirror no longer publishes placeholder values as a zone's real state. A zone with no reading yet shows fan speed high (there is no "auto" fan speed; Auto is a climate mode), and an unknown heat source is left out (#101).

## 1.0.25

- New **Generator State** sensor: Running, Stopped, or Starting/Stopping while the auto-gen smart start is starting or stopping the generator (#20).
- New **Generator Start Output Fault** and **Generator Stop Output Fault** problem sensors for a gas generator. They report a fault on the line that drives the generator's start or stop input, which the BMPro app shows as a marker on its Start and Stop buttons (#20).

## 1.0.24

- The cloud mirror now reports the generator's start and stop output faults separately from its general fault, and reports whether the auto-gen smart start is starting or stopping the generator, using the BMPro app's own keys. The BMPro app, when it reads the vehicle through the cloud, can now show the fault markers on a gas generator's Start and Stop buttons and its Starting/Stopping state. Showing these in Home Assistant comes in a later version (#20).

## 1.0.23

- Downloading the vehicle configuration (`app.json`) from the BMPro node is more robust (#17). A late reply to a retried request can no longer put the wrong bytes into the file, and a node that doesn't report the file's length no longer gets its configuration cut off after the first chunk.

## 1.0.22

- Stopping the add-on, or rebuilding the connection, while the Bluetooth connection is being set up now ends the setup after the step in progress. Before, it carried on through authentication retries and every notification subscription, and a cancelled connect could start a full Bluetooth scan. A deliberate stop is also no longer logged as the Bluetooth provider failing (#18).
- Shutting down the Bluetooth connection while a request is still in flight no longer risks corrupting the connection's internal state (#18).

## 1.0.21

- Stopping the add-on, or pressing **Force BLE Reconnect**, while it is connecting to the BMPro node now takes effect straight away. A connect that hangs gives up after a bounded time instead of blocking the next attempt (#11).
- A cancelled connect is no longer reported as a pairing failure that asks you to put the node into pairing mode.

## 1.0.20

- Stopping the add-on or rebuilding its Bluetooth connection, for example with **Force BLE Reconnect**, now actually disconnects from the BMPro node, so the BMPro phone app can connect straight away (#12). Before, the node stayed connected to the Home Assistant host until BlueZ timed the link out.
- A Bluetooth disconnect that fails, for example straight after the host's Bluetooth service restarts, is now logged instead of crashing the add-on (#120).

## 1.0.19

- The bridge can find a BMPro node that the host's Bluetooth stack has never seen, so a fresh Home Assistant install pairs without first running `scan le` in `bluetoothctl` (#113). Every Bluetooth scan used to fail on Linux, so the bridge only found a node that BlueZ already had cached. It now runs its own scan for the BMPro service and stops as soon as it finds the node.
- If the node can't be found, the log still names the Bluetooth adapter that was searched.
- Bluetooth scan warnings, such as a powered-off adapter, now appear in the log at the default `info` level. They used to be visible only with `debug` logging.
- A remembered `ble_device_id` no longer stops the scan when the host's Bluetooth stack has forgotten the node, for example after a backup is restored onto new hardware.

## 1.0.18

- The bridge works with a Bluetooth adapter that isn't named `hci0`, for example a USB dongle the host numbers `hci1` (#15). Before, pairing and connecting failed on such hosts because the bridge sent its BlueZ calls to `hci0`. With more than one adapter, the bridge picks one, not necessarily `hci0`. Once the bridge has found the node, it logs which adapter it is connecting through at the information level: `using Bluetooth adapter /org/bluez/hciN`.

## 1.0.17

- Rebuilt with updated dependencies, including the Bluetooth library (InTheHand.BluetoothLE 4.0.45), its D-Bus layer (Tmds.DBus 0.95.1), the Firestore client (4.6.0), MQTTnet (5.2) and the .NET 10.0.14 runtime libraries. The bridge's behaviour is unchanged.

## 1.0.16

- Rebuilt against the updated shared Bluetooth transport, which fixes reconnecting in the Android app (#110). The new code only runs on Android, so the bridge behaves the same.
- Disconnecting the Firebase connection straight after it connects no longer fails with an `ObjectDisposedException` (#118).

## 1.0.15

- Home Assistant entities keep updating after the vehicle connection is rebuilt, including after pressing **Force BLE Reconnect**. Before, any rebuild silently stopped the bridge until the add-on was restarted.
- The bridge keeps retrying the MQTT broker until it comes back, so a Mosquitto restart or a short network outage no longer leaves it offline. Once the broker is back, the bridge finishes any entity discovery that was cut off and re-sends the vehicle's current state, so entities don't keep showing values from before the outage.
- The bridge publishes `offline` to its availability topic when it stops.

## 1.0.14

- A Bluetooth command that fails because the link is gone now marks the connection disconnected straight away, so the bridge starts reconnecting instead of waiting for the next liveness check.
- The Bluetooth liveness check runs every 15 seconds (was 30), so the "BLE Connected" sensor reacts to a silent drop within about half a minute.
- The bridge recovers on its own after the system D-Bus or BlueZ restarts (for example a Home Assistant OS update of the Bluetooth stack); it used to need an add-on restart.

## 1.0.13

- Rebuilt against the updated shared Firebase transport (read retries, session reuse, adjustable polling). The bridge's own cloud mirror is unchanged, so there is no difference in behaviour.

## 1.0.12

- A standby connection is configured with the vehicle's climate zone count before its first status refresh.
- Reconnect timing counts planned retry delays instead of wall-clock time.

## 1.0.0

Initial Release
