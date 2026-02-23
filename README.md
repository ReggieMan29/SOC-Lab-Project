# 🛡️ 60-Day SOC Analyst Lab

## 📌 Overview

This project simulates a real-world Security Operations Center (SOC) environment over a structured 60-day period. The objective is to replicate practical SOC workflows including alert monitoring, triage, investigation, escalation, and formal incident documentation.

The lab is designed to strengthen hands-on detection and response skills while developing structured documentation aligned with industry frameworks.

---

## 🎯 Project Objectives

- Simulate real-world SOC alert workflows  
- Investigate and document **10+ security alerts**  
- Produce **3 formal incident reports**  
- Develop standardized SOC playbooks  
- Map confirmed activity to MITRE ATT&CK techniques  
- Identify detection gaps and tuning recommendations  

---

## 🧭 Scope

This lab will simulate the following security scenarios:

- Brute force authentication attempts  
- Suspicious login behavior  
- Phishing-related activity  
- Malicious or suspicious PowerShell execution  
- Privilege escalation attempts  
- Endpoint detection alerts  

Each alert will follow a structured triage and escalation process.

---

## 🔎 Investigation Methodology

All alerts will follow this standardized workflow:

1. **Alert Identification**
2. **Initial Triage**
   - False positive vs credible threat determination
3. **Log & Telemetry Analysis**
4. **MITRE ATT&CK Mapping**
5. **Containment & Remediation Recommendations**
6. **Formal Incident Documentation**

---

## 📂 Repository Structure

.   
├── blog/ # Weekly updates   
├── docs/ # Project scope, methodology, final summary   
├── playbooks/ # SOC investigation playbooks   
├── incidents/ # Formal incident reports   
├── detections/ # Detection logic and alert criteria   
├── scripts/ # Log parsing / automation scripts   
├── screenshots/ # SIEM dashboards and investigation evidence   
└── README.md

---

## 🛠️ Tools (Planned)

- SIEM platform (Splunk / Microsoft Sentinel)
- Windows & Linux virtual machines
- MITRE ATT&CK Framework
- Python / Bash for log analysis
- GitHub for documentation and version control

---

## 📊 Deliverables

- Alert triage documentation  
- Investigation timelines  
- Indicators of Compromise (IOCs)  
- SOC Playbooks:
  - Phishing Investigation
  - Suspicious Login / Brute Force
  - Malware / Endpoint Alert
- Detection improvement recommendations  
- Executive-style SOC summary report  

---

## 📅 Project Phases

| Phase | Focus |
|-------|-------|
| Phase 1 | Planning & Lab Setup |
| Phase 2 | Alert Simulation |
| Phase 3 | Investigation & Reporting |
| Phase 4 | Detection Tuning & Optimization |
| Phase 5 | Final Summary & Lessons Learned |

---

## 📈 Success Criteria

- 10+ alerts investigated
- Clear false positive analysis included
- MITRE ATT&CK mapping for confirmed threats
- Consistent escalation documentation
- Repeatable investigation workflow established

---

## 🎓 Purpose

This project is designed to simulate entry-level SOC analyst responsibilities while developing technical analysis, documentation discipline, and security process improvement skills.

---

## 📖 Blog Progress

- [Week 1 - Lab Setup](blog/week-1-setup.md)

*Project in Progress – Updates will be committed throughout the 60-day timeline.*
