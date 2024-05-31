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
## Task 2. Inspect the network traffic of a network interface with tcpdump
1. Using `tcpdump` to filter live network packet traffic on the `eth0` interface.
   `sudo tcpdump -i eth0 -v -c5`

This command will run tcpdump with the following options:
+ -i eth0: Capture data specifically from the eth0 interface.
+ -v: Display detailed packet data.
+ -c5: Capture 5 packets of data.
  
Now, let's take a detailed look at the packet information that this command has returned below: 
```
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
16:37:16.647596 IP (tos 0x0, ttl 64, id 19241, offset 0, flags [DF], proto TCP (6), length 119)
    22667c8ec94a.5000 > nginx-us-east1-b.c.qwiklabs-terminal-vms-prod-00.internal.44010: Flags [P.], cksum 0x5891 (incorrect -> 0xf1d7), seq 285494408:285494475, ack 1926277971, win 501, options [nop,nop,TS val 3672580546 ecr 4041935978], length 67
16:37:16.647817 IP (tos 0x0, ttl 63, id 62011, offset 0, flags [DF], proto TCP (6), length 52)
    nginx-us-east1-b.c.qwiklabs-terminal-vms-prod-00.internal.44010 > 22667c8ec94a.5000: Flags [.], cksum 0xe615 (correct), ack 67, win 507, options [nop,nop,TS val 4041936265 ecr 3672580546], length 0
16:37:16.658157 IP (tos 0x0, ttl 64, id 19242, offset 0, flags [DF], proto TCP (6), length 148)
    22667c8ec94a.5000 > nginx-us-east1-b.c.qwiklabs-terminal-vms-prod-00.internal.44010: Flags [P.], cksum 0x58ae (incorrect -> 0x676f), seq 67:163, ack 1, win 501, options [nop,nop,TS val 3672580556 ecr 4041936265], length 96 
16:37:16.658333 IP (tos 0x0, ttl 63, id 62012, offset 0, flags [DF], proto TCP (6), length 52)
    nginx-us-east1-b.c.qwiklabs-terminal-vms-prod-00.internal.44010 > 22667c8ec94a.5000: Flags [.], cksum 0xe5a1 (correct), ack 163, win 507, options [nop,nop,TS val 4041936275 ecr 3672580556], length 016:37:16.670549 IP (tos 0x0, ttl 64, id 50936, offset 0, flags [DF], proto UDP (17), length 69)
    22667c8ec94a.55661 > metadata.google.internal.domain: 30560+ PTR? 2.0.18.172.in-addr.arpa. (41)
5 packets captured 
8 packets received by filter 
0 packets dropped by kernel
```
**2. Exploring the network packet details**

Lets identify some of the properties that `tcpdump` outputs from the packet capture data output above.

1. In the example data at the start of the packet output, tcpdump reported that it was listening on the `eth0` interface, and it provided information on the `link-type` and the `capture size` in bytes:
`tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes`
2. On the next line, the first field is the packet's timestamp, followed by the protocol type, IP:
`16:37:16.647596 IP`
3. The verbose option, -v, has provided more details about the IP packet fields, such as TOS, TTL, offset, flags, internal protocol type (in this case, TCP (6)), and the length of the outer IP packet in bytes:
`(tos 0x0, ttl 64, id 19241, offset 0, flags [DF], proto TCP (6), length 119)`
These fields are properties that relate to the IP network packet. For details on these visit <a href='https://www.tcpdump.org/manpages/tcpdump.1.html#lbAL'>IPv4 Packets</a>
5. In the next section, the data shows the systems that are communicating with each other:
7acb26dc1f44.5000 > nginx-us-east1-c.c.qwiklabs-terminal-vms-prod-00.internal.59788:
By default, tcpdump will convert IP addresses into names, as in the screenshot. The name of your Linux virtual machine, also included in the command prompt, appears here as the source for one packet and the destination for the second packet. In your live data, the name will be a different set of letters and numbers.
The direction of the arrow (>) indicates the direction of the traffic flow in this packet. Each system name includes a suffix with the port number (.5000 in the screenshot), which is used by the source and the destination systems for this packet.
6. The remaining data filters the header data for the inner TCP packet:
Flags [P.], cksum 0x5851 (incorrect > 0x30d3), seq 1080713945:1080714027, ack 62760789, win 501, options [nop,nop,TS val 1017464119 ecr 3001513453], length 82
The flags field identifies TCP flags. In this case, the P represents the push flag and the period indicates it's an ACK flag. This means the packet is pushing out data.
The next field is the TCP checksum value, which is used for detecting errors in the data.
This section also includes the sequence and acknowledgment numbers, the window size, and the length of the inner TCP packet in bytes.



























