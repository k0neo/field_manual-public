# PatMenu3 Installation and Maintenance

## Revision History

2026-11-12 - Initial deployment, source maintenance,
security improvements, and integration testing on portable workstation.

## Purpose

Document the locally maintained PatMenu3 implementation
used to manage portable Winlink operations.

## Platform

Host: portable workstation
Operating system: Linux Mint 22.3
Architecture: x86_64
Maintainer: K0NEO

## Upstream Repository

https://github.com/km4ack/patmenu3

Original upstream commit: 2a64a69

Local maintenance branch:

    k0neo/filesystem-layout

## Filesystem Layout

| Purpose | Location |
|---|---|
| Application repository | ~/.local/share/patmenu3 |
| Application launcher | ~/.local/bin/patmenu |
| Desktop entry | ~/.local/share/applications/patmenu.desktop |
| Application configuration | ~/.local/share/patmenu3/data/config |
| ARDOP logs | ~/.local/state/ardopcf |
| Configuration backups | ~/backups |

The application repository remains user-owned because
PatMenu3 maintains configuration and runtime data within
its own directory structure.

The original installation was relocated from ~/patmenu3.

## Application Launcher

The launcher resolves the application directory without
depending on symbolic link behavior.

    #!/bin/bash
    exec "$HOME/.local/share/patmenu3/patmenu" "$@"

Launcher permissions: 0755.

## Maintenance Commit History

| Commit | Description |
|---|---|
| 8d29c4b | Correct legacy paths and improve relocatable gateway scheduling |
| d4765cc | Replace stored sudo credentials with on-demand graphical authentication |
| 17fffda | Correct ARDOP CF process detection and shutdown |
| d471593 | Implement shared Hamlib radio state and daemon lifecycle management |
| 358660c | Harden rigctld lifecycle with locking, backend verification, and failed-start cleanup |
| 39583a0 | Integrate shared radio control and safe ARDOP startup and shutdown |

These commits exist on the local maintenance branch.
Publication to a separate GitHub repository remains pending.

## Filesystem Corrections

Legacy PatMenu2 references were removed from gateway
management routines.

Gateway scripts now derive their application directory
from their own filesystem location.

Automatic gateway scheduling was revised to preserve
unrelated crontab entries and support relocation.

Mock testing verified daily scheduling, boot scheduling,
schedule removal, legacy entry replacement, cancellation,
and invalid time rejection.

The actual user crontab was not modified during testing.

## Authentication Improvements

The original implementation collected and retained
encoded sudo credentials in runtime files.

This behavior was removed.

Administrative authentication now uses an on-demand
graphical password prompt.

Helper:

    ~/.local/share/patmenu3/bin/sudo-askpass

The helper invokes:

    /usr/bin/zenity --password \
        --title="PatMenu3 Administrative Authentication"

PatMenu3 exports the SUDO_ASKPASS environment variable
and privileged operations use sudo -A.

Graphical authentication was independently verified
using a privileged id command.

No sudoers modifications were required.

## ARDOP CF Process Management

Legacy references to ardopc64 and piardopc were corrected
where appropriate.

The active modem executable is:

    /usr/local/bin/ardopcf

The startup and shutdown routines were subsequently
replaced to integrate shared radio state management.

## Radio Control Architecture

Hamlib version: 4.7.0
Hamlib build: 554e02b39

The radio control architecture uses rigctld as the sole
owner of the physical CAT interface.

Pat and PatMenu3 communicate with rigctld through TCP.

    Pat -----------+
                   |
    PatMenu3 ------+--- rigctld --- DigiRig --- Radio
                   |
    rigctl --------+

The Hamlib NET rigctl client uses model 2.

Operational client command:

    /usr/local/bin/rigctl -m 2 -r 127.0.0.1:4532

The intended physical radio backend is model 16007,
corresponding to the Ten-Tec Argonaut V.

Operational radio control remains disabled until
physical transceiver testing is complete.

## Shared Radio State Library

Location:

    bin/modems/rig-state.sh

The library provides:

- Network radio commands.
- Radio state capture and restoration.
- Frequency, mode, and passband syntax validation.
- Daemon readiness verification.
- Process identity and ownership verification.
- Managed daemon startup and shutdown.
- Remote Hamlib backend identification.
- Advisory lifecycle locking.
- Failed-start cleanup.

The library eliminates direct serial access by the
integrated ARDOP startup and shutdown routines.

## Runtime Files

Default runtime files:

    /run/user/1000/patmenu-rig.state
    /run/user/1000/patmenu-rig.owner
    /run/user/1000/patmenu-rig.owner.lock

The actual runtime directory is derived from
XDG_RUNTIME_DIR or the current user identifier.

The state file contains three lines:

    Frequency in Hz
    Operating mode
    Passband in Hz

Example:

    14074000
    USB
    3000

State files are created with permissions 0600.

The ownership record contains:

    Process identifier
    Kernel process start time
    Executable path

Ownership records are also created with permissions
0600.

The lock file is retained between operations.

State and ownership records are parsed as data rather
than executed as shell code.

## Backend Identification

The NET rigctl client reports its own backend model
rather than the underlying radio model when queried
through get_rig_info.

The underlying daemon model is obtained directly
through the rigctld TCP protocol using dump_state.

The response includes:

    rig_model=1

for the dummy radio backend.

The library verifies the expected backend before
accepting an existing daemon.

Backend verification does not independently identify
individual physical radios of the same model.

## Daemon Lifecycle

An independently started compatible rigctld daemon
may be reused without claiming ownership.

A daemon started by PatMenu3 receives an ownership
record.

Before terminating a managed daemon, the library
verifies its process identifier, owner, executable,
and kernel process start time.

Lifecycle operations are serialized using flock.

The background daemon does not inherit the lifecycle
lock descriptor.

## ARDOP Startup Integration

Location:

    bin/modems/start-pat-ardop

The revised startup routine:

1. Checks for conflicting modem processes.
2. Verifies the configured executable.
3. Establishes radio control if enabled.
4. Saves the existing radio settings.
5. Configures the requested HF operating mode.
6. Starts ARDOP CF in a graphical terminal.
7. Verifies modem process creation.
8. Restarts Pat.
9. Opens the Pat HTTP interface.

With radio control disabled, no CAT commands are
issued by the startup routine.

## ARDOP Shutdown Integration

Location:

    bin/modems/stop-modems

The revised shutdown routine:

1. Terminates ARDOP CF processes belonging to the
   operating user.
2. Verifies modem termination.
3. Restores saved radio settings when applicable.
4. Removes successfully restored state records.
5. Stops an owned rigctld daemon when applicable.
6. Preserves independently managed rigctld daemons.
7. Preserves the Pat HTTP service.

The shutdown routine no longer evaluates saved
radio settings as executable shell code.

## Simulation Testing

Hamlib's dummy radio backend was used to verify
radio control without connecting a physical radio.

Start an independent test daemon:

    /usr/local/bin/rigctld -m 1 \
        -T 127.0.0.1 -t 14532

Query the simulated radio:

    /usr/local/bin/rigctl -m 2 \
        -r 127.0.0.1:14532 f m

A second test daemon on port 14533 was used for
managed lifecycle and failure-path testing.

## Verified Library Tests

| Test | Result |
|---|---|
| Network CAT communication | PASS |
| Frequency capture and restoration | PASS |
| Mode capture and restoration | PASS |
| Passband capture and restoration | PASS |
| State file permissions | PASS |
| Malformed state rejection | PASS |
| Unavailable daemon rejection | PASS |
| Disabled radio control bypass | PASS |
| Process identity verification | PASS |
| Stale process ownership rejection | PASS |
| External daemon preservation | PASS |
| Managed daemon startup | PASS |
| Managed daemon shutdown | PASS |
| Backend identification | PASS |
| Backend mismatch rejection | PASS |
| Mandatory model selection | PASS |
| Locked daemon lifecycle | PASS |
| Failed ownership-record cleanup | PASS |

## Integrated Graphical Testing

ARDOP startup and shutdown were verified with radio
control disabled.

Verified results:

- Correct audio interface selection.
- Successful modem initialization.
- Dedicated logging directory.
- Pat service restart.
- HTTP response 200.
- Continued GPS integration.
- Successful modem termination.
- Release of both modem TCP sockets.
- Preservation of an independent rigctld daemon.

Integrated CAT testing used the independent dummy
daemon on port 14532.

Initial radio state:

    14074000
    USB
    3000

State during ARDOP operation:

    14074000
    LSB
    2400

State following graphical modem shutdown:

    14074000
    USB
    3000

The original radio state was restored successfully.

The independently managed daemon remained active,
and no ownership record was created.

The saved radio state file was removed after successful
restoration.

Following testing, the original operational configuration
was restored and the dummy daemon was terminated.

## Known Limitations

This maintenance branch is not a complete modernization
of PatMenu3.

The Dire Wolf and VARA startup routines retain legacy
radio control implementations.

Automatic shutdown of these modem types is temporarily
unsupported by the revised shutdown routine.

The legacy configuration wizard assumes FLRig and can
overwrite customized settings. It should not be used
with the current portable workstation configuration.

ARDOP startup verifies process creation rather than
complete modem protocol readiness.

The shutdown routine terminates all ARDOP CF processes
belonging to the current operating user.

Lifecycle locking does not serialize complete modem
sessions.

Process identity verification reduces, but does not
eliminate, process-management race conditions.

Stale runtime records may require manual intervention.

Configuration command parsing assumes arguments do
not contain embedded spaces.

Integrated shutdown of a PatMenu3-owned rigctld daemon
has not been independently verified through the complete
graphical startup and shutdown sequence.

Physical CAT operation, PTT, radio audio levels, and
Pat-to-ARDOP protocol integration remain unverified.

## Future Development

Further PatMenu3 maintenance should be undertaken as
a separate development project.

Outstanding work includes:

- Dire Wolf integration.
- VARA HF and VARA FM integration.
- Configuration wizard modernization.
- Complete session locking.
- Individual modem process ownership.
- Protocol readiness verification.
- Additional failure-path testing.
- Publication of the maintenance branch.

## References

https://github.com/km4ack/patmenu3

https://github.com/Hamlib/Hamlib

https://github.com/pflarue/ardop

https://github.com/la5nta/pat
