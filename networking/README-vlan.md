# 🔥 VLAN basics

- VLANs segment the network into different broadcast domains.
- IP ranges are assigned per VLAN, but VLAN

- VLAN = container → then you assign a specific subnet/IP range to it.

- Some IP ranges would be only for HR and some would be only for DevOps team.

For example:
```bash
VLAN 10 (HR) → 192.168.10.0/24
VLAN 20 (DevOps) → 192.168.20.0/24
```

- For new devices we can select on which VLAN it should be belonged.

- You usually configure the switch port or Wi-Fi network to assign a device to a specific VLAN

- The responsibility of the DHCP server is to give appropriate IP address for the devices based on the IP range that is defined by VLAN.

- But the switch or router must tell the DHCP server which VLAN (scope) the request came from.