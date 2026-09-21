# TODO

Open issues found while writing the Bayou section of `index.html`.

## Demo feed keys are effectively public

The receiver firmware has a demo Bayou feed's keys built in
(`BAYOU_DEFAULT_PUBLIC_KEY` / `BAYOU_DEFAULT_PRIVATE_KEY` in
`MeshCore-simple-sensor/examples/v3-ultrasonic/companion_sensor_receiver/main.cpp`).
The private key is also printed on the micro-config setup page
(`simple-sensor.json`, receiver notes). Anyone can write to or clear that feed.

- [ ] Decide whether to keep shipping default keys, or ship none so the gateway
      waits until the builder sets their own
- [ ] Remove the private key from the micro-config page copy
- [ ] Consider rotating the demo feed's keys

## `source` field is dropped by Bayou

The gateway sends the sender's name as `"source"` in its POST body, but Bayou
silently ignores field names it doesn't recognise, so the name is never stored.

- [ ] Either stop sending it, fold it into `log` (e.g. `source=Sensor path=... route=...`),
      or add a `source` column to Bayou (schema + `postNewMeasurement` INSERT +
      `restructureJSON`)

## Readings are lost while WiFi or Bayou is down

The gateway holds only the most recent reading and retries it every 15 s
(`serviceBayou()`); a newer reading replaces an unsent one. Bayou also assigns
timestamps server-side and won't accept client timestamps, so missed readings
can't be backfilled later.

- [ ] Queue unsent readings on the gateway (RAM ring buffer, or flash for power cuts)
- [ ] Add optional client timestamps to Bayou's `POST /data/:feed_pubkey/` so queued
      readings keep their real measurement time
