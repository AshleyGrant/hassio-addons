# Changelog

## 1.0.123

- Each log line now names its source with `StonerControl.` instead of `RiverstoneControl.`, for example `info: StonerControl.HomeAssistant.…`, and some of the names after that prefix have changed as well. If you search or filter the add-on's log by these names, update the search. The messages themselves are unchanged.
- Each time the add-on connects to your RV, the log now records at info level which Cloud gateways are on and the `cloud_backend` value in use, for example `Cloud gateways for this connection: Firebase off, Appwrite on (cloud_backend: appwrite).` At debug level, it also records when the add-on finds, saves, replaces or deletes its saved Appwrite session in `/data`. These lines never include the session itself, the account e-mail or the password.
- The other changes in this release are internal and do not change how the add-on behaves.

## 1.0.122

Maintenance release: no change in how the add-on behaves.

## 1.0.121

Maintenance release: no change in how the add-on behaves.

## 1.0.120

Maintenance release: no change in how the add-on behaves.

## 1.0.119

- Home Assistant now allows the add-on 15 seconds to stop, rather than the usual 10, and the add-on finishes closing its connection to the RV within 13 seconds. Previously, in a rare case where the Bluetooth link was slow to close, stopping could take about 12 seconds, and Home Assistant stopped the add-on by force at 10.
- The add-on still marks the RV's gateway as offline on Appwrite before it closes Bluetooth, even when its connection to the MQTT broker is slow to close, as long as that connection closes within about 12 seconds.
- If a step of stopping runs out of time, the log records a warning ending "didn't stop within the add-on's 13s stop budget; abandoning it." A normal stop is unchanged.

## 1.0.118

- When the add-on stops, it now waits up to 2 seconds for connection work in progress (a connection attempt, a status update, or the clean-up after a dropped link) to finish, so that the connection has fully stopped. The wait applies only when that work does not stop immediately. The gateway is still marked as offline in the Cloud first.

## 1.0.117

- If only one of `cloud_email` and `cloud_password` is set, the log now reports it and names the empty option. Previously, the add-on ran without the Cloud and logged nothing about it.

## 1.0.116

Maintenance release: no change in how the add-on behaves.

## 1.0.115

Maintenance release: no change in how the add-on behaves.

## 1.0.114

- If a Bluetooth connection attempt fails in an unexpected way, the log now records a Bluetooth warning immediately. Previously, the failure was logged much later or not at all.

## 1.0.113

- The red `fail:` line about a missing Bluetooth "Appearance" property, logged a few seconds after every start, no longer appears. It came from the Bluetooth library reading a property that some nearby devices lack, and it never affected Bluetooth. That fault is now logged only at debug level, with an explanation. Any other unexpected failure still appears as `fail:`.

## 1.0.112

Maintenance release: no change in how the add-on behaves.

## 1.0.111

Maintenance release: no change in how the add-on behaves.

## 1.0.110

Maintenance release: no change in how the add-on behaves.

## 1.0.109

Maintenance release: no change in how the add-on behaves.

## 1.0.108

Maintenance release: no change in how the add-on behaves.

## 1.0.107

Maintenance release: no change in how the add-on behaves.

## 1.0.106

Maintenance release: no change in how the add-on behaves.

## 1.0.105

Maintenance release: no change in how the add-on behaves.

## 1.0.104

Maintenance release: no change in how the add-on behaves.

## 1.0.103

Maintenance release: no change in how the add-on behaves.

## 1.0.102

Maintenance release: no change in how the add-on behaves.

## 1.0.101

Maintenance release: no change in how the add-on behaves.

## 1.0.100

Maintenance release: no change in how the add-on behaves.

## 1.0.99

- The remaining Bluetooth lines in the log no longer carry `[DBG]` and `[WRN]` tags, since each line already shows its level. For example, `[BLE] [WRN] PairAsync threw: …` now reads `[BLE] PairAsync threw: …`. Each message keeps its wording and its level.

## 1.0.98

Maintenance release: no change in how the add-on behaves.

## 1.0.97

- A few Appwrite lines in the log show a different source name in front of the message. Their wording and levels are unchanged.

## 1.0.96

Maintenance release: no change in how the add-on behaves.

## 1.0.95

- Many Bluetooth lines in the log no longer carry `[DBG]`, `[INF]`, `[WRN]` and `[ERR]` tags, since each line already shows its level. For example, `[BLE] [INF] LinuxPlatformBle: using Bluetooth adapter /org/bluez/hci0.` now reads `[BLE] LinuxPlatformBle: using Bluetooth adapter /org/bluez/hci0.` Each message keeps its wording and its level. The remaining lines follow in 1.0.99.

## 1.0.94

Maintenance release: no change in how the add-on behaves.

## 1.0.93

Maintenance release: no change in how the add-on behaves.

## 1.0.92

Maintenance release: no change in how the add-on behaves.

## 1.0.91

- 1.0.90 logged `A background task failed and nothing observed it`, with a long stack trace, several times a minute. The cause is harmless and predates 1.0.90: the Bluetooth library reads an "Appearance" property from every nearby Bluetooth device, and many devices lack one. Each kind of failure is now logged as an error once, and any repeat only at debug level. About one error line about "Appearance" is still expected shortly after each start.

## 1.0.90

- Background work, such as a Cloud update or a retry, now logs a warning if it fails, for example `Background work failed: …`. Previously, such a failure could go unrecorded. A failure that nothing else handles is logged once as an error: `A background task failed and nothing observed it`. Neither message is expected in normal use; each names what failed.

## 1.0.89

- On BMPro's newer cloud, Appwrite, changing `cloud_email` to another account now signs out the old account's session before the add-on signs in to the new one. Previously, the add-on continued as the old account, and that session occupied one of the old account's ten until it expired. If `cloud_email` was changed on an earlier version, the first start after this update signs out the old session and signs in to the account in the options.
- Appwrite keeps at most ten sessions for an account and signs out the oldest when an eleventh is created, which may be one of your other devices. After signing in, the add-on now checks how many sessions the account has, and logs a warning at 8 or more, for example `This BMPro account has 9 of Appwrite's 10 sessions.` If you see this warning, sign out on devices you no longer use.
- A restart, a stop or a password change still keeps a session that Appwrite accepts, as before.

## 1.0.88

- On Appwrite, a command sent while the add-on was starting, or while its live connection to the Cloud was briefly down, now runs once the connection returns. Previously, such a command was missed. The log records `Catching up Cloud command …` for each.
- On Appwrite, a command may now be up to 60 seconds old when it runs, where the limit was 10 seconds, and only the newest command for each light or device runs. For example, if a light is switched on and then off while the add-on is away, the add-on only switches it off, and logs `Skipping Cloud command …` for the other. A later All Lights command also replaces an earlier command for a single light. Firebase keeps its 10-second limit.
- The add-on remembers which Cloud commands it ran for two minutes, across reconnects and restarts, so no command runs twice.

## 1.0.87

Maintenance release: no change in how the add-on behaves.

## 1.0.86

Maintenance release: no change in how the add-on behaves.

## 1.0.85

- **Action needed:** move your BMPro account from `firebase_username` and `firebase_password` to the new `cloud_email` and `cloud_password` options, since the same account signs in to both of BMPro's clouds. On the add-on's Configuration tab, turn on **Show unused optional configuration options** to show the new options, copy the e-mail address and password into them, clear the old ones, and save.
- `firebase_username` and `firebase_password` are deprecated but still work. Until they are moved, the log shows a reminder at each start, naming the old options it still reads.

## 1.0.84

- On Appwrite, after a quiet period, the add-on now answers an app's first status check immediately, rather than within the next second, so the RV appears live sooner.

## 1.0.83

- On Appwrite, a change made on the RV now reaches the Cloud within about a second, even when no app has been watching. Previously, the add-on's quiet mode could delay it by up to 20 seconds, so the Cloud could briefly show the old state. This applies to lights and dimming, the water pump and heaters, a climate zone's mode, fan, heat source or setpoint, and the generator starting or stopping, whether the change comes from the panel, Home Assistant or another device on Bluetooth. Readings that change on their own, such as voltages, temperatures and tank levels, still follow the quiet cycle.
- The log's `Appwrite publish loop alive` line now also counts those changes, as `N change(s) seen on the RV`.

## 1.0.82

- Four new diagnostic sensors on the RV device show the versions contained in the control node's firmware string, which Home Assistant shows in full as the device's firmware: **Controller Firmware**, **Companion Firmware**, **Panel (RVMC) Version** and **BLE Protocol Version**. For `1.1.0E3.0C2.1P4.5.0` they show `1.1.0`, `3.0`, `2.1` and `4.5.0`. A version the node does not report shows as unknown.
- The RV device in Home Assistant now shows the control node's serial number and hardware version, which were previously blank.

## 1.0.81

- The add-on now publishes the control node's serial number, firmware versions and hardware revision to Appwrite, where they were previously blank, along with the RV's own serial number if the RV reports one.

## 1.0.80

- Commands relayed from the Cloud now show in the log which cloud they came from, as in `Appwrite OnOffCommand relayed: output 23 -> ON`. Previously, every one said `Firebase`, even with `cloud_backend: appwrite`.

## 1.0.79

- With `log_level: debug`, each Appwrite request that BMPro's cloud limits now also shows how many such requests remain and when the limit resets, as in `…: 200 in 84 ms, rate limit 118 of 120 left, resets in 42 s`. Nothing changes at `info`.

## 1.0.78

- On Appwrite, the log now shows the add-on's activity. Once a minute it writes `Appwrite publish loop alive: …`, with when it last wrote, how many writes and failures it had, whether it is active or dormant, and how many commands and pulses it handled. With `log_level: debug` it also logs every request it makes to Appwrite and every live update it receives. Previously, it logged nothing unless something failed.

## 1.0.77

- New `cloud_backend` value, `none`: turns BMPro's cloud off, even when a BMPro account is set. The add-on then signs in to neither cloud, publishes nothing to them and relays no commands from BMPRO Connect. Bluetooth control and Home Assistant work as before.
- With `none`, the **Last Firebase Publish** and **Last Successful Firebase Publish** sensors are removed when the RV next connects. Choosing a cloud again restores them. See the Documentation tab.

## 1.0.76

- **Action needed** if BMPro has not yet moved your BMPro account to Appwrite: set `cloud_backend` to `firebase` or `both`. With the new default alone, the RV is then on neither cloud. If you still need the RV on Firebase after your account has moved, set `cloud_backend` to `both`.
- `cloud_backend` now defaults to `appwrite`, BMPro's newer cloud, where BMPRO Connect looks for the RV. Where no value was chosen, the add-on becomes the RV's gateway on Appwrite after this update and stops publishing to Firebase. A saved `cloud_backend` value is kept. See the Documentation tab.

## 1.0.75

Maintenance release: no change in how the add-on behaves.

## 1.0.74

Maintenance release: no change in how the add-on behaves.

## 1.0.73

Maintenance release: no change in how the add-on behaves.

## 1.0.72

- New `cloud_backend` option: chooses which of BMPro's clouds the add-on is the RV's gateway on, `firebase`, `appwrite` or `both`. It defaults to `firebase`, so nothing changes until another value is chosen.
- With `appwrite` or `both`, the add-on moves the BMPro account to Appwrite the way BMPRO Connect does, with the same e-mail address and password, and keeps the session across restarts. If BMPro has not yet moved the account, the log says so and the add-on checks again every 30 minutes; keep `firebase` or `both` until then. See the Documentation tab.

## 1.0.71

Maintenance release: no change in how the add-on behaves.

## 1.0.70

Maintenance release: no change in how the add-on behaves.

## 1.0.69

- A faulted sensor is now published to the Cloud as a fault. Previously, the Cloud kept a faulted tank's last level, or never received one, and kept a faulted outside temperature's last value.
- Fixed a failed Bluetooth subscription never being retried: the next attempt treated it as already subscribed.

## 1.0.68

- Fixed faulted sensors appearing as real readings. An open temperature sensor read 128.0 °C and a faulted tank sender read over 100 %, and Home Assistant recorded those values in its history. A faulted sensor now reads as unknown, so any faulted sensor changes to unknown after this update.
- Fixed the RV's device in Home Assistant being named "Disconnected", with firmware "unknown", after a Bluetooth drop during the first connection, until the add-on restarted. A later connection now corrects the name and firmware. Entity IDs do not change.

## 1.0.67

- Fixed a dimming level or setpoint being lost when the Bluetooth command before it failed. The newer value is now sent, and the 10-second status re-read continues.

## 1.0.66

- Fixed Cloud commands sent together being dropped. Turning a zone on and setting its mode and temperature arrive at the same time, and the add-on could keep the temperature and discard the power and mode commands. Commands now run in the order they were sent, and a newer command replaces an older one only if it is of the same kind for the same light or zone, so a slider still ends on its last value.

## 1.0.65

- Fixed the add-on connecting to any nearby BMPro node rather than the RV's own. It searched by name alone, so a second RV in range could answer first and receive Home Assistant's commands. The node in `ble_device_id`, or the one the add-on remembers, is now the only node it connects to.
- `ble_device_id` now pins the add-on to that node: if the node is not found, the add-on keeps retrying rather than use another. If you replace the control node, set `ble_device_id` to the new node's Bluetooth address. A blank `ble_device_id` means no pin, and with no node remembered, the first connection finds one by name as before.

## 1.0.64

- Fixed the **All Lights** switch, which only reflected the last command sent to it. It now reads on only when every light is on, whether the lights were changed from Home Assistant, a wall switch or elsewhere, and it is restored correctly after the MQTT broker restarts.
- The RV's description in the Cloud now includes its temperature sensors and whether its generator supports auto start (AGS).

## 1.0.63

- Fixed false warnings when the add-on stops while reading the node's serial number or firmware version, or while downloading the RV's configuration. It could log a false "no serial number" message or report the firmware as unknown while stopping. It now stops without them. A read that fails while the add-on is running still falls back as before.

## 1.0.62

Maintenance release: no change in how the add-on behaves.

## 1.0.61

- Fixed dimming at 30% and 70%. The add-on sent one step less than BMPRO Connect at those two levels. The difference is not visible on a light, but the commands now match BMPRO Connect's exactly.

## 1.0.60

Maintenance release: no change in how the add-on behaves.

## 1.0.59

- Fixed a Firebase stall that was not always counted. When Cloud updates stop getting through, the add-on restarts its Firebase session, and after five stalls in a row it also rebuilds the Bluetooth connection. A stall that cleared just as the add-on intervened was treated as an ordinary failure, with an extra sign-in, and did not count toward the five. It now always counts.
- Fixed the add-on giving up on a Bluetooth connection that dropped just after connecting, while a reconnect was about to start. It logged "The vehicle connection ended Faulted instead of connecting", waited 15 seconds and rebuilt the whole connection. It now waits for that reconnect.
- A connection attempt that was stopped and then failed late is no longer logged as having "exhausted all retry attempts". That warning now means only that the retries ran out.

## 1.0.58

Maintenance release: no change in how the add-on behaves.

## 1.0.57

Maintenance release: no change in how the add-on behaves.

## 1.0.56

Maintenance release: no change in how the add-on behaves.

## 1.0.55

- The add-on now re-reads every status from the RV every 10 seconds while connected, in addition to reporting the changes the node sends. A missed update now corrects itself within 10 seconds, in Home Assistant and in the Cloud. Commands still take priority.
- Fixed the Cloud showing lights as off when they were on. After a reconnect, the add-on could miss the light states the RV sent first, and reported those lights as off in the Cloud until they changed.

## 1.0.54

- Closing a connection now completes even if one of its steps fails, rather than stopping at the first failure. This is not noticeable in normal use.

## 1.0.53

- Bluetooth problems that were previously not recorded are now logged at debug level, with the reason, so they appear only with `log_level: debug`. They include a failed link check (with its count, "1/2"), a part of the node that could not be read while connecting or downloading the RV's configuration, and a node detail (such as the RV's serial number, maker or model) that could not be read.
- A connection that fails to close properly now logs a warning.

## 1.0.52

- With Firebase on, when the Cloud rejects a state update because it is busy (HTTP 429 or 503), the add-on now waits before the next one: 2 seconds, doubling each time the Cloud refuses again, up to 10 seconds. Previously, it tried again a second later, which the Cloud's response asks clients not to do. Each refusal logs a warning with the wait.
- Any other failed update, such as a network error or a timeout, is now sent again about a second later. Previously, if nothing on the RV changed, it reached the Cloud only up to 10 seconds later, despite the log's "will retry on next cycle".

## 1.0.51

- With Firebase on, the add-on no longer misses the RV's state when the first status arrives just as the Cloud connection starts. Previously, if the RV then sent nothing new, the add-on published nothing to the Cloud for that connection, so the RV appeared offline in the Cloud until something on the RV changed.

## 1.0.50

- **Action needed** if your RV's configuration leaves a light unnamed: its entity is now named "Light N", matching its name in the Cloud, instead of "Output N". Home Assistant keeps the old entity, which stops responding, and ignores the new one. To fix this, delete the add-on's RV device in Home Assistant's MQTT integration, then restart the add-on so that the device is discovered again. Customisations of its entities are lost.
- The add-on now reads the RV's configuration with stricter rules. A typical configuration gives the same entities as before. An unusual one may read differently, and one that repeats a setting is now refused: the add-on connects but creates no entities for it.
- With Firebase on, the RV's description in the Cloud is now built with the same rules. It is unchanged for a typical configuration. In an unusual one, a battery or voltage source with no type now appears in the Cloud as unknown, as it does over Bluetooth, and a motor's RV-C setting is now sent correctly, where it was always sent as off.
- Fixed a Bluetooth connection that dropped just as it connected being kept as the link, so that the add-on reported connected while every command failed, until it restarted or **Force BLE Reconnect** was pressed. It now counts as a failed attempt and is retried.

## 1.0.49

Maintenance release: no change in how the add-on behaves.

## 1.0.48

- Stopping, updating or restarting the add-on no longer logs a warning that watching for Bluetooth service restarts failed. The update to 1.0.48 itself still shows it once, because Home Assistant stops the previous version to update it.

## 1.0.47

- With Firebase on, when the Cloud is limiting sign-ins (too many attempts, or a quota used up), the add-on now waits for the limit to pass. It logs a warning and makes no further attempt for 5 minutes, doubling each time the Cloud refuses again up to 30 minutes, and then resumes on its own. Previously, it tried again after a few seconds and on every new Bluetooth connection, and each attempt extended the lockout.
- A Cloud refusal of the e-mail address or password is now recognised even when the Cloud adds an explanation to it. Previously, such a refusal was retried instead of turning the Cloud off.

## 1.0.46

- The add-on is now released under the MIT licence. Its documentation uses the add-on's name and links to the repository it is installed from.

## 1.0.45

- With Firebase on, the Cloud now records the time on the state the add-on writes, and the age of a remote command is judged by the Cloud's clock. Previously, a Home Assistant clock more than 30 seconds out made the RV appear offline in the Cloud, and made the add-on drop valid commands or run stale ones.
- Remote commands more than 10 seconds old are now discarded, as BMPro's own gateway does. Previously, the limit was 30 seconds.

## 1.0.44

- With Firebase on, a publish the Cloud refuses now counts as failed. Previously, it was treated as published, so **Last Successful Firebase Publish** kept advancing while the Cloud held stale state.
- When the Cloud rejects the add-on's sign-in, the add-on now renews it immediately, instead of continuing with it for up to 55 minutes. After five refused updates in a row, it signs in again, and pauses remote commands until it has.

## 1.0.43

- With Firebase on, a wrong e-mail address or password is now reported once, with the reason, and the Cloud stays off until the add-on restarts. The same applies if the password changes or the account is disabled while the add-on runs. Previously, the add-on retried the sign-in every few minutes, which could get the account locked.

## 1.0.42

Maintenance release: no change in how the add-on behaves.

## 1.0.41

- When the Bluetooth link drops, dimming and setpoint changes still waiting to be sent are discarded immediately, instead of each failing with a *worker failed* warning.
- A dropped connection is now logged under the RV's name, and a failed connection attempt while the RV is out of reach is no longer also logged as "Connection to Disconnected lost."

## 1.0.40

- With Firebase on, the add-on now stops cleanly. Previously, stopping, restarting or updating it hung until Home Assistant stopped it by force after 10 seconds, and the add-on's page then showed an error.
- With Firebase on, the add-on now reconnects to the RV after the Bluetooth link drops, or after **Force BLE Reconnect**. Previously, the same hang could prevent it from reconnecting until it was restarted.
- Each disconnect now logs how long it took.

## 1.0.39

- With Firebase on, the add-on no longer logs a *PermissionDenied* warning every time it starts. It had been listening for remote commands in a second location that Firebase always refused. A listener that keeps failing now waits longer between restarts, up to a minute.

## 1.0.38

- Home Assistant now shows the add-on's entities as unavailable, rather than their last values, while the add-on is stopped or has lost the MQTT broker. The RV's entities are also unavailable while the Bluetooth link is down or reconnecting.
- **BLE Connected**, the Firebase publish sensors, the vehicle configuration entities and the two Force buttons stay available, so **Force BLE Reconnect** can still be pressed while the RV is out of reach.

## 1.0.37

- With Firebase on, a Bluetooth outage no longer counts as a stalled Cloud update. Previously, every outage longer than 45 seconds caused a new Firebase sign-in, about every 50 seconds, with misleading "stall" warnings.
- A real stall now keeps the add-on signed in. Five stalls in a row rebuild the Bluetooth connection, as intended; previously, the count never passed one.

## 1.0.36

- The add-on now connects to the MQTT broker before the RV. While the RV is out of reach, **BLE Connected** now shows Off within a few seconds of start-up, instead of its value from when the add-on last stopped.
- If no MQTT broker can be found yet, for example while the Mosquitto broker is still starting, the add-on logs the reason and keeps looking. Previously, it stopped and had to be restarted.

## 1.0.35

- **Slideout Lockout** now shows On while the slide-outs are locked out (on a trailer, while it is hitched for towing) and Off otherwise, with a trailer icon. Previously, it read backwards, showing Unlocked while locked out. The entity keeps its ID and history.

## 1.0.34

- New `mqtt_ssl` option: connects over TLS to an external MQTT broker set with `mqtt_host`. With TLS on and no `mqtt_port`, the add-on connects to port 8883. Without `mqtt_host`, the add-on continues to use the Mosquitto broker without TLS, and the log notes that `mqtt_ssl` was ignored.
- The `mqtt_protocol` option, which had no effect, is removed. The add-on always uses MQTT 5, as it already did.

## 1.0.33

- When the Bluetooth link drops and the add-on is still reconnecting 10 minutes later, it now rebuilds the connection from scratch. Previously, this safeguard never ran.

## 1.0.32

- If the RV is out of range or switched off when the add-on starts, the add-on now keeps trying to connect and creates no entities until it has connected. Previously, it created a placeholder device named "Disconnected", and the next successful start added a second, real device with a duplicate of every entity.
- The add-on deletes that placeholder "Disconnected" device, and its entities, the first time it connects. The real device and its entities are not affected.

## 1.0.31

- The add-on's image now shows its source and its licence (MIT) on its GitHub Container Registry page. The images are public, so pulling them needs no login.
- The Documentation tab explains where the image comes from and how to build it.

## 1.0.30

- Fixed a fresh install failing to start with "Missing required option 'firebase_username'". `firebase_username` and `firebase_password` are now optional, with no default, and appear on the Configuration tab when **Show unused optional configuration options** is on. Existing installs keep their settings.

## 1.0.29

- On newer BMPro control nodes, downloading the RV's configuration can no longer be disrupted by a stray Bluetooth message that dropped or corrupted part of it. The RVMN101C is not affected.

## 1.0.28

Maintenance release: no change in how the add-on behaves.

## 1.0.27

- New **Heat Source** select for each climate zone that has both a heat pump and a furnace: chooses Furnace, Heat Pump or Both for Heat mode. As in BMPRO Connect, a choice takes effect only while the zone is in Heat. Zones with one heat source do not have one.

## 1.0.26

- A climate zone whose node reports heat source 0 is now treated as a heat pump, as BMPRO Connect does. Previously, it was treated as an unknown heat source, which the Cloud showed as a furnace.
- The Cloud no longer receives placeholder values as a zone's real state. A zone with no reading yet shows fan speed high, and an unknown heat source is left out.

## 1.0.25

- New **Generator State** sensor: Running, Stopped, or Starting/Stopping while the auto-gen smart start is starting or stopping the generator.
- New **Generator Start Output Fault** and **Generator Stop Output Fault** problem sensors for a gas generator. They report a fault on the line that drives the generator's start or stop input, which BMPRO Connect shows as a marker on its Start and Stop buttons.

## 1.0.24

- The Cloud now receives the generator's start and stop faults separately from its general fault, and whether the auto-gen smart start is starting or stopping it. BMPRO Connect, reading the RV through the Cloud, can then show its fault markers and its Starting/Stopping state. Home Assistant shows these from 1.0.25.

## 1.0.23

- Downloading the RV's configuration from the control node is more reliable. A late reply to a retried request can no longer corrupt it, and the configuration from a node that does not report its size is no longer cut short.

## 1.0.22

- Stopping the add-on, or rebuilding the connection, while Bluetooth is being set up now ends the setup after the step in progress. Previously, it continued through every remaining step, and could start a full Bluetooth scan. A deliberate stop is no longer logged as a Bluetooth failure.
- Stopping the Bluetooth connection while a request is still in progress is now safe.

## 1.0.21

- Stopping the add-on, or pressing **Force BLE Reconnect**, while it is connecting to the control node now takes effect immediately. A connection attempt that hangs now gives up after a time limit instead of blocking the next attempt.
- A cancelled connection attempt is no longer reported as a pairing failure with a request to put the node into pairing mode.

## 1.0.20

- Stopping the add-on, or pressing **Force BLE Reconnect**, now disconnects from the control node, so BMPRO Connect can connect over Bluetooth immediately. Previously, the node stayed connected to the Home Assistant host until the host's Bluetooth timed the link out.
- A Bluetooth disconnect that fails, for example just after the host's Bluetooth service restarts, is now logged instead of crashing the add-on.

## 1.0.19

- The add-on can now find a control node the host's Bluetooth has never seen, so a fresh Home Assistant install pairs without first running `scan le` in `bluetoothctl`. Previously, it could only find a node the host already knew.
- If the node cannot be found, the log names the Bluetooth adapter that was searched.
- Bluetooth scan warnings, such as an adapter being switched off, now appear at the default `info` log level, not only at `debug`.
- A remembered node no longer prevents the scan when the host has forgotten it, for example after a backup is restored onto new hardware.

## 1.0.18

- The add-on now works with a Bluetooth adapter not named `hci0`, such as a USB dongle the host numbers `hci1`. Previously, pairing and connecting failed on such hosts. With more than one adapter, the add-on picks one, not necessarily `hci0`, and logs which: `using Bluetooth adapter /org/bluez/hciN`.

## 1.0.17

Maintenance release: no change in how the add-on behaves.

## 1.0.16

Maintenance release: no change in how the add-on behaves.

## 1.0.15

- Home Assistant entities now keep updating after the connection to the RV is rebuilt, including after **Force BLE Reconnect**. Previously, any rebuild stopped updates without warning until the add-on was restarted.
- The add-on keeps retrying the MQTT broker until it returns, so a Mosquitto restart or a short network outage no longer leaves it offline. Once the broker is back, the add-on finishes setting up any entities that were interrupted and re-sends the RV's current state.
- The add-on now publishes `offline` to its MQTT availability topic when it stops.

## 1.0.14

- A Bluetooth command that fails because the link is gone now starts a reconnect immediately, instead of waiting for the next link check.
- The Bluetooth link is now checked every 15 seconds, where it was 30, so **BLE Connected** reflects a silent drop within about half a minute.
- The add-on now recovers on its own after the host's Bluetooth service restarts, for example during a Home Assistant OS update. Previously, it needed a restart.

## 1.0.13

Maintenance release: no change in how the add-on behaves.

## 1.0.12

Maintenance release: no change in how the add-on behaves.

## 1.0.11

- A climate zone that reports an unknown fan speed is now sent High, the default, and Home Assistant shows High rather than an unknown speed.

## 1.0.10

- Connecting is faster: the add-on looks up each of the node's Bluetooth features once per connection, and no longer reads every status twice at start-up.

## 1.0.9

- Connecting is faster when the host's Bluetooth link to the node is already up: the add-on uses it instead of resetting it.
- The version banner at start-up is framed to stand out in the log.

## 1.0.8

- The add-on no longer resets the node's pairing on the host when there is nothing to reset, which caused aborted connection attempts and retries.

## 1.0.7

- Status reads requested at the same time are combined into one.
- The add-on logs its version when it starts.

## 1.0.6

- Connecting is faster.

## 1.0.5

- The add-on can now connect to a node the host is already paired with.

## 1.0.4

- The add-on waits for the node to settle after the host's Bluetooth finds it again, before connecting.
- The first status read requests the correct number of climate zones, using the RV's saved configuration.

## 1.0.3

- Downloading the RV's configuration from the node is faster.

## 1.0.2

- Pairing and connecting are more reliable: a refused pairing is recognised, and the add-on waits for the host's Bluetooth to find the node again instead of waiting a fixed time, and logs it if the node does not reappear.
- After a Bluetooth drop, the add-on now reconnects. Previously, every attempt failed until the add-on was restarted.
- A connection that is still reconnecting no longer prevents reading the RV's configuration.

## 1.0.1

- Water tank levels now update as the node reports them.

## 1.0.0

Initial release.
