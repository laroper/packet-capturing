# Packet Capturing using tcpdump
**Description:** We will perform tasks associated with using tcpdump to capture network traffic on a linux vitual machine. You’ll capture the data in a packet capture (p-cap) file and then examine the contents of the captured packet data to focus on specific types of traffic.

## Task 1. Identify network interfaces
1. Use `ifconfig` to identify the interfaces that are available:
   `sudo ifconfig`
The command returns the following output:
```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 748  bytes 13731779 (13.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 392  bytes 36326 (35.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 166  bytes 18849 (18.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 166  bytes 18849 (18.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
The Ethernet network interface is identified by the entry with the `eth` prefix. Therefore you'll use `eth0` as the interface that you will capture network packet data.

2. Alternatively for system that doensat have ifconfig command you can use `tcpdump` to identify the interface options available for packet capture:
  `sudo tcpdump -D`
The command returns the following output:
```
1.eth0 [Up, Running]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.nflog (Linux netfilter log (NFLOG) interface)
5.nfqueue (Linux netfilter queue (NFQUEUE) interface)

```
