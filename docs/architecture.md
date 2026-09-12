# Project Orion Architecture



## Overview



Project Orion is a virtualized healthcare infrastructure lab designed to model core enterprise infrastructure and DICOM medical-imaging workflows.



The environment currently consists of three virtual machines hosted in VMware Workstation on a Windows workstation.



All Orion systems communicate through the VMware VMnet8 NAT network.



## Network Architecture



**Network:** `192.168.174.0/24`  

**Default gateway:** `192.168.174.2`  

**VMware host interface:** `192.168.174.1`  

**Internal DNS domain:** `orion.lab`



| System | IPv4 Address | Operating System | Primary Role |
|---|---|---|---|
| ORION-DC01 | 192.168.174.10 | Windows Server 2025 | Active Directory Domain Services / DNS |
| ORION-PACS01 | 192.168.174.20 | Ubuntu Server 24.04 LTS | Orthanc PACS |
| ORION-MOD01 | 192.168.174.30 | Ubuntu Server 24.04 LTS | Simulated DICOM modality |



## ORION-DC01



ORION-DC01 provides the identity and name-resolution foundation for the lab.



### Services



\- Active Directory Domain Services

\- DNS

\- Global Catalog



### Active Directory



**Forest:** `orion.lab`  

**Domain:** `orion.lab`  

**NetBIOS domain:** `ORION`



The lab uses Windows Server 2025 forest and domain functional levels.



ORION-DC01 currently holds all FSMO roles because Project Orion uses a single-domain-controller architecture.



### DNS



The `orion.lab` zone is an Active Directory-integrated primary DNS zone.



Current infrastructure records include:



| Host | Address |
|---|---|
| `orion-dc01.orion.lab` | 192.168.174.10 |
| `orion-pacs01.orion.lab` | 192.168.174.20 |
| `orion-mod01.orion.lab` | 192.168.174.30 |



Linux systems use ORION-DC01 as their DNS server.



The Windows VMware host uses a Name Resolution Policy Table (NRPT) rule that forwards only `.orion.lab` queries to ORION-DC01. This allows Project Orion DNS resolution without making the lab domain controller the host's global DNS resolver.



## ORION-PACS01



ORION-PACS01 provides the central DICOM/PACS workload.



### Platform



\- Ubuntu Server 24.04 LTS

\- Orthanc 1.13.0

\- DCMTK utilities

\- systemd-managed Orthanc service



### Network Services



| Service | TCP Port |
|---|---:|
| SSH | 22 |
| DICOM | 4242 |
| Orthanc HTTP / REST API | 8042 |



### Storage Architecture



ORION-PACS01 uses Linux LVM to separate application storage from the root filesystem.



The Orthanc data volume is mounted at:



`/var/lib/orthanc`



This provides a dedicated filesystem for PACS data and allows the application-data volume to be managed independently of the operating-system root filesystem.



## ORION-MOD01



ORION-MOD01 simulates a network-connected medical imaging modality.



### Platform



\- Ubuntu Server 24.04 LTS

\- DCMTK 3.6.7



### DICOM Identity



**AE Title:** `ORION-MOD01`  

**Storage SCP port:** `11112`



A persistent `storescp` process is managed by systemd and runs under a dedicated `oriondicom` service identity.



Received DICOM objects are written to:



`/var/lib/orion-mod01/incoming`



## DICOM Communication



The primary DICOM endpoints are:



| System | AE Title | Address | Port |
|---|---|---|---:|
| ORION-PACS01 | ORION-PACS01 | 192.168.174.20 | 4242 |
| ORION-MOD01 | ORION-MOD01 | 192.168.174.30 | 11112 |



Validated workflows currently include:



\- C-ECHO

\- C-STORE

\- C-FIND

\- C-MOVE



C-MOVE uses two separate DICOM associations:



1\. ORION-MOD01 sends the C-MOVE request to ORION-PACS01.

2\. ORION-PACS01 resolves the requested destination AE.

3\. ORION-PACS01 opens a new association to ORION-MOD01.

4\. The requested DICOM object is delivered using C-STORE.



## Security Boundaries



Project Orion currently implements several infrastructure and DICOM security controls:



\- Default-deny UFW host firewalls

\- Restricted inbound service ports

\- Dedicated Linux service identities

\- systemd service hardening

\- Orthanc HTTP authentication

\- Called AE validation

\- Calling AE validation

\- Source-host validation

\- Per-modality DICOM service authorization

\- Global Query/Retrieve authorization disabled



Project Orion is a lab environment and is not presented as a production healthcare deployment.



TLS-protected application access and additional security controls are planned for later project phases.



