# Evidence Index

## Lab Setup
- Ubuntu/OpenPLC VM IP: 192.168.79.148
- Kali assessment VM IP: 192.168.79.149
- Connectivity confirmed using ping between both VMs.

## OpenPLC Setup
- OpenPLC installed on Ubuntu.
- Program `modbus_lab` compiled successfully.
- PLC runtime started successfully.
- Modbus/TCP confirmed listening on port 502.

## Nmap Enumeration
- Nmap confirmed open ports: 502, 8080, 8443, and 44818.
- Service detection identified port 502 as Modbus/TCP.
- `modbus-discover` confirmed Modbus/TCP was reachable, although device information returned `ILLEGAL FUNCTION`.

## Metasploit Modbus Testing
- Read coils from address 0 returned `[0, 0]`.
- Read holding register address 0 returned `[0]`.
- Write test changed holding register 0 from `0` to `75`.
- Read-back confirmed `[75]`.
- Register was restored back to `0`.

## Wireshark Traffic Analysis
- Wireshark captured Modbus/TCP read traffic.
- Wireshark captured Modbus/TCP write traffic.
- Write request used function code 6, Write Single Register.
- Read request used function code 3, Read Holding Registers.
