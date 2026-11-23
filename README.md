# K0NEO Field Manual — Public Edition

Verified technical procedures, engineering notes, and amateur-radio software deployment records maintained by K0NEO.

This is a **curated public edition**, not a mirror of the private operational Field Manual. Private network inventories, device identifiers, exact backup locations, and host-specific deployment records are intentionally omitted or generalized. Tests reported in these documents refer to the original documented deployments; publication does not imply that the instructions have been independently retested on other platforms.

## Engineering practice

- [Engineering and Troubleshooting SOP](procedures/engineering-sop.md)

## Amateur radio

### Satellite and GPSD

- [Gpredict with Native GPSD Support](radio/satellite/gpredict.md)
- [73Linux Gpredict GPSD Enhancement](radio/satellite/73linux-gpredict.md)

### Propagation

- [VOACAP and PythonProp](radio/propagation/voacap-pythonprop.md)

### Digital, packet, and Winlink

- [Dire Wolf and YAAC VHF APRS](radio/packet/direwolf-yaac.md)
- [ARDOP CF Configuration](radio/winlink/ardopcf-configuration.md)
- [PatMenu3 Installation and Maintenance](radio/winlink/patmenu3-maintenance.md)
- [DigiRig Mobile Interface](radio/hardware/digirig-mobile.md)

## Reading and applying the procedures

The material records particular versions, hardware, observations, and verification results. Confirm that the software and hardware match your own system before running commands, and make a recoverable backup of your own configuration first. Replace sample paths, callsigns, network settings, and device identifiers with your own values.

Some public articles omit private-site backup and restoration sections. Such omissions are deliberate; do not assume the remaining text is a complete disaster-recovery procedure.

## Contributions and attribution

Original project and upstream links are retained where provided. The repository uses the [MIT License](LICENSE). Third-party software remains subject to its own license.

## Publication

Initial public selection prepared 2026-11-23 from the current private Field Manual documentation. The public repository has independent history; it does not contain the private repository's earlier commits.
