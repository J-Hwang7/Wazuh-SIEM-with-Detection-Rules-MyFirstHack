# Wazuh-SIEM-with-Detection-Rules
This home project creates a Wazuh SIEM: allowing the accumulation and logging of events from attacks on a Windows victim VM, creating custom Sigma detection rules, and utilizing Atomic Red Team to test the SIEM. This project provided a great opportunity to familiarize myself with Sigma rules, creating a victim VM, and telemetry.

**Author:**@J-Hwang7

# Overview
For the project, I deployed Wazuh with Docker, created a target Windows 10 VM with Sysmon, installed a Wazuh agent on the VM, wrote three MITRE detection rules, and tested each rule with Atomic Red Team.

# Wazuh SIEM Procedures
**Prequiste Installations**
- Docker Desktop
- Windows 10 or 11 iso file
- VirtualBox or VMware

**Deploying Wazuh**
``` git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.0 
```
