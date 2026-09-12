# Wazuh-SIEM-with-Detection-Rules-MyFirstHack
Within Cybersecurity, Telemetry is one of the fundamental aspects to catch and detect threats to a syste,. This project creates a Wazuh SIEM: allowing the accumulation and logging of events from attacks on a Windows victim VM to test the SIEM. This project provided a great opportunity to familiarize with Sigma-style detection rules, creating a victim VM, telemetry, and running security tests.

**Author:** @J-Hwang7 **Date** May 2026

# Overview
For the project, I deployed Wazuh with Docker, created a target Windows 10 VM with Sysmon, installed a Wazuh agent on the VM, wrote three MITRE detection rules, converted the detection rules into Sigma-signature style, and tested each rule with Atomic Red Team.

# How it works 
Wazuh acts as a data collection sensor at endpoints (VM), meaning it can receive security logs and changes. By utilizing custom MITRE ATT&CK detection rules, the SIEM is able to create alerts based on specific events. MITRE detection rules serve to ensure that not every activity is flagged by the SIEM, as specific actions that break the detection rules will create an alert

# Simulating Attacks

1. Install Atomic Red Team onto the VM
     * Run the following commands in the VM Administrator PowerShell terminal
```
Set-ExecutionPolicy Bypass -Scope Process -Force
Install-Module -Name Invoke-AtomicRedTeam -Force
Import-Module Invoke-AtomicRedTeam

//Download Atomic Red Team Tests
mkdir \AtomicRedTeam
Install-AtomicRedTeam -InstallPath [path to AtomicRedTeam directory]
```
2. Run Atomic Red Team Tests with the following commands
```
//For each detection rule
Invoke-AtomicTest [MITRE of detection rule]
```
3. Check the logs of the attacks
     * Navigate to Wazuh
     * On the Wazuh dashboard, enter Security events
Example logs: [Screenshots](Screenshots)
# Detection Rules
For the project, I created three detection rules that follow the Sysmon EventID 1(ProcessCreate). 
* Detection Rules Used: [local_rules.xml](local_rules.xml)

**Detection Rule 100001 - Scheduled task creation detected**

**MITRE:** T1053.005

**Level 10**

**Description:** Monitors for suspicious task creation and execution

\
**Detection Rule 100002 - Registry Run key modification**

**MITRE:** T1547.001 

**Level 12**

**Description:** Monitors for suspicious modifications to the Windows Registry Run keys. Helps to catch malware.

\
**Detection Rule 100003 - Firewall Manipulation**

**MITRE:** T1562.004

**Level 11**

**Description:** Monitors for attempts to bypass or alter the system's firewall

# What I learned
- Telemetry is not protection but detection
     * Detection Rules are only able to notify, not prevent.
     * If detection rules are written incorrectly, it could lead to SIEM logging failure.
- Conflicts with the GUI and CLI installation can cause failure of the application setup
     * When setting up the Wazuh agent on the VM, I attempted to install the agent on the CLI when I had already installed the agent on the GUI. This resulted in a service conflict.
     * Residual files from the GUI installation prevented the CLI from properly downloading 
- Absence of toggle protection leads to modification of the device
     * Through the Atomic Red Team security tests, I witnessed core functions being tampered with.
     * According to Wazuh, the VM received modifications from threats outside of the security test. (Even though I had not used the VM)
       
# Licenses
The content of this repository is to be used for educational purposes. Sysmon config, Atomic Red Team, and sigma have their own licenses (View Respective GitHub repositories).
