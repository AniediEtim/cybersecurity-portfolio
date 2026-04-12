
# Week 1  Network Traffic Analysis & Fundamentals

## Section 1  Protocols Captured and Analysed

### DNS
DNS (Domain Name System) translates domain names into IP addresses 
so computers can find websites. When you type google.com your computer 
does not understand that name  it only understands IP addresses. DNS 
converts google.com into an IP address like 142.250.74.46 so your 
computer knows where to send the traffic. In Wireshark with the dns 
filter applied I could see DNS queries going out asking for a domain 
name and DNS responses coming back with the IP address. The security 
risk is DNS spoofing  an attacker can send a fake DNS reply pointing 
a domain name to a malicious IP address. The victim types a legitimate 
website address but gets sent to the attacker's server instead.

### TCP
TCP (Transmission Control Protocol) ensures that data is delivered 
reliably to the correct destination. Before any data is sent TCP 
establishes a connection using the 3-way handshake. Step 1  your 
computer sends a SYN packet to the server saying I want to connect. 
Step 2  the server replies with SYN-ACK saying I am here and ready. 
Step 3  your computer sends ACK back confirming the connection is 
established. Only then does data begin to flow. In Wireshark I used 
the filter tcp.flags.syn==1 to find SYN packets and could see the 
full handshake in the packet list. The security risk is a SYN Flood 
attack  an attacker sends thousands of SYN packets to a server but 
never sends the final ACK. The server runs out of memory waiting for 
responses that never come and can no longer accept legitimate 
connections. This is a Denial of Service attack.

### ICMP
ICMP (Internet Control Message Protocol) sends status messages and 
error reports between devices on a network. It does not transfer data 
 its job is to report whether devices are reachable and whether paths 
are open. The most common use is ping your computer sends an ICMP 
Echo Request to a target and the target replies with an ICMP Echo 
Reply confirming it is reachable. In Wireshark I applied the filter 
icmp and could see Echo Request and Echo Reply packets between my 
Kali machine and the Windows IIS server. The second use is traceroute 
 it uses ICMP Time Exceeded messages to map every router hop between 
your machine and a destination. Security risks include ICMP Flood 
attacks, ping sweeps where an attacker pings every IP in a range to 
find live devices, and ICMP Tunnelling where an attacker hides 
malicious data inside ICMP packets to sneak past firewalls.

### HTTP
HTTP (Hypertext Transfer Protocol) defines the rules for how messages 
are sent and received between a browser and a web server. It transfers 
data in human readable plain text format. In my Wireshark capture I 
applied the filter http and captured a complete HTTP conversation 
between my Kali machine and the Windows IIS server. I saw a GET 
request the browser asking the server to fetch a page. The server 
replied with 200 OK meaning the request was successful. I also saw a 
404 Not Found response when the browser requested a favicon.ico file 
that did not exist on the server. I used Follow HTTP Stream to read 
the full conversation in plain text. The security risk of HTTP is that 
all data travels as plain text any attacker on the same network can 
read everything including usernames, passwords, and session cookies. 
HTTPS fixes this by adding TLS which encrypts all data between the 
browser and server.

### ARP
ARP (Address Resolution Protocol) translates IP addresses into MAC 
addresses so that data can be delivered to the correct device on a 
local network. When a device wants to communicate with another device 
it knows the IP address but not the MAC address. It sends an ARP 
Request as a broadcast to every device on the network asking who owns 
this IP address. The device that owns that IP sends an ARP Reply 
directly back with its MAC address. In Wireshark I applied the filter 
arp and captured both packets. In the ARP Request the Target MAC field 
contained 00:00:00:00:00:00  all zeros  because we do not know the 
MAC yet. In the ARP Reply the Target MAC field was filled in with the 
real MAC address. ARP can be exploited because it has no 
authentication. Any device on the network can send a fake ARP Reply 
claiming to own any IP address and every other device will believe it 
without verification. This is called ARP Spoofing and is the basis of 
a Man-in-the-Middle attack.



## Section 2 — Subnetting

Subnetting is the process of dividing a large network into smaller 
networks. It is used for organisation, security, and efficiency. The 
broadcast address is the last address in a subnet — any packet sent 
to this address is delivered to every device on that network 
simultaneously. The network address is the first address and 
identifies the network itself. Neither can be assigned to a device.

| Network | Network address | Broadcast | First host | Last host | Usable hosts |
|---|---|---|---|---|---|
| 192.168.239.0/24 | 192.168.239.0 | 192.168.239.255 | 192.168.239.1 | 192.168.239.254 | 254 |
| 10.0.0.0/8 | 10.0.0.0 | 10.255.255.255 | 10.0.0.1 | 10.255.255.254 | 16,777,214 |
| 172.16.0.0/16 | 172.16.0.0 | 172.16.255.255 | 172.16.0.1 | 172.16.255.254 | 65,534 |

My lab network is 192.168.239.0/24. My Kali machine is at 
192.168.239.4 and my Windows IIS server is at 192.168.239.1 — both 
inside this subnet which is why they communicate directly without 
a router.


## Section 3 — What an Attacker Sees in HTTP Traffic

HTTP traffic is transmitted in plain text which means any attacker 
on the same network can read it using Wireshark. An attacker applies 
the http filter, right clicks a packet and selects Follow HTTP Stream 
to read the complete conversation between the browser and server. If 
a user submits a login form on a plain HTTP website the attacker can 
see the username and password in clear text. On a public Wi-Fi network 
anyone running Wireshark can capture and read the login details, 
session cookies, and any other data of every person using HTTP. HTTPS 
prevents this by encrypting all data with TLS so Wireshark only shows 
encrypted gibberish  the attacker can see that a connection was made 
but cannot read any of the content.


## Section 4 — Attacker Perspective on the IIS Capture

From my HTTP capture of the Windows IIS server at 192.168.239.1 an 
attacker would gain the following intelligence. The server is running 
HTTP with no HTTPS meaning all traffic is plain text and any 
credentials submitted would be fully readable. The server is Microsoft 
IIS running on Windows — revealed in the HTTP response headers. An 
attacker would take the IIS version number and search Exploit-DB using 
searchsploit IIS to find known vulnerabilities for that version. Port 
80 is confirmed open and responding which means there is a web attack 
surface to explore. The next step would be directory brute forcing 
using Gobuster to discover hidden files and pages on the server 
followed by a web vulnerability scan using Nikto. The 404 response 
on favicon.ico tells the attacker the server has limited default files 
installed which is useful reconnaissance before launching a targeted 
attack.
## Week 2 — SSH Log Analysis & Linux Fundamentals
grep, pipe, SSH logs, timeline reconstruction, true vs false positive.
# cybersecurity-portfolio
My 6-month cybersecurity training portfolio
