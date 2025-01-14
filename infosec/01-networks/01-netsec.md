# Network Security

### Contents

[Ways of Attacking Networks](#ways-of-attacking-networks)

[Network Sniffing - An Introduction](#network-sniffing---an-introduction)

[Network Sniffing - Hardware Tools](#network-sniffing---hardware-tools)

[Network Sniffing - Software Tools](#network-sniffing---software-tools)

[Network Sniffing - Defense](#network-sniffing---defense)

[Network Scanning - An Introduction](#network-scanning---an-introduction)

[Network Scanning - Software Tools](#network-scanning---software-tools)

[Network Scanning - Defense](#network-scanning---defense)

[DDoS Attacks - An Introduction](#ddos-attacks---an-introduction)

[DDoS Attacks - Techniques](#ddos-attacks---techniques)

[DDoS Attacks - Defense](#ddos-attacks---defense)

[A Quick Note on Packet Spoofing](#a-quick-note-on-packet-spoofing)

*[Back](../01-networks#networks)*


## Ways of Attacking Networks
- Sniffing
- Network reconnaissance
- Denial of Service (DoS)
- Impersonation (spoofing)
- Hijacking (information access, delivery tampering)


## Network Sniffing - An Introduction
__Network sniffing__: looking at network traffic and analyzing packets
  - Most traffic on a network is still in plaintext
  - Uses of network sniffing
    - Troubleshoot networking issues
    - Record communications
    - Catch sensitive information

__.pcap__: a common file extension for packet captures
  - Is commonly used in apps like Wireshark, ettercap, and tcpdump
  - A 100 mb PCAP file contains tens of thousands of packets

### Setting Up Your Lab
Step 1: Set your network card to promiscuous mode
- __Promiscuous mode__: a NIC mode that allows one to look at all packets
  regardless of destination address

Step 2: Disable the use of the ARP
```bash
# For Unix/Linux/Macs
foo@bar:~$ sudo ifconfig -i <INTERFACE> promisc -arp
```


## Network Sniffing - Hardware Tools
- A computer with wired or wireless networking (admin access required)
- __Span port__: AKA port mirroring; a network switch setting in which a copy of
  all packets transmitted to and from a source port are forwarded to a single 
  destination port
- __LAN tap__: A small physical device used for monitoring ethernet
  communications. Contains at least 3 ports: 2 for allowing normal
  communications traffic to move unimpeded; 1 for monitoring the traffic moving
  between the previous 2 ports
- __Network hub__: A network hardware device containing multiple I/O ports in
  which the input at one port outputs to all other ports. Used for unswitched
  networks


## Network Sniffing - Software Tools
### `tcpdump`
A command-line packet analyzer.
```bash
# Usage
foo@bar:~$ sudo tcpdump -a <INTERFACE>

# Manual
foo@bar:~$ man tcpdump

# Ex. Reading a PCAP file
foo@bar:~$ tcpdump -r file.pcap

# Ex. Splitting a PCAP file into smaller, 10 mb files
foo@bar:~$ tcpdump -r old_file.pcap -w new_files -C 10
```

### Wireshark
An open source, free graphical and entensive packet analyzer, similar to
`tcpdump`.

Site: [wireshark.org](https://www.wireshark.org)

### `tshark`
A command-line version of Wireshark.
```bash
# Manual
foo@bar:~$ man tshark

# Ex. Listing hosts in a PCAP file
foo@bar:~$ tshark -r file.pcap -1 -z hosts,ipv4
```

### `ettercap`
A graphical and command-line suite of man-in-the-middle (MITM) attack tools. 
Uses include:
- Capturing passwords
- Conducting MITM attacks
- Hijacking sessions

```bash
# Manual
foo@bar:~$ man ettercap

# Ex. Listing plaintext passwords captured in a PCAP file
foo@bar:~$ ettercap -T -r set3.pcap | grep "PASS:"
```

### `bettercap`
A ruby-based suite of MITM attack tools. Similar to `ettercap`.

Site: [bettercap.org](https://www.bettercap.org)

### `dsniff`
A command line suite of network sniffing tools including: 
- `dsniff # sniffs passwords`
- `webspy # intercepts entered URIs`
- `mailsnarf # intercepts POP or SMTP-based mail`

```bash
# Usage
foo@bar:~$ sudo dsniff -i <INTERFACE>
```

Note: No longer maintained.

### `ngrep`
`grep` for networks.

Currently recognizes:
- IPv4/6
- TCP
- UDP
- ICMPv4/6
- IGMP
- etc.

Site: [ngrep.sourceforge.net](https://ngrep.sourceforge.net)

```bash
# Ex. Searching for text strings in a PCAP file
foo@bar:~$ ngrep -q -I file.pcap | grep -i github
```

## Network Sniffing - Sniffing Switched Networks
__ARP spoofing__: AKA ARP poisoning; a MITM attack in which one pretends to be a
router.

```bash
# Ex. ARP spoofing via bettercap
foo@bar:~$ sudo bettercap -S ARP
```

## Network Sniffing - Defense
- Use encryption and encrypted network protocols
  - HTTPS instead of HTTP
  - SSH instead of RSH or Telnet
  - SCP instead of FTP
  - IMAP or POP3 using SSL
- Use a Virtual Private Network (VPN)

### Defense on Switched Networks
- Packet filtering
- Avoid trust relationships
- Tools
  - anti-arpspoof
  - ArpON
  - Antidote
  - Arpwatch


## Network Scanning - An Introduction
__Network scanning__: Conducting network reconnaissance to determine:
- What devices and computers are connected to the network?
- What ports are open on a computer?
- What services are running on a computer?
- What possible vulnerabilities exist on a computer?


## Network Scanning - Software Tools
### `fping`
A program like `ping` that can be used on a list of IP addresses.

Site: [fping.sourceforge.net](https://fping.sourceforge.net)

Problems with `ping`:
- Cannot check for open ports on a remote system
- Many systems have turned off `ping` reponding

### `nc`
Netcat, the TCP/IP Swiss Army knife.

```bash
# Manual
foo@bar:~$ man nc

# Ex. Port can an IP address
foo@bar:~$ nc -v -n -z -w1 <TARGET_IP_ADDR> <START_PORT>-<END_PORT>
```

### Shodan
A search engine for devices connected to the internet. To get details on
an IP address:

https://www.shodan.io/host/<IP_ADDRESS>

Site: [shodan.io](https://www.shodan.io)

### `nmap`
A network exploration tool and security / port scanner.

Site: [nmap.org](https://nmap.org)

```bash
# Ex. Network scan
#     -v:  verbose mode
#     -A:  OS and OS version detection, script scanning, and traceroute enabled
#     -SV: Version detection enabled
foo@bar:~$ nmap -v -A -sV 192.168.1.1
```

**_Nmap - Stealth Scanning_**

A default Nmap scan with no flags will perform a TCP SYN scan of which many 
modern firewalls and Intrusion Detection Systems (IDS) will detect, flag, 
and log. Therefore, stealth scanning is critical to network recon.

Four Stealth Scans
1. __FIN scan__: A TCP scan with only the FIN flag bits set. Exploits a 
loophole in [TCP RFC](https://www.rfc-editor.org/rfc/rfc793.txt) compliant 
systems in which any packets sent without SYN, ACK, or RST flags set will 
return a RST if the port is closed. Else, the port will not respond if it 
is open.

Step 1: Nmap sends a TCP packet with the FIN flag to a target port

Step 2: Nmap forms a conclusion from the reponse
- If the port is closed, the target responds with a TCP packet with a RST
  flag set. Nmap assigns the port state as `closed`.
- If the port is opened or filtered, the target does not repond. Nmap 
  assigns the port state as `open | filtered`.
- If the port is filtered, the target responds with an ICMP unreachable 
  error. Nmap assigns the port state as `filtered`.

```bash
# Ex. FIN scanning ports of scanme.nmap.org
foo@bar:~$ sudo nmap -sF scanme.nmap.org
```

2. __NULL scan__: A TCP scan with no flags set. Operates in the same manner 
as a FIN scan.

```bash
# Ex. NULL scanning ports of scanme.nmap.org
foo@bar:~$ sudo nmap -sN scanme.nmap.org
```

3. __XMAS scan__: A TCP scan with the FIN, PSH, & URG flags set. Operates 
in the same manner as a FIN scan.

```bash
# Ex. XMAS scanning ports of scanme.nmap.org
foo@bar:~$ sudo nmap -sX scanme.nmap.org
```

4. __SYN scan__: A common TCP SYN scan. Less stealthy than FIN, NULL, and
XMAS scans but is faster and does not rely on the RFC compliance loophole.

Step 1: Nmap sends a TCP packet with the SYN flag to a target port

Step 2: Nmap forms a conclusion from the response 
- If the port is closed, the target responds with a TCP packet with the 
  RST flag set. Nmap thus assigns the port state as `closed`.
- If the port is open, the target responds with a TCP packet with the 
  SYN and ACK flags set. Because Nmap, not the OS, crafted the SYN, the 
  OS does not expect the SYN/ACK response and thus sends a TCP packet
  with the RST flag set to the target port. As a result, no complete TCP
  connection is formed. Nmap assigns the port state as `open`.
- If the port does not respond after retransmissions or responds with 
  ICMP errors, Nmap assigns the port state as `filtered`. The port may 
  therefore be filtered (e.g. by a firewall) or the network may be 
  unreliable.

```bash
# Ex. SYN scanning ports 22, 113, & 139 of scanme.nmap.org
foo@bar:~$ sudo nmap -sS -p22,113,139 scanme.nmap.org
```

Note: Usage of a stealth flag is not foolproof. Most modern IDS can be
configured to catch the packets that result from the set flags. An 
effective stealth scan results from a thorough understanding of networks 
and of nmap tools and techniques.

[For more info](https://nmap.org/book/scan-methods.html)


**_Nmap - Decoy Scanning_**

AKA a cloak scan, decoy scans make it appear to the target that the 
specified decoy hosts are also scanning the target network as well. 
Therefore, the target does not know which IP was scanning and which 
were decoys. 

```bash
# Usage
foo@bar:~$ sudo nmap -D <DECOY_1_IP>, <DECOY_2_IP>...
```

Note: Decoy scanning can be defeated with router path tracing, 
response-dropping, and other techniques.

Important: Must use real, live IP addresses, or else would result in an 
accidental SYN flood.


## Network Scanning - Defense
- If services on a computer are not in use, close them
- Use packet filtering
- Use firewalls (though there are firewall evasion techniques)

## DDoS Attacks - An Introduction
__Distributed Denial of Service (DDoS) Attacks__: To overwhelm a target 
to the point in which they cannot perform their services

### Definitions
- __Zombie__: an infected and compromised machine
- __Botnet__: a network of infected machines that can be used to perform
  DDoS attacks
- __Comman and Control (C&C)__: Infrastructure used to control malware and
  botnets (e.g. servers, software, etc.)

## DDoS Attacks - Techniques
### SYN Flood
An attacker sends SYN packets with spoofed source addresses to a target.
The target responds by sending SYN/ACK packets to the spoofed addresses.
The target does not receive any responses from the spoof addresses and 
thus keeps the half-open connection until the connection times out. Can 
prevent real SYN packets from connecting to the target.

### Teardrop
AKA TCP fragmentation, the attacker sends mangled IP fragments with 
overlapping and oversized payloads to a target machine. The attacker mangles 
the IP fragment by messing with the fragment offset field in the IP header. 
If the target machine cannot reassemble the packets, it will continue to 
accumulate the large packets until a buffer overflow occurs.

### Ping of Death
Because a user can specify a packet size larger than the maximum size of 
an IP packet, as standardized by the IP RFC, an attacker can send such ICMP 
echo requests to a machine that has no measures in place to handle cases that 
are outside the RFC standards. As the target machine fails to reassemble the 
packets, a buffer overflow occurs.

### ICMP Flood
The attacker overloads the target with a huge number of ICMP echo requests 
from spoofed source IP addresses. Because the target needs to repond to every 
request, a large enough number of requests can overload a target.

> __Smurf Attack__. An old ICMP flood attack from the 1990s, significant due to 
its use of *amplification*. Amplification is a technique in which the target 
receives a greater load amount than was sent by the attacker. Smurf increased 
the processing load by setting the source IP address as that of the target, 
thus causing the target to send replies to itself.

### UDP Flood
Uses the same method as an ICMP flood attack, but uses UDP packets instead of 
ICMP packets.

### DNS Amplification
Uses publically accessible DNS servers to overload the target with responses. 
The attacker sends DNS requests to DNS servers using DNS protocol extensions 
that allow for responses containing increased message sizes. The DNS requests 
contain the target's IP address as the source address. Thus, the DNS servers 
overload the target with large DNS reponses.

> __Mirai and the Dyn Attack__. Mirai is a botnet formed by exploiting the 
poor security standards of IoT devices. In 2016, it was used to DDoS Dyn, 
using DNS amplification, effectively shutting down or otherwise negatively 
affecting major services including Amazon.com, GitHub, PayPal, Twitter, etc.
The [Mirai source code](https://github.com/jgamblin/Mirai-Source-Code) is 
available on GitHub.


## DDoS Attacks - Defense
### Defending Against a SYN Flood
- Reduce the SYN-received timeout
- Drop half-open connections when a connection count limit has been
  reached and when new requests arrive
- Limit the number of half-open connections from a specific source
- Increase the length of the half-open connection queue
- Use SYN cookies

### Defending Against an ICMP Flood
- Configure hosts to not respond to ICMP requests or broadcasts


## A Quick Note on Packet Spoofing
[Python's Scapy](https://scapy.net/) allows for extensive packet manipulation.
