# Day 1 – Splunk Server Deployment

This entry documents the deployment of a Splunk Enterprise server on Ubuntu Server as part of my 60-Day SOC Lab project. This marks the foundation of a simulated Security Operations Center environment built within VMware.

---

## Lab Environment

### Host Machine
- AMD Ryzen 7 7700X (8-Core)
- 32GB RAM
- Windows 11
- VMware Workstation

### Splunk Server Virtual Machine
- Operating System: Ubuntu Server 22.04 LTS
- vCPU: 2
- RAM: 8GB
- Disk: 60GB
- Network Mode: NAT (used during installation)

---

## Ubuntu Server Installation

Ubuntu Server was installed with a minimal configuration and OpenSSH enabled for remote management.

Post-installation updates:

`sudo apt update && sudo apt upgrade -y`
`sudo apt install curl wget unzip -y`

This ensured all system packages were updated before deploying Splunk.

---

## Initial Networking Issue

During setup, the following errors occurred:

- `Failed to fetch`
- `Unable to resolve host address www.splunk.com`

---

## Root Cause

The VM was initially configured using **Host-Only networking**, which isolates the machine from the internet. Because of this:

- DNS resolution failed  
- Package repositories were unreachable  
- Splunk could not be downloaded  

---

## Resolution

The VM network adapter was changed to **NAT mode**. After rebooting, internet connectivity was restored:

This highlighted the importance of understanding virtualization networking modes when building isolated lab environments.

---

## Splunk Enterprise Installation

Splunk Enterprise was downloaded and installed.


During startup, an administrative account was created.

---

## Splunk Web Interface Troubleshooting

After installation, Splunk reported:

`Web interface available at http://'hostname':8000`

However, the interface did not initially load in the browser.

---

## Troubleshooting Steps

Verified Splunk service status:

`sudo /opt/splunk/bin/splunk status`

Verified port 8000 was listening:

`sudo ss -tulnp | grep 8000`

Confirmed the VM IP address:

The issue was due to using the incorrect IP address in the browser. Once the correct NAT-assigned IP was used:

`http://<ubuntu_ip>:8000`

The Splunk web interface loaded successfully.

---

## Current Status

At the completion of this phase:

- Ubuntu Server deployed  
- Splunk Enterprise installed  
- Web interface accessible  
- Networking configuration validated  
- Core SIEM infrastructure operational

This completes Phase 1 of the SOC Lab deployment.

---

## Lessons Learned

- Virtualization network mode directly impacts DNS resolution and external connectivity.  
- Service validation using `splunk status` and `ss` helps isolate connectivity issues.  
- Confirming the correct VM IP address is critical when accessing hosted services.  
- Documenting troubleshooting steps provides clarity and reinforces learning.  

---

## Next Phase

The next stage of the lab will include:

- Deploying a Windows 10 endpoint VM  
- Installing the Splunk Universal Forwarder  
- Ingesting Windows Event Logs  
- Beginning detection engineering and log analysis
- Remove NAT dependency 

This will establish the first functional SOC ingestion pipeline.

# Day 2 - Network Segmentation, Static IP Configuration & Windows VM Deployment


## Overview

Day 2 focused on completing the endpoint deployment and stabilizing the SOC lab network. This included creating the Windows 11 virtual machine, configuring Splunk Universal Forwarder, resolving VMware subnet conflicts, and implementing static IP addressing for long-term lab stability.

### Objectives

- Deploy Windows 11 endpoint VM
- Install Splunk Universal Forwarder
- Ensure proper network segmentation
- Remove NAT dependency
- Implement static IP addressing
- Validate full SIEM connectivity


---

# Windows 11 VM Deployment

## VM Creation

A new Windows 11 virtual machine was created in VMware with the following configuration:

- 2–4 CPU cores
- 8 GB RAM
- 60+ GB disk
- Network Adapter attached to Host-Only (VMnet1)

The Windows ISO was mounted and installation completed through the standard Windows setup process.

## Initial Network Configuration

Although both virtual machines were configured under "Host-Only," the Windows machine was receiving an IP in the `192.168.162.x` range, while the Ubuntu Splunk server was operating in the `192.168.56.x` range.

Because these were different VMware virtual subnets, the systems could not communicate.

### Symptoms

- Failed ping attempts between VMs
- Splunk Web not loading from Windows
- Forwarder unable to reliably communicate

---

## Root Cause

VMware had multiple virtual networks configured:

- VMnet1 → 192.168.56.0 (Host-Only)
- VMnet8 → 192.168.162.0 (NAT)

The Windows VM was attached to a different VMnet than the Ubuntu VM.

Even when selecting "Host-Only," VMware can map VMs to different internal networks unless explicitly configured.

---

## Resolution

Both VMs were explicitly attached to the same virtual network (VMnet1).

To eliminate future DHCP inconsistencies, static IP addressing was configured.

### Ubuntu (Splunk Server)

IP Address: `192.168.56.x`  
Subnet Mask: `255.255.255.0`

### Windows Endpoint

IP Address: `192.168.56.x`  
Subnet Mask: `255.255.255.0`

No default gateway was required because the lab is fully isolated.

---

## Connectivity Validation

From the Windows machine:

`ping 192.168.56.x`

`Test-NetConnection -ComputerName 192.168.56.10 -Port 9997`

Both returned successful results.

Splunk Web was accessible at:

`http://192.168.56.x:8000`

Log ingestion resumed immediately after network stabilization.

---

## Current Architecture

Windows 11 Endpoint (192.168.56.x)  
|  
Host-Only Network  
192.168.56.0/24  
|  
Ubuntu Splunk Enterprise (192.168.56.x)     
|   
NAT Network
192.168.162.0/24

The environment is now fully internal and segmented.

---

## Why Static IPs Were Chosen

Using static IP addresses provides:

- Predictable forwarder configuration
- No DHCP lease dependency
- Reduced troubleshooting complexity
- Enterprise-style infrastructure stability

This mirrors how production SIEM and security infrastructure is typically deployed.

---

## Lessons Learned

- "Host-Only" does not guarantee identical VMnet usage
- VMware virtual networking must be explicitly verified
- DHCP can introduce instability in lab environments
- Static addressing is preferred for core security infrastructure
- Validating connectivity step-by-step prevents cascading troubleshooting

---

## Lab Status After Day 2

- Splunk Enterprise operational
- Windows Forwarder communicating
- Logs successfully ingesting
- Network fully isolated
- Infrastructure stabilized

The SOC lab foundation is now properly engineered and ready for detection engineering and incident simulation.

---

## Next Phase

Upcoming objectives:

- Create first detection rule (Failed Logon Monitoring)
- Build alerting workflow
- Begin structured incident documentation
- Map detections to MITRE ATT&CK

Day 2 completed the infrastructure hardening phase.  
The lab is now stable, segmented, and production-structured.

# Day 3 - Advanced Audit Policy & First Detection Rule

## Overview

Day 3 marked the transition from infrastructure deployment to detection engineering. With the Splunk server and Windows endpoint stabilized, the focus shifted to enabling advanced audit logging, validating security telemetry, and creating the first SOC detection rule.

The objective was to ensure full visibility into Windows authentication activity and build a foundational brute-force detection in Splunk.

---

## Identity & Access Structure Implementation

To simulate a realistic enterprise-style environment, additional users and role-based groups were created on the Windows 11 endpoint.

### Users Created

- Kelly (HR)
- Jimmy (Finance)
- Hellen (IT_Helpdesk)
- Ethan (Workstations_LocalAdmins)

### Groups Created

- HR
- Finance
- IT_Helpdesk
- Workstations_LocalAdmins

To simulate role-based access control (RBAC), the Workstations_LocalAdmins group was nested inside the built-in Administrators group using PowerShell:

Add-LocalGroupMember -Group "Administrators" -Member "Workstations_LocalAdmins"

This created the following privilege hierarchy:

Ethan → Workstations_LocalAdmins → Administrators

The IT_Helpdesk group was granted limited elevated permissions by adding it to:

- Remote Desktop Users
- Event Log Readers

This allowed elevated access without full administrative privileges and created realistic opportunities for privilege monitoring and detection.

---

## Advanced Audit Policy Configuration

To ensure proper security logging, advanced audit subcategories were enabled using:

auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Security Group Management" /success:enable /failure:enable
auditpol /set /subcategory:"Special Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Other Logon/Logoff Events" /success:enable /failure:enable

The Security event log size was increased to 100MB to prevent log rollover during testing.

---

## Log Validation in Splunk

Security log ingestion was verified using:

index=* sourcetype="WinEventLog:Security"

Failed logon activity was confirmed using:

index=* sourcetype="WinEventLog:Security" EventCode=4625

During validation, it was discovered that the username field in this environment is Account_Name rather than TargetUserName. Detection queries were adjusted accordingly.

This reinforced the importance of validating field names before building detection logic.

---

## Detection Rule - Failed Logon Monitoring

### Detection Objective

Detect 5 or more failed logon attempts against the same account within a 5-minute window.

### SPL Query

index=* sourcetype="WinEventLog:Security" EventCode=4625
| search NOT Account_Name IN ("SYSTEM","ANONYMOUS LOGON")
| bin _time span=5m
| stats count by _time, Account_Name
| where count >= 5

This detection identifies potential brute-force or password-spraying behavior.

### MITRE ATT&CK Mapping

- Tactic: Credential Access (TA0006)
- Technique: Brute Force (T1110)

### Alert Configuration

- Trigger Condition: Number of results > 0
- Time Range: Last 5 minutes
- Schedule: Every 5 minutes
- Severity: High

---

## Lessons Learned

- Field validation is critical before writing detection queries.
- Advanced audit subcategories must be explicitly enabled for full visibility.
- Group nesting enables realistic privilege escalation simulation.
- Detection rules should be time-bound to reduce false positives.
- Telemetry validation must occur before alert creation.

---

## Lab Status After Day 3

- Tiered identity structure implemented
- Advanced audit policies enabled
- Security logs validated in Splunk
- First detection rule created and tested
- Alerting workflow established

Day 3 marks the beginning of structured detection engineering within the SOC lab. The environment is now capable of detecting credential-based attack behavior and generating SOC-style alerts.

# Day 4 - Detection Formalization & Incident Documentation

## Overview

Day 4 focused on transforming detection logic into structured SOC artifacts. While Day 3 introduced the first detection rule, Day 4 formalized that detection into documented engineering output and introduced incident reporting and response playbooks.

The objective was to move from simple alert creation to full SOC workflow simulation.

---

## Detection Formalization

The previously created Failed Logon Brute Force detection was moved into the `detections/` directory and documented as a formal detection artifact.

The detection monitors Windows Security Event ID 4625 and triggers when five or more failed logon attempts occur against the same account within a five-minute window.

### Detection Logic

index=* sourcetype="WinEventLog:Security" EventCode=4625
| search NOT Account_Name IN ("SYSTEM","ANONYMOUS LOGON")
| bin _time span=5m
| stats count by _time, Account_Name
| where count >= 5

### MITRE ATT&CK Mapping

- Tactic: Credential Access (TA0006)
- Technique: Brute Force (T1110)

Formalizing detections improves consistency, tuning, and future scalability.

---

## Incident Simulation

To validate the detection workflow, multiple failed logon attempts were generated against the Kelly account. The alert triggered as expected.

An incident report was created in the `incidents/` directory documenting:

- Alert details
- Timeline of events
- Investigation steps
- Determination
- Containment actions

This simulates real SOC analyst documentation practices.

### Incident Determination

The alert was classified as a False Positive due to intentional lab simulation activity.

However, the detection logic functioned correctly and triggered at the defined threshold.

---

## Playbook Creation

A structured response playbook was created in the `playbooks/` directory to outline investigative and containment procedures for brute-force authentication attempts.

The playbook includes:

- Investigation workflow
- Log validation steps
- Containment recommendations
- Escalation criteria
- Documentation requirements

This introduces repeatable and standardized incident response procedures into the lab environment.

---

## Repository Structure Implementation

The following directories are now active components of the SOC lab:

- detections/
- incidents/
- playbooks/
- screenshots/

This structure mirrors enterprise SOC documentation practices and supports detection lifecycle management.

---

## Lessons Learned

- Detections should be documented formally, not just written as queries.
- Incident documentation is essential for tracking alert effectiveness.
- Playbooks ensure consistent investigative methodology.
- Structured repositories increase portfolio professionalism.
- Detection engineering includes validation, documentation, and tuning.

---

## Lab Status After Day 4

- Detection formalized in repository
- Incident report created
- Response playbook documented
- Screenshot evidence captured
- SOC workflow simulation established

Day 4 marks the transition from simple SIEM usage to structured SOC operations simulation. The lab now includes detection engineering, alert validation, incident documentation, and response playbooks.

# Day 5 - Privilege Escalation Detection & Group Monitoring

## Overview

Day 5 focused on detecting privilege escalation through local group membership changes. While previous days concentrated on authentication monitoring, this phase introduced monitoring of high-risk authorization events.

The objective was to detect when users are added to privileged groups such as Administrators or Workstations_LocalAdmins and simulate a privilege escalation scenario.

---

## Detection Engineering - Group Membership Monitoring

A new detection was created to monitor Windows Security Event ID 4732, which indicates a member was added to a local security-enabled group.

Privileged groups monitored:

- Administrators
- Workstations_LocalAdmins

### SPL Query

index=* sourcetype="WinEventLog:Security" EventCode=4732
| search TargetUserName="Administrators" OR TargetUserName="Workstations_LocalAdmins"
| table _time, Account_Name, Member_Name, TargetUserName

This query identifies:

- When a user is added
- Which group was modified
- Which account performed the action

---

## Privilege Escalation Simulation

To validate detection functionality:

1. Jimmy was added to the Workstations_LocalAdmins group.
2. Jimmy was removed.
3. Jimmy was re-added.

This generated Event ID 4732 in the Windows Security log.

The detection triggered as expected, confirming that group membership monitoring is functioning correctly.

---

## MITRE ATT&CK Mapping

- Tactic: Privilege Escalation (TA0004)
- Technique: Abuse Elevation Control Mechanism (T1548)

Monitoring group membership changes is critical because attackers frequently elevate privileges after initial access.

---

## Incident Documentation

An incident report was created in the incidents directory documenting:

- Alert details
- Modified group
- Initiating account
- Investigation steps
- Determination

The event was classified as a False Positive due to controlled lab simulation activity.

---

## Playbook Implementation

A structured response playbook was created to standardize investigation steps for privilege escalation alerts.

The playbook includes:

- Identification of modified group
- Validation of initiating account
- Correlation with authentication events
- Containment procedures
- Escalation criteria

This ensures repeatable and structured response methodology for high-severity alerts.

---

## Lessons Learned

- Privileged group changes represent high-value security telemetry.
- Monitoring authorization events is as important as monitoring authentication failures.
- Detection engineering must include validation and documentation.
- Correlating group changes with logon events strengthens investigative capability.
- Structured playbooks improve consistency in incident handling.

---

## Lab Status After Day 5

- Privilege escalation detection implemented
- Event ID 4732 validated in Splunk
- Incident documentation completed
- Response playbook created
- High-severity monitoring expanded

Day 5 introduces authorization monitoring into the SOC lab. The environment now detects both authentication-based attacks and privilege escalation activity, significantly increasing detection coverage.

# Day 6 – Kali Setup and RDP Validation

## Objective
Set up Kali Linux as the attacker system and validate remote access to the Windows target over RDP.

## Actions Completed
- Confirmed Windows Remote Desktop service was enabled
- Verified RDP port availability on the Windows machine
- Installed and configured an RDP client on Ubuntu
- Troubleshot authentication and connection issues
- Successfully established an RDP session into the Windows machine

## Validation
- Ubuntu machine successfully connected to Windows over RDP
- Windows target accepted remote login over port 3389
- Lab systems confirmed capable of generating authentication and remote access events

## Outcome
The lab environment now supports realistic remote access activity that can be monitored and analyzed in Splunk.
