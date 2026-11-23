# ARDOP CF Configuration

## Revision History

2026-11-12 - Standalone and integrated modem testing.

## Software

Version: ARDOP CF 1.0.4.1.3
Executable: /usr/local/bin/ardopcf

The installed executable matches the binary on development workstation.

Verified SHA256 fingerprint:

    caf256ca1138ea992323bf8c6729a06aaaacb1b62181d0a7b2abba3dffaa9ad8

## Audio Interface

The modem uses the DigiRig Mobile USB audio interface.

Capture and playback device:

    plughw:CARD=Device,DEV=0

ARDOP CF successfully initialized both audio streams
at 12000 samples per second.

## Logging

Dedicated logging directory:

    ~/.local/state/ardopcf/

Create the directory:

    mkdir -p ~/.local/state/ardopcf

The installed executable supports the --logdir option.

## Startup Command

    /usr/local/bin/ardopcf \
        --logdir "$HOME/.local/state/ardopcf" \
        8515 \
        plughw:CARD=Device,DEV=0 \
        plughw:CARD=Device,DEV=0

PatMenu3 uses this configuration.

## Network Interfaces

| Interface | TCP port |
|---|---|
| Host control | 8515 |
| Host data | 8516 |

Both interfaces were observed listening on 0.0.0.0.

This exposes the modem interfaces to other hosts on
connected networks. Firewall restrictions remain an
outstanding portable workstation security task.

## Verification

With the transceiver disconnected, start the modem
in a terminal.

From a second terminal:

    pgrep -a ardopcf

    ss -ltnp | grep -E ':(8515|8516)'

Test the modem control interface:

    printf 'VERSION\r' | nc -w 3 127.0.0.1 8515

The modem returned an ARDOP CF version response and
reported a connected host control session.

Carriage-return termination interfered with terminal
display of the complete response.

## Shutdown

For standalone testing, terminate the modem using
Ctrl-C in its foreground terminal.

For PatMenu3 operation, use the Stop Modems function.

Successful shutdown was verified by process inspection
and closure of TCP ports 8515 and 8516.

## Integrated Testing

PatMenu3 successfully launched ARDOP CF with the
persistent audio device identifier and dedicated
logging directory.

The modem initialized both audio streams and entered
ARQ protocol mode.

Pat was restarted successfully and remained active
after modem shutdown.

## Outstanding Verification

- Pat-to-ARDOP protocol communication.
- Physical transceiver audio and PTT.
- Receive decoding and transmitted signal quality.
- Actual HF Winlink sessions.
- Firewall restrictions for modem TCP interfaces.

## References

https://github.com/pflarue/ardop

## Successful HF RMS operational validation

Date: 2026-11-15

Workstation: reference workstation
Transceiver: Ten-Tec Argonaut V (TT-516)
Interface: DigiRig
Client: Pat 1.0.0
Modem: ARDOP CF 1.0.4.1.3

RMS gateway: KD7UHR
Frequency: 7062.5 kHz
Connection established: 13:22:31 CST
Remote software: RMS Trimode 1.4.4.2

The initial connection attempt timed out.
The subsequent attempt established a successful
ARDOP connection.

The Winlink session completed normally and
outbound requests were transmitted successfully.

This verifies the operational HF Winlink transmit
path, including radio control, audio interface,
PTT, ARDOP communication, and message submission.

Inbound message retrieval remains to be verified.
