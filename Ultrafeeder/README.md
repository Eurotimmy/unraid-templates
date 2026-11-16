# Ultrafeeder (ADS-B) for Unraid

Unraid Community Applications template for [sdr-enthusiasts/docker-adsb-ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder). It's an all-in-one ADS-B container with readsb, a tar1090 map, graphs1090 and multi-feeder/MLAT support.

This is a community template. It is not affiliated with SDR-Enthusiasts. Container questions are best asked on the [SDR-Enthusiasts project](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder); template questions go to the support link in the template.

## Quick start

1. Plug your RTL-SDR dongle into the Unraid server. Stop any other container that uses it, such as dump1090 or fr24feed-piaware, because only one container can open a dongle at a time.
2. Install **Ultrafeeder** from Community Applications.
3. Fill in **Receiver latitude**, **Receiver longitude** and **Receiver altitude** (for example `18m`).
4. Start it and open the WebUI (default `http://SERVER:8080`). Aircraft should appear within a minute. Graphs are at `/graphs1090/`.

## Powering an LNA with the bias-tee

Bias-tee powered LNAs and filters, such as the RTL-SDR Blog ADS-B LNA or a SAWbird, need DC power up the coax from the dongle (RTL-SDR Blog V3/V4).

Set both:

| Setting | Value |
|---|---|
| Enable Bias-Tee (`READSB_ENABLE_BIASTEE`) | `true` |
| Extra readsb Arguments (`READSB_EXTRA_ARGS`) | `--enable-biastee` |

Why both? Some decoders switch the bias-tee on before starting, then reset it when they open the dongle. Your LNA ends up unpowered, and an unpowered LNA *blocks* most of the signal. Passing `--enable-biastee` to readsb makes the decoder itself hold the bias-tee on.

**How to tell if your LNA is unpowered:** few aircraft, and a noise floor (in `/data/stats.json` or graphs1090) that barely moves when you change the gain. You can confirm with a multimeter: with the coax disconnected and the container running, expect about 4.5 V DC between the dongle's SMA centre pin and its outer threads.

## Using Ultrafeeder as the receiver for other feeders

If you already run an all-in-one feeder such as `thomx/fr24feed-piaware` for FR24 and FlightAware, let Ultrafeeder own the dongle and point the other container at Ultrafeeder's Beast output.

In fr24feed-piaware set:

```
SERVICE_ENABLE_BIAST=false
DUMP1090_ADDITIONAL_ARGS=--device-type none
SERVICE_ENABLE_IMPORT_OVER_NETCAT=true
DUMP1090_LOCAL_PORT=30004
DUMP1090_REMOTE_HOST=<your Unraid IP>
DUMP1090_REMOTE_PORT=30005
```

That container keeps all its feeds (FR24, FlightAware, and the rest) but no longer touches the dongle.

## Gain

`auto` (default) uses readsb's built-in auto-gain, which SDR-Enthusiasts recommend. If you prefer a fixed gain, step it up until you see a small percentage of "strong signals" (above −3 dBFS) during busy traffic, then back off one step.

## Feeding aggregators

Add feeds with `ULTRAFEEDER_CONFIG` and set a `UUID`. See the [project README](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder#configuring-the-ultrafeeder-container) for the full list. For example:

```
adsb,feed.adsb.fi,30004,beast_reduce_plus_out;mlat,feed.adsb.fi,31090
```
