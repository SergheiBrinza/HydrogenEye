# HydrogenEye

A portable 1420 MHz hydrogen line receiver built from a 28-euro RTL-SDR dongle. Detects the 21-centimeter emission line of neutral hydrogen across the galactic plane. Runs on a Raspberry Pi Zero W with an Android companion app for real-time spectrum and control over Wi-Fi.

![Hydrogen line detection](images/hydrogen-line-detection.png)

## Why the 21 cm line matters

Neutral hydrogen atoms have a tiny quantum transition where the electron flips its spin relative to the proton. The energy difference is minuscule: 5.9 microelectronvolts. A photon emitted in this transition has a wavelength of 21 centimeters and a frequency of 1,420,405,751.7667 Hz, known as the hydrogen line or H I line.

It's the most fundamental signal in radio astronomy. It lets you map the distribution of hydrogen across the Milky Way, measure the rotation curve of the galaxy, see structures that are invisible in the optical spectrum. This isn't esoteric, even a small antenna can pick it up, because hydrogen clouds in the galactic plane are massive and emit continuously.

For under 100 euros you can be listening to galactic hydrogen from your backyard.

## Hardware

| Component | Price (2023) |
|---|---|
| RTL-SDR Blog V.3 (R820T2 + TCXO) | 28 EUR |
| Raspberry Pi Zero W | 15 EUR |
| SMA cable + adapters | 7 EUR |
| Power bank 10000 mAh | 12 EUR |
| **Total** | **~62 EUR** |

The RTL-SDR Blog V.3 is the critical piece. It has a 0.5 ppm TCXO (temperature compensated crystal oscillator) which is what makes hydrogen line work feasible on this hardware class. A regular RTL-SDR with a cheap crystal drifts too much to lock onto a 1420 MHz narrow emission feature.

![RTL-SDR V.3 Dongle](images/rtl-sdr-dongle.png)

The antenna is not in this BOM. I built a separate 1420 MHz feed horn for capture, see the [1420MHz-Feed-Horn](https://github.com/SergheiBrinza/1420MHz-Feed-Horn) repository for the waveguide design.

## Signal chain

```
Galaxy (H I emission at 1420.406 MHz)
    v
Feed horn antenna (see 1420MHz-Feed-Horn repo)
    v
SMA cable
    v
RTL-SDR Blog V.3 dongle (0.5 ppm TCXO, R820T2 tuner)
    v
USB port on Raspberry Pi Zero W
    v
Python pipeline (pyrtlsdr, NumPy FFT, integration buffer)
    v
Flask HTTP API + WebSocket streaming
    v
Wi-Fi
    v
Android companion app (Kotlin, real-time spectrum view)
```

The Pi Zero does all the heavy lifting: 2.4 MHz of bandwidth around 1420 MHz, FFT at ~2048 bins, power spectral density averaging, drift-scan buffer, calibration math. The phone is just a display and control panel.

## Features

**Real-time spectrum view.** Waterfall and FFT at the 1420 MHz band, streamed to the Android app with roughly 10 frames per second latency over local Wi-Fi.

**Automatic H I peak detection.** The hydrogen peak sits in a ~1 MHz wide feature around 1420.406 MHz. The detector watches for it rising above the noise floor by a user-set threshold and pushes an alert.

**Drift scan mode.** Because the earth rotates, any fixed antenna sweeps through the sky over 24 hours. Drift scan records the spectrum continuously and builds a strip chart of galactic H I intensity over right ascension. Simple, no motor, no tracking, very effective for galactic structure studies.

**Signal integration.** Long-exposure stacking up to 60 minutes to pull weak emission features out of thermal noise. Variance of the averaged spectrum decreases as 1/sqrt(N) where N is the number of independent samples. At 2.4 MHz bandwidth with 1 Hz resolution this gives roughly 40 dB improvement over a single snapshot in a one-hour integration.

**GPS-tagged recordings.** Every observation is stamped with coordinates and UTC. Galactic coordinates (l, b) are derived from RA/Dec and observation time, so you can map your data to the galactic plane directly.

**One-tap calibration.** Reference the system against known bright radio sources like Cassiopeia A (3C461) or Cygnus A (3C405) when they pass through the beam. The software fits the expected flux versus what's measured and computes a calibration factor.

**CSV and FITS export.** For anyone wanting to analyze the data elsewhere, the app exports raw spectral data in CSV for Excel or pandas, or in FITS for professional astronomy tools.

**Night-vision UI.** Deep red text on black background, zero eye strain under a dark sky. Also looks kind of sinister in a good way.

## Software stack

| Layer | Technology |
|---|---|
| Signal I/O | pyrtlsdr (Python binding for librtlsdr) |
| FFT and analysis | NumPy, SciPy |
| Integration and calibration | Custom Python pipeline |
| API server | Flask (HTTP) + WebSocket for streaming |
| Mobile app | Kotlin (Android native) |
| Data formats | CSV (raw), FITS (for astronomy tools) |

Pi Zero W is underpowered by modern standards (single-core 1 GHz ARM11, 512 MB RAM), so the pipeline is tuned: no per-sample Python loops, all heavy lifting in vectorized NumPy, integration buffer kept as a ring buffer in float32.

## Best detection

My cleanest capture:

| Measurement | Value |
|---|---|
| Center frequency | 1,420,405,751 Hz |
| Integration time | 30 seconds |
| Signal-to-noise ratio | ~35 dB |
| Bandwidth | 2.4 MHz centered on H I |
| FFT resolution | 1171 Hz per bin |

The narrow feature at the hydrogen line sat clearly above the galactic continuum. Doppler shift of the line versus the rest frequency allowed rough velocity mapping of the sampled galactic region (line-of-sight velocity of local hydrogen clouds).

## What it can't do

Honest limits.

No arcminute resolution. A feed horn at 1420 MHz has a beam on the order of 10 to 20 degrees. This means HydrogenEye sees the combined emission of huge patches of sky. For anything resembling a proper radio map you need an interferometer or a large dish, neither of which fits in 62 euros.

No pulsar timing. Pulsars emit at much lower flux, and timing them requires precise GPS synchronization plus a larger aperture. HydrogenEye has the GPS hook, but the antenna is the limit.

No CMB or cosmic structure. Those require colder receivers, bigger apertures, and usually different frequencies.

It handles the 21 cm line. That's the scope.

## Why I built it

I wanted a starting project for radio astronomy under 100 euros that actually worked, not a demo that sat at 10 dB below detection. The RTL-SDR V.3 + Pi Zero combination turned out to be the sweet spot: just enough stability at 1420 MHz thanks to the TCXO, small enough to throw in a backpack with a feed horn and go observe from a rural spot with low RFI.

The Android app was the fun part. Processing signals on a 15-euro computer and watching a galactic hydrogen line show up on a phone in real time never stopped being satisfying.

## Build it yourself

Clone this repo, follow the hardware BOM, pair it with an antenna from [1420MHz-Feed-Horn](https://github.com/SergheiBrinza/1420MHz-Feed-Horn). You need a clear view of the galactic plane. Best observations are at night in a location with minimal RFI. Urban observations work but require longer integration.

## License

MIT

---

Author: Serghei Brinza, AI engineer. Other projects: [github.com/SergheiBrinza](https://github.com/SergheiBrinza)
