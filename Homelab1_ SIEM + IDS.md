---
title: 'Homelab1: SIEM + IDS'

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
![{AEECF608-BD9E-4A25-9CBD-8BA27B0A60D0}](https://hackmd.io/_uploads/H1J8e3F6-l.png)



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
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
Once finised, open Wazuh Dashboard by access: 
```
https://<WAZUH_SERVER_IP>
```

#### Step 2: Install Wazuh agent
On Dashboard, select Wazuh agent -> Deploy new agent and follow the instructions

#### Step 3: Install Suricata
In the device's terminal (Linux), run:
```
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```
In this home-lab, I used ET ruleset. Download it from:
https://rules.emergingthreats.net/open/suricata-6.0.8/
Then open suricata.yaml and configure:
- Rule files
- Log files (IDS log and Suricata self-log)
*(more detail in docs/)*

#### Step 4: Integrate with Wazuh:
Modify *ossec.conf* to make Wazuh Agent read eve.json, since it is in json format
*(more detail in docs/)*

## Test
### Port scanning
On Attacker, start scanning Victim using nmap tool:
```
nmap -sS -sV Pn 10.0.2.8
```
(image 1: Running nmap)

On Victim, view eve.json to see Suricata detection result:
(image 2: Viewing eve.json)

On Wazuh Dashboard, view that alert:
(image 3: Alert on Wazuh Dashboard)


## Troubleshoot:
- Once installed Wazuh agent, if Wazuh Server does not recognize new agent, open ossec.conf in agent device, modify **MANAGER_IP**, then restart Wazuh Agent
(image 4: Modifying MANAGER_IP )

## What is included:
- /docs: config files
- /images: images
- /results: result files

## Note (delete before push)
- docs: suricata.yaml, ossec.conf.
- Problem: Không lấy được 2 file trên ra.