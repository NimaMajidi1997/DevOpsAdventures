# 🧰Kea DHCP Server with VLAN Support

This guide explains how to run a VLAN-aware Kea DHCP server in Docker with sub-interfaces (like `eth1.20`) and enabling `bridge_vlan_aware` in Proxmox.

---

## ✅ System Requirements

* Docker installed (inside LXC)
* Kea Docker image

---

#### 📁 Docker Compose Configuration [docker-compose.yaml](docker-compose.yaml)
#### ⚙️ kea-dhcp4.conf Example [kea-dhcp4.json](config/kea-dhcp4.json)

---

## 💡 Why This Works

* `ipvlan` in Docker treats `parent: eth1.20` as "listen for VLAN 20 on eth1"
* Docker (via `ipvlan`) can "borrow" a VLAN interface from the host if it exists, here eth1
* So Docker itself took care of:

  * **Creating the VLAN sub-interface (**\`eth1.20\`**)**
  * Binding the container’s interface to that VLAN
  * No need to manually define `eth1.20` in `/etc/network/interfaces`
  * Need to make `vmbr1` VLAN-aware

Two Network Adaptors created in LXC container:

| Name  | Bridge | VLAN Tag | Ip address|
|-------|--------|----------|-----------|
| eth0  | vmbr0  |    none  |    dhcp   |
| eth1  | vmbr1  |    none  |  static   | 

---

🧪 Optional: Make VLAN IP reachable from host

Add to `/etc/network/interfaces` (Proxmox host):

```ini
auto eth1.20
iface eth1.20 inet static
    address 10.0.0.1
    netmask 255.255.255.0
```

This step is only needed if you want the **host itself** to participate in VLAN 20, meaning **Proxmox host itself** can act as a **router** between the two VLANs.

If the **Proxmox host should have a presence inside VLAN 20**, we define `eth1.20` in `/etc/network/interfaces` . If the host is just a **pass-through for VM/container VLANs**, we do not define it, and let the VMs/containers handle the VLAN traffic
