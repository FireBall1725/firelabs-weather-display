# FireLabs Weather Display

Firmware for a battery-powered e-paper weather display: a LilyGo T-Energy-S3 (ESP32-S3) on an 18650 cell, driving a 4.2-inch GoodDisplay GDEY042T81 panel (400x300, black and white, SSD1683 controller). It wakes every 30 minutes, pulls a weather bundle from Home Assistant, repaints the panel, and drops back into deep sleep.

This device consumes data from Home Assistant rather than reporting to it, which runs the opposite direction from the [S31 plug](https://github.com/FireLabsCA/firelabs-s31-firmware). It replaces the ESPHome config the display used to run, so it now shares the S31's captive-portal setup, web UI, OTA, and GPIO0 button behaviour through [firelabs-core](https://github.com/FireLabsCA/firelabs-core).

The companion Home Assistant integration lives at [FireLabsCA/firelabs-hass](https://github.com/FireLabsCA/firelabs-hass). It owns the entity mapping, so pointing the display at different sensors is a settings change instead of a reflash:

[![Open your Home Assistant instance and open the FireLabs repository inside HACS.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=FireLabsCA&repository=firelabs-hass&category=integration)
[![Open your Home Assistant instance and start setting up FireLabs.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=firelabs)

## What a wake does

One HTTP request per wake covers both directions. The device POSTs its battery voltage, firmware version, and wake reason to a Home Assistant webhook, and the response carries the weather bundle it paints. Reporting telemetry costs no extra awake time, and there is no broker in the path.

The panel draws the current temperature with a condition glyph, a feels-like and high/low column, six stats (humidity, wind, UV, gust, PoP, precipitation) with units taken from the Home Assistant sensors, and a five-slot hourly forecast strip.

## Power

The display is asleep nearly all the time, which is what turns one 18650 into weeks of runtime rather than a day.

- 30 minutes of deep sleep between refreshes, woken on a timer.
- Quiet hours from 21:00 to 06:00. A timer wake inside that window works out the minutes until 06:00 and sleeps that long in one shot, without repainting.
- Battery percentage comes off GPIO3 through a hardware divider, scaled 3.0 V = 0% to 4.2 V = 100%.

## Button

GPIO0 carries two actions, the same split as the S31:

- **Tap:** wakes into a 60-second window for OTA and the web UI, then refreshes and sleeps.
- **Hold about 5 seconds:** factory reset. Clears the config in LittleFS and reboots into the setup AP.

Home Assistant can hold the device awake the same way with the force-wake switch. Its value rides back in the bundle, and the device honours it on the next wake.

## Build

PlatformIO:

```
pio run -e wx
```

The binary lands at `.pio/build/wx/firmware.bin`. Every GitHub release also ships a prebuilt `firelabs-wx-<tag>.bin`.

## First boot

With no saved wifi the display starts an open AP named `FireLabs WX <mac-suffix>`, and a captive portal walks through picking a network. An unconfigured unit stays awake for 30 minutes and then sleeps, so one left in a drawer does not flatten the cell. Once it is on the network but has no webhook URL yet, it holds that same 30-minute window on a config prompt so setup can be finished from the web UI.

See [SPEC.md](SPEC.md) for the full design, including the parts that are not built yet.

## Support

Questions, updates, and works in progress: [FireBall Codes on Discord](https://discord.gg/QpV82CFfVD).

If this saved you some time, you can [buy me a sushi roll](https://ko-fi.com/fireball1725).

## Licence

AGPL-3.0, matching the rest of FireLabs.
