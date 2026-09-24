# PaperGhost — Windows DFIR

> A forensic reconstruction of a Windows endpoint compromise using multiple Windows artifact families.

## Overview

PaperGhost is a Windows forensics CTF investigation involving a planted USB device and a disguised update package.

The investigation reconstructs the activity through four major artifact families:

- `SYSTEM`
- `NTUSER.DAT`
- `SRUDB.DAT`
- `Windows.edb`

The objective was to connect individual forensic findings into a chronological activity sequence rather than treating each artifact independently.

## Investigation Timeline

```text
USB Connected
     ↓
Payload Executed
     ↓
Microphone Access
     ↓
Webcam Activity
     ↓
Outbound Network Traffic
     ↓
Credential Evidence Recovered

## Findings
01 — USB Connection

The SYSTEM registry hive was used to determine when the removable device was connected.

Artifact: SYSTEM
Result: USB connection timestamp recovered from the device properties FILETIME.

02 — USB Serial Number

USBSTOR information was examined to identify the complete device instance identifier.

Artifact: SYSTEM / USBSTOR

03 — Payload Path

UserAssist execution history was examined to recover the path of the executed payload.

Artifact: NTUSER.DAT / UserAssist

The UserAssist value name required ROT13 decoding to recover the executable path.

04 — Execution Timestamp

The UserAssist record contained a Windows FILETIME associated with the payload execution.

The timestamp was converted from the Windows epoch to UTC.

Artifact: NTUSER.DAT / UserAssist

05 — USB Asset Name

The SWD WPDBUSENUM registry information was correlated with the USBSTOR instance to recover the device's asset label.

Artifact: SYSTEM / WPDBUSENUM

06 — Microphone Activity

ConsentStore data was examined to determine when the payload accessed the microphone.

Artifact: NTUSER.DAT / ConsentStore

07 — Webcam Activity

The webcam ConsentStore record contained start and stop FILETIME values.

The difference between these timestamps was used to determine the duration of webcam activity.

Artifact: NTUSER.DAT / ConsentStore

08 — Outbound Traffic

SRUM network usage data was correlated with the malicious application to determine the amount of outbound traffic recorded for the payload.

Artifact: SRUDB.DAT

09 — Credential Evidence

Windows Search index data was examined for evidence relating to the missing DIOGENES document.

The indexed text preserved credential-related information even though the original PDF was absent.

Artifact: Windows.edb / SystemIndex_PropertyStore

Artifact Map
Finding	Artifact	Technique
USB connection	SYSTEM	Device Properties / FILETIME
USB serial	SYSTEM	USBSTOR
Payload path	NTUSER.DAT	UserAssist / ROT13
Execution time	NTUSER.DAT	UserAssist / FILETIME
USB asset	SYSTEM	WPDBUSENUM
Microphone	NTUSER.DAT	ConsentStore
Webcam duration	NTUSER.DAT	ConsentStore / FILETIME
Outbound traffic	SRUDB.DAT	SRUM network usage
Credential evidence	Windows.edb	Search index
Investigation Takeaway

The individual artifacts become more useful when correlated into a single timeline.

The evidence connects removable-media arrival with payload execution, followed by microphone and webcam activity, outbound traffic, and credential exposure.

This demonstrates the value of combining Windows registry artifacts, application-usage evidence, privacy-consent records, SRUM data, and Windows Search indexing during forensic reconstruction.
