# SOC Home Lab — SIEM Detection (Splunk)

## Objective
Demonstrate an end-to-end SOC workflow in a controlled home lab: simulate a basic payload delivery and post-execution activity between an attacker VM and a Windows victim, collect Windows telemetry (Sysmon / event logs) into Splunk, and create searches/dashboards to detect and analyze the activity.

---

## Environment / Tools 
- **Windows VM (victim)** — Splunk (collector / instance) + Sysmon configured for detailed process creation and command-line telemetry.  
- **Kali Linux VM (attacker)** — used to simulate delivery and control of a payload (testing only).  
- **Splunk** — SIEM for ingesting, searching, and visualizing Windows telemetry.  
- **Sysmon** — Windows process creation / command-line visibility.  
- **msfvenom / Metasploit (msfconsole)** — used in the lab environment to generate/handle a test payload for simulation purposes only.  
- **Python simple HTTP server** — used to host the test payload in the isolated lab.

> Note: Offensive tools were used only to generate realistic telemetry for detection testing. This repository focuses on the defensive detection, analysis, and lessons learned. No exploit code or step-by-step offensive instructions are included.

---

## Lab flow
1. Prepare two isolated VMs: Kali (attacker) and Windows (victim with Sysmon + Splunk).  
2. Simulate payload transfer from attacker → victim (hosted via a simple HTTP server in the lab).  
3. Execute the simulated payload on the Windows host in the lab environment.  
4. Post-execution, collect process creation and command-line events (examples observed: `net user`, `net localgroup`, `ipconfig`) in Splunk.  
5. Use Splunk searches and dashboards to detect and build a timeline of the suspicious activity.

---

## What I observed / Evidence
- **Process creation events** captured the executed binary and full `CommandLine` arguments.  
- **Command-line artifacts** included enumeration commands that aligned with post-compromise behavior (`net user`, `net localgroup`, `ipconfig`).  
- **Correlation in Splunk** allowed pivoting from the initial download/new executable to subsequent suspicious commands and host context.

---

## Example defensive Splunk searches
Find suspicious enumeration commands:
```spl
index=endpoint {bdb8f82f-e70b-68df-0e0c-000000001000}
| search CommandLine="*net user*" OR CommandLine="*net localgroup*" OR CommandLine="*ipconfig*"
| table _time,ParentImage,Image,CommandLine
