# tcpdump

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