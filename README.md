from pathlib import Path
readme = r'''# Cisco DHCP Snooping Security Lab

A Cisco Packet Tracer lab that demonstrates a basic secure LAN using a router-based DHCP server and DHCP Snooping on access switches.

## Overview

This project simulates a small LAN consisting of one router, two switches, and three client PCs. The router provides IP addresses through DHCP, while DHCP Snooping is enabled on the switches to help prevent unauthorized DHCP server responses on untrusted access ports.

## Topology

```text
                    +-------------------+
                    |        R1         |
                    | DHCP Server       |
                    | 192.168.1.1/24    |
                    +---------+---------+
                              |
                              |
                    +---------+---------+
                    |       SW1         |
                    | DHCP Snooping     |
                    +---------+---------+
                              |
                    +---------+---------+
                    |       SW2         |
                    | DHCP Snooping     |
                    +----+---------+----+
                         |         |
                       PC1        PC2 / PC3
```

> The physical interface numbers may differ depending on the Packet Tracer cabling. The interface facing the trusted DHCP path must be configured as trusted on each switch.

## Addressing Plan

| Device | Interface / Role | Address / Configuration |
|---|---|---|
| R1 | LAN gateway | `192.168.1.1/24` |
| R1 | DHCP excluded range | `192.168.1.1` - `192.168.1.9` |
| R1 | DHCP pool network | `192.168.1.0/24` |
| PCs | DHCP clients | Dynamic addresses, starting from `192.168.1.10` |
| SW1 / SW2 | Management | Not configured in this lab |

## Technologies Used

- Cisco Packet Tracer
- Cisco IOS
- IPv4 addressing
- DHCP
- DHCP Snooping
- VLAN 1 (default access VLAN)

## Router Configuration (R1)

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

ip dhcp excluded-address 192.168.1.1 192.168.1.9

ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
exit

! Allows relay information / Option 82 information when required.
ip dhcp relay information trust-all

end
copy running-config startup-config
```

## DHCP Snooping Configuration

DHCP Snooping is enabled on VLAN 1. Only the switch interface that faces the legitimate DHCP server or the trusted upstream path should be trusted.

### SW1

```cisco
enable
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 1

! Replace Fa0/1 if your R1/upstream connection uses another interface.
interface fastEthernet0/1
 ip dhcp snooping trust
exit

! Option 82 was disabled for Packet Tracer compatibility in this lab.
no ip dhcp snooping information option

end
copy running-config startup-config
```

### SW2

```cisco
enable
configure terminal

ip dhcp snooping
ip dhcp snooping vlan 1

! Replace Fa0/1 if the trusted upstream connection uses another interface.
interface fastEthernet0/1
 ip dhcp snooping trust
exit

! Option 82 was disabled for Packet Tracer compatibility in this lab.
no ip dhcp snooping information option

end
copy running-config startup-config
```

## Verification

### On a PC

Set the PC network configuration to **DHCP**, then run:

```text
ipconfig /release
ipconfig /renew
ipconfig
ping 192.168.1.1
```

Expected DHCP result:

```text
IP Address:       192.168.1.10 or higher
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
```

Example successful lease:

```text
IP Address......................: 192.168.1.12
Subnet Mask.....................: 255.255.255.0
Default Gateway.................: 192.168.1.1
```

### On a Switch

```cisco
show ip dhcp snooping
show running-config
show cdp neighbors
```

Expected behavior:

- DHCP Snooping is enabled for VLAN 1.
- The uplink toward R1 or the trusted upstream switch is listed as `Trusted`.
- PC-facing access ports remain untrusted.

## Troubleshooting Notes

### Client receives a `169.254.x.x` address

This is an APIPA address, which means the client did not receive a valid DHCP lease. Check the following:

1. The router LAN interface is `up/up` and has `192.168.1.1/24`.
2. The DHCP pool network and default gateway are correct.
3. DHCP Snooping is enabled for the client VLAN.
4. The interface toward the DHCP server/trusted upstream path is configured with:

```cisco
ip dhcp snooping trust
```

5. The trusted interface is the **actual physical interface** connected toward R1/upstream, not merely an interface that looks plausible in the config.
6. If Packet Tracer causes Option 82-related DHCP issues, disable Option 82 insertion on both switches:

```cisco
no ip dhcp snooping information option
```

## Security Concept

DHCP Snooping classifies switch ports as either **trusted** or **untrusted**:

- **Trusted ports** can receive DHCP server messages such as DHCPOFFER and DHCPACK.
- **Untrusted ports** are intended for client devices and should not be allowed to provide DHCP server responses.

This reduces the risk of rogue DHCP servers distributing malicious or incorrect network settings to clients.

## Planned Improvement: Port Security

Port Security can be applied to PC-facing access interfaces to limit the number of permitted MAC addresses.

Example configuration:

```cisco
interface fastEthernet0/2
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

> Do not apply Port Security to switch uplinks or router-facing interfaces.

## Project Files

```text
.
├── README.md
└── DHCP-Snooping-Lab.pkt    # Add/export your Packet Tracer topology file
```

## Author
EHSAN 
Created as a Cisco Packet Tracer networking and switch-security practice lab.
'''
Path('/mnt/data/README.md').write_text(readme, encoding='utf-8')
print('Created /mnt/data/README.md')
