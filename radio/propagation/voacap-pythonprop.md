# VOACAP and PythonProp

## Purpose

Provide autonomous HF propagation prediction capabilities
for field radio workstations without requiring continuous
Internet connectivity.

## Reference deployment

- Workstation: reference workstation
- Operating system: Linux Mint 22.3
- VOACAPL release: 0.7.5
- VOACAP engine: 16.1207W
- PythonProp release: 0.30.1
- Python: 3.12
- Matplotlib: 3.6.3
- Cartopy: 0.22.0
- Radio: Ten-Tec Argonaut V (TT-516)
- Deployment date: 2026-11-15

## Architecture

PythonProp provides a graphical interface to the VOACAP
FORTRAN propagation prediction engine.

VOACAP performs the propagation calculations. PythonProp
generates input files, invokes the engine, processes its
output, and renders point-to-point and geographic area
predictions.

## VOACAPL installation

The reference workstation already had VOACAPL installed.

Executable locations:

    /usr/local/bin/voacapl
    /usr/local/bin/wwl

System data directory:

    /usr/local/share/voacapl/itshfbc

User working directory:

    ~/itshfbc

The user working directory contains local antenna models,
input files, prediction results, and symbolic links to
system-installed coefficient and geographic databases.

## PythonProp dependencies

Install the required development and runtime dependencies:

    sudo apt install python3-matplotlib python3-cartopy \
        python3-scipy python3-gi-cairo gir1.2-gtk-3.0 \
        yelp-tools automake

The reference deployment also uses the existing system
development toolchain.

## Source repository

Upstream:

    https://github.com/jawatson/pythonprop

Local repository:

    ~/src/pythonprop

Base release:

    v0.30.1
    6da590d9d941f7c047c0d3ff3675988bfa77164f

Compatibility branch:

    k0neo/python312-compat

## Compatibility corrections

### Commit 1bbb7e0

Fix Python 3.12 module loading and Matplotlib GTK3
compatibility.

Changes:

- Replace obsolete module loading with importlib.
- Correct template-loading diagnostic messages.
- Prevent duplicate registration of the portland colormap.
- Explicitly select the GTK3Agg graphical backend.

### Commit 90c0cf7

Fix AxesGrid labeling for multiple P2P prediction plots.

Replace the unsupported label_outer() method on AxesGrid
axes with the native AxesGrid labeling mechanism.

Use label mode L for grids containing more than four
subplots and all for smaller grids.

## Build and installation

From the patched source repository:

    ./autogen.sh
    ./configure --prefix=/usr/local --without-online-help
    make -j2
    sudo make install

Launch the graphical application:

    voacapgui

Application modules are installed under:

    /usr/local/lib/python3.12/site-packages/pythonprop

Shared application data is installed under:

    /usr/local/share/pythonprop

## Operational data

PythonProp preferences and solar data:

    ~/.voacapgui/

Cached sunspot data:

    ~/.voacapgui/ssn.json

Cartopy geographic resources:

    ~/.local/share/cartopy/

VOACAP working data:

    ~/itshfbc/

The Natural Earth 110m coastline shapefiles are cached
locally after their initial retrieval.

## Autonomous operation

Point-to-point and worldwide area predictions were
generated during offline testing.

The application uses locally cached solar data and
geographic resources. The VOACAP calculation engine
does not require Internet connectivity.

The multi-subplot compatibility correction was developed
following the initial offline test and subsequently
verified through graphical runtime testing and a final
six-subplot offline area prediction regression test.

Cached solar predictions should be refreshed when
Internet connectivity is available.

## Validation

Verified on reference workstation:

- VOACAPL standalone execution.
- PythonProp graphical startup.
- Point-to-point predictions.
- Repeated point-to-point plotting.
- Area prediction template loading.
- Worldwide area predictions.
- Repeated area plotting.
- Multi-subplot point-to-point rendering.
- Local geographic data availability.
- Offline point-to-point and area predictions.
- Offline six-subplot area prediction rendering.
- Repository and configuration backup integrity.

Reference area prediction parameters:

    Frequency: 14.100 MHz
    Transmitter power: 20 W
    Transmit antenna: isotropic
    Receive antenna: isotropic
    Month: November 2026
    Time: 1500 UTC
    Sunspot number: 48
    VOACAP method: 30
    Grid resolution: 50 x 50

These parameters were used for software validation,
not as a calibrated model of the installed antenna.

## Known limitations

- Large multi-subplot grids suffer from excessive title
  and legend crowding.
- Cartopy emits nonfatal deprecation warnings.
- Offline predictions depend on the currency of cached
  solar data.
- Geographic datasets not previously cached may require
  Internet access.

The compatibility patches are maintained locally.
No upstream submission has been made.

## Final offline regression validation

Validation date: 2026-11-15.

The final PythonProp compatibility patch,
commit 90c0cf7, was tested on reference workstation with
Ethernet disconnected and Wi-Fi disabled.

The patch replaces the incompatible
ax.label_outer() calls with the AxesGrid
label_mode setting.

Six area predictions were calculated and
rendered simultaneously.

| Parameter | Value |
|---|---|
| Year | 2026 |
| Month | November |
| Frequency | 14.100 MHz |
| Transmitter Power | 20 W |
| Transmitter Antenna | Isotropic |
| Grid Resolution | 50 x 50 |
| Prediction Subplots | 6 |

### Regression results

| Test | Result |
|---|---|
| Application Startup | PASS |
| Cached Solar Data | PASS |
| VOACAP Calculations | PASS |
| Cached Geographic Data | PASS |
| Six-Subplot Rendering | PASS |
| Matplotlib Compatibility Patch | PASS |
| Internet Independence | PASS |

The previously failing multi-subplot rendering
path completed successfully.

Crowded subplot titles remain a cosmetic
limitation and do not prevent successful
prediction calculations or graphical rendering.

No additional source code changes were required.

