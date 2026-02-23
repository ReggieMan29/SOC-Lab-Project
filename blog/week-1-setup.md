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

sudo apt update && sudo apt upgrade -y
sudo apt install curl wget unzip -y

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

Web interface available at http://'hostname':8000 

However, the interface did not initially load in the browser.

---

## Troubleshooting Steps

Verified Splunk service status:

sudo /opt/splunk/bin/splunk status

Verified port 8000 was listening:

sudo ss -tulnp | grep 8000

Confirmed the VM IP address:

The issue was due to using the incorrect IP address in the browser. Once the correct NAT-assigned IP was used:

http://<ubuntu_ip>:8000

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

This will establish the first functional SOC ingestion pipeline.
