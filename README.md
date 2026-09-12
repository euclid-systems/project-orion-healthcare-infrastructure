# Project Orion

## Healthcare Infrastructure & DICOM Engineering Lab

Project Orion is a multi-system healthcare infrastructure lab designed to simulate core enterprise and medical-imaging workflows.

The environment integrates Windows Server Active Directory and DNS, Ubuntu Linux infrastructure, an Orthanc PACS, and a DCMTK-based simulated imaging modality.

The project focuses on infrastructure engineering, systems administration, networking, security, troubleshooting, and healthcare interoperability.

> **Note:** All DICOM data used in Project Orion is synthetic. No protected health information (PHI) is used or stored.

---

## Architecture

Project Orion currently consists of three virtual servers running within VMware Workstation on an isolated NAT network.

| System | Address | Platform | Role |
|---|---|---|---|
| ORION-DC01 | 192.168.174.10 | Windows Server 2025 | Active Directory / DNS |
| ORION-PACS01 | 192.168.174.20 | Ubuntu Server 24.04 LTS | Orthanc PACS |
| ORION-MOD01 | 192.168.174.30 | Ubuntu Server 24.04 LTS | Simulated DICOM modality |

**Network:** `192.168.174.0/24`  
**Internal DNS domain:** `orion.lab`

---

## Current Capabilities

### Infrastructure

- Windows Server 2025 Active Directory Domain Services
- AD-integrated DNS for `orion.lab`
- Static addressing for infrastructure systems
- VMware NAT-based virtual network
- Windows host split-DNS using NRPT
- Ubuntu Server infrastructure
- LVM-backed dedicated Orthanc storage

### DICOM

- Orthanc 1.13.0 PACS
- DCMTK 3.6.7 simulated modality
- DICOM Verification (C-ECHO)
- DICOM Storage (C-STORE)
- DICOM Query (C-FIND)
- DICOM Retrieve (C-MOVE)
- Bidirectional DICOM associations

### Security

- UFW host firewalls
- Dedicated Linux service accounts
- systemd service hardening
- Orthanc HTTP authentication
- Calling AE validation
- Called AE validation
- Source-host validation
- Per-modality DICOM service authorization
- Global DICOM Query/Retrieve permissions disabled

---

## Verified DICOM Workflows

| Workflow | Result |
|---|---|
| ORION-MOD01 → ORION-PACS01 C-ECHO | Pass |
| ORION-MOD01 → ORION-PACS01 C-STORE | Pass |
| ORION-MOD01 → ORION-PACS01 C-FIND | Pass |
| ORION-MOD01 → ORION-PACS01 C-MOVE | Pass |
| ORION-PACS01 → ORION-MOD01 C-STORE | Pass |
| Unknown Calling AE | Rejected |
| Incorrect Called AE | Rejected |
| Valid AE from incorrect source host | Rejected |

---

## Engineering Approach

Project Orion is built around verification rather than configuration alone.

Changes are validated through:

- Positive functional testing
- Negative authorization testing
- Network-level validation
- Application-level protocol testing
- Service restart and cold-boot testing
- Filesystem and storage verification
- Log analysis
- DICOM metadata comparison
- Cryptographic hashing and byte-level comparison

The goal is to demonstrate not only that services can be installed, but that their behavior, security boundaries, persistence, and failure modes are understood.

---

## Project Status

**Version 1.0 — Core Infrastructure & DICOM**

Current milestone includes:

- Core Windows and Linux infrastructure
- Active Directory and DNS
- Orthanc PACS
- Simulated imaging modality
- DICOM storage
- DICOM Query/Retrieve
- Persistent Linux services
- Host firewalling
- DICOM access control
- Security validation

Future phases will include monitoring and centralized logging, PostgreSQL-backed Orthanc storage, backup and recovery testing, DICOMweb, TLS-protected application access, automation, and hybrid AWS infrastructure.