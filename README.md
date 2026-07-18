# Modbus/TCP OT Security Assessment Lab

## Project Overview

This project is a small OT/ICS security assessment lab built in a virtual environment. The goal was to simulate a basic industrial control system using OpenPLC and assess the security risk of exposed Modbus/TCP access.

The lab demonstrates how an assessment workstation can identify an exposed Modbus/TCP service, read PLC data, perform a controlled write to a holding register, validate the change, capture the network traffic, and document the associated OT/ICS risk.

This project was completed as a hands-on learning exercise to strengthen understanding of OT/ICS security assessment concepts, industrial protocols, and risk-based analysis.

---

## Project Objective

The objective of this project was to build a small virtual OT/ICS lab and assess the security risk of direct Modbus/TCP access to a simulated PLC. The project focused on understanding how exposed industrial services can be discovered, how Modbus data can be read or modified in a controlled lab, and how technical findings can be translated into OT/ICS risk, impact, and defensive recommendations.

The project included:

- Building an isolated virtual OT/ICS lab using Ubuntu, OpenPLC, and Kali Linux.
- Configuring OpenPLC as a simulated PLC with mapped process variables.
- Enumerating exposed services using Nmap and Nmap NSE scripts.
- Performing read-only Modbus/TCP testing using Metasploit.
- Performing a controlled write test against a simulated holding register.
- Validating the register change through read-back testing and local Ubuntu verification.
- Capturing and analyzing Modbus/TCP traffic using Wireshark.
- Documenting the security risk, OT/ICS impact, and recommended defensive controls.

---

## Lab Environment

| Role | System | IP Address | Purpose |
|---|---|---|---|
| Simulated PLC / Target | Ubuntu Desktop 22.04.5 with OpenPLC | `192.168.79.148` | Runs OpenPLC and exposes Modbus/TCP |
| Assessment Workstation | Kali Linux | `192.168.79.149` | Runs Nmap, Metasploit, and Wireshark |

Both virtual machines were configured on the same virtual NAT network for lab communication.

---

## Tools Used

| Tool | Purpose |
|---|---|
| OpenPLC | Simulated PLC runtime |
| Ubuntu Desktop | Host system for OpenPLC |
| Kali Linux | Assessment workstation |
| Nmap | Port scanning and service enumeration |
| Nmap NSE Scripts | Modbus, EtherNet/IP, and HTTP enumeration |
| Metasploit | Modbus/TCP read and write testing |
| Wireshark | Packet capture and protocol analysis |
| Python | Local Modbus validation on Ubuntu |

---

## Lab Architecture

The lab was built using two virtual machines on the same virtual NAT network. Ubuntu/OpenPLC acted as the simulated PLC, while Kali Linux was used as the assessment workstation for enumeration, Modbus testing, and packet capture.

![Lab Architecture](lab-architecture/lab-architecture.png)

---

## Simulated PLC Program

A simple OpenPLC Structured Text program was created to define simulated process values.

```text
Pump      AT %QX0.0 : BOOL;
Valve     AT %QX0.1 : BOOL;
TankLevel AT %MW0   : INT;
```

For the lab scenario:

| Variable | Address | Type | Meaning |
|---|---|---|---|
| `Pump` | `%QX0.0` | Boolean | Simulated pump output |
| `Valve` | `%QX0.1` | Boolean | Simulated valve output |
| `TankLevel` | `%MW0` | Integer | Simulated tank-level register |

The `TankLevel` value was used for the controlled Modbus write test.

---

## Repository Structure

```text
modbus-ot-security-assessment/
│
├── README.md
│
├── setup/
│   ├── setup.md
│   └── screenshots/
│
├── nmap-results/
│   ├── nmap-enumeration-summary.md
│   ├── output-files/
│   └── screenshots/
│
├── metasploit-results/
│   ├── metasploit-commands-used.md
│   └── screenshots/
│
├── wireshark-results/
│   ├── wireshark-analysis.md
│   ├── pcaps/
│   └── screenshots/
│
└── notes/
    ├── evidence-index.md
    └── risk-assessment.md
```

---

## Assessment Workflow

The project followed a simple OT/ICS assessment workflow:

```text
1. Build isolated virtual lab
2. Install and configure OpenPLC
3. Confirm Modbus/TCP service is running
4. Enumerate exposed services with Nmap
5. Perform Modbus read testing with Metasploit
6. Perform controlled Modbus write testing
7. Validate register value change
8. Capture Modbus traffic with Wireshark
9. Assess OT/ICS security risk
10. Document findings and recommendations
```

---

## Key Technical Results

### 1. OpenPLC Setup

OpenPLC was installed and configured on Ubuntu. The PLC runtime was started successfully, and Modbus/TCP was confirmed listening on port `502`.

Evidence:

- [Lab setup documentation](setup/setup.md)

---

### 2. Nmap Enumeration

Nmap identified the following open services on the OpenPLC target:

```text
502/tcp   open  modbus
8080/tcp  open  http
8443/tcp  open  ssl/http
44818/tcp open  EtherNetIP-2?
```

The service scan confirmed that TCP port `502` was running Modbus/TCP.

Evidence:

- [Nmap enumeration summary](nmap-results/nmap-enumeration-summary.md)

---

### 3. Modbus-Specific Discovery

The `modbus-discover` Nmap NSE script confirmed that Modbus/TCP was reachable on port `502`.

The target returned:

```text
ILLEGAL FUNCTION
```

This indicated that the OpenPLC lab target responded to the Modbus request but did not support the specific device-identification function used by the script.

---

### 4. Metasploit Read Testing

Metasploit’s Modbus client module was used to read coil and holding register values.

Module used:

```text
auxiliary/scanner/scada/modbusclient
```

Read coils result:

```text
[0, 0]
```

Read holding register result:

```text
[0]
```

Interpretation:

- `Pump` and `Valve` were both initially `0` / `FALSE`.
- `TankLevel` was initially `0`.

Evidence:

- [Metasploit Modbus test results](metasploit-results/metasploit-commands-used.md)

---

### 5. Controlled Write Test

A controlled Modbus write test was performed against holding register address `0`.

The register value was changed from:

```text
0
```

to:

```text
75
```

Metasploit confirmed:

```text
Value 75 successfully written at registry address 0
```

A read-back test confirmed:

```text
[75]
```

The value was later restored back to:

```text
[0]
```

---

### 6. Ubuntu Local Validation

A local Python Modbus/TCP read was executed on the Ubuntu/OpenPLC VM to confirm that the value changed inside the OpenPLC runtime.

Result after write:

```text
Holding register 0 value: 75
```

Result after restore:

```text
Holding register 0 value: 0
```

This confirmed that the register change was reflected on the simulated PLC side.

---

### 7. Wireshark Packet Analysis

Wireshark captured the Modbus/TCP read and write traffic between Kali and OpenPLC.

Observed Modbus function codes:

| Function Code | Name | Observed Activity |
|---:|---|---|
| `3` | Read Holding Registers | Kali read holding register address `0` |
| `6` | Write Single Register | Kali wrote value `75` to holding register address `0` |

Evidence:

- [Wireshark Modbus/TCP traffic analysis](wireshark-results/wireshark-analysis.md)

---

## Key Finding

The Kali assessment workstation was able to directly communicate with the OpenPLC simulated PLC over Modbus/TCP. It successfully read PLC values and performed a controlled write to holding register address `0`.

The main finding is:

> Direct access to Modbus/TCP can allow a workstation to read or modify process-related PLC values when access controls and segmentation are not enforced.

---

## Risk Assessment Summary

A lightweight qualitative risk assessment approach was used. The assessment considered likelihood, impact, and OT/ICS-specific consequences such as safety, availability, reliability, and process impact.

| Category | Rating | Justification |
|---|---|---|
| Likelihood | Medium | The risk depends on whether unauthorized systems can reach the PLC network. In this lab, Kali had direct network access to Modbus/TCP. |
| Impact | High | Unauthorized writes to PLC registers could affect industrial process behavior depending on the controlled equipment. |
| Overall Risk | High | Direct Modbus write access from an unauthorized workstation creates a serious OT security concern. |

Detailed assessment:

- [OT/ICS Modbus/TCP risk assessment](notes/risk-assessment.md)

---

## Defensive Recommendations

Based on the lab findings, relevant defensive controls include:

- Restrict TCP port `502` to approved engineering workstations and HMIs only.
- Segment OT networks from general IT or user networks.
- Monitor for Modbus write function codes such as `6` and `16`.
- Alert when new or unauthorized hosts communicate with PLC assets.
- Restrict access to PLC management interfaces.
- Maintain an OT asset inventory and expected communication baseline.
- Use formal change-control procedures for PLC logic and process value changes.

---

## Evidence Index

A short evidence index was created to summarize the major evidence collected during the project.

Evidence index:

- [Evidence index](notes/evidence-index.md)

---

## Project Scope and Limitations

This project was intentionally limited in scope so it could be completed quickly and documented clearly.

The project did not include:

- A real physical PLC.
- A production OT network.
- Full IEC 62443 or NIST SP 800-82 assessment.
- Full firewall mitigation testing.
- Full IDS/SIEM detection engineering.
- Multi-protocol OT monitoring beyond the selected Modbus/TCP testing.

The lab was designed as a controlled learning project, not a production-grade assessment.

---

## Safety and Authorization Note

All testing was performed only inside an isolated virtual lab environment owned and controlled by the project author.

The techniques shown in this project should only be used in environments where explicit authorization has been granted.

In real OT/ICS environments, active scanning and protocol testing should be carefully planned and approved because some industrial systems may be sensitive to unexpected traffic.

---

## Interview Summary

This project demonstrates hands-on interest in OT/ICS security risk assessment.

The project shows the ability to:

- Build a small virtual OT/ICS lab.
- Configure OpenPLC as a simulated PLC.
- Identify exposed industrial services.
- Use Nmap for OT service enumeration.
- Use Metasploit for controlled Modbus read/write testing.
- Validate findings using packet capture.
- Translate technical findings into OT/ICS risk language.
- Recommend practical defensive controls.

A concise interview explanation:

> I built a small virtual OT/ICS lab using Ubuntu with OpenPLC as a simulated PLC and Kali as the assessment workstation. I used Nmap to identify exposed services, including Modbus/TCP on port 502. I then used Metasploit’s Modbus client module to perform read-only testing and a controlled write to a simulated tank-level register. Wireshark confirmed the Modbus function codes and packet-level activity. Finally, I documented the risk using a lightweight qualitative risk assessment approach and recommended controls such as segmentation, access restriction, monitoring for Modbus write functions, and asset communication baselining.

---

## Detailed Documentation

| Section | Link |
|---|---|
| Lab setup | [setup/setup.md](setup/setup.md) |
| Nmap enumeration | [nmap-results/nmap-enumeration-summary.md](nmap-results/nmap-enumeration-summary.md) |
| Metasploit testing | [metasploit-results/metasploit-commands-used.md](metasploit-results/metasploit-commands-used.md) |
| Wireshark analysis | [wireshark-results/wireshark-analysis.md](wireshark-results/wireshark-analysis.md) |
| Risk assessment | [notes/risk-assessment.md](notes/risk-assessment.md) |
| Evidence index | [notes/evidence-index.md](notes/evidence-index.md) |

---

## Final Outcome

The project successfully demonstrated a basic OT/ICS assessment lifecycle:

```text
Discovery → Enumeration → Read Test → Controlled Write Test → Packet Analysis → Risk Assessment
```

The most important outcome was understanding that exposed Modbus/TCP access can create risk when unauthorized systems can communicate directly with PLC assets.

This lab reinforced the importance of secure network architecture, segmentation, access control, monitoring, and risk-based thinking in OT/ICS environments.
