# Security

## Overview

Project Orion applies layered security controls at the network, operating-system, application, and DICOM protocol levels.

The environment is a lab and is not presented as a production healthcare deployment. The purpose of these controls is to model least-privilege design, validate security boundaries, and provide hands-on experience troubleshooting authorization behavior.

## Network Security

Both Ubuntu systems use UFW with a default-deny inbound policy.

### ORION-PACS01

| Service | Port | Allowed Source |
|---|---:|---|
| SSH | 22/tcp | 192.168.174.0/24 |
| Orthanc HTTP | 8042/tcp | 192.168.174.0/24 |
| DICOM | 4242/tcp | 192.168.174.0/24 |

### ORION-MOD01

| Service | Port | Allowed Source |
|---|---:|---|
| SSH | 22/tcp | 192.168.174.0/24 |
| DICOM Storage SCP | 11112/tcp | 192.168.174.20 |

The MOD01 DICOM receive port is restricted specifically to ORION-PACS01 rather than the entire lab subnet.

## Linux Service Isolation

Orthanc runs under a dedicated `orthanc` service account rather than root.

The MOD01 Storage SCP runs under a dedicated `oriondicom` service account.

The systemd-managed MOD01 service also uses controls including:

- `NoNewPrivileges=true`
- `PrivateTmp=true`
- `ProtectHome=true`
- `ProtectSystem=strict`
- Restricted write access through `ReadWritePaths`

The service can write to its DICOM receive directory without receiving unnecessary write access to the rest of the operating system.

## Filesystem Permissions

Orthanc application data is stored under:

`/var/lib/orthanc/db-v6`

The active storage and log directories are restricted to the Orthanc service identity.

MOD01 received objects are stored under:

`/var/lib/orion-mod01/incoming`

The receive directory is owned by `oriondicom:oriondicom` and is not writable by ordinary users.

During the portfolio audit, Orthanc configuration and backup-file permissions were also tightened to reduce unnecessary local access to application configuration and credential-bearing files.

## Orthanc HTTP Authentication

Remote Orthanc access is enabled for the lab network.

HTTP authentication is explicitly enabled.

Validation confirmed:

| Request | Result |
|---|---|
| Unauthenticated `/system` request | HTTP 401 |
| Authenticated `/system` request | HTTP 200 |

Orthanc currently uses authenticated HTTP without TLS.

This is acceptable for the isolated current lab milestone, but credentials and application traffic are not encrypted in transit. TLS-protected access is planned for a later Project Orion security phase.

## DICOM Authorization

Project Orion denies DICOM services globally and selectively grants required services to the registered ORION-MOD01 endpoint.

| Service | Global Policy | ORION-MOD01 |
|---|---|---|
| C-ECHO | Deny | Allow |
| C-STORE | Deny | Allow |
| C-FIND | Deny | Allow |
| C-MOVE | Deny | Allow |
| C-GET | Deny | Deny |
| Worklist | Deny | Deny |

Additional controls include:

- Called AE title validation
- Calling AE title recognition
- Registered modality source-host validation
- Per-modality service authorization

## Negative Security Validation

Security controls were tested with intentionally invalid DICOM associations.

| Test | Expected Result | Observed Result |
|---|---|---|
| Unknown Calling AE `ROGUE-MODALITY` | Reject | Calling AE Title Not Recognized |
| Incorrect Called AE `WRONG-PACS` | Reject | Called AE Title Not Recognized |
| Valid `ORION-MOD01` AE from wrong source host | Reject | Calling AE Title Not Recognized |

These tests demonstrate that basic TCP connectivity to port 4242 is not sufficient to gain access to the DICOM service.

## Authorization Troubleshooting Case

During Query/Retrieve testing, C-FIND and C-MOVE unexpectedly succeeded even though the registered ORION-MOD01 modality configuration showed both operations as disabled.

The investigation compared:

- On-disk Orthanc configuration
- Effective configuration exposed by the running Orthanc API
- Orthanc service restart timestamps
- Global DICOM authorization settings
- Per-modality authorization settings

The investigation identified the cause:

`DicomAlwaysAllowFind` and `DicomAlwaysAllowMove` had been enabled globally.

This meant Query/Retrieve succeeded because of global authorization rather than the intended per-modality policy.

The configuration was corrected so that:

- Global C-FIND authorization is disabled
- Global C-MOVE authorization is disabled
- C-FIND is granted specifically to ORION-MOD01
- C-MOVE is granted specifically to ORION-MOD01

After the change, legitimate C-ECHO, C-FIND, and C-MOVE operations were retested successfully.

Negative Calling AE, Called AE, and source-host tests were also repeated and continued to be rejected.

This produced a least-privilege authorization model and demonstrated the importance of validating effective behavior rather than assuming that a successful test proves the intended security control is responsible for that success.

## Current Security Limitations

Project Orion v1.0 intentionally leaves several security improvements for later phases:

- Orthanc HTTP traffic is not yet protected by TLS
- SSH password authentication has not yet been replaced completely by key-based authentication
- Centralized security logging is not yet implemented
- The lab currently operates on a single VMware NAT segment
- Certificate lifecycle management has not yet been introduced

These limitations are documented rather than treated as production-ready controls.
