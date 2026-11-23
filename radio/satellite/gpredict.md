# Gpredict with Native GPSD Support

## Purpose

Gpredict provides satellite tracking, orbital predictions,
ground station visualization, and interfaces for radio and
antenna rotator control.

Native GPSD support allows automatic ground station positioning
during portable amateur radio operations.

## Validated Environment

Tested November 14, 2026.

| Component | Version |
|---|---|
| Workstation | reference workstation |
| Operating system | Linux Mint 22.3 |
| Distribution base | Ubuntu 24.04 Noble |
| Architecture | x86-64 |
| Gpredict | 2.6 |
| GPSD | 3.25 |
| GTK+ | 3.24.41 |
| GLib | 2.80.0 |
| libcurl | 8.5.0 |

## Distribution Package Limitation

The Ubuntu Noble Gpredict 2.3 package does not include
native GPSD support.

This was verified by examining the ground station configuration
interface and the executable's shared library dependencies.

The upstream Gpredict 2.6 release compiles successfully with
GPSD support without source modifications.

## Build Dependencies

Install the required development packages:

```bash
sudo apt-get install \
    build-essential \
    libtool \
    intltool \
    autoconf \
    automake \
    pkg-config \
    wget \
    bzip2 \
    libcurl4-openssl-dev \
    libglib2.0-dev \
    libgtk-3-dev \
    libgps-dev
```

GooCanvas is not required by Gpredict 2.6.

## Source Acquisition

Upstream repository:

https://github.com/csete/gpredict

Download the pinned release:

```bash
mkdir -p ~/src
cd ~/src

wget https://github.com/csete/gpredict/releases/download/v2.6/gpredict-2.6.tar.bz2
```

Verify the archive:

```bash
printf '%s  %s\n' \
    f4e6967cb2eeb6d74b8e8208c90dc8f4b1c00955ae1049753ad8352c927e54a3 \
    gpredict-2.6.tar.bz2 | sha256sum -c -
```

The checksum was recorded from the upstream release archive
downloaded during development.

Do not proceed if checksum verification fails.

## Compilation

```bash
tar -xjf gpredict-2.6.tar.bz2
cd gpredict-2.6

./configure --prefix=/usr/local

make -j2
```

The configuration summary should identify libgps.

Verify the compiled executable:

```bash
ldd src/gpredict | grep -i gps
ldd src/gpredict | grep 'not found'
```

On the validated system, Gpredict links against libgps.so.30.

No unresolved library dependencies should be present.

## Installation

```bash
sudo make install
hash -r
command -v gpredict
```

Expected executable:

```text
/usr/local/bin/gpredict
```

Verify GPSD linkage:

```bash
ldd /usr/local/bin/gpredict | grep -i gps
```

The source installation can coexist with the distribution
package during initial testing.

## Desktop Integration

The upstream installation creates:

```text
/usr/local/share/applications/gpredict.desktop
```

The desktop entry uses Exec=gpredict.

Cinnamon correctly selected the executable under /usr/local/bin
during testing.

Verify the running executable:

```bash
pgrep -a -x gpredict
readlink -f /proc/$(pgrep -n -x gpredict)/exe
```

Invoking gpredict --version through SSH without a graphical
display produces a GTK display warning.

## GPSD Configuration

GPSD must be installed, configured, and operational.

Verify the service and receiver:

```bash
systemctl is-active gpsd.service gpsd.socket
gpspipe -w -n 10
```

A TPV report with mode 2 or 3 indicates a position fix.

In Gpredict, open:

Edit -> Preferences -> Ground Stations

Create or edit the ground station and select:

| Parameter | Setting |
|---|---|
| QTH Type | GPSD |
| GPSD Server | 127.0.0.1 |
| GPSD Port | 2947 |

Enter valid fallback latitude, longitude, and altitude.

Restart Gpredict after changing the ground station type.

During testing, changing an existing ground station from
Static to GPSD did not establish a connection until Gpredict
was restarted.

## Ground Station Configuration File

Ground station files reside under:

```text
~/.config/Gpredict/
```

Example configuration:

```ini
[QTH]
DESCRIPTION=Portable ground station
LAT=38.0
LON=-90.0
ALT=150
QTH_TYPE=1
GPSDSERVER=127.0.0.1
GPSDPORT=2947
```

The coordinates above are illustrative, not an operational
station location.

Close Gpredict before modifying its configuration files.

## Dynamic Positioning and Fallback

Gpredict initially uses the coordinates stored in its QTH file.

Once GPSD supplies a valid position, Gpredict updates the
ground station location automatically.

GPS-acquired coordinates were not written back to the QTH
file during operation or normal application shutdown.

Consequently, valid fallback coordinates should be configured
rather than leaving latitude and longitude at zero.

The fallback position may be inaccurate when operating away
from the stored location. Verify live GPS acquisition before
relying on satellite pass predictions in the field.

## Verify GPSD Connection

```bash
ss -tnp | grep ':2947'
```

An established connection owned by Gpredict confirms
communication with GPSD.

Verify that the displayed ground station position agrees with
the receiver's current position.

## GPSD Service Interruption Test

Close Gpredict and stop both GPSD units:

```bash
sudo systemctl stop gpsd.socket gpsd.service
```

Verify:

```bash
systemctl is-active gpsd.service gpsd.socket
ss -ltnp | grep ':2947'
```

Start Gpredict.

The application should use its stored fallback coordinates.

Restore GPSD while Gpredict remains running:

```bash
sudo systemctl start gpsd.socket gpsd.service
```

Verify the connection:

```bash
ss -tnp | grep ':2947'
```

On reference workstation, Gpredict automatically reconnected and acquired
the live position without requiring an application restart.

GPSD interruption also affects GPS-assisted system time.
Verify Chrony recovery when applicable:

```bash
chronyc sources -v
chronyc tracking
```

## Troubleshooting

### GPSD options are missing

Verify that the source-built executable is being used and
that it links against libgps.

### Ground station appears at zero latitude and longitude

Verify the saved QTH configuration, GPS receiver fix, and
established GPSD connection.

Restart Gpredict after changing the QTH type.

Configure valid fallback coordinates.

### No GPSD connection

Verify that both GPSD service units are active and that
GPSD is listening on port 2947.

### GTK cannot open display

Launch Gpredict from the local graphical desktop rather
than an ordinary SSH session.

## Manual Source Uninstallation

Close Gpredict before removal.

From the configured source directory:

```bash
sudo make uninstall
```

Gpredict 2.6 leaves four documentation files behind.
Remove these explicitly:

```bash
sudo rm -f \
    /usr/local/share/gpredict/AUTHORS \
    /usr/local/share/gpredict/COPYING \
    /usr/local/share/gpredict/NEWS \
    /usr/local/share/gpredict/README
```

User configuration is preserved.

If the distribution package remains installed, its executable
will become available again after removal of the source version.

Installations managed by the revised 73Linux recipe should
normally be removed through that recipe instead.

## Configuration Backup

Close Gpredict before creating a backup.

```bash
mkdir -p ~/backups

tar -czf ~/backups/gpredict-working-$(date +%Y%m%d).tar.gz \
    -C "$HOME" .config/Gpredict
```

The configuration archive includes ground stations,
tracking modules, satellite data, and application preferences.

Treat ground station coordinates as private information.

## Validation Summary

Verified on reference workstation:

- Source compilation with GPSD 3.25.
- Installation and desktop integration.
- Automatic GPS positioning.
- Stored fallback positioning when GPSD is unavailable.
- Automatic reconnection after GPSD service restoration.
- Source uninstallation.
- User configuration preservation.

Additional test coverage and scope:

- GPSD connected without a satellite fix:
  optional resilience test, not a deployment blocker.
- Satellite radio Doppler control:
  deferred; not currently required.
- Antenna rotator control:
  not applicable; no rotator is installed.
- Raspberry Pi compatibility:
  not tested; outside the current laptop deployment scope.

## References

Gpredict upstream:
https://github.com/csete/gpredict

Gpredict releases:
https://github.com/csete/gpredict/releases

GPSD:
https://gpsd.gitlab.io/gpsd/

73Linux:
https://github.com/km4ack/73Linux

