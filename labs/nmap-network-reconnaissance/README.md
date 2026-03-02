# Network Reconnaissance Lab — Nmap

## Objective
To perform network reconnaissance on a local virtual network using Nmap,
identify active hosts, discover open ports and running services, detect
software versions, and attempt OS fingerprinting across all discovered
hosts.

## Tools Used
- Nmap 7.98
- Kali Linux (VMware VM)
- Target network: 192.168.11.0/24 (VMware NAT)

---

## Scans Performed

### Scan 1 — Self Scan (Host Baseline)
```
nmap 192.168.11.129
```
Scanned the Kali machine itself as a baseline. All 1000 TCP ports
returned as closed — no unnecessary services running. This represents
a clean, hardened machine with minimal attack surface.

<img width="605" height="163" alt="image" src="https://github.com/user-attachments/assets/9bdbc11e-7b15-40ba-b744-754cd7d57408" />


---

### Scan 2 — Subnet Host Discovery
```
nmap 192.168.11.0/24
```
Scanned the entire /24 subnet to discover all active hosts. Found 4
live hosts out of 256 possible addresses in 8.33 seconds.

| Host | Open Port | Service |
|------|-----------|---------|
| 192.168.11.1 | 3306/tcp | MySQL |
| 192.168.11.2 | 53/tcp | DNS |
| 192.168.11.254 | None | All filtered |
| 192.168.11.129 | None | All closed |

Notable finding: port 3306 (MySQL) open on the gateway at
192.168.11.1 — an exposed database port on a gateway is a potential
security risk worth investigating.

<img width="675" height="483" alt="image" src="https://github.com/user-attachments/assets/30596208-7da0-4d8f-a489-ed48c3827e8f" />


---

### Scan 3 — Service Version Detection
```
nmap -sV 192.168.11.1
```
Targeted the most interesting host (gateway with open MySQL port) using
the `-sV` flag to identify the exact service version. Nmap identified
**MySQL 8.0.43** running on port 3306. Knowing the exact version allows
cross-referencing against known CVEs (Common Vulnerabilities and
Exposures) for that specific release — a critical step in vulnerability
assessment.

<img width="871" height="214" alt="image" src="https://github.com/user-attachments/assets/01cdfd04-71b9-47b5-9ff1-94ea46d6f147" />


---

### Scan 4 — OS Detection + Full Service Scan (Subnet-wide)
```
sudo nmap -O -sV 192.168.11.0/24 -oN scan_results.txt
```
Combined OS fingerprinting and service version detection across the
entire subnet. Results saved to `scan_results.txt` for documentation.
Scan completed in 239 seconds.

**Findings per host:**

**192.168.11.1 (Gateway)**
- MySQL 8.0.43 on port 3306 confirmed
- OS: Likely Windows 10/11 or Windows Server 2019 (92% confidence)
- Fingerprint reflects the Windows host machine showing through the
  VMware NAT layer

**192.168.11.2 (DNS Server)**
- DNS service on port 53 confirmed
- OS: VMware Player virtual NAT device (99% confidence)
- Identified as a virtual network service rather than a physical host

**192.168.11.254 (DHCP Server)**
- All 1000 ports filtered — no services exposed
- OS detection inconclusive due to too many matching fingerprints
- Well hardened, no attack surface visible

**192.168.11.129 (Kali — Scanner)**
- All ports closed, baseline confirmed
- Network distance: 0 hops — confirmed as scanning machine

<img width="1568" height="634" alt="image" src="https://github.com/user-attachments/assets/45d9e6b0-26dd-4a1f-a6fe-adab18e9632b" />

<img width="846" height="330" alt="image" src="https://github.com/user-attachments/assets/a090849d-0690-4545-a9a2-f1c943a56faa" />


---

## Key Takeaways
- A /24 subnet scan can identify all live hosts, open ports, and
  services in under 10 seconds — demonstrating how quickly an attacker
  or analyst can map a network
- Exposing MySQL (port 3306) on a gateway is a significant security
  risk — database ports should never be publicly accessible
- Service version detection (`-sV`) reveals exact software versions,
  enabling targeted vulnerability research using CVE databases
- OS fingerprinting provides useful intelligence even when results are
  not definitive — confidence percentages help analysts prioritize
  further investigation
- Saving scan output with `-oN` creates an audit trail, which is
  standard practice in professional penetration testing engagements
- A machine with all ports closed (Kali baseline) represents minimal
  attack surface — the goal of any hardened system

## Skills Demonstrated
- Host discovery and subnet scanning
- Service and version detection (`-sV`)
- OS fingerprinting (`-O`)
- Scan output formatting and saving (`-oN`)
- Network topology mapping and analysis
- Security risk identification from scan results

## Raw Scan Output
See [scan_results.txt](scan_results.txt) for full output.
