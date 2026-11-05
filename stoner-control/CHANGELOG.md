# Changelog

## 1.0.63

- Fixed stopping the app while it reads the node's serial number or firmware version. A serial read carried on to the next source and could return the serial from the advertised name, or none at all, which could log a false "no serial number" message on the way out. A firmware read could log a false warning and report the version as unknown. A read that has started can't be interrupted, so a stop still waits for it to end, but the app now stops there instead of moving on. If you haven't stopped the app, a read that fails or times out still moves on to the next source (#389).
- Fixed stopping the app while it downloads the RV's profile (`app.json`). A download that ended because of the stop was treated as a failed one, so the app logged false warnings and looked for its cached copy on the way out. It now just stops. If you haven't stopped the app, a failed download still falls back to the cached copy (#389).
- Internal: a rejected Bluetooth MTU request is now compared with the MTU that was asked for, so Android Oreo no longer logs a false warning. This doesn't apply to the add-on, and nothing changes in how the app behaves (#389).

## 1.0.62

- Internal: the default Firebase web API key is now stored Base64-encoded in the source, so the plain key no longer appears in the source. The app decodes it at start-up and signs in with the same key as before, and a `firebase_api_key` you set yourself still takes precedence. Nothing changes in how the app behaves.

## 1.0.61

- Fixed dimming levels at 30% and 70%. The bridge rounded those two levels down where the BMPro app rounds them up, so it wrote one step less than the app (19660 instead of 19661, and 45874 instead of 45875, out of 65535). You won't see the difference on a light, but the commands now match the app's exactly (#370).

## 1.0.60

- Internal: the Linux (BlueZ) Bluetooth layer no longer checks at every log line whether it was given somewhere to log. When it has none, it now logs to a sink that discards everything, so its unit tests no longer need a second copy of each scenario without a log. Nothing changes in how the app behaves (#368).

## 1.0.59

- Fixed a cloud stall that wasn't always counted. When updates to the cloud stop getting through, the app restarts its Firebase session, and after five such stalls in a row it also rebuilds the Bluetooth connection. If the stuck update finished just as the app stepped in, the stall was treated as an ordinary failure instead: the app signed in to Firebase again and didn't count it toward the five. It now always counts, and the extra sign-in is gone (#332).
- Fixed two mistakes in how the app retries its Bluetooth connection:
  - When the Bluetooth link dropped just after connecting, the connect gave up with a fault even though a reconnect was about to start. The app logged "The vehicle connection ended Faulted instead of connecting", waited 15 seconds and rebuilt the whole connection. It now waits for that reconnect instead (#319).
  - A connection attempt that was stopped, then failed late, was logged as having "exhausted all retry attempts". It's now logged at debug level, and that warning means only that the retries really ran out (#320).
- Internal: the shared connection layer now reports Connecting, not Connected, when one of two connections drops while the other is still connecting. This app uses only Bluetooth, so nothing changes here (#318).
- Internal: the Bluetooth transport, device discovery and the Linux (BlueZ) Bluetooth layer now reach InTheHand and D-Bus through small interfaces, so their connect, scan, pairing and recovery logic can be unit tested without hardware. The authentication cipher's code is split into two methods, with the same choice between them. Two unused setters in the device catalog are removed. Saving a configuration archive straight to a file now leaves the document marked as changed when the save fails; nothing in the apps saves that way yet. None of this changes how the app behaves (#286, #288, #294, #297).

## 1.0.58

- Internal: the code that relays commands from the cloud to the RV now reaches Firebase's live listener through a small interface, so its checks (how old a command may be, which of two commands for the same light or zone wins, waiting for Bluetooth, not relaying a command twice) can be tested without Firebase. Three helpers nothing used are removed. Nothing changes in how commands are relayed (#290).

## 1.0.57

- Internal: the app's start-up setup (log level, services, Bluetooth device and retry settings) moved into its own class, and the options loader can now be tested without Supervisor, for the unit test coverage work (#290). Nothing changes in how the app starts or behaves.

## 1.0.56

- Internal: shared cloud code changed for the phone app, which no longer reports the cloud as stale just after Android wakes it (#280). Nothing changes in this app.

## 1.0.55

- The app now re-reads every status from the RV every 10 seconds while connected, as well as reporting the changes the node sends. A missed update now corrects itself within 10 seconds, in Home Assistant and in the cloud. Commands still go first: a refresh waits while dimming or setpoint changes are being sent, and otherwise holds a command up for no more than the read in progress (#279).
- Fixed the cloud showing lights as off when they were on. After a reconnect, the app could miss the light states its first status read sent, and kept reporting those lights as off in the cloud until they changed. Home Assistant only looked right because it still had the values from before the reconnect (#279).

## 1.0.54

- Internal: closing a connection is sturdier. If a disconnect fails or is cancelled, the app still releases the rest of that connection and finishes closing down, instead of stopping at the first failure. No change you'd notice in normal use: the Bluetooth disconnect already logged its own failures rather than throwing them (#275).

## 1.0.53

- Bluetooth failures that used to vanish are now logged at debug level, with the reason. You'll see them only with `log_level: debug`. That covers a failed liveness check (with its count, "1/2"), a GATT service whose characteristics couldn't be listed, a skipped service during the configuration download, and a node identity field (the RV's OEM serial, manufacturer, model or revisions) that couldn't be read. Such a field still reads as empty, as before. Once whatever asked for an identity read, a GATT lookup or a configuration lookup is cancelled (the app stopping, or a connection torn down), a failure there is no longer skipped past: it goes up to the caller instead (#89).
- Internal: the app's constructors now reject a missing argument at once, instead of failing later somewhere unrelated (#89). A time seam in the configuration download's reader makes its retry tests deterministic (#258). Neither changes behaviour. A connection teardown that fails to close the connection, past what the Bluetooth layer already logs at debug, now logs a warning instead of dropping it (#89).

## 1.0.52

- With Firebase cloud mirroring on, a state update the cloud refuses because it's busy (HTTP 429 or 503) is now followed by a wait before the next one: 2 seconds, doubling each time the cloud refuses again, up to 10 seconds. The app goes back to its usual pace once an update gets through. Before, it wrote again a second later, which is what the cloud's refusal asks clients not to do. Each refusal logs a warning with the wait (#228).
- Any other failed state update, such as a network error, a timeout or another server error, is now sent again on the next pass, about a second later. Before, if nothing on the RV changed, it reached the cloud only with the next heartbeat, up to 10 seconds later, so the log's "will retry on next cycle" wasn't true (#228).

## 1.0.51

- With Firebase cloud mirroring on, the app no longer misses the RV's state when the first status reaches it just as mirroring starts. Before, if the RV then sent nothing new, the app published nothing to the cloud for that connection, not even its heartbeat, so the apps' Cloud mode showed the RV as offline until something on the RV changed (#252).
- Internal: the live vehicle status store moves out of the shared contracts library into its own, and the app depends on it through an interface (#78). Unused code is removed: an MQTT transport kind the app never used (#68), and a list of Bluetooth services that only the deleted Web Bluetooth project used (#69). The app's own MQTT connection to Home Assistant is unchanged. Package pins that changed nothing are dropped (#83). No change in behaviour.

## 1.0.50

- The app now reads the RV's configuration with the same rules as the Stoner Control apps, so all of them show the same devices (#71). Every sample configuration, the 425FO among them, gives the same entities as before. Only an unusual file reads differently. An item with no index is skipped, and a field of the wrong type is treated as missing; before, the item became index 0, or the whole file was refused. A temperature sensor's `analogIndex` is now read. A property name spelled with different capitals is now ignored. A file that repeats a key is now refused, so the app connects but publishes no entities for it.
- If the RV's configuration leaves a light unnamed, its entity is now named "Light N", as in the apps and the cloud, instead of "Output N", and its MQTT topics change with the name. Home Assistant keeps the old entity, which stops responding, and ignores the new one, because both have the same unique ID. To recover, delete the Stoner Control device in Home Assistant's MQTT integration, which clears its discovery messages, then restart the app so it's discovered again. Customisations of its entities are lost (#71).
- With Firebase cloud mirroring on, the vehicle document for the apps' Cloud mode is now built from the same configuration reader (#236). The document is unchanged for every sample configuration. On an unusual file, a tank type is normalised, so "gray" is written as `grey` and an unrecognised type as `fresh`. A voltage rail with no type is written as `unknown`, so Cloud mode shows it as an unknown rail, as Bluetooth does; before, it was shown as the system rail. Each motor's RV-C flag is read from the file's real key, where before it was always written as false. Temperature sensors are read by `analogIndex`, then `index`. Comments, trailing commas and wrong-typed fields no longer stop mirroring from starting.
- A Bluetooth connection that drops just as it connects now counts as a failed attempt, retried with the usual delays and give-up time. Before, the app could keep it as its link and report connected while every command failed, until it restarted or *Force BLE Reconnect* was pressed (#243).
- Internal: fixes to the shared status store's start and stop races (#70), unused configuration loaders removed (#73), and archive caps and saved timestamps in the configuration library (#72, #188). The app creates a new store for each connection and reads its configuration straight from the RV, so none of these changes its behaviour.

## 1.0.49

- Internal: updated libraries. Polly, which times the reconnect retries, is now 8.9.0, with builds for current .NET versions. Tmds.DBus, under the Bluetooth connection on Linux, is now 0.95.2, a packaging-only release. No change in behaviour is expected.

## 1.0.48

- From this version on, stopping, updating or restarting the app no longer logs a warning that watching for Bluetooth service restarts failed. The update to 1.0.48 itself still shows it once, because Home Assistant stops the previous version to update. The watch simply ends with the connection, and that is now logged at debug level; a watch that fails for any other reason is still a warning (#159).
- Internal: an unused cloud option is removed from the Firebase mirror's settings. No change in behaviour (#67).

## 1.0.47

- With Firebase cloud mirroring on, a cloud that is limiting sign-ins (too many attempts, or a quota used up) is waited out. The app logs a warning and makes no sign-in attempt for 5 minutes, doubling each time the cloud refuses again up to 30 minutes, and mirroring resumes on its own. Before, it signed in again after 5 seconds, then 10, and so on, and on every new Bluetooth connection, each attempt extending the lockout (#66).
- A cloud refusal of the e-mail address or password is recognised when the cloud adds an explanation after its error code. Before, such a refusal was retried like a passing failure instead of turning cloud mirroring off (#66).

## 1.0.46

- The app is MIT licensed, and its assemblies carry the owner's copyright. Its documentation uses the app's name and points at the repository installs come from (#56).
- The changelog now covers 1.0.1 to 1.0.11 (#56).

## 1.0.45

- With Firebase cloud mirroring on, the cloud now stamps the time on the state the app writes, and remote commands are judged by the cloud's clock. Before, a Home Assistant clock more than 30 seconds out made the apps show the RV as offline, and made the app drop valid commands or relay stale ones (#62).
- Remote commands more than 10 seconds old are discarded, as the BMPro gateway does. Before, the limit was 30 seconds (#62).

## 1.0.44

- With Firebase cloud mirroring on, a publish the cloud refuses now counts as failed. Before, it was logged and treated as published, so *Last Successful Firebase Publish* kept advancing while the apps saw stale state (#61).
- A token the cloud rejects is refreshed before the next request, instead of being sent until it expires on its own up to 55 minutes later. A publish refused because of the session (its token, or the document it writes) is retried a second later, and after five in a row the app signs in to the cloud again; remote commands pause while it waits to retry. Other failures, such as the cloud being busy or unavailable, are retried with the regular heartbeat and leave remote commands running (#61).

## 1.0.43

- With Firebase cloud mirroring on, a wrong e-mail address or password is reported once, with the reason, and cloud mirroring stays off until the app restarts. The same happens if the password changes or the account is disabled while the app runs. Before, the app retried the sign-in every few minutes for as long as it ran, which could get the account locked (#63).
- With Firebase cloud mirroring on, the cloud sign-in token is refreshed once when it nears expiry, instead of by each part of the mirror at the same time (#60).

## 1.0.42

- Internal: the Firebase sign-in and token refresh used by the cloud transport move into one shared class, which the add-on's cloud mirror will use next. No change in behaviour (#63).

## 1.0.41

- When the Bluetooth link drops, light dimming and climate setpoint changes still waiting to be sent are dropped at once, instead of each failing against the lost connection with a *worker failed* warning (#57).
- A dropped connection is logged under the vehicle's name, and a connection attempt that fails while the RV is out of reach is no longer also logged as "Connection to Disconnected lost." (#57).

## 1.0.40

- With Firebase cloud mirroring on, the add-on stops cleanly. Before, stopping, restarting or updating it hung until Supervisor killed it after 10 seconds, and the app page then showed an error (#148).
- With Firebase cloud mirroring on, the add-on reconnects to the RV after the Bluetooth link drops, or after **Force BLE Reconnect**. Before, the same hang most likely kept it from ever reconnecting until it was restarted (#148).
- Each disconnect now logs how long tearing the connection down took (#148).

## 1.0.39

- With Firebase cloud mirroring on, the add-on listens for remote commands only on the collection the apps write them to. The second listener it opened, which Firebase always refused, logged a *PermissionDenied* warning every time it started and was restarted straight away. Now a refused listener is logged once and left stopped. A listener that fails again straight after restarting waits longer each time, up to a minute, and one that ends without an error waits before restarting too (#59).

## 1.0.38

- Home Assistant now shows the add-on's entities as unavailable, instead of their last values, while the add-on is stopped or has lost the MQTT broker. The vehicle's entities are also unavailable while the Bluetooth link to the RV is down or reconnecting. *BLE Connected*, the Firebase timestamps, the vehicle configuration entities and the two Force buttons stay available, so you can still press **Force BLE Reconnect** (#52).

## 1.0.37

- With Firebase cloud mirroring on, a Bluetooth outage no longer counts as a stalled cloud publish. Before, every outage longer than 45 seconds tore down the cloud session and signed in to Firebase again, about every 50 seconds, with misleading "stall" warnings (#51).
- A real stall now keeps the signed-in cloud session and only restarts the publishing and the remote-command listeners. Five stalls in a row without a successful publish rebuild the Bluetooth connection, as intended; before, the count never got past one (#51).

## 1.0.36

- The add-on connects to the MQTT broker before the vehicle. While the RV is out of reach, *BLE Connected* now shows Off within a few seconds of start-up, instead of keeping whatever it showed when the add-on last stopped (#53).
- If no MQTT broker can be found yet, for example while the Mosquitto app is still starting, the add-on logs why and keeps looking. Before, it stopped and had to be restarted (#53).

## 1.0.35

- *Slideout Lockout* now shows On while the slide-outs are locked out (on a trailer, while it's hitched for towing) and Off otherwise, with a trailer icon. It used Home Assistant's lock class, which shows on as Unlocked, so it read backwards. The entity keeps its ID and history (#116).

## 1.0.34

- New `mqtt_ssl` option: connect over TLS to an external MQTT broker set with `mqtt_host`. It was documented but couldn't be set, because it wasn't in the app's configuration schema. With TLS on and no `mqtt_port`, the app connects to port 8883, the standard MQTT-over-TLS port. Without `mqtt_host` the app keeps using the Mosquitto app's broker without TLS, and the log says `mqtt_ssl` was ignored (#54).
- The unused `mqtt_protocol` option is gone. The app always uses MQTT 5, as it already did (#54).

## 1.0.33

- When the Bluetooth link to the node drops and the add-on is still stuck reconnecting 10 minutes later, it now rebuilds the connection from scratch. That safety net never actually ran before. It was also meant to fire after 60 seconds, which is shorter than the add-on's ordinary reconnect attempts, so it would have interrupted connections that were about to recover (#49).
- Rebuilding the connection no longer cuts the old connection's clean-up short (#49).

## 1.0.32

- If the RV is out of range or powered off when the add-on starts, the add-on now keeps trying to connect and creates no entities until it has really connected. Before, it created a placeholder device named "Disconnected", and the next good start added a second, real device with a duplicate of every entity (#48).
- The add-on deletes that placeholder "Disconnected" device, and its entities, the first time it connects. Your real device and its entities are left alone (#48).

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

## 1.0.11

- A climate zone that reports an unknown fan speed is sent High, the default, and Home Assistant shows High rather than an unknown speed.
- Internal: the shared cloud transport reads Firebase errors and timestamps more carefully. The bridge's own cloud mirror is unchanged.

## 1.0.10

- Status refreshes are answered from one that has just finished instead of starting another, and each Bluetooth service's characteristics are looked up once per connection.

## 1.0.9

- A Bluetooth link that is already up is reused instead of being rebuilt.
- The version banner at start-up is framed so it stands out in the log.

## 1.0.8

- The add-on leaves the node's Bluetooth pairing alone when the host has no key for it.

## 1.0.7

- Concurrent status refreshes are combined into one.
- The add-on logs its version when it starts.

## 1.0.6

- Connecting uses the host's Bluetooth link directly, which is quicker.

## 1.0.5

- A node that is already bonded to the host can be connected to.

## 1.0.4

- The add-on waits for the node to settle after BlueZ rediscovers it, before connecting.
- Status refresh counts come from the cached vehicle profile before the first connection, so the first refresh asks for the right number of zones.

## 1.0.3

- Downloading the vehicle configuration (`app.json`) is quicker: each chunk request now fits in one Bluetooth write instead of three.

## 1.0.2

- More reliable pairing and connection on Linux: a refused bond is recognised, stale Bluetooth state is cleared at most once per run instead of on every connect, and the add-on waits for BlueZ to rediscover the node instead of a fixed delay, saying so if it doesn't reappear.
- A connection that is still reconnecting no longer fails the vehicle's configuration, and the first status refresh asks for at most five climate zones when none are configured.
- After a Bluetooth drop, the add-on reconnects. Before, every attempt failed with an `ObjectDisposedException` until the add-on was restarted (#6).
- A reconnect's first status refresh uses the vehicle's cached climate zone and generator counts.

## 1.0.1

- Water tank levels update as the node reports them.

## 1.0.0

Initial Release
