# Sysmon Config

> **Note**: For once, I'm doing a blue team contribution! 
> The goal of this configuration is to propose a Sysmon config that fills the gaps between EDR/XDR endpoint agents and native Windows telemetry.

## Overview

This Sysmon configuration is specifically designed to complement XDR agent by filling telemetry gaps. While XDR agent already collects extensive data through Windows Security Auditing and Windows Filtering Platform, this configuration adds critical detection capabilities for advanced threats including:

## Sysmon Envent IDs Reference

| Event ID | Event Name | What It Monitors |
|----------|------------|------------------|
| 1 | ProcessCreate | Process creation (executable launched with command line) |
| 2 | FileCreateTime | File creation timestamp modified (timestomping) |
| 3 | NetworkConnect | Network connection initiated (TCP/UDP) |
| 4 | Sysmon Service State Changed | Sysmon service started or stopped |
| 5 | ProcessTerminate | Process terminated |
| 6 | DriverLoad | Driver loaded into kernel |
| 7 | ImageLoad | DLL/module loaded by a process |
| 8 | CreateRemoteThread | Remote thread created in another process (injection) |
| 9 | RawAccessRead | Raw disk access (bypass file system) |
| 10 | ProcessAccess | Process opened another process (memory access) |
| 11 | FileCreate | File created or overwritten |
| 12 | RegistryEvent | Registry key or value created/deleted |
| 13 | RegistryEvent | Registry value set/modified |
| 14 | RegistryEvent | Registry key or value renamed |
| 15 | FileCreateStreamHash | Alternate Data Stream (ADS) created |
| 16 | Sysmon Config Change | Sysmon configuration changed |
| 17 | PipeEvent | Named pipe created |
| 18 | PipeEvent | Named pipe connection established |
| 19 | WmiEvent | WMI event filter registered |
| 20 | WmiEvent | WMI event consumer registered |
| 21 | WmiEvent | WMI consumer bound to filter |
| 22 | DnsQuery | DNS query executed |
| 23 | FileDelete | File deleted and archived |
| 24 | ClipboardChange | Clipboard contents changed |
| 25 | ProcessTampering | Process image modified (hollowing/herpaderping) |
| 26 | FileDeleteDetected | File deleted (logged only, not archived) |
| 27 | FileBlockExecutable | Executable file creation blocked |
| 28 | FileBlockShredding | File shredding blocked |
| 29 | FileExecutableDetected | Executable file created |
| 255 | Error | Sysmon error occurred |

---

## Configuration Status

### ENABLED - Event IDs

| Event ID | Event Name | Description | Priority | MITRE ATT&CK |
|----------|------------|-------------|----------|--------------|
| **2** | FileCreateTime | Detects timestomping (anti-forensic technique to modify file creation dates) | MEDIUM | [T1070.006](https://attack.mitre.org/techniques/T1070/006/) |
| **6** | DriverLoad | Detects kernel driver loading (rootkits, malicious drivers) | MEDIUM | [T1014](https://attack.mitre.org/techniques/T1014/) |
| **8** | CreateRemoteThread | Detects process injection via remote thread creation | HIGH** | [T1055](https://attack.mitre.org/techniques/T1055/) |
| **9** | RawAccessRead | Detects raw disk access (credential dumping, bypassing file system auditing) | HIGH | [T1003](https://attack.mitre.org/techniques/T1003/) |
| **10** | ProcessAccess | Detects process memory access (credential dumping from LSASS) | HIGH | [T1003.001](https://attack.mitre.org/techniques/T1003/001/) |
| **12-14** | RegistryEvent | Detects registry object creation, deletion, modification, and renaming | MEDIUM | [T1112](https://attack.mitre.org/techniques/T1112/) |
| **15** | FileCreateStreamHash | Detects Alternate Data Stream (ADS) creation to hide malware | HIGH | [T1564.004](https://attack.mitre.org/techniques/T1564/004/) |
| **17-18** | PipeEvent | Detects named pipe creation and connection (C2 communication) | HIGH | [T1055.001](https://attack.mitre.org/techniques/T1055/001/) |
| **22** | DnsQuery | Detects DNS queries (C2 beaconing, data exfiltration) | MEDIUM | [T1071.004](https://attack.mitre.org/techniques/T1071/004/) |
| **24** | ClipboardChange | Detects clipboard changes (sensitive data collection) | LOW | [T1115](https://attack.mitre.org/techniques/T1115/) |
| **29** | FileExecutableDetected | Detects creation of executable files (malware staging) | HIGH | [T1204.002](https://attack.mitre.org/techniques/T1204/002/) |

### DISABLED - Event IDs (Covered by basic telemetry)

| Event ID | Event Name |
|----------|------------|
| **1** | ProcessCreate | 
| **3** | NetworkConnect |
| **5** | ProcessTerminate |
| **11** | FileCreate | 
| **19-21** | WmiEvent |
| **23** | FileDelete |
| **25** | ProcessTampering |
| **28** | FileBlockShredding |

### DISABLED - Event IDs (Too Verbose / Not Suitable for Production)

| Event ID | Event Name | Reason for Disabling |
|----------|------------|---------------------|
| **7** | ImageLoad | Extremely verbose - generates massive amounts of DLL loading events |
| **27** | FileBlockExecutable | Preventive blocking mode - requires extensive testing before production use |
| **28** | FileBlockShredding | Preventive blocking mode - may cause false positives with legitimate cleanup tools |

### FORCED - Event IDs (Can't Be Disabled)

| Event ID | Event Name | Description |
|----------|------------|-------------|
| **4** | Sysmon Service State Changed | Detects when Sysmon service is started or stopped (tamper detection) |
| **16** | Sysmon Configuration Change | Detects changes to Sysmon configuration (tamper detection) |
| **255** | Sysmon Error | Logs Sysmon internal errors |

---

## Deployment

### Installation
```bash
# Download Sysmon from Microsoft Sysinternals
# https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon

# Install with configuration
sysmon.exe -accepteula -i sysmon-config.xml

# Update existing Sysmon installation
sysmon.exe -c sysmon-config.xml
```

### Verification
```powershell
# Check Sysmon service status
Get-Service Sysmon64

# View Sysmon events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10

# Check current configuration
sysmon.exe -c
```
