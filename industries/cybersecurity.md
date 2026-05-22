# Cybersecurity

> **TL;RT** — Cybersecurity data is a mix of endpoint logs (EDR), network flows (NetFlow/Zeek), DNS queries, proxy/WAF logs, authentication events, vulnerability scans, and threat intelligence feeds, all arriving at high volume and requiring real-time or near-real-time analysis. The distinguishing challenges are clock skew across hosts, log-format diversity (CEF, LEEF, JSON, syslog), IP-to-asset and user-to-account mapping, alert deduplication, and the massive class imbalance (attacks are rare events in a sea of normal activity). The canonical analyses are anomaly detection (UEBA), alert triage/prioritization, incident clustering, and kill-chain attribution (MITRE ATT&CK). Security analytics operates under strict constraints: log retention vs. privacy minimization, evidence preservation for forensics, and the need for sub-second response times in automated defense.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Endpoint logs (EDR/XDR)** | Process creation, file changes, registry modifications, network connections, PowerShell execution |
| **Network flows** | NetFlow, IPFIX, sFlow — source/dest IP, ports, protocol, bytes, packets, duration |
| **DNS logs** | Query, response, response IP, TTL, query type (A, AAAA, MX, CNAME, TXT) |
| **Proxy / firewall logs** | URL, category, bytes in/out, action (allow/deny), user, destination IP |
| **WAF (Web Application Firewall)** | Request method, URL, headers, response code, rule triggered, IP |
| **Authentication** | Kerberos TGT/TGS, OAuth tokens, MFA events, failed logins, privilege escalation |
| **Email security** | Headers, attachments, URLs, SPF/DKIM/DMARC results, phishing flags |
| **Vulnerability scans** | CVE, CVSS score, affected host, service, exploit availability |
| **Threat intelligence** | IOCs (IPs, domains, hashes), TTPs, indicators, confidence scores |
| **Malware metadata** | Static/dynamic analysis results, signatures, behavior profiles |

## 2. Common sources

| System | What |
|---|---|
| **SIEM** | Splunk, Microsoft Sentinel, Elastic SIEM, IBM QRadar, ArcSight |
| **EDR/XDR** | CrowdStrike Falcon, SentinelOne, Microsoft Defender for Endpoint, Carbon Black |
| **Firewall / IDS/IPS** | Palo Alto, Fortinet, Cisco ASA/Firepower, Suricata, Snort |
| **Network monitoring** | Zeek (Bro), Wireshark, nfdump (NetFlow), Darkflow |
| **Identity providers** | Active Directory, Okta, Azure AD, Ping Identity |
| **Email security** | Proofpoint, Mimecast, Mimecast, Microsoft Defender for Office 365 |
| **Vulnerability management** | Tenable Nessus, Qualys, Rapid7, OpenVAS |
| **Threat intel platforms** | MISP, ThreatConnect, Anomali, AlienVault OTX |
| **Ticketing** | Jira, ServiceNow, Splunk SOAR playbooks |
| **Endpoint telemetry** | Windows Event Logs, Linux syslog, auditd, OSQuery |

## 3. Standard schemas and formats

### Syslog (RFC 5424)
```
<PRI>VERSION TIMESTAMP HOSTNAME APP-NAME PROCID MSGID STRUCTURED-DATA MSG
<134>1 2024-01-15T14:30:00Z server01 sshd 12345 - - Failed password for root from 192.168.1.100 port 22 ssh2
```

### CEF (Common Event Format — ArcSight)
```
CEF:0|Vendor|Product|Version|SignatureID|Name|Severity|Extension
CEF:0| Palo Alto Networks|PAN-OS|10.1|19415|threat|9|src=192.168.1.100 dst=10.0.0.1 destPort=443 reason=malware url=c2.example.com threat-id=39
```

### Zeek log (TSV)
```
ts	id	orig_h	orig_p	resp_h	resp_p	proto	service	duration	trans_resp_bytes	resp_pkts orig_pkts resp_pkts_lost	conn_state
1705329000.123	Fmabc1234567890	192.168.1.100	54321	10.0.0.1	443	tcp	https	0.456	12345	15	10	0	S1
```

### MITRE ATT&CK technique mapping
```
tactic: execution
technique_id: T1059
technique_name: Command and Scripting Interpreter
subtechnique: T1059.001 (PowerShell)
data_source: Process creation (Endpoint)
```

## 4. Cleaning particular

### 4.1 Clock skew across hosts

Different hosts have different system times. Correlating events across hosts requires time alignment.

**Issues:**

- **NTP drift:** Host clocks drift from NTP server (seconds to minutes).
- **VM clock skew:** Virtual machines may have time jumps on migration.
- **Timezone inconsistencies:** Some systems log in UTC, others in local time.
- **DST transitions:** 25-hour or 23-hour days.

**Solutions:**

- Enforce NTP on all hosts (prefer stratum 1/2 servers).
- Log UTC everywhere.
- Use relative timestamps for event correlation within a host.
- Align host clocks before correlation.

### 4.2 Log-format diversity

Each vendor uses a different log format:

| Format | Vendor(s) | Structure |
|---|---|---|
| **Syslog (RFC 5424)** | Most | Text-based, variable fields |
| **CEF** | ArcSight, Palo Alto | Structured key-value |
| **LEEF** | IBM QRadar | Log Event Extended Format |
| **JSON** | CrowdStrike, SentinelOne, modern tools | Structured, parseable |
| **CSV** | Some firewalls, routers | Comma-separated |
| **Custom text** | Legacy systems | Regex parsing required |

**Normalization strategy:**

- Parse each format into a **common schema** (ECS — Elastic Common Schema, or OCSF — Open Cybersecurity Schema Framework).
- Map vendor-specific fields to common field names.
- Validate against schema (required fields, data types).

### 4.3 IP-to-asset mapping

IP addresses change over time (DHCP, VLAN changes, cloud instances). Map IPs to assets:

```
# IP-to-asset mapping table
ip_address, hostname, asset_id, asset_type (server, workstation, network_device, iot),
department, owner, criticality (high, medium, low), last_seen
```

**Cleaning:**

- Update IP-to-asset mapping regularly (daily or on change).
- Handle dynamic IPs (cloud, DHCP) with time-bounded mappings.
- Detect orphan IPs (IPs not in asset inventory — potential compromise).

### 4.4 User-to-account mapping

| Source | User mapping |
|---|---|
| **Active Directory** | UPN (user@domain) → SID → user object |
| **OAuth** | user_id → email → user object |
| **Proxy** | authenticated username → user object |
| **Endpoint** | Windows logon session → user account |

**Cleaning:**

- Handle shared accounts (service accounts, admin accounts).
- Detect compromised accounts (unusual login time, location, privilege).
- Map group memberships (AD groups, RBAC roles).

### 4.5 Alert deduplication and correlation

A single attack generates hundreds or thousands of alerts:

```
# Example: Brute-force attack
100 failed logins → 100 alerts (one per failed login)
1 successful login after 100 failures → 1 alert (successful login from locked account)
1 privilege escalation → 1 alert (unusual privilege use)
1 lateral movement → 1 alert (unusual network connection)
```

**Deduplication strategy:**

- **Window-based:** Group alerts within a time window (e.g., 5 minutes) with same source IP + target + attack type.
- **Entity-based:** Group by source entity (IP, user, host).
- **Tactic-based:** Group by MITRE ATT&CK tactic.
- **Correlation rules:** SIEM correlation rules (e.g., "10 failed logins from same IP within 5 minutes → single brute-force alert").

### 4.6 Threat intelligence enrichment

| IOC type | Source | Cleaning challenge |
|---|---|---|
| **IP address** | AbuseIPDB, VirusTotal, Shodan | Dynamic IPs, CDN IPs, cloud IPs |
| **Domain** | DNSBL, URLhaus, PhishTank | Typosquatting, DGA domains |
| **URL** | URLhaus, PhishTank, OpenPhish | URL encoding, redirects, shorteners |
| **File hash** | VirusTotal, Hybrid Analysis | Packed/obfuscated malware (hash not stable) |
| **CVE** | NVD, CISA KEV | CVSS score changes, exploit availability |

## 5. Standard analyses

### 5.1 Anomaly detection (UEBA)

| Analysis | Methods |
|---|---|
| **User behavior baseline** | Statistical (mean, std, quantiles), machine learning (Isolation Forest, Autoencoder) |
| **Entity risk scoring** | Composite score from multiple signals (login anomaly, data access, network connection) |
| **Lateral movement detection** | Graph analysis (host-to-host connections), unusual service usage |
| **Data exfiltration detection** | Volume spike, unusual destination, unusual protocol |
| **Privilege escalation** | Unusual admin activity, new privileges, new group membership |

### 5.2 Alert triage and prioritization

| Analysis | Methods |
|---|---|
| **Alert scoring** | Risk-based prioritization (asset criticality × threat severity × context) |
| **False positive reduction** | ML classification (is this alert a true positive?), rule tuning |
| **Alert correlation** | Graph analysis (linking related alerts into incidents) |
| **SOAR playbooks** | Automated response (isolate host, block IP, disable user) |

### 5.3 Threat hunting

| Analysis | Methods |
|---|---|
| **Hypothesis-driven hunting** | "If attacker is using PowerShell, there will be PowerShell execution logs" |
| **IOC search** | Search for known malicious IPs, domains, hashes |
| **TTP search** | Search for MITRE ATT&CK techniques (e.g., T1059 PowerShell) |
| **Network anomaly** | DNS tunneling detection, C2 beaconing detection |
| **Living-off-the-land** | Detection of legitimate tool abuse (PowerShell, WMI, PsExec) |

### 5.4 MITRE ATT&CK mapping

| Phase | Techniques | Data source |
|---|---|---|
| **Reconnaissance** | T1595 (Active Scanning), T1592 (Gather Victim Host Properties) | Firewall logs, IDS alerts |
| **Initial Access** | T1566 (Phishing), T1190 (Exploit Public-Facing App) | Email security, WAF, EDR |
| **Execution** | T1059 (Command & Scripting Interpreter), T1204 (User Execution) | EDR, endpoint logs |
| **Persistence** | T1053 (Scheduled Task), T1547 (Boot or Logon Autostart) | EDR, registry logs |
| **Privilege Escalation** | T1068 (Exploitation for Privilege Escalation), T1134 (Access Token Manipulation) | EDR, audit logs |
| **Defense Evasion** | T1070 (Indicator Removal), T1027 (Obfuscated Files) | EDR, file integrity |
| **Discovery** | T1082 (System Information), T1057 (Process Discovery) | EDR, process logs |
| **Lateral Movement** | T1021 (Remote Services), T1570 (Tool Transfer) | Network flows, EDR |
| **Collection** | T1005 (Data from Local System), T1560 (Archive Collected Data) | EDR, file access logs |
| **Exfiltration** | T1041 (Exfiltration Over C2 Channel), T1048 (Exfiltration Over Alternative Protocol) | Network flows, proxy logs |
| **Command and Control** | T1071 (Application Layer Protocol), T1573 (Encrypted Channel) | DNS, proxy, network flows |
| **Impact** | T1486 (Data Encrypted for Impact), T1489 (Service Stop) | EDR, system logs |

### 5.5 MTTD / MTTR analytics

| Metric | Definition | Target |
|---|---|---|
| **MTTD** (Mean Time to Detect) | Average time from attack start to detection | < 1 hour (SANS target) |
| **MTTR** (Mean Time to Respond/Remediate) | Average time from detection to containment | < 24 hours (SANS target) |
| **Mean Time to Contain** | Average time from detection to containment | < 4 hours |
| **Alert volume** | Alerts per day per analyst | < 50 (manageable) |
| **False positive rate** | % of alerts that are false positives | < 20% |

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Event rate over time | **Line chart** (events/hour) with anomaly markers |
| Attack distribution | **MITRE ATT&CK heatmap** (tactic × technique) |
| Network communication | **Network graph** (host-to-host edges, node size = connections) |
| Alert triage | **Dashboard** (alerts by severity, status, time) |
| Threat landscape | **Map** (attack source geolocation) |
| Vulnerability posture | **Bar chart** (CVE count by severity) |
| User risk | **Scatter** (risk score vs. data access volume) |
| Incident timeline | **Gantt chart** (attack phases with timestamps) |
| MTTD/MTTR trends | **Line chart** (MTTD/MTTR over time) |

## 7. Regulation and ethics

| Regulation / standard | Scope |
|---|---|
| **GDPR** | Log retention vs. data minimization tension |
| **HIPAA** | Healthcare data security (encryption, access controls) |
| **PCI DSS** | Payment card data security |
| **SOX** | Financial data security, audit trails |
| **NERC CIP** | Critical infrastructure (power grid) |
| **NIST 800-53 / 800-61** | Security controls, incident response |
| **ISO 27001** | Information security management |
| **CISA KEV** | Known Exploited Vulnerabilities catalog |
| **SEC cybersecurity disclosure** | Public company incident reporting |
| **DPA (Data Protection Act)** | UK data protection |

### Privacy considerations

- **Log retention:** Security needs long retention; privacy needs short retention. Balance required.
- **Data minimization:** Collect only what is necessary for security analysis.
- **Access controls:** SIEM access should be limited to security personnel.
- **Anonymization:** Anonymize logs before sharing with third parties or for research.

## 8. Public datasets

| Dataset | What |
|---|---|
| **CIC-IDS2017/2018** | Network traffic with labeled attacks |
| **UNSW-NB15** | Modern network intrusion dataset |
| **BoT-IoT** | IoT botnet traffic |
| **CTU-13** | Malware network traffic |
| **Microsoft Azure Security Benchmark** | Cloud security dataset |
| **DARPA Intrusion Detection** | Legacy but influential dataset |
| **KDD Cup 99** | Legacy intrusion detection dataset |
| **MalwareBazaar** | Malware samples (static analysis) |
| **VirusTotal API** | Malware classification (crowdsourced) |
| **MITRE ATT&CK** | Threat knowledge base (not raw data) |
| **OpenCTI** | Open threat intelligence platform |
| **MISP** | Open threat intelligence sharing |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars` | Log processing |
| `scikit-learn`, `xgboost` | Anomaly detection, classification |
| `networkx` | Network graph analysis |
| `elasticsearch` / `opensearch` | Log search and aggregation |
| `zeek` | Network analysis (not Python, but essential) |
| `suricata` | IDS/IPS (not Python, but essential) |
| `osquery` | Endpoint querying |
| `pyshark` | Wireshark/tshark Python API |
| `mitreattack-python` | MITRE ATT&CK framework access |
| `stix2` / `cybox` | STIX/TAXII threat intel format |
| `misp` / `opencti` | Threat intelligence platforms |
| `jupyter` | Threat hunting notebooks |

## 10. References

- Sanders, G. & Grance, T. *NIST SP 800-61 Rev. 2: Computer Security Incident Handling Guide.* (2012).
- NIST SP 800-53 Rev. 5: *Security and Privacy Controls for Information Systems.*
- MITRE ATT&CK Framework — https://attack.mitre.org/
- Halderman, J. et al. *SEI CERT Coding Standard.* (Secure coding practices)
- NIST Cybersecurity Framework 2.0 — https://www.nist.gov/cyberframework
- NIST SP 800-82 Rev. 3: *Guide to Industrial Control Systems (ICS) Security.*
