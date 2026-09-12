\# DICOM Workflows



\## Overview



Project Orion implements a simulated DICOM environment consisting of an Orthanc PACS and a DCMTK-based imaging modality.



The lab is used to validate DICOM association behavior, storage, query/retrieve operations, access controls, and failure scenarios using synthetic data only.



\## DICOM Endpoints



| System | AE Title | Address | Port | Role |

|---|---|---|---:|---|

| ORION-PACS01 | ORION-PACS01 | 192.168.174.20 | 4242 | PACS / DICOM SCP |

| ORION-MOD01 | ORION-MOD01 | 192.168.174.30 | 11112 | Simulated modality / Storage SCP |



\## Validated DIMSE Services



\### C-ECHO — Verification



C-ECHO is used to verify that two DICOM application entities can establish an association and successfully perform the DICOM Verification service.



A successful C-ECHO validates more than basic TCP connectivity. It demonstrates that the remote DICOM service is reachable, an association can be negotiated, and the Verification SOP Class is supported.



Project Orion has validated C-ECHO in both directions between the registered PACS and modality endpoints.



\### C-STORE — Storage



C-STORE is used to transmit DICOM objects from one application entity to another.



ORION-MOD01 successfully transmits synthetic DICOM objects to ORION-PACS01 for storage.



ORION-PACS01 can also establish a separate association back to ORION-MOD01 and transmit DICOM objects to the modality's persistent Storage SCP.



\### C-FIND — Query



C-FIND allows ORION-MOD01 to query the PACS for matching DICOM resources.



Project Orion has validated study-level queries using identifiers including Patient ID and Study Instance UID.



C-FIND access is denied globally and granted specifically to the registered ORION-MOD01 modality.



\### C-MOVE — Retrieve



C-MOVE allows a DICOM requester to instruct the PACS to send matching objects to a designated destination AE.



Unlike a simple request/response exchange, C-MOVE uses two DICOM associations:



1\. ORION-MOD01 establishes an association to ORION-PACS01 and sends the C-MOVE request.

2\. ORION-PACS01 resolves the requested destination AE from its modality configuration.

3\. ORION-PACS01 establishes a separate association to ORION-MOD01.

4\. The requested DICOM object is delivered to ORION-MOD01 using C-STORE.

5\. ORION-MOD01's systemd-managed Storage SCP writes the object to persistent storage.



```mermaid

sequenceDiagram

&#x20;   participant MOD as ORION-MOD01

&#x20;   participant PACS as ORION-PACS01



&#x20;   MOD->>PACS: C-MOVE request

&#x20;   Note over MOD,PACS: Study Instance UID requested

&#x20;   PACS-->>MOD: C-MOVE pending

&#x20;   PACS->>MOD: New DICOM association

&#x20;   PACS->>MOD: C-STORE object

&#x20;   MOD-->>PACS: C-STORE success

&#x20;   PACS-->>MOD: C-MOVE final success

