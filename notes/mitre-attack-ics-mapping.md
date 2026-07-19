# MITRE ATT&CK for ICS Mapping

## Project Context

This document maps the activities performed in the Modbus/TCP OT security assessment lab to relevant MITRE ATT&CK for ICS tactics and techniques.

The purpose of this mapping is to connect the hands-on lab activity to recognized OT/ICS adversary behaviors. This mapping is not intended to claim that a full adversary simulation was performed. Instead, it shows how the observed actions in the lab relate to ATT&CK for ICS techniques.

## Lab Summary

| Component | Description |
|---|---|
| Simulated PLC / Target | Ubuntu VM running OpenPLC |
| Assessment Workstation | Kali Linux VM |
| Target IP | `192.168.79.148` |
| Assessment IP | `192.168.79.149` |
| Protocol Tested | Modbus/TCP |
| Port | TCP `502` |
| Main Tools Used | Nmap, Metasploit, Wireshark |

---

## Mapping Summary

| Lab Activity | MITRE ATT&CK for ICS Tactic | Technique ID | Technique Name | Mapping Confidence |
|---|---|---:|---|---|
| Nmap scan against OpenPLC target | Discovery | `T0846.001` | Remote System Discovery: Port Scan | High |
| Identifying reachable OT services and open ports | Discovery | `T0840` | Network Connection Enumeration | Medium |
| Reading Modbus register values | Collection | `T0861` | Point & Tag Identification | Medium |
| Capturing Modbus/TCP packets in Wireshark | Collection | `T0842` | Network Sniffing | High |
| Writing value `75` to holding register address `0` | Impair Process Control | `T0836` | Modify Parameter | High |
| Sending Modbus write request to OpenPLC | Impair Process Control | `T1692.001` | Unauthorized Message: Command Message | Medium |
| Potential effect of register modification | Impair Process Control | `T0831` | Manipulation of Control | Related / Contextual |

---

## Detailed Technique Mapping

## 1. Remote System Discovery: Port Scan

| Field | Details |
|---|---|
| Tactic | Discovery |
| Technique | `T0846.001 - Remote System Discovery: Port Scan` |
| Lab Evidence | Nmap scan against `192.168.79.148` |
| Tool Used | Nmap |
| Confidence | High |

### Lab Activity

Nmap was used from the Kali assessment workstation to scan selected ports on the Ubuntu/OpenPLC target.

Command example:

```text
sudo nmap -Pn -p 502,8080,8443,44818 192.168.79.148
```

The scan identified exposed services, including:

```text
502/tcp   open  modbus
8080/tcp  open  http
8443/tcp  open  ssl/http
44818/tcp open  EtherNetIP-2?
```

### Why This Maps

This maps strongly to port scanning because the activity involved checking network ports on the target to identify reachable services. In the lab, this was used for authorized assessment and asset discovery.

### Defensive Relevance

In a real OT/ICS environment, port scanning should be monitored carefully because industrial devices may be sensitive to unexpected traffic. Detection opportunities include monitoring for hosts attempting connections to multiple OT ports or scanning PLC subnets.

---

## 2. Network Connection Enumeration

| Field | Details |
|---|---|
| Tactic | Discovery |
| Technique | `T0840 - Network Connection Enumeration` |
| Lab Evidence | Identification of exposed services and communication paths |
| Tool Used | Nmap |
| Confidence | Medium |

### Lab Activity

The assessment identified that Kali could reach the OpenPLC target over the virtual network and communicate with services including Modbus/TCP, HTTP/HTTPS, and EtherNet/IP-related service.

### Why This Maps

This maps to network connection enumeration because the lab identified reachable services and communication paths between the assessment workstation and the simulated PLC.

### Defensive Relevance

In a real OT environment, defenders should maintain a baseline of expected communication paths. Unexpected hosts communicating with PLCs or OT protocol ports should be investigated.

---

## 3. Point & Tag Identification

| Field | Details |
|---|---|
| Tactic | Collection |
| Technique | `T0861 - Point & Tag Identification` |
| Lab Evidence | Reading coil and holding register values |
| Tool Used | Metasploit Modbus client |
| Confidence | Medium |

### Lab Activity

Metasploit was used to read Modbus coil and holding register values from the OpenPLC target.

Read coils result:

```text
[0, 0]
```

Read holding register result:

```text
[0]
```

The holding register represented the simulated `TankLevel` variable mapped in OpenPLC as:

```text
TankLevel AT %MW0 : INT;
```

### Why This Maps

This is a medium-confidence mapping because the lab read process-related values from the simulated PLC. However, the lab did not perform broad tag discovery or full process mapping. Therefore, this should be treated as related to point/tag identification rather than a complete implementation of the technique.

### Defensive Relevance

Defenders should monitor for unusual hosts reading PLC values, unexpected increases in read volume, or queries to values that are not normally accessed by that system.

---

## 4. Network Sniffing

| Field | Details |
|---|---|
| Tactic | Collection |
| Technique | `T0842 - Network Sniffing` |
| Lab Evidence | Wireshark capture of Modbus/TCP traffic |
| Tool Used | Wireshark |
| Confidence | High |

### Lab Activity

Wireshark was used on the Kali assessment workstation to capture Modbus/TCP traffic generated during the read and write tests.

Display filter used:

```text
ip.addr == 192.168.79.148 && tcp.port == 502
```

The capture showed Modbus/TCP requests and responses between:

```text
Kali:           192.168.79.149
Ubuntu/OpenPLC: 192.168.79.148
```

### Why This Maps

This maps strongly to network sniffing because network packets were captured and inspected to observe protocol-level Modbus activity.

### Defensive Relevance

In authorized defensive work, packet capture is useful for validating traffic and creating detections. In adversarial use, network sniffing could reveal process values, protocol behavior, and communication patterns.

---

## 5. Modify Parameter

| Field | Details |
|---|---|
| Tactic | Impair Process Control |
| Technique | `T0836 - Modify Parameter` |
| Lab Evidence | Holding register address `0` changed from `0` to `75` |
| Tool Used | Metasploit Modbus client |
| Confidence | High |

### Lab Activity

Metasploit was used to write a new value to holding register address `0`.

Command example:

```text
set ACTION WRITE_REGISTER
set DATA_ADDRESS 0
set DATA 75
run
```

The output confirmed:

```text
Value 75 successfully written at registry address 0
```

Read-back verification confirmed:

```text
[75]
```

Ubuntu local validation also confirmed:

```text
Holding register 0 value: 75
```

### Why This Maps

This maps strongly to Modify Parameter because the lab changed a simulated process-related value. In the lab, register address `0` represented `TankLevel`, a simulated tank-level value.

### Defensive Relevance

Defenders should monitor for unexpected parameter changes, especially Modbus function code `6` and function code `16`. Alerting should consider source IP, destination PLC, function code, register address, value, and whether the source system is approved to make changes.

---

## 6. Unauthorized Message: Command Message

| Field | Details |
|---|---|
| Tactic | Impair Process Control |
| Technique | `T1692.001 - Unauthorized Message: Command Message` |
| Lab Evidence | Modbus write request sent from Kali to OpenPLC |
| Tool Used | Metasploit Modbus client |
| Confidence | Medium |

### Lab Activity

Kali sent a Modbus write request to OpenPLC using function code `6`, Write Single Register.

Wireshark showed:

```text
Function Code: 6 - Write Single Register
Reference Number: 0
Register Value: 75
```

### Why This Maps

This is a medium-confidence mapping because the lab generated a command-style Modbus message from a workstation that was not acting as an HMI or engineering workstation. However, because this was a controlled lab and OpenPLC was intentionally configured for testing, it should be documented as a lab simulation of unauthorized command behavior rather than a real unauthorized command in production.

### Defensive Relevance

In real OT networks, defenders should monitor for command messages from unexpected sources, especially when a non-approved host sends write commands to PLCs.

---

## 7. Manipulation of Control

| Field | Details |
|---|---|
| Tactic | Impair Process Control |
| Technique | `T0831 - Manipulation of Control` |
| Lab Evidence | Controlled modification of simulated `TankLevel` value |
| Tool Used | Metasploit Modbus client |
| Confidence | Related / Contextual |

### Lab Activity

The lab modified a simulated process value from `0` to `75`.

### Why This Is Contextual

This is included as a related technique because the register modification could represent manipulation of a process value in a real environment. However, this lab did not control physical equipment or change a real industrial process. Therefore, `T0831` should be treated as contextual impact mapping rather than a direct claim.

### Defensive Relevance

In a real environment, defenders should correlate write commands with operational context, expected operator actions, change windows, and process state changes.

---

## Techniques Considered but Not Directly Mapped

| Technique ID | Technique Name | Reason Not Directly Mapped |
|---:|---|---|
| `T0806` | Brute Force I/O | The lab did not repeatedly or successively change I/O point values. Only controlled single-register writes were performed. |
| `T0821` | Modify Controller Tasking | The lab did not modify controller task scheduling or PLC task execution behavior. |
| `T0843` | Program Download | The OpenPLC program was uploaded as part of authorized lab setup, not as an adversary behavior during the assessment phase. |
| `T0855` / `T1692.001` legacy naming | Unauthorized Command Message / Command Message | Mapped as `T1692.001` because ATT&CK now identifies it under Unauthorized Message: Command Message. |

---

## Detection Opportunities

| Detection Idea | Relevant ATT&CK for ICS Technique | Example Logic |
|---|---|---|
| Port scanning against PLC subnet | `T0846.001` | Alert when a host attempts connections to multiple OT service ports or multiple PLCs |
| New host communicating with PLC | `T0840` | Alert when a new source IP communicates with PLC TCP/502 |
| Unusual Modbus read activity | `T0861` | Alert on unusual read volume, new read source, or access to uncommon registers |
| Packet capture / sniffing tool execution | `T0842` | Monitor for unauthorized packet capture tools on engineering or user workstations |
| Modbus write activity | `T0836`, `T1692.001` | Alert on Modbus function code `6` or `16`, especially from non-approved hosts |
| Register value outside expected range | `T0836` | Alert when register values exceed expected operational limits |
| Command from unauthorized host | `T1692.001` | Alert when write commands originate from systems outside approved HMI/engineering workstation list |

---

## Defensive Recommendations

| Control Area | Recommendation |
|---|---|
| Network Segmentation | Place PLCs in restricted OT network zones and prevent direct access from general user networks. |
| Firewall Rules | Restrict TCP/502 to approved HMI and engineering workstation IP addresses only. |
| Asset Inventory | Maintain a list of approved PLCs, HMIs, engineering workstations, and expected communication paths. |
| Protocol Monitoring | Monitor Modbus function codes, especially write operations such as function code `6` and `16`. |
| Change Control | Require approval and documentation for PLC logic changes, register modifications, and configuration updates. |
| Alerting | Alert when a new or unauthorized host communicates with PLCs or sends write commands. |
| Packet Capture Review | Use Wireshark or an OT IDS sensor to validate suspicious protocol activity during investigations. |
| Management Interface Restriction | Limit access to PLC web interfaces such as OpenPLC HTTP/HTTPS management ports. |

---

## Final Mapping Summary

This lab maps most strongly to the following MITRE ATT&CK for ICS techniques:

| Technique ID | Technique Name | Why It Matters |
|---:|---|---|
| `T0846.001` | Remote System Discovery: Port Scan | Nmap identified exposed OT services. |
| `T0842` | Network Sniffing | Wireshark captured Modbus/TCP traffic. |
| `T0836` | Modify Parameter | A Modbus holding register was changed from `0` to `75`. |
| `T1692.001` | Unauthorized Message: Command Message | A Modbus write command was sent from Kali to the simulated PLC. |
| `T0861` | Point & Tag Identification | Process-related values were read from the simulated PLC. |

The strongest direct mappings are `T0846.001`, `T0842`, and `T0836`. The other mappings are included because they are related to the lab behavior but should be explained carefully to avoid overstating the project scope.

---

## Notes on Responsible Use

All activities in this project were performed in an isolated virtual lab environment. The mapping is intended for learning, documentation, and interview discussion.

Active scanning, Modbus testing, and write operations should only be performed in environments where explicit authorization has been granted. In real OT/ICS environments, testing should be carefully planned because industrial systems may be sensitive to unexpected traffic.
