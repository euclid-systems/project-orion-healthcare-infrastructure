# Configuration Samples

This directory contains sanitized configuration examples from Project Orion.

These files are intended to document the lab's infrastructure and application design without exposing credentials, private keys, backup files, or other sensitive information.

## Included Configurations

### Netplan

`netplan/orion-pacs01.yaml`

Static network configuration for ORION-PACS01.

`netplan/orion-mod01.yaml`

Static network configuration for ORION-MOD01.

Both systems use:

- Static addressing on `192.168.174.0/24`
- VMware NAT gateway `192.168.174.2`
- ORION-DC01 at `192.168.174.10` for DNS
- `orion.lab` as the DNS search domain

### systemd

`systemd/orion-dicom-scp.service`

Persistent DCMTK Storage SCP service used by ORION-MOD01.

The service demonstrates:

- A dedicated non-root service account
- Automatic startup
- Restart-on-failure behavior
- Restricted filesystem access
- systemd hardening controls
- Persistent DICOM reception on TCP/11112

### Orthanc

`orthanc/orthanc-sanitized.json`

A sanitized representation of the Project Orion-specific Orthanc configuration.

It documents:

- PACS identity and ports
- Storage paths
- HTTP authentication
- DICOM AE configuration
- Called AE validation
- Source-host validation
- Global DICOM authorization policy
- ORION-MOD01 registration
- Per-modality DICOM permissions

This file is not a verbatim copy of the live `/etc/orthanc/orthanc.json`.

Default Orthanc settings, comments, credentials, and unrelated configuration have been intentionally omitted.

## Security

No credentials or private keys should be committed to this repository.

Repository exclusions are defined in the root `.gitignore`.

Configuration examples should always be reviewed and sanitized before being added to source control.
