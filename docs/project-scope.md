# Project Scope – 60-Day SOC Lab

## 1. Purpose

The purpose of this lab is to simulate a functional Tier 1 SOC analyst role by recreating real-world alert triage, investigation, documentation, and escalation workflows.

This lab will prioritize structured analysis, repeatable processes, and professional-grade documentation.

---

## 2. Lab Environment

### Virtual Machines
- 1 Windows VM (Primary endpoint)
- 1 Linux VM (Authentication & system log source)

### Logging Sources
- Windows Security Event Logs
- Windows PowerShell Logs
- Linux auth.log / syslog
- Simulated network activity logs

### Central Log Aggregation
- SIEM platform (TBD)

---

## 3. Simulated Attack Scenarios

The following scenarios will be executed and investigated:

1. Brute force login attempts
2. Suspicious PowerShell execution
3. Privilege escalation attempts
4. Simulated phishing-related activity
5. Unauthorized persistence mechanisms
6. Suspicious outbound network connections

---

## 4. Investigation Workflow

Each alert will follow this structured process:

1. Alert Triggered
2. Initial Triage
   - Determine severity
   - Validate legitimacy
3. Deep Investigation
   - Timeline construction
   - User/account review
   - Process and command-line analysis
   - Log correlation
4. MITRE ATT&CK mapping
5. Escalation or Closure
6. Incident Documentation

---

## 5. Deliverables

By Day 60, the following will be completed:

- 10+ documented alert investigations
- 3 full formal incident reports
- 3 SOC playbooks
- Detection tuning recommendations
- Executive summary report

---

## 6. Success Metrics

- Accurate alert classification (true positive vs false positive)
- Complete investigation documentation
- Proper MITRE technique mapping
- Clear escalation criteria defined
- Repeatable investigation methodology established

---

## 7. Constraints

- Lab environment only (no live production systems)
- Simulated attacks for learning purposes
- Focus on analysis and documentation over tool complexity
