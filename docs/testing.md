# Validation and Testing

## Overview

Project Orion uses repeatable functional, security, persistence, and protocol-level testing to verify that infrastructure changes behave as intended.

Testing is performed with synthetic DICOM data only.

The objective is to validate not only successful workflows, but also expected failures and security boundaries.

---

## Test Categories

Project Orion currently uses four primary categories of validation:

- Infrastructure validation
- DICOM functional validation
- Security and authorization validation
- Persistence and recovery validation

---

## Infrastructure Validation

### DNS Resolution

The `orion.lab` DNS zone is hosted by ORION-DC01.

Validated records include:

| Hostname | Expected Address | Result |
|---|---|---|
| `orion-dc01.orion.lab` | 192.168.174.10 | Pass |
| `orion-pacs01.orion.lab` | 192.168.174.20 | Pass |
| `orion-mod01.orion.lab` | 192.168.174.30 | Pass |

The Windows VMware host also resolves `.orion.lab` through an NRPT split-DNS rule targeting ORION-DC01.

### Service Availability

Validated services include:

| System | Service | Port / State | Result |
|---|---|---|---|
| ORION-DC01 | DNS | Running / Automatic | Pass |
| ORION-PACS01 | SSH | TCP/22 | Pass |
| ORION-PACS01 | DICOM | TCP/4242 | Pass |
| ORION-PACS01 | Orthanc HTTP | TCP/8042 | Pass |
| ORION-MOD01 | SSH | TCP/22 | Pass |
| ORION-MOD01 | DICOM Storage SCP | TCP/11112 | Pass |

---

## DICOM Functional Validation

### C-ECHO

**Purpose:** Verify DICOM association establishment and Verification SOP Class support.

| Source | Destination | Result |
|---|---|---|
| ORION-MOD01 | ORION-PACS01 | Pass |
| ORION-PACS01 | ORION-MOD01 | Pass |

A successful C-ECHO is treated as application-layer validation rather than merely a TCP connectivity test.

### C-STORE

**Purpose:** Verify transmission and persistent storage of DICOM objects.

| Source | Destination | Result |
|---|---|---|
| ORION-MOD01 | ORION-PACS01 | Pass |
| ORION-PACS01 | ORION-MOD01 | Pass |

Synthetic DICOM objects were successfully stored at both endpoints.

### C-FIND

**Purpose:** Verify study-level Query/Retrieve functionality.

ORION-MOD01 successfully queried ORION-PACS01 using study-level identifiers including Patient ID and Study Instance UID.

**Result:** Pass

### C-MOVE

**Purpose:** Verify DICOM retrieve behavior and destination AE resolution.

ORION-MOD01 submitted a study-level C-MOVE request to ORION-PACS01.

ORION-PACS01 then opened a separate DICOM association back to ORION-MOD01 and delivered the requested object using C-STORE.

**Result:** Pass

---

## Security Validation

Security testing includes intentionally invalid DICOM association attempts.

| Test | Expected Result | Observed Result |
|---|---|---|
| Registered ORION-MOD01 | Accept | Pass |
| Unknown Calling AE | Reject | Calling AE Title Not Recognized |
| Incorrect Called AE | Reject | Called AE Title Not Recognized |
| Correct AE from incorrect source host | Reject | Calling AE Title Not Recognized |
| C-FIND with authorized MOD01 policy | Accept | Pass |
| C-MOVE with authorized MOD01 policy | Accept | Pass |

These tests verify that TCP access to the DICOM listener does not automatically grant application-level access.

---

## Orthanc HTTP Authentication

Authentication behavior was tested directly against the Orthanc REST API.

| Request | Expected | Result |
|---|---|---|
| Unauthenticated `/system` request | HTTP 401 | Pass |
| Authenticated `/system` request | HTTP 200 | Pass |

This confirms that remote access requires authentication.

TLS is not yet implemented and remains a documented limitation of the current lab milestone.

---

## Storage Validation

ORION-PACS01 uses a dedicated LVM logical volume mounted at:

`/var/lib/orthanc`

The filesystem mount was verified after restart.

ORION-MOD01 stores received DICOM objects under:

`/var/lib/orion-mod01/incoming`

Received objects were verified on disk with restricted ownership and permissions.

---

## Service Persistence Validation

### ORION-PACS01

The Orthanc service is:

- Enabled at boot
- Active after restart
- Running under the dedicated `orthanc` service identity

### ORION-MOD01

The DICOM Storage SCP is:

- Managed by systemd
- Enabled at boot
- Active after reboot
- Running under the dedicated `oriondicom` service identity
- Listening on TCP/11112 after reboot

Post-reboot C-ECHO and DICOM storage tests were successfully repeated.

---

## DICOM Integrity Validation

Stored and retrieved DICOM objects were compared using:

- SHA-256 hashing
- File-size comparison
- DICOM metadata inspection
- Study Instance UID comparison
- Series Instance UID comparison
- SOP Instance UID comparison
- Byte-level comparison

Differences were isolated to DICOM Part 10 File Meta Information.

No differences were found in the underlying DICOM dataset during the analyzed round-trip test.

See [Troubleshooting and Engineering Investigations](troubleshooting.md) for the full investigation.

---

## Validation Philosophy

Project Orion treats a system as validated only when observable behavior supports the intended configuration.

This includes:

- Testing expected success
- Testing expected failure
- Verifying application-level protocols
- Correlating results with service logs
- Confirming persistent storage
- Repeating critical tests after service restart or system reboot
- Comparing intended and effective authorization policy

The goal is repeatable evidence rather than configuration alone.
