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
The Ethernet network interface is identified by the entry with the `eth` prefix. Therefore use `eth0` as the interface that will capture network packet data.

2. Alternatively for systems that does not have ifconfig command `tcpdump` can be used to identify the interface options available for packet capture:
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
- `tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes`
2. On the next line, the first field is the packet's timestamp, followed by the protocol type, IP:
- `16:37:16.647596 IP`
3. The verbose option, `-v`, has provided more details about the IP packet fields, such as TOS, TTL, offset, flags, internal protocol type (in this case, TCP (6)), and the length of the outer IP packet in bytes:
- `(tos 0x0, ttl 64, id 19241, offset 0, flags [DF], proto TCP (6), length 119)`
- These fields are properties that relate to the IP network packet. For details on these visit <a href="https://www.tcpdump.org/manpages/tcpdump.1.html#lbAL" >IPv4 Packets</a>
5. In the next section, the data shows the systems that are communicating with each other:
- `22667c8ec94a.5000 > nginx-us-east1-b.c.qwiklabs-terminal-vms-prod-00.internal.44010:`
- By default, tcpdump will convert IP addresses into names, as in the result output. The name of the Linux virtual machine, also included in the command prompt, appears here as the source for one packet and the destination for the second packet.
- The direction of the arrow `>` indicates the direction of the traffic flow in this packet. Each system name includes a suffix with the port number (.5000 in the screenshot), which is used by the source and the destination systems for this packet.
> [!NOTE]
> In your live data, the name will be a different set of letters and numbers.
6. The remaining data filters the header data for the inner TCP packet:
- `Flags [P.], cksum 0x5891 (incorrect -> 0xf1d7), seq 285494408:285494475, ack 1926277971, win 501, options [nop,nop,TS val 3672580546 ecr 4041935978], length 67`
- The flags field identifies TCP flags. In this case, the `P` represents the push flag and the period `.` indicates it's an ACK flag. This means the packet is pushing out data.
  The next field is the TCP checksum value, which is used for detecting errors in the data.
  This section also includes the sequence and acknowledgment numbers, the window size, and the length of the inner TCP packet in bytes.

## Task 3. Capture network traffic with tcpdump
   Using `tcpdump` to save the captured network data to a packet capture file.
   In Task 2 `tcpdump` was used to stream all network traffic. Here, a filter and other tcpdump configuration options will be used to save
   a small sample that contains only web (TCP port 80) network packet data.
   
1.	Capture packet data into a file called capture.pcap:
   
`sudo tcpdump -i eth0 -nn -c9 port 80 -w capture.pcap &`

This command returns:
```
analyst@22667c8ec94a:~$ sudo tcpdump -i eth0 -nn -c9 port 80 -w capture.pcap &
[1] 12884
analyst@22667c8ec94a:~$ tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```
> [!IMPORTANT]
> You must press the ENTER key to get your command prompt back after running this command.

This command will run `tcpdump` in the background with the following options:
+ -i eth0: Capture data from the eth0 interface.
+ -nn: Do not attempt to resolve IP addresses or ports to names.This is best practice from a security perspective, as the lookup data may not be valid.
  It also prevents malicious actors from being alerted to an investigation.
+ -c9: Capture 9 packets of data and then exit.
+ port 80: Filter only port 80 traffic. This is the default HTTP port.
+ -w capture.pcap: Save the captured data to the named file.
+ &: This is an instruction to the Bash shell to run the command in the background.

> This command runs in the background, but some output text will appear in your terminal. 
Dont worry about the text, it will not affect the commands used in the next step when we generate some traffic.

2.	Use `curl` to generate some HTTP (port 80) traffic:
   
`curl opensource.google.com`

The command returns output:
```
analyst@22667c8ec94a:~$ curl opensource.google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://opensource.google/">here</A>.
</BODY></HTML>
analyst@22667c8ec94a:~$ 9 packets captured
12 packets received by filter
0 packets dropped by kernel
```

When the curl command is used like this to open a website, it generates some HTTP (TCP port 80) traffic that can be captured.
Now lets verify the packet was captured adn save to our pcap file

3.	Verify that packet data has been captured using: `ls -l capture.pcap`
   
```
analyst@22667c8ec94a:~$ ls -l capture.pcap
-rw-r--r-- 1 root root 1410 May 29 16:58 capture.pcap
[1]+  Done                    sudo tcpdump -i eth0 -nn -c9 port 80 -w capture.pcap
```
> [!NOTE]
> The "Done" in the output indicates that the packet was captured.




























