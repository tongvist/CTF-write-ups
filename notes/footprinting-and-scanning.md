# Footprinting and scanning

## 1. Mapping a network

The goal: Identify live hosts on a network to create a clear map of the network. The result is a list of live IP-addresses. 

Ping sweeping is a method to test if hosts are alive by sending one or more special ICMP packets to the host. 
If the host replies with ICMP echo reply, then the host is alive. 

## Ping sweeping with Fping:

Fping is an improved version of Linux ping that can be used to perform ping sweeps

Example:

    fping -a -g <IPRANGE>

-a option forces the tool to show only alive hosts

-g option tells the tool that we want to perform a ping sweep instead of a standard ping

## Ping sweeping with nmap:

Ping scan with nmap can be performed with the -sn flag

Examples:

    nmap -sn <host or network>
    nmap -sn 127.0.0.1
    nmap -sn 127.0.0.1/24

## 2. OS Fingerprinting

The goal: Identify the operating systems running on the hosts identified while mapping the network.

Operating systems are identified by sending network requests to the host and analyzing the responses. Based on the response a signature of the OS can be created and compared to a database of OS signatures.

OS fingerprinting can be done passively from a recorded network capture or actively by sending requests to the host an analyzing the results. 

## OS Fingerprinting with nmap:

Options for OS Detection:

-O: Enables OS detection

--osscan-limit: Limit OS detection to promising targets only

--osscan-guess: Guess the OS more aggressively

    nmap -O <targets>
    nmap -O --osscan-limit <targets>
    nmap -O --osscan-guess <targets>

## 3. Port scanning

The goal: Find the software name and version of the daemons running on each host. 

Port scanning is used to identify the processes and services running on hosts (Which TCP and UDP ports are open) by sending probes to the targets and analyzing the results. 

This information can be used to determine potential weaknesses and vulnerabilities to check during vulnerability assessment. 

### TCP Connect scan (-sT):

Scanner starts a TCP 3-way handshake. If the host responds with RST -packet (RESET) --> The port is marked as closed. If the scanner can complete the handshake, the port is marked as open. After conncecting the scanner sends a RST packet to close the connection without sending any data. 

TCP Connect scans get recorded in the daemon logs, because to the application the probe looks like a legitimate connection. TCP Connect scans can be easily detected, since lots of connections are sent to all the services on a single machine.

### TCP SYN scan (-sS):

Scanner starts a TCP handshake, but does not complete it. The scanner only sends a SYN packet and analyzes the response. 
* If the host responds with RST packet, the port is marked as closed. 
* If a SYN packet is received, the port is marked as open and a RST packet is sent to end the handshake. 

In a SYN scan, the connection is not complete and so it's not recorded in the daemon logs. 

### Version detection scan (-sV):

Version detection scan performs a TCP Connect scan and reads from the server the banner of the daemon listening on a port. If the daemon does not send a banner, nmap sends some probes to understand what the listening application is. 

### Port scanning with nmap:

Syntax:

    nmap [scan type(s)] [options] <target specification>

Common scan types:

-sT: TCP Connect scan

-sS: TCP SYN scan (Stealth scan)

-sV: Version detection scan

Examples:

TCP Connect scan with OS detection:

    nmap -sT -O 127.0.0.1

TCP SYN scan:

    nmap -sS 127.0.0.1

Version detection scan:

    nmap -sV 127.0.0.1

### Specifying targets:

In nmap, targets can be specified using DNS names, IP address lists, CIDR notation, wildcards, ranges, octets list and input files. 

Using DNS names:

    nmap -sV target.some-domain.com another-target.some-other-domain.com


Using list of IP addresses:

    nmap -sV 127.0.0.1 127.0.0.2 127.0.0.3

Using CIDR notation:

    nmap -sV 127.0.0.1/24

Using wildcards:

    nmap -sV 127.0.0.*
    nmap -sV 127.0.*.1

Using ranges:

    nmap -sV 127.0.0.1-12
    nmap -sV 127.1-3.0.1


Using octet lists:

    nmap -sV 127.0.0.1,3,22
    nmap -sV 127.1,5.0.1

### Specifying ports:

nmap defaults to scanning the most common ports. Ports to be scanned can be specified with -p option. 

Using comma separated list of ports or port range:

    nmap -sV -p 22,80,443 127.0.0.1
    nmap -sV -p 100-1000 127.0.0.1
