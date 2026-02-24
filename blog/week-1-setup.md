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




