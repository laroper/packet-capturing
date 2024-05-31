# Packet Capturing using tcpdump
**Description:** I performed tasks associated with using tcpdump to capture network traffic on a linux vitual machine. I capture the data in a packet capture (p-cap) file and then examine the contents of the captured packet data to focus on specific types of traffic.

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

## Task 4. Filter the captured packet data

1.	Use the `tcpdump` command to filter the packet header data from the capture.pcap file:
   
`sudo tcpdump -nn -r capture.pcap -v`

This command will run tcpdump with the following options:
+ -nn: Disable port and protocol name lookup.
+ -r: Read capture data from the named file.
+ -v: Display detailed packet data.

> [!IMPORTANT]
> You must specify the -nn switch again here, as you want to make sure tcpdump does not perform name lookups of either IP addresses or ports, since this can alert threat actors.

This returns following output data:
```
analyst@22667c8ec94a:~$ sudo tcpdump -nn -r capture.pcap -v
reading from file capture.pcap, link-type EN10MB (Ethernet)
16:58:19.107053 IP (tos 0x0, ttl 64, id 26436, offset 0, flags [DF], proto TCP (6), length 60)
    172.17.0.2.52462 > 142.251.107.113.80: Flags [S], cksum 0xa6ae (incorrect -> 0x6350), seq 1182088039, win 65320, options [mss 1420,sackOK,TS val 2475842689 ecr 0,nop,wscale 7], length 0
16:58:19.108794 IP (tos 0x60, ttl 126, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    142.251.107.113.80 > 172.17.0.2.52462: Flags [S.], cksum 0x55c3 (correct), seq 1150855157, ack 1182088040, win 65535, options [mss 1420,sackOK,TS val 1106959899 ecr 2475842689,nop,wscale 8], length 0
16:58:19.108839 IP (tos 0x0, ttl 64, id 26437, offset 0, flags [DF], proto TCP (6), length 52)
    172.17.0.2.52462 > 142.251.107.113.80: Flags [.], cksum 0xa6a6 (incorrect -> 0x8267), ack 1, win 511, options [nop,nop,TS val 2475842691 ecr 1106959899], length 0
16:58:19.108906 IP (tos 0x0, ttl 64, id 26438, offset 0, flags [DF], proto TCP (6), length 137)
    172.17.0.2.52462 > 142.251.107.113.80: Flags [P.], cksum 0xa6fb (incorrect -> 0xf11a), seq 1:86, ack 1, win 511, options [nop,nop,TS val 2475842691 ecr 1106959899], length 85: HTTP, length: 85
        GET / HTTP/1.1
        Host: opensource.google.com
        User-Agent: curl/7.64.0
        Accept: */*

16:58:19.109038 IP (tos 0x60, ttl 126, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    142.251.107.113.80 > 172.17.0.2.52462: Flags [.], cksum 0x8310 (correct), ack 86, win 256, options [nop,nop,TS val 1106959900 ecr 2475842691], length 0
16:58:19.112017 IP (tos 0x80, ttl 126, id 0, offset 0, flags [DF], proto TCP (6), length 599)
    142.251.107.113.80 > 172.17.0.2.52462: Flags [P.], cksum 0xee71 (correct), seq 1:548, ack 86, win 256, options [nop,nop,TS val 1106959903 ecr 2475842691], length 547: HTTP, length: 547
        HTTP/1.1 301 Moved Permanently
        Location: https://opensource.google/
        X-Content-Type-Options: nosniff
        Server: sffe
        Content-Length: 223
        X-XSS-Protection: 0
        Date: Wed, 29 May 2024 16:56:51 GMT
        Expires: Wed, 29 May 2024 17:26:51 GMT
        Cache-Control: public, max-age=1800
        Content-Type: text/html; charset=UTF-8
        Age: 88

        <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
        <TITLE>301 Moved</TITLE></HEAD><BODY>
        <H1>301 Moved</H1>
        The document has moved
        <A HREF="https://opensource.google/">here</A>.
        </BODY></HTML>
16:58:19.112025 IP (tos 0x0, ttl 64, id 26439, offset 0, flags [DF], proto TCP (6), length 52)
    172.17.0.2.52462 > 142.251.107.113.80: Flags [.], cksum 0xa6a6 (incorrect -> 0x7fec), ack 548, win 507, options [nop,nop,TS val 2475842694 ecr 1106959903], length 0
16:58:19.113281 IP (tos 0x0, ttl 64, id 26440, offset 0, flags [DF], proto TCP (6), length 52)
    172.17.0.2.52462 > 142.251.107.113.80: Flags [F.], cksum 0xa6a6 (incorrect -> 0x7fea), seq 86, ack 548, win 507, options [nop,nop,TS val 2475842695 ecr 1106959903], length 0
16:58:19.113508 IP (tos 0x80, ttl 126, id 0, offset 0, flags [DF], proto TCP (6), length 52)
    142.251.107.113.80 > 172.17.0.2.52462: Flags [F.], cksum 0x80e3 (correct), seq 548, ack 87, win 256, options [nop,nop,TS val 1106959904 ecr 2475842695], length 0
```

2. Use the `tcpdump` command to filter the extended packet data from the capture.pcap file:
3. 
`sudo tcpdump -nn -r capture.pcap -X`

This command will run tcpdump with the following options:
+ -nn: Disable port and protocol name lookup.
+ -r: Read capture data from the named file.
+ -X: Display the hexadecimal and ASCII output format packet data. Security analysts can analyze hexadecimal and ASCII output to detect patterns or anomalies during malware analysis or forensic analysis.

> [!NOTE]
> Hexadecimal, also known as hex or base 16, uses 16 symbols to represent values, including the digits 0-9 and letters A, B, C, D, E, and F. American Standard Code for Information Interchange (ASCII) is a character encoding standard that uses a set of characters to represent text in digital form.
```
analyst@22667c8ec94a:~$ sudo tcpdump -nn -r capture.pcap -X
reading from file capture.pcap, link-type EN10MB (Ethernet)
16:58:19.107053 IP 172.17.0.2.52462 > 142.251.107.113.80: Flags [S], seq 1182088039, win 65320, options [mss 1420,sackOK,TS val 2475842689 ecr 0,nop,wscale 7], length 0
        0x0000:  4500 003c 6744 4000 4006 2cf8 ac11 0002  E..<gD@.@.,.....
        0x0010:  8efb 6b71 ccee 0050 4675 3b67 0000 0000  ..kq...PFu;g....
        0x0020:  a002 ff28 a6ae 0000 0204 058c 0402 080a  ...(............
        0x0030:  9392 5c81 0000 0000 0103 0307            ..\.........
16:58:19.108794 IP 142.251.107.113.80 > 172.17.0.2.52462: Flags [S.], seq 1150855157, ack 1182088040, win 65535, options [mss 1420,sackOK,TS val 1106959899 ecr 2475842689,nop,wscale 8], length 0
        0x0000:  4560 003c 0000 4000 7e06 55dc 8efb 6b71  E`.<..@.~.U...kq
        0x0010:  ac11 0002 0050 ccee 4498 a7f5 4675 3b68  .....P..D...Fu;h
        0x0020:  a012 ffff 55c3 0000 0204 058c 0402 080a  ....U...........
        0x0030:  41fa de1b 9392 5c81 0103 0308            A.....\.....
16:58:19.108839 IP 172.17.0.2.52462 > 142.251.107.113.80: Flags [.], ack 1, win 511, options [nop,nop,TS val 2475842691 ecr 1106959899], length 0
        0x0000:  4500 0034 6745 4000 4006 2cff ac11 0002  E..4gE@.@.,.....
        0x0010:  8efb 6b71 ccee 0050 4675 3b68 4498 a7f6  ..kq...PFu;hD...
        0x0020:  8010 01ff a6a6 0000 0101 080a 9392 5c83  ..............\.
        0x0030:  41fa de1b                                A...
16:58:19.108906 IP 172.17.0.2.52462 > 142.251.107.113.80: Flags [P.], seq 1:86, ack 1, win 511, options [nop,nop,TS val 2475842691 ecr 1106959899], length 85: HTTP: GET / HTTP/1.1
        0x0000:  4500 0089 6746 4000 4006 2ca9 ac11 0002  E...gF@.@.,.....
        0x0010:  8efb 6b71 ccee 0050 4675 3b68 4498 a7f6  ..kq...PFu;hD...
        0x0020:  8018 01ff a6fb 0000 0101 080a 9392 5c83  ..............\.
        0x0030:  41fa de1b 4745 5420 2f20 4854 5450 2f31  A...GET./.HTTP/1
        0x0040:  2e31 0d0a 486f 7374 3a20 6f70 656e 736f  .1..Host:.openso
        0x0050:  7572 6365 2e67 6f6f 676c 652e 636f 6d0d  urce.google.com.
        0x0060:  0a55 7365 722d 4167 656e 743a 2063 7572  .User-Agent:.cur
        0x0070:  6c2f 372e 3634 2e30 0d0a 4163 6365 7074  l/7.64.0..Accept
        0x0080:  3a20 2a2f 2a0d 0a0d 0a                   :.*/*....
16:58:19.109038 IP 142.251.107.113.80 > 172.17.0.2.52462: Flags [.], ack 86, win 256, options [nop,nop,TS val 1106959900 ecr 2475842691], length 0
        0x0000:  4560 0034 0000 4000 7e06 55e4 8efb 6b71  E`.4..@.~.U...kq
        0x0010:  ac11 0002 0050 ccee 4498 a7f6 4675 3bbd  .....P..D...Fu;.
        0x0020:  8010 0100 8310 0000 0101 080a 41fa de1c  ............A...
        0x0030:  9392 5c83                                ..\.
16:58:19.112017 IP 142.251.107.113.80 > 172.17.0.2.52462: Flags [P.], seq 1:548, ack 86, win 256, options [nop,nop,TS val 1106959903 ecr 2475842691], length 547: HTTP: HTTP/1.1 301 Moved Permanently
        0x0000:  4580 0257 0000 4000 7e06 53a1 8efb 6b71  E..W..@.~.S...kq
        0x0010:  ac11 0002 0050 ccee 4498 a7f6 4675 3bbd  .....P..D...Fu;.
        0x0020:  8018 0100 ee71 0000 0101 080a 41fa de1f  .....q......A...
        0x0030:  9392 5c83 4854 5450 2f31 2e31 2033 3031  ..\.HTTP/1.1.301
        0x0040:  204d 6f76 6564 2050 6572 6d61 6e65 6e74  .Moved.Permanent
        0x0050:  6c79 0d0a 4c6f 6361 7469 6f6e 3a20 6874  ly..Location:.ht
        0x0060:  7470 733a 2f2f 6f70 656e 736f 7572 6365  tps://opensource
        0x0070:  2e67 6f6f 676c 652f 0d0a 582d 436f 6e74  .google/..X-Cont
        0x0080:  656e 742d 5479 7065 2d4f 7074 696f 6e73  ent-Type-Options
        0x0090:  3a20 6e6f 736e 6966 660d 0a53 6572 7665  :.nosniff..Serve
        0x00a0:  723a 2073 6666 650d 0a43 6f6e 7465 6e74  r:.sffe..Content
        0x00b0:  2d4c 656e 6774 683a 2032 3233 0d0a 582d  -Length:.223..X-
        0x00c0:  5853 532d 5072 6f74 6563 7469 6f6e 3a20  XSS-Protection:.
        0x00d0:  300d 0a44 6174 653a 2057 6564 2c20 3239  0..Date:.Wed,.29
        0x00e0:  204d 6179 2032 3032 3420 3136 3a35 363a  .May.2024.16:56:
        0x00f0:  3531 2047 4d54 0d0a 4578 7069 7265 733a  51.GMT..Expires:
        0x0100:  2057 6564 2c20 3239 204d 6179 2032 3032  .Wed,.29.May.202
        0x0110:  3420 3137 3a32 363a 3531 2047 4d54 0d0a  4.17:26:51.GMT..
        0x0120:  4361 6368 652d 436f 6e74 726f 6c3a 2070  Cache-Control:.p
        0x0130:  7562 6c69 632c 206d 6178 2d61 6765 3d31  ublic,.max-age=1
        0x0140:  3830 300d 0a43 6f6e 7465 6e74 2d54 7970  800..Content-Typ
        0x0150:  653a 2074 6578 742f 6874 6d6c 3b20 6368  e:.text/html;.ch
        0x0160:  6172 7365 743d 5554 462d 380d 0a41 6765  arset=UTF-8..Age
        0x0170:  3a20 3838 0d0a 0d0a 3c48 544d 4c3e 3c48  :.88....<HTML><H
        0x0180:  4541 443e 3c6d 6574 6120 6874 7470 2d65  EAD><meta.http-e
        0x0190:  7175 6976 3d22 636f 6e74 656e 742d 7479  quiv="content-ty
        0x01a0:  7065 2220 636f 6e74 656e 743d 2274 6578  pe".content="tex
        0x01b0:  742f 6874 6d6c 3b63 6861 7273 6574 3d75  t/html;charset=u
        0x01c0:  7466 2d38 223e 0a3c 5449 544c 453e 3330  tf-8">.<TITLE>30
        0x01d0:  3120 4d6f 7665 643c 2f54 4954 4c45 3e3c  1.Moved</TITLE><
        0x01e0:  2f48 4541 443e 3c42 4f44 593e 0a3c 4831  /HEAD><BODY>.<H1
        0x01f0:  3e33 3031 204d 6f76 6564 3c2f 4831 3e0a  >301.Moved</H1>.
        0x0200:  5468 6520 646f 6375 6d65 6e74 2068 6173  The.document.has
        0x0210:  206d 6f76 6564 0a3c 4120 4852 4546 3d22  .moved.<A.HREF="
        0x0220:  6874 7470 733a 2f2f 6f70 656e 736f 7572  https://opensour
        0x0230:  6365 2e67 6f6f 676c 652f 223e 6865 7265  ce.google/">here
        0x0240:  3c2f 413e 2e0d 0a3c 2f42 4f44 593e 3c2f  </A>...</BODY></
        0x0250:  4854 4d4c 3e0d 0a                        HTML>..
16:58:19.112025 IP 172.17.0.2.52462 > 142.251.107.113.80: Flags [.], ack 548, win 507, options [nop,nop,TS val 2475842694 ecr 1106959903], length 0
        0x0000:  4500 0034 6747 4000 4006 2cfd ac11 0002  E..4gG@.@.,.....
        0x0010:  8efb 6b71 ccee 0050 4675 3bbd 4498 aa19  ..kq...PFu;.D...
        0x0020:  8010 01fb a6a6 0000 0101 080a 9392 5c86  ..............\.
        0x0030:  41fa de1f                                A...
16:58:19.113281 IP 172.17.0.2.52462 > 142.251.107.113.80: Flags [F.], seq 86, ack 548, win 507, options [nop,nop,TS val 2475842695 ecr 1106959903], length 0
        0x0000:  4500 0034 6748 4000 4006 2cfc ac11 0002  E..4gH@.@.,.....
        0x0010:  8efb 6b71 ccee 0050 4675 3bbd 4498 aa19  ..kq...PFu;.D...
        0x0020:  8011 01fb a6a6 0000 0101 080a 9392 5c87  ..............\.
        0x0030:  41fa de1f                                A...
16:58:19.113508 IP 142.251.107.113.80 > 172.17.0.2.52462: Flags [F.], seq 548, ack 87, win 256, options [nop,nop,TS val 1106959904 ecr 2475842695], length 0
        0x0000:  4580 0034 0000 4000 7e06 55c4 8efb 6b71  E..4..@.~.U...kq
        0x0010:  ac11 0002 0050 ccee 4498 aa19 4675 3bbe  .....P..D...Fu;.
        0x0020:  8011 0100 80e3 0000 0101 080a 41fa de20  ............A...
        0x0030:  9392 5c87                                ..\.
```

## SUMMARY 
> [!IMPORTANT]
> This project highlighted the practical experience I gained to:
> + identify network interfaces,
> + use the `tcpdump` command to capture network data for inspection,
> + interpret the information that `tcpdump` outputs regarding a packet, and
> + save and load packet data for later analysis.





















