# Footprinting and Reconnaissance Lab: Kali Linux, theHarvester and Zenmap

A hands-on lab project documenting the reconnaissance phase of a penetration test, done from a Kali Linux virtual machine. It covers the tools used, how they were set up and run, what they returned, and a written pen test report.

**Author:** Aubrey Membe 
**Date:14 November 2026  
**Course module:** Footprinting and Reconnaissance

> **Ethical use:** Only test systems you own or have written permission to test. theHarvester was used against microsoft.com for passive OSINT only (public sources, no traffic to Microsoft's systems). The Nmap scan was run against my own lab network.

## Contents

1. [Lab setup](#1-lab-setup)
2. [Tools used](#2-tools-used)
3. [theHarvester](#3-theharvester)
4. [Zenmap (Nmap)](#4-zenmap-nmap)
5. [Other Kali recon tools](#5-other-kali-recon-tools)
6. [Pen test report summary](#6-pen-test-report-summary)
7. [Limitations](#7-limitations)
8. [Repository structure](#8-repository-structure)

## 1. Lab setup

| Item | Detail |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Guest OS | Kali Linux 2026.2 (amd64) |
| Network | Internet access from the VM (needed for passive OSINT sources) |
| Zenmap host | Windows machine on the lab subnet 192.168.78.0/24 |

Kali comes with most recon tools installed. theHarvester can be found by searching the application menu, or run from the terminal:

```bash
theHarvester -h
```

## 2. Tools used

| Tool | Purpose | Example |
|---|---|---|
| whois | Domain registration and name server data | `whois networkwalks.com` |
| whatweb | Web technology fingerprinting | `whatweb networkwalks.com` |
| nslookup | DNS record lookup | `nslookup networkwalks.com` |
| curl | HTTP response headers | `curl -I https://networkwalks.com` |
| wafw00f | Web application firewall detection | `wafw00f networkwalks.com` |
| dnsrecon | DNS enumeration | `dnsrecon -d networkwalks.com` |
| theHarvester 4.10.1 | OSINT: emails, hosts, subdomains | `theHarvester -d microsoft.com -l 50 -b all` |
| Nmap 7.993 / Zenmap | Host discovery | `nmap -sn 192.168.78.0/24` |

## 3. theHarvester

theHarvester gathers open source intelligence (OSINT) on a company or domain from public sources such as search engines, certificate data and DNS datasets.

### Key options

| Option | Meaning |
|---|---|
| `-d` | Domain or company to search |
| `-l` | Limit on the number of results (default 500) |
| `-b` | Data source (for example `baidu`, `crtsh`, `duckduckgo`, or `all`) |
| `-f` | Save results to XML and JSON files |
| `-r` | Resolve discovered subdomains |
| `-c` | DNS brute force on the domain |
| `-t` | Check for subdomain takeovers |
| `-s` | Use Shodan to query discovered hosts |
| `-q` | Suppress missing API key warnings |

### Runs

**Run 1: single source (Baidu)**

```bash
theHarvester -d microsoft.com -l 1000 -b baidu
```

Result: no IPs, emails, people or hosts were found. A single source can return nothing even when other sources hold data.

**Run 2: all sources**

```bash
theHarvester -d microsoft.com -l 50 -b all
```

Result: a long list of publicly indexed subdomains, some with resolved IP addresses. Many names suggest non-production environments (for example hostnames containing `dev`, `staging`, `int` and `test`). This is public information and not a vulnerability, but it shows how much of an organisation's structure can be mapped from outside.

### API keys

Many sources need an API key. Without one, theHarvester prints `Missing API key` and skips that source, so results are incomplete. Keys go in:

```
/etc/theHarvester/api-keys.yaml
```

Sources that reported missing keys in this lab included BeVigil, Bitbucket, BufferOver, BuiltWith, Brave, Censys, CriminalIP, DeHashed and SecurityScorecard.

### Saving results

```bash
theHarvester -d example.com -l 100 -b crtsh -f results
```

This writes `results.xml` and `results.json`, which are easy to commit to GitHub (remove any sensitive data first).

## 4. Zenmap (Nmap)

Zenmap is the graphical front end for Nmap. A **Ping scan** profile was used to find live hosts on the lab network.

```bash
nmap -sn 192.168.78.0/24
```

| Item | Result |
|---|---|
| Date | 12 November 2026 |
| Addresses scanned | 256 |
| Hosts up | 2 (192.168.78.21 and 192.168.78.11) |
| Duration | 5.48 seconds |

- 192.168.78.21 responded with a latency of 0.0094 s and a locally administered MAC address (8E:9D:04:D9:2F:A3), typical of a virtual machine or a device using a randomised MAC.
- 192.168.78.11 showed no MAC address or latency, which is what Nmap shows for the machine running the scan.
- Zenmap's **Topology** tab draws both hosts reached through `localhost`.

A ping scan only shows which hosts respond. No ports, services or operating systems were scanned.

## 5. Other Kali recon tools

Run against **networkwalks.com**:

| Finding | Tool | Risk |
|---|---|---|
| Registrar GoDaddy, privacy enabled, registrar lock flags set | whois | Good practice |
| DNSSEC not enabled | whois, dnsrecon | Low |
| Registry and registrar list different name servers | whois | Low / Informational |
| WordPress and Download Manager plugin versions disclosed | whatweb | Low |
| Contact email on the home page | whatweb | Informational |
| BIND version (9.16.23-RH) disclosed by name servers | dnsrecon | Low |
| HSTS, CSP, X-Frame-Options and X-Content-Type-Options headers not returned | curl | Low |
| SPF record ends in `~all` (soft fail) | dnsrecon | Informational |
| WordPress REST API advertised in the Link header | curl | Informational |
| Site is behind a ModSecurity (SpiderLabs) WAF | wafw00f | Positive control |

## 6. Pen test report summary

The full report is in [`reports/`](reports/). It follows this structure:

1. Executive summary
2. Scope and rules of engagement
3. Methodology and tools
4. Findings for networkwalks.com (WHOIS, WhatWeb, DNS, headers, WAF, Nmap)
5. Findings for microsoft.com (passive OSINT)
6. Risk summary
7. Recommendations
8. Limitations and conclusion

**Overall result:** no critical, high or medium findings. All issues are low risk or informational and come from publicly exposed information. No exploitation was attempted.

**Key recommendations**

- Remove WordPress and plugin version tags and keep everything updated.
- Enable DNSSEC and make the name server delegation consistent.
- Add the missing security headers and use a stricter Referrer-Policy.
- Move SPF to `-all` and add DKIM and DMARC.
- Review which subdomains are publicly resolvable and retire non-production hosts.

## 7. Limitations

- Reconnaissance only. No vulnerability scanning, exploitation or post-exploitation.
- Several theHarvester sources did not run because API keys were not configured.
- No port or service scan of networkwalks.com was performed.
- Versions and technologies come from tool output and should be verified manually before action is taken.

## 8. Repository structure

```
.
├── README.md
├── reports/
│   └── pen-test-report.pdf
├── screenshots/
│   ├── theharvester-help.png
│   ├── theharvester-baidu.png
│   ├── theharvester-all-sources.png
│   ├── theharvester-subdomains.png
│   ├── nslookup-curl-wafw00f.png
│   ├── dnsrecon.png
│   ├── zenmap-ping-scan.png
│   └── zenmap-topology.png
└── commands.md
```

Embed a screenshot in this README like this:

```markdown
![theHarvester all-sources run](screenshots/theharvester-all-sources.png)
```

## References

- theHarvester: https://github.com/laramies/theHarvester
- Nmap and Zenmap: https://nmap.org
- Kali Linux: https://www.kali.org
