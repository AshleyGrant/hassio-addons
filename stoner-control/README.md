# Stoner Control RV Controller Bridge

[![App](https://img.shields.io/badge/Home%20Assistant-App-41BDF5?logo=home-assistant&logoColor=white)](https://github.com/AshleyGrant/hassio-addons)

Home Assistant app that bridges a compatible [BMPro][bmpro] control node to
Home Assistant via MQTT discovery. Once paired, every light, climate zone,
slide/awning, water tank, voltage source, temperature sensor, water pump,
line heater, generator, inverter, shore power sensor, and diagnostic entity
the node knows about appears as native HA entities — no manual YAML required.

Optional Firebase credentials let the app mirror live BLE state back to the
BMPro cloud and relay fresh BMPro mobile-app commands to the RV over Bluetooth.
Firebase is not used as the source of truth for current state.

See [`DOCS.md`](DOCS.md) for full setup, pairing, and configuration details.
[`CHANGELOG.md`](CHANGELOG.md) tracks per-release notes.

## How to install

1. In Home Assistant: *Settings → Apps → App Store → ⋮ → Repositories*
   and add `https://github.com/AshleyGrant/hassio-addons`.
2. Install the **Stoner Control RV Controller Bridge** app from the new repository.
3. Make sure the official **Mosquitto broker** app is installed and running.
4. Put the BMPro node into pairing mode from the wall display, then start the
   app. The bridge will pair, authenticate, and populate HA entities.

[bmpro]: https://www.bmpro.com.au/
