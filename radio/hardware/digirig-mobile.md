# DigiRig Mobile Interface

## Revision History

2026-11-12 - Initial installation and verification on portable workstation.

## Purpose

Document the DigiRig Mobile interface used for portable
amateur radio digital communications.

## Platform

Host: portable workstation
Operating system: Linux Mint 22.3
Architecture: x86_64
Operator: K0NEO

Intended HF transceiver: Ten-Tec Argonaut V, model 516.

## USB Hardware

The DigiRig provides independent USB serial and USB audio
interfaces.

| Component | USB identifier | Linux driver |
|---|---|---|
| Silicon Labs CP2102N | 10c4:ea60 | cp210x |
| C-Media USB Audio | 0d8c:0012 | snd-usb-audio |

## Serial Interface

The following persistent device path was verified:

    /dev/serial/by-id/REPLACE_WITH_YOUR_DEVICE_SYMLINK

During testing, this resolved to /dev/ttyUSB0.

The serial device is owned by root:dialout with
permissions 0660.

Applications should use the persistent device path rather
than assuming a particular ttyUSB device number.

## User Permissions

The operating account belongs to the following groups:

    dialout
    audio
    plugdev

Group membership was verified after reboot.

## Audio Interface

The DigiRig USB audio interface was identified as ALSA
card 3 during testing.

The persistent ALSA device identifier is:

    plughw:CARD=Device,DEV=0

This identifier avoids dependence on the numerical ALSA
card index. Additional USB audio devices with identical
card identifiers may require further disambiguation.

## Verification

Identify the USB interfaces:

    lsusb

Identify persistent serial devices:

    ls -l /dev/serial/by-id/

Enumerate audio devices:

    arecord -l
    aplay -l

Test audio capture:

    arecord -D plughw:CARD=Device,DEV=0 \
        -f S16_LE -r 48000 -c 1 -d 5 \
        /tmp/digirig-test.wav

Verify the recording format:

    file /tmp/digirig-test.wav

The recording was successfully created as 16-bit,
monophonic PCM audio at 48000 samples per second.

ARDOP CF also successfully initialized both capture
and playback using the persistent ALSA identifier
at 12000 samples per second.

## Hamlib

Installed Hamlib version: 4.7.0

Hamlib identifies the Ten-Tec Argonaut V as:

    Model: 16007
    Backend: RIG_MODEL_TT516

## Outstanding Verification

The physical transceiver was disconnected during these
tests.

The following remain unverified:

- CAT serial parameters.
- Serial electrical interface configuration.
- Transceiver operating mode support.
- PTT configuration and operation.
- Receive and transmit audio levels.
- Actual RF communication.

Audio device initialization does not establish correct
radio audio levels or successful RF operation.
