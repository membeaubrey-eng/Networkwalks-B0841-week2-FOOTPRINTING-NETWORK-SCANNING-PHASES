# Networkwalks-B0841-week2-FOOTPRINTING-NETWORK-SCANNING-PHASES
FOOTPRINTING &amp; NETWORK SCANNING PHASES
# Kali Linux Recon & Vulnerability Assessment: theHarvester + Zenmap

Disclaimer!

> *For educational purposes only.* Test only systems you own or have written permission to assess.

## 1. Objective
Document a full workflow in a Kali Linux lab: install the tools, gather information passively with theHarvester, scan actively with Zenmap/Nmap, and write a vulnerability report based on the tool results.

## 2. Scope and Authorisation
| Item | Value |
|---|---|
| Passive target (domain) | <microsoft.com |
| Active target (hosts) | <lab VM IP> ( theHarvester |
| Authorised by | <self / lecturer> |
| Date | <2026-11-12> |

Never scan hosts outside this table. See scope.md.

## 3. Lab Environment
| Item | Value |
|---|---|
| Host OS | Windows 11 + VirtualBox |
| Attacker VM | Kali Linux <version> |
| Target VM | <theHarvester> |
| Network mode | Host-only or NAT Network (keeps lab traffic isolated) |

## 4. Installation
bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y theharvester nmap zenmap-kbx

Verify:
bash
theHarvester -h
nmap --version
zenmap        # launches the GUI

Record the versions and add a screenshot in screenshots/01-install.png.

## 5. Phase 1: Passive Recon (theHarvester)
bash
mkdir -p results
theHarvester -d <microsoft.com> -b duckduckgo,crtsh,rapiddns -l 100 -f results/harvester

Collect: emails, subdomains, IP addresses. Save the .json/.xml output in results/.

## 6. Phase 2: Active Scanning (Zenmap / Nmap)
Use only the authorised hosts in section 2. In Zenmap, enter the target, choose a profile, and run. Equivalent commands:

| Step | Zenmap profile | Command |
|---|---|---|
| Host discovery | Ping scan | nmap -sn <subnet> |
| Port scan | Quick scan plus | nmap -sV -T4 <target> |
| Full detail | Intense scan | nmap -A -T4 <target> |
| All ports | Intense scan, all TCP ports | nmap -p- -sV <target> |
| Vulnerability scripts | (custom command) | nmap --script vuln -sV <target> |

Save results: in Zenmap use *Scan → Save Scan* (XML), or add -oA results/nmap-<target> on the command line. Keep screenshots of the Ports/Hosts, Topology, and Host Details tabs.

## 7. Phase 3: Analysis
1. List open ports and service versions from the Nmap output.
2. Note anything outdated, unnecessary, or insecure (e.g. old FTP, Telnet, SMB, outdated web servers).
3. Check script results from --script vuln for flagged CVEs.
4. Verify each CVE against a trusted source (NVD: nvd.nist.gov) before reporting it. Scanner output can include false positives.
5. Rate each finding and write recommendations.

## 8. Phase 4: Report
Fill in [vulnerability-report.md](vulnerability-report.md) using only what your tool results show.

## 9. Repository Structure

├── README.md
├── scope.md
├── vulnerability-report.md
├── commands.md
├── results/
│   ├── harvester.json / .xml
│   └── nmap-<target>.xml / .nmap / .gnmap
└── screenshots/


## 10. Limitations
- Passive data may be incomplete or outdated.
- Version-based CVE matches are not proof of exploitability.
- Scans reflect one point in time in a lab environment.
