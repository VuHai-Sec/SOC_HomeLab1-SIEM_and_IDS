---
title: 'Technical Documentation - Homelab1: SIEM + IDS'

---

## Overall

This is my home-lab SOC system, including:
- SIEM: Wazuh
- IDS: Suricata

## Project goals
The primary goals of this home-lab are to:
- Deploy a centralized SIEM platform - Wazuh
- Deploy an IDS (Suricata) and Wazuh Agent on victim machine
- Simulate cyber attacks and check Suricata & Wazuh detection and responses
- Develop hands-on troubleshooting skills by identifying and resolving technical issues during the system's setup and deployment.

## Lab architecture
<img width="879" height="277" alt="image" src="https://github.com/user-attachments/assets/79f803ca-d3b4-4cee-937e-c76459b5133a" />




### Devices used:
- Ubuntu Server (Wazuh_server): Contains Wazuh Indexer, Wazuh Manager and Wazuh Dashboard
- Another Ubuntu Server (Wazuh_agent): Contains Suricata and Wazuh Agent, simulate victim device
- Kali Linux: Simulate attack device

| Device | Description | IP Address | Note (also known as) |
| :--- | :--- | :--- | :--- |
| Ubuntu Server | SIEM (Wazuh) | 10.0.2.6 | Wazuh Server|
| Ubuntu Server | IDS (Suricata) | 10.0.2.8 | Victim |
| Kali Linux | Attacker Device | 10.0.2.9 | Attacker |

### Technique used:
- Wazuh: An open-source HIDS and SIEM platform
- Suricata: An open-source NIDS, known for balance and suitable for Wazuh.

### Setup process:
#### Step 1: Install Wazuh Server (all)
In the device's terminal (Linux), run:
```
$ curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
Once finished, access the Wazuh Dashboard via: 
```
https://<WAZUH_SERVER_IP>
```

#### Step 2: Install Wazuh agent
On Dashboard, select Wazuh agent -> Deploy new agent and follow the instructions

#### Step 3: Install Suricata
In the device's terminal (Linux), run:
```
$ sudo add-apt-repository ppa:oisf/suricata-stable
$ sudo apt-get update
$ sudo apt-get install suricata -y
```
This home-lab uses ET ruleset for Suricata. Download it from:
https://rules.emergingthreats.net/open/suricata-6.0.8/
Then open *suricata.yaml* and configure:
- Rule files
- Log files (IDS log and Suricata self-log)
*(more detail in docs/)*

#### Step 4: Integrate with Wazuh:
Modify *ossec.conf* to make Wazuh Agent read *eve.json*, since it is in json format
*(more detail in docs/)*

## Test
### Port scanning
On Attacker, start scanning Victim device using nmap tool:
```
$ nmap -sS -sV -Pn 10.0.2.8
```
<img width="931" height="242" alt="image 1 nmap" src="https://github.com/user-attachments/assets/02fdb464-8867-4e9d-afb7-1f7f2006e240" />


On Victim device, view *eve.json* to see Suricata detection result:
<img width="630" height="604" alt="image 2 eve_json" src="https://github.com/user-attachments/assets/3d934c76-a2ac-4ecf-b635-5675d9c71120" />


On Wazuh Dashboard, view that alert:
<img width="906" height="526" alt="image 3 Alerts on Wazuh Dashboard" src="https://github.com/user-attachments/assets/e6ce3cde-6b59-4e2f-9d83-2570d24b9f84" />



## Troubleshoot:
### Set Wazuh Manager IP
- Problem: Once installed Wazuh agent, Wazuh Manager did not recognize new agent from agent. Agent device failed to register with Manager.
- Investigation: On Agent device, *ossec.log* indicated a connection failure. In *ossec.conf*, it was noticed that **MANAGER_IP** was set to default.
- Troubleshooting: *ossec.conf* was updated with correct **MANAGER_IP**, then restarted Wazuh Agent
<img width="726" height="303" alt="image 4 Troubleshoot" src="https://github.com/user-attachments/assets/b0784335-2945-4f12-9f59-b18fe4112570" />

- Verify: Agent's status changed to Active. 

### Permission issues
- Problem: In Victim device, Suricata detected the attack and wrote logs to *eve.json* as usual. However, Wazuh Dashboard did not get any alerts, though *ossec.conf* had been configured properly.
- Investigation: On victim device, *ossec.log* revealed a permission issue. It **could not open eve.json due to lack of permission**
```
Wazuh-logcollector: ERROR: (1103): Could not open file '/var/log/suricata/eve.json' due to [(13)-Permission denied]
```
- Troubleshooting: Give wazuh user permission to view *eve.json* by adding it into suricata group:
```
$ sudo usermod -aG suricata wazuh
$ sudo chmod 640 /var/log/suricata/eve.json
```
Then restart Suricata and Wazuh Agent
- Verification: Alerts were sucessfully shown on Dashboard

## What is included:
- /docs: config files
- /images: images
- /results: result files

