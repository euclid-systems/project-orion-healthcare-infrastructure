# Troubleshooting and Engineering Investigations

## Overview

Project Orion documents technical failures and unexpected behavior as part of the engineering process.

Rather than recording only successful configuration steps, troubleshooting cases are preserved to demonstrate how problems were isolated, tested, and resolved.

---

## Case Study 1 — DICOM File Hash Mismatch After Storage and Retrieval

### Observation

A synthetic DICOM object was created on ORION-MOD01 and transmitted to ORION-PACS01.

After retrieving the stored object, its SHA-256 hash did not match the original source file.

At first glance, this raised the possibility that the DICOM object had been modified during storage or retrieval.

### Investigation

The investigation compared:

- SHA-256 hashes
- File sizes
- DICOM metadata
- Study Instance UID
- Series Instance UID
- SOP Instance UID
- Raw byte differences
- Part 10 File Meta Information
- Parsed DICOM datasets

The original and retrieved files contained the same DICOM dataset and preserved the same Study, Series, and SOP identifiers.

Byte-level comparison showed that the differences were limited to DICOM Part 10 File Meta Information.

### Findings

The receiving DICOM implementation regenerated file-level metadata while preserving the underlying DICOM dataset.

During a later round-trip test, the source and returned objects differed by exactly 20 bytes.

The additional metadata was:

`(0002,0016) SourceApplicationEntityTitle`

with the value:

`ORION-PACS01`

The File Meta Information Group Length changed accordingly.

No differences were found in the DICOM dataset itself.

### Conclusion

Whole-file binary identity is not equivalent to DICOM dataset identity.

A DICOM receiver can legitimately regenerate or augment Part 10 File Meta Information without altering the clinical dataset or DICOM object identity.

This investigation demonstrated the importance of understanding protocol-specific file structure before treating a hash mismatch as evidence of corruption.

---

## Case Study 2 — Query/Retrieve Authorization Applied at the Wrong Policy Layer

### Observation

C-FIND and C-MOVE operations succeeded from ORION-MOD01 even though the registered modality configuration showed:

- `AllowFind` set to false
- `AllowMove` set to false

The observed behavior appeared to contradict the per-modality configuration.

### Investigation

The following were examined:

- `/etc/orthanc/orthanc.json`
- Effective modality configuration exposed by the Orthanc REST API
- Orthanc service restart timestamps
- Other JSON configuration files
- Database-backed modality configuration
- Global DICOM authorization settings

The running Orthanc configuration confirmed that the per-modality values were genuinely false.

The investigation then identified these global settings:

- `DicomAlwaysAllowFind` set to true
- `DicomAlwaysAllowMove` set to true

### Root Cause

C-FIND and C-MOVE were succeeding because they had been authorized globally.

The successful tests therefore did not prove that ORION-MOD01's per-modality permissions were functioning as intended.

### Corrective Action

The authorization model was changed to:

- Disable global C-FIND authorization
- Disable global C-MOVE authorization
- Enable C-FIND specifically for ORION-MOD01
- Enable C-MOVE specifically for ORION-MOD01

The resulting policy follows a least-privilege model.

### Validation

After restarting Orthanc:

- C-ECHO from ORION-MOD01 succeeded
- C-FIND from ORION-MOD01 succeeded
- C-MOVE from ORION-MOD01 succeeded
- Unknown Calling AE was rejected
- Incorrect Called AE was rejected
- Correct AE identity from the wrong source host was rejected

### Conclusion

Successful application behavior does not necessarily prove that the intended security control caused the success.

Effective authorization may result from multiple policy layers.

The investigation reinforced the need to validate both configuration state and observed behavior when testing access controls.

---

## Case Study 3 — TCP Connectivity vs. DICOM Protocol Validation

### Observation

A TCP connection test to the Orthanc DICOM port succeeded from another system.

However, Orthanc logged the connection as an invalid DICOM association.

### Explanation

Tools such as `Test-NetConnection` and `nc` validate TCP reachability only.

They can prove that:

- DNS resolution works
- Routing is functional
- A TCP port is reachable
- A process is listening

They do not prove that a valid DICOM association can be established.

### Validation Method

DICOM connectivity was instead validated using DCMTK `echoscu`.

A successful C-ECHO demonstrates:

- TCP connectivity
- DICOM association negotiation
- Valid Called and Calling AE behavior
- Support for the Verification SOP Class
- A valid DICOM response from the peer

### Conclusion

Network-layer success and application-layer success must be tested independently.

Project Orion therefore distinguishes between TCP connectivity tests and DICOM protocol tests.

---

## Troubleshooting Principles Used in Project Orion

Project Orion follows several recurring troubleshooting principles:

- Verify observed state before changing configuration
- Separate network-layer and application-layer testing
- Compare intended configuration with effective configuration
- Use logs to correlate events across systems
- Change one variable at a time when validating policy
- Preserve negative tests as evidence
- Validate persistence after restart or reboot
- Use protocol-aware tools instead of relying only on generic connectivity tests
- Treat unexpected successful behavior as something worth investigating
- Document root cause and corrective action
