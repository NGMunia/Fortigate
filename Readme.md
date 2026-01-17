# Basic Zone based firewall using Fortigate

## Project Overview

This project demonstrates the configuration of a basic **zone-based firewall** on a **FortiGate NGFW**, using **Inside** and **Outside** security zones.  

The objective is to showcase practical skills in **network security, firewall policy design, and FortiGate administration**.

---

## Key Concepts Demonstrated
- Zone-based firewalling (Inside / Outside)
- Interface-to-zone assignment
- Inter-zone firewall policies
- PAT
- Secure management access
- FortiGate CLI configuration
---

## Topology Overview


![Topology](/Topology.PNG)

- **INSIDE Zone**: Trusted internal LAN
- **OUTSIDE Zone**: Untrusted external / WAN network
- FortiGate functions as the perimeter security device

---

## Configuration Summary

### 1. Zone Configuration
- Created two security zones:
  - `INSIDE`
  - `OUTSIDE`

```bash
  FortiGate-VM64-KVM (zone) # show
  config system zone
    edit "Inside"
        set intrazone allow
        set interface "port2" "port3"
    next
    edit "Outside"
        set intrazone allow
        set interface "port4"
    next
  end
```

- Interfaces assigned to zones instead of using interface-based firewall policies


```bash
FortiGate-VM64-KVM # config firewall policy

FortiGate-VM64-KVM (policy) # show
config firewall policy
    edit 1
        set name "Inside-Internet-Policy"
        set uuid 7daace9a-f3a6-51f0-a656-ee721d3b3df2
        set srcintf "Inside"
        set dstintf "Outside"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic-start enable
        set fsso disable
        set nat enable
    next
end
```

### 2. Interface Configuration
- Inside interface configured with a private IP address (with adminstrative access limited to PING only)
- Outside interface configured toward the ISP / upstream router
- Management access (SSH, HTTPS, PING) allowed on MGT

```bash
FortiGate-VM64-KVM (interface) # show
config system interface
    edit "port1"
        set vdom "root"
        set ip 192.168.100.110 255.255.255.240
        set allowaccess ping https ssh http
        set type physical
        set snmp-index 1
    next
    edit "port2"
        set vdom "root"
        set ip 10.255.0.1 255.255.255.0
        set allowaccess ping
        set type physical
        set alias "LAN-1"
        set device-identification enable
        set role lan
        set snmp-index 2
```

### 3. Firewall Policies
- **INSIDE → OUTSIDE**
  - Permits outbound traffic
  - Source NAT enabled
- **OUTSIDE → INSIDE**
  - Denied by default (implicit deny)
  - No inbound access unless explicitly permitted

### 4. NAT Configuration
- Source NAT configured using PAT
- Internal IP addresses translated to the Outside interface address
```bash
FortiGate-VM64-KVM # get system session list
PROTO   EXPIRE SOURCE           SOURCE-NAT       DESTINATION      DESTINATION-NAT
udp     175    192.168.122.202:3308 -                54.232.58.26:53  -
udp     175    192.168.122.202:3308 -                65.0.232.185:53  -
udp     176    192.168.122.202:3308 -                154.52.12.53:53  -
udp     175    192.168.122.202:3308 -                154.52.30.55:53  -
udp     176    192.168.122.202:3308 -                210.7.96.53:53   -
udp     165    192.168.122.202:3308 -                208.91.112.220:53 -
```
---

## Tools and Technologies
- FortiGate NGFW
- Zone-Based Firewalling
- NAT (PAT)
- GNS3 / Lab Environment
