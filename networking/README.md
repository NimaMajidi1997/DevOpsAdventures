# Network

#### 💻 NIC = Network Interface Card
It’s the physical (or virtual) hardware that connects a device to a network.

On a real PC: it’s the Ethernet port or Wi-Fi chip

On a VM: it’s a virtual NIC, like virtio-net, e1000, etc.

🧠 Think: the card that sends/receives packets

---
### 🔌 Network Interface = The software representation of the NIC
It’s how the operating system talks to the NIC, the software-side handle to configure, monitor, and use the NIC.

Examples:

In Linux: you see eth0, ens18, enp0s3

In Windows: you see Ethernet, Ethernet 2, Ethernet 3 (via ipconfig or Get-NetAdapter)

🧠 Think: the OS’s name for the NIC so it can configure IP/DHCP/etc.

### Network Interface Name
Linux systems use two different styles of naming the network interfaces. The first style is the old-style name, such as eth0, eth1, and wlan0. The new ones are based on hardware locations like enp3s0 and wlp2s0.

## 🧱 Physical network interface = en, eth, ...
This is your actual network card on the Proxmox server — the one plugged into your LAN or router.

Example:

```bash
ip a
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    inet 10.78.1.5/23
```
That’s your real NIC, used by the host itself.


## 🌉 vmbrX = Virtual Linux bridge interface
connect guests to the underlying physical network. This is a software switch inside Proxmox. It acts like a virtual network switch which the guests and physical interfaces are connected to.

It connects VMs, LXCs, and the host together

You assign VMs and containers to it instead of assigning them directly to eth0

It can forward VLAN-tagged traffic

It can have an IP address (for the host to reach the outside world)

### 🧠 What’s inside /etc/network/interfaces:
You’ll see something like:
```bash
auto vmbr0     # Create a network interface named vmbr0 with a static IP
iface vmbr0 inet static
    address 8.18.1.1
    netmask 255.255.254.0
    bridge_ports eth0
    bridge_vlan_aware yes
```
"inet" → use IPv4

"static" → don't use DHCP, assign a manual IP

vmbr0 is using eth0 as its physical uplink

Everything plugged into vmbr0 (VMs, LXCs) can access the outside network via eth0

---
understanding how DHCP + VLANs can go wrong 🧠🔥

🧠 First: How DHCP works

When a VM boots up and needs an IP, it sends this:
```bash
DHCPDISCOVER → to 255.255.255.255 (broadcast)
```
It’s like yelling:

“Is there any DHCP server out there?! Give me an IP!”

Any DHCP server that hears that can reply:
```bash
DHCPOFFER → Here! Take this IP: 10.0.0.55
```

We can attach one interface to multiple VLANs
That’s the point of a trunk port.

On your Proxmox host (or switch), you can have:
```bash
eth0       ← physical NIC
eth0.20    ← VLAN 20 sub-interface
eth0.30    ← VLAN 30 sub-interface
```
All these ride over the same wire (eth0) — but carry separate traffic.

🔍 SCENARIO 1: A VM is tagged with VLAN 20
Does eth0.20 see it? What about eth0 or eth0.30?

Answer:

- ✅ eth0.20 sees the traffic (because it filters VLAN ID 20)
- ❌ eth0 (untagged) does not see it
- ❌ eth0.30 (VLAN 30) also does not see it

Each VLAN sub-interface only sees packets for its own tag.
So no — there's no leakage or overlap if you configure VLANs properly.

🔍 SCENARIO 2: A VM is untagged
What happens?
Proxmox forwards this as untagged traffic through vmbr0

It reaches the physical NIC eth0, but not any VLAN interface (like eth0.20)

❌ eth0.20, eth0.30 won’t see it

✅ Only eth0 or other VMs on untagged bridge ports will see it

So untagged traffic is your weakest point if you're running multi-VLAN setups. Best to either:

- Isolate untagged traffic into a different bridge
Or
- block untagged traffic entirely at switch/firewall level


## `tcpdump`

For observing the traffic, like putting a stethoscope on the network, we can use tcpdump:


```bash
docker run -it --rm \
  --net vlan_20 \
  --ip 10.0.0.99 \
  --cap-add=NET_ADMIN \
  nicolaka/netshoot
```
and now you can check the traffic on the following ports:

```bash
tcpdump -i eth0 -n -vvv port 67 or port 68
```
PS1: you are assuming that another container is running on the same network (vlan_20)

PS2: `--ip 10.0.0.99` assigns a static IP address to the netshoot container inside your custom Docker network (vlan_20)
