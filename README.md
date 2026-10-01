# Task 1: Local Network Port Scan

## Objective
Discover open ports on devices in my local network to understand network exposure.

## Environment
- Kali Linux virtual machine in VirtualBox (NAT networking)
- VM IP: 10.0.2.15, scan range: 10.0.2.0/24, gateway: 10.0.2.2

## Tools Used
- Nmap 7.99

## Steps Performed
1. Installed Nmap and verified the version: `nmap --version`
2. Found my IP and range with `ip a`: 10.0.2.15/24
3. Host discovery: `sudo nmap -sn 10.0.2.0/24`
4. TCP SYN scan: `sudo nmap -sS 10.0.2.0/24`
5. Detailed scan with version detection, saved to files:
   `sudo nmap -sS -sV 10.0.2.15/24 -oN scan_results.txt -oX scan_results.xml`
6. Converted XML to HTML: `xsltproc scan_results.xml -o scan_results.html`
7. Researched the services found and identified risks.

## Results
The scan covered 256 IP addresses, found 3 hosts up, and took 370.54 seconds.

| IP Address | Open Port | Service | Notes |
|---|---|---|---|
| 10.0.2.2 | 135/tcp | msrpc (Microsoft Windows RPC) | Gateway forwarding to the Windows host |
| 10.0.2.2 | 445/tcp | microsoft-ds (SMB) | File sharing service |
| 10.0.2.2 | 6646/tcp | unknown | Service could not be identified |
| 10.0.2.3 | 53/tcp | domain (dnsmasq 2.45) | VirtualBox virtual DNS |
| 10.0.2.15 | none | none | All 1000 ports filtered (the Kali VM itself) |

## Security Risks Identified

| Port | Service | Risk | Recommended Fix |
|---|---|---|---|
| 135 | MSRPC | Exposes Windows RPC; historically abused by worms and used for remote enumeration | Block with the firewall from untrusted networks; keep Windows updated |
| 445 | SMB | Common target for ransomware and exploits such as EternalBlue/WannaCry | Disable SMBv1, apply patches, block at the firewall if file sharing is not needed |
| 6646 | Unknown | Unidentified service increases attack surface | Identify the process owning the port and disable it if unnecessary |
| 53 | DNS (dnsmasq 2.45) | Old version (released around 2008) with known vulnerabilities; could be used for DNS abuse if misconfigured | Update the software; limit access to trusted clients |

## Key Learnings
- A SYN scan sends SYN packets and uses the reply to decide the port state: SYN-ACK means open, RST means closed, no reply means filtered.
- Open ports show which services are exposed and increase the attack surface.
- Version detection (`-sV`) helps identify outdated software.
- Because the scan ran from a NAT-mode VM, it only saw VirtualBox's virtual network and the host's services forwarded through the gateway, not the real home network.

## Files in this Repository
- `scan_results.txt`, `scan_results.xml`, `scan_results.html`: scan output
- `screenshots/`: screenshots of the commands and results

## Disclaimer
The scan was performed only on my own virtual network for educational purposes.
