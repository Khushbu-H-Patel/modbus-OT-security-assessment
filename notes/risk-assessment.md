# OT/ICS Modbus/TCP Risk Assessment

## Project Context

This risk assessment is based on a controlled virtual OT/ICS lab using OpenPLC as a simulated PLC and Kali Linux as an assessment workstation.

The objective was to understand how an exposed Modbus/TCP service could be identified, interacted with, and assessed from a security risk perspective. All testing was performed in an isolated lab environment.

## Lab Summary

| Component | Description |
|---|---|
| Simulated PLC | Ubuntu VM running OpenPLC |
| Assessment Workstation | Kali Linux VM |
| Target IP | `192.168.79.148` |
| Assessment IP | `192.168.79.149` |
| Protocol Tested | Modbus/TCP |
| Port | TCP `502` |
| Main Tools Used | Nmap, Metasploit, Wireshark |

## Risk Assessment Methodology

This project used a lightweight qualitative risk assessment approach based on likelihood, impact, and overall risk rating.

The assessment was not intended to represent a full formal audit or compliance assessment. Instead, it was used to evaluate the security relevance of the lab findings in a practical OT/ICS context.

The approach was inspired by common NIST-style risk assessment concepts, where risk is evaluated by considering:

- The likelihood that a weakness could be exploited
- The potential impact if exploitation occurs
- The operational consequence to the environment

For the OT/ICS context, the assessment also considered safety, availability, reliability, and process impact, not only confidentiality.

Risk ratings used in this project:

| Rating | Meaning |
|---|---|
| Low | Limited likelihood or limited operational impact |
| Medium | Plausible likelihood or moderate operational impact |
| High | Directly exploitable condition with potentially serious operational impact |

This lab used a qualitative rating model rather than numerical scoring because the environment was simulated and did not represent a real production process.

## Key Finding

The Kali assessment workstation was able to directly communicate with the OpenPLC simulated PLC over Modbus/TCP on port `502`.

The assessment confirmed that Kali could:

- Identify Modbus/TCP using Nmap.
- Read coil values from the PLC.
- Read holding register values from the PLC.
- Write a new value to holding register address `0`.
- Confirm the changed value through Metasploit, Ubuntu local validation, and Wireshark packet capture.

## Technical Evidence Summary

| Test Area | Evidence |
|---|---|
| Nmap enumeration | Port `502` identified as Modbus/TCP |
| Modbus discovery | `modbus-discover` confirmed Modbus reachability |
| Metasploit read test | Holding register address `0` returned `[0]` |
| Metasploit write test | Register address `0` changed from `0` to `75` |
| Read-back verification | Register address `0` returned `[75]` after write |
| Ubuntu-side validation | Local Modbus read confirmed register value `75` |
| Wireshark analysis | Function code `6`, Write Single Register, observed in traffic |

## Risk Statement

If an unauthorized workstation can directly access a PLC over Modbus/TCP, it may be able to read or modify process-related values. In this lab, the Kali workstation successfully wrote the value `75` to holding register address `0`, which represented a simulated `TankLevel` value in OpenPLC.

In a real OT/ICS environment, the impact would depend on what the register controls or represents. Unauthorized modification of process values could affect process visibility, equipment behavior, reliability, or safety.

## Risk Rating

Using the qualitative methodology described above, the main risk was rated as follows:

| Category | Rating | Justification |
|---|---|---|
| Likelihood | Medium | The risk depends on whether unauthorized systems can reach the PLC network. In this lab, Kali had direct network access to Modbus/TCP. |
| Impact | High | Unauthorized writes to PLC registers could affect industrial process behavior depending on the controlled equipment. |
| Overall Risk | High | Direct Modbus write access from an unauthorized workstation creates a serious OT security concern. |

## Risk Register

| ID | Finding | Risk | Potential Impact | Likelihood | Impact | Recommendation |
|---|---|---|---|---|---|---|
| R1 | Modbus/TCP was reachable from the Kali workstation | Unauthorized PLC access | An unauthorized host may read process values | Medium | Medium | Restrict TCP `502` to approved engineering workstations and HMIs only |
| R2 | Holding register value could be modified | Unauthorized process value manipulation | Process values or control parameters may be changed | Medium | High | Use segmentation, firewall rules, and strict access control |
| R3 | Modbus write activity was visible on the network | Lack of monitoring could delay detection | Unauthorized changes may go unnoticed | Medium | High | Monitor for Modbus write function codes such as function code `6` |
| R4 | OpenPLC web interface was reachable from Kali | Management interface exposure | Unauthorized users may attempt access to PLC management functions | Medium | Medium | Restrict management interfaces to trusted administrative systems |
| R5 | Additional OT-related service on port `44818` was exposed | Expanded attack surface | More exposed services increase assessment and monitoring scope | Low | Medium | Review required services and disable unused protocols where possible |

## OT/ICS Impact Considerations

In IT environments, security findings are often evaluated mainly around confidentiality, integrity, and availability. In OT/ICS environments, the assessment must also consider operational and physical process impact.

For this lab, the modified register represented a simulated tank-level value. In a real environment, similar values could represent process measurements, setpoints, equipment states, or control parameters.

Possible OT/ICS consequences could include:

- Incorrect process visibility for operators.
- Unauthorized modification of control values.
- Equipment operating outside expected conditions.
- Disruption to normal process behavior.
- Increased safety or reliability concerns depending on the process.

## Recommended Controls

### 1. Network Segmentation

The PLC network should be separated from general IT/user networks. Only approved engineering workstations, HMIs, or authorized systems should be able to communicate with PLCs.

### 2. Firewall Access Control

Firewall rules should restrict access to Modbus/TCP port `502`.

Example control objective:

```text
Allow TCP/502 only from approved HMI or engineering workstation IP addresses.
Deny TCP/502 from all other systems.
```

### 3. Protocol Monitoring

Security monitoring should include detection of Modbus write operations.

Important Modbus function codes to monitor include:

| Function Code | Name | Why it matters |
|---:|---|---|
| `3` | Read Holding Registers | May indicate process data collection |
| `6` | Write Single Register | Can modify process-related values |
| `16` | Write Multiple Registers | Can modify multiple process values |

### 4. Asset Inventory and Communication Baseline

Maintain an approved inventory of OT assets and expected communication flows.

Examples:

| Asset | Expected Communication |
|---|---|
| HMI | Allowed to communicate with PLC on TCP/502 |
| Engineering Workstation | Allowed only when required |
| User Workstations | Should not communicate directly with PLC |
| Unknown Hosts | Should be blocked or investigated |

### 5. Restrict Management Interfaces

The OpenPLC web interface was reachable on ports `8080` and `8443` in the lab. In a real OT environment, PLC management interfaces should not be broadly accessible.

Access should be limited to trusted administrative systems and protected with strong authentication.

### 6. Change Control and Logging

Changes to PLC logic, registers, or configuration should follow a formal change-control process. Write activity should be logged and reviewed where possible.

## Detection Ideas

The following detection ideas could be developed from this lab:

| Detection Idea | Logic |
|---|---|
| Unauthorized Modbus access | Alert when a non-approved source IP communicates with PLC TCP/502 |
| Modbus write activity | Alert on Modbus function code `6` or `16` |
| New PLC communication source | Alert when a new host communicates with a PLC for the first time |
| Management interface access | Alert on unexpected access to PLC web management ports |
| Scanning activity | Alert on repeated connection attempts to OT protocol ports |

## Final Assessment Summary

The lab demonstrated that when Modbus/TCP is reachable from an unauthorized workstation, PLC process values may be read or modified. The controlled write test changed holding register address `0` from `0` to `75`, and the change was confirmed using Metasploit, a local Ubuntu validation script, and Wireshark packet analysis.

The main risk is not the OpenPLC lab system itself, but the broader OT/ICS security lesson: direct and unrestricted access to industrial protocols can create opportunities for unauthorized process manipulation. In a real environment, this risk should be reduced through segmentation, firewall restrictions, protocol monitoring, asset inventory, and strong change-control practices.
