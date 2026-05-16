# Wazuh-SIEM-with-Detection-Rules-MyFirstHack
This home project creates a Wazuh SIEM: allowing the accumulation and logging of events from attacks on a Windows victim VM, creating custom Sigma detection rules, and utilizing Atomic Red Team to test the SIEM. This project provided a great opportunity to familiarize myself with Sigma-style detection rules, creating a victim VM, and telemetry.

**Author:** @J-Hwang7 **Date** May 2026

# Overview
For the project, I deployed Wazuh with Docker, created a target Windows 10 VM with Sysmon, installed a Wazuh agent on the VM, wrote three MITRE detection rules, and tested each rule with Atomic Red Team.

# Wazuh SIEM Procedures
**Prequiste Installations**
- Docker Desktop
- Windows 10 or 11 iso file
- VirtualBox or VMware
- Sysmon v15.2

\
**Deploying Wazuh**

In a Linux terminal, run these commands.
```
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.0;
cd wazuh-docker/single-node;
docker-compose -f generate-indexer-certs.yml run -rm generator
```

Afterwards, navigate to the browser and search https://localhost

**Login credentials for Wazuh**
- Username: admin
- Password: SecretPassword

\
**Creating Target VM**

1. In VirtualBox or VMware, create a new VM with the Windows iso file.
     * Set BaseMemory to 4096MB
     * Set Number of CPUs to 2
     * Set Disk Size to 80.0GB
     
2. In the VM, navigate to Windows Security.
    - Next, navigate to Virus & threat protection.
    - Afterwards, navigate to virus & threat protection settings and disable Tamper protection.
3. In an Administrator PowerShell terminal on the VM, run these commands to disable Windows Defender. (Allows Atomic Red Team security tests to function)
```
Set-MpPreference -DisableRealtimeMonitoring $true -DisableBehaviorMonitoring $true -DisableIOAVProtection $true -DisableScriptScanning $true
Get-MpPreference | Select-Object DisableRealtimeMonitoring, DisableBehaviorMonitoring
```
4. In an Administrator PowerShell terminal, install Sysmon with the SwiftOnSecurity config with these commands
```
cd \Sysmon
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml" -OutFile "sysmonconfig.xml"
.\sysmon64.exe -accepteula -i sysmonconfig.xml
```
5. Navigate to https://localhost and click on add agent.
    * Select "MSI 32/64 bits" as the package to download and install
    * In an Administrator PowerShell terminal, run the following command to receive the host IP address
      ```
      ipconfig
      ```
    * Host IP should be under **Ethernet adapter vEthernet (WSL (Hyper-V firewall)):** and **IPv4 Address**
    * Enter the host IP address as the server address
    * Assign "WindowsVictim" as the agent name
    * Run the prompted commands in the Victim VM's PowerShell
      
\
**Creating Sigma Detection rules**

1. Write each MITRE detection rule into an .xml file   
2. Run the following commands to convert each detection rule into Sigma-style 
```
pip install sigmatools
//For each detection rule file, run this command
sigma convert -t wazuh [detection rule file name].xml
```
* Output should be similar to detection rules in [local_rules.xml](local_rules.xml)
3. Place all sigma detection rules into one file
     * In **/var/ossec/etc/rules** directory, create the file **local_rules.xml** 
         * Copy contents of each Sigma detection rule into **local_rules.xml**        
4. Run the following command to restart Wazuh to utilize the rules
```
docker-compose restart wazuh.manager
```

**Simulating Attacks**

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
     * Navigate to https://localhost
     * On the Wazuh dashboard, enter Security events

# Detection Rules
For the project, I created three detection rules that follow the Sysmon EventID 1(ProcessCreate). 
* Detection Rules Used: [local_rules.xml](local_rules.xml)

**Detection Rule 100001 - Scheduled task creation detected**
**MITRE:** T1053.005
**Level 10**
**Description:** Monitors for suspicious task creation and execution

**Detection Rule 100002 - Registry Run key modification**
**MITRE:** T1547.001 
**Level 12**
**Description:** Monitors for suspicious modifications to the Windows Registry Run keys. Helps to catch malware.

**Detection Rule 100003 - Firewall Manipulation**
**MITRE:** T1562.004
**Level 11**
**Description:** Monitors for attempts to bypass or alter the system's firewall
