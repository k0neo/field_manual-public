# Dire Wolf and YAAC VHF APRS

## Purpose

Provide autonomous VHF APRS reception, transmission,
position reporting, and geographic station mapping
for the field radio workstation.

## Reference deployment

- Workstation: reference workstation
- Operating system: Linux Mint 22.3
- Deployment date: 2026-11-15
- Station callsign: K0NEO-12
- Dire Wolf: 1.8.2
- YAAC: 1.0-beta229 (2026-06-09)
- Java: OpenJDK 25.0.4.1
- Radio: BTECH VHF/UHF handheld
- Antenna: Outdoor 2m/70cm J-pole
- Frequency: 144.390 MHz FM
- Modem: 1200 bps AFSK

## Architecture

The VHF APRS subsystem operates independently of the
Argonaut V HF digital subsystem.

RF receive path:

    Antenna
      |
    BTECH handheld
      |
    APRS audio cable
      |
    TRRS-to-dual-TRS adapter
      |
    Internal sound card
      |
    Dire Wolf
      |
    KISS TCP
      |
    YAAC

Transmit audio follows the reverse path.

The DigiRig remains dedicated to the Argonaut V.
No changes to its audio or CAT configuration are
required for VHF APRS operation.

## Software locations

    /usr/local/bin/direwolf
    ~/direwolf-aprs.conf
    ~/YAAC/
    ~/YAAC/YAAC
    ~/.java/.userPrefs/org/ka2ddo/yaac/

Dire Wolf includes optional GPSD, Hamlib, and
CM108 PTT support.

The existing AX.25 configuration for Winlink
remains unchanged.

## Dire Wolf configuration

A dedicated configuration file separates VHF APRS
from the original Dire Wolf configuration.

File:

    ~/direwolf-aprs.conf

Contents:

    ADEVICE plughw:CARD=PCH,DEV=0
    ACHANNELS 1

    CHANNEL 0
    MYCALL K0NEO-12
    MODEM 1200

    AGWPORT 8000
    KISSPORT 8001

    TXDELAY 60
    TXTAIL 10

The persistent ALSA card identifier PCH is used
instead of a numerical card index.

TXDELAY specifies 600 milliseconds of transmit
preamble. TXTAIL specifies 100 milliseconds of
trailing transmission.

These timing values are provisional settings for
operation with radio-controlled VOX.

## Audio configuration

Internal audio device:

    plughw:CARD=PCH,DEV=0

DigiRig audio device:

    plughw:CARD=Device,DEV=0

The DigiRig is not used for VHF APRS.

### Receive settings

    Capture:          +22.50 dB
    Mic Boost:          0.00 dB
    Digital:            0.00 dB
    Mic playback:       Muted
    Loopback Mixing:    Disabled

Configure the capture gain:

    amixer -c PCH sset Capture 22.5dB
    amixer -c PCH sset 'Mic Boost' 0

The external microphone input must be selected.

Receive calibration results:

    Capture gain    Dire Wolf audio level
      0.0 dB          5
     15.0 dB         20-21
     22.5 dB         47-48

The final settings produced consistent decoding
of local APRS test packets.

### Transmit settings

    Master:           -30.00 dB
    Headphone:          0.00 dB
    PCM:               -0.60 dB

Set the Master control:

    amixer -c PCH -- sset Master -30dB

The double hyphen prevents amixer from interpreting
the negative gain value as command-line options.

Save internal sound card settings:

    sudo alsactl store PCH

Actual RF deviation has not been measured.
The transmit setting was validated by successful
over-the-air packet reception.

## YAAC configuration

Launch:

    cd ~/YAAC
    ./YAAC

### GPSD port

    Port type: GPSD
    Host: localhost
    Enabled: Yes

YAAC obtains position information from the existing
local GPSD service.

### KISS TNC port

    Port type: KISS-over-TCP
    Host: localhost
    TCP port: 8001
    TNC channel: 0
    Callsign: K0NEO-12
    Transmit: Enabled for RF operation
    Protocol: APRS

Dire Wolf must be running before enabling the
KISS port.

### Beacon configuration

Use GPS-derived position information.

The station beacon must be enabled for position
reporting.

During controlled transmit testing, use a direct
path and suspend automatic beaconing between tests.

### Digipeating

reference workstation is intended to operate as a field APRS station,
not as an unattended digipeater.

Disable digipeater aliases unless digipeating is
specifically required.

In particular, disable:

    TEMP1-1
    WIDE1-1
    WIDE2-2

Outgoing digipeater paths and the digipeating of
received packets are separate configuration items.

An unintended WIDE1-1 retransmission was observed
during initial testing and the configuration was
subsequently corrected.

## Startup procedure

Connect the handheld to the APRS audio interface
and a suitable VHF antenna.

Tune the handheld to 144.390 MHz FM.

For receive-only operation, disable VOX.

Start Dire Wolf:

    direwolf -c ~/direwolf-aprs.conf

Start YAAC:

    cd ~/YAAC
    ./YAAC

Verify the KISS connection:

    ss -tnp | grep ':8001'

Confirm GPS operation and incoming packet decoding.

Enable VOX only when RF transmission is required.

Avoid routing desktop notification sounds through
the radio-connected headphone output while VOX
is enabled.

## Operational verification

The following were verified during deployment:

- Dire Wolf initialization.
- AGWPE and KISS TCP listeners.
- YAAC graphical operation.
- GPSD integration.
- VHF RF reception.
- AFSK packet decoding.
- Receive audio calibration.
- KISS packet delivery to YAAC.
- YAAC geographic station mapping.
- Outgoing APRS beacon generation.
- K0NEO-12 station identification.
- VOX-controlled transmission.
- Direct RF reception by an external IGate.
- APRS-IS delivery and APRS.fi reporting.

At 03:22:11 CST on 2026-11-15, KD0ZEA-6
successfully received a K0NEO-12 beacon directly
over RF at a distance of approximately five miles.

This verified the complete transmit path without
requiring a digipeater.

## Map data

YAAC application and downloaded geographic data:

    ~/YAAC/

Downloaded tiles:

    ~/YAAC/tiledir/

The downloaded maps are included in the operational
backup.

Complete offline geographic coverage has not yet
been verified.

## Known limitations

- Radio-controlled VOX introduces transmit timing delays.
- VOX is unsuitable for reliable connected-mode packet
  operation without further timing validation.
- Hardware-controlled PTT remains a future improvement.
- RF modulation deviation has not been measured.
- Overnight operation verified; long-term packet delivery reliability remains unmeasured.
- Complete offline map operation remains untested.
- Mic Boost was observed to change following reboot.
- Restoration on another workstation remains untested.
- The backup does not contain the Dire Wolf executable,
  Java runtime, or system ALSA state.

## Extended operational validation

2026-11-15

The reference workstation APRS subsystem operated successfully
throughout the night, as confirmed by the operator.

### Audio persistence

Following an autonomous system reboot, the
internal PCH mixer was inspected.

| Control | Observed Setting |
|---|---|
| Capture | +22.5 dB |
| Master | -30 dB |
| Mic Boost | +10 dB |

Capture and Master retained their calibrated
settings. Mic Boost unexpectedly changed from
0 dB to +10 dB.

The correct Mic Boost setting was restored using:

    amixer -c PCH sset 'Mic Boost' 0

The corrected setting remained stable after
starting the APRS applications.

The cause of the reboot-related change remains
unconfirmed.

Verify mixer settings before field operation.

### Additional geographic data

Additional pre-imported YAAC geographic tiles
were downloaded to improve local map coverage.

Local streets and geographic features rendered
correctly following the download.

Offline rendering of the additional tiles has
not been explicitly verified.
