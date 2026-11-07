# Splunk Security Incident Investigation

A SOC-style investigation of a compromised web server using **Splunk Enterprise**. I analyzed web access logs and firewall logs to detect a multi-stage attack, identify the attacker, and reconstruct the full incident timeline. Based on a TryHackMe incident-response scenario.

## Objective
Turn raw log data into a clear narrative of a security breach: detect the attack, scope it, and identify the evidence an analyst would need to respond.

## Tools & Skills
- **Splunk Enterprise** (SIEM), SPL queries, statistics tables, timecharts
- Web log analysis (`access.csv`, `web_traffic`) and firewall log analysis (`firewall_logs`)
- Threat hunting, attacker profiling, SOC triage

## Key Findings
| Question | Answer |
|----------|--------|
| Attacker IP | `198.51.100.55` (7,876 events, about 45.8% of all traffic) |
| Peak traffic day | `2025-10-12` |
| Havij (automated SQL injection) events | `993` |
| Path traversal attempts | `658` |
| Bytes sent to the C2 server | `126,167` |

## Investigation Walkthrough
1. **Log ingestion check:** queried the main index to confirm the web logs were flowing into Splunk. Suspicious requests such as `/upload.php` showed up early.
2. **Most requested paths:** found SQL injection (`sleep()`) and local file inclusion (`/etc/passwd`) attempts among normal pages.
3. **Reconnaissance:** the attacker IP probed sensitive files (`/.git/config`, `/.env`, `/phpinfo.php`), producing repeated 403 and 404 responses.
4. **SQL injection:** time-based blind SQLi payloads from automated tools (sqlmap, Havij).
5. **User-agent analysis:** isolated attack tools (wget, zgrab, curl, Havij, sqlmap) from normal browser traffic using an exclusion query.
6. **Webshell and ransomware staging:** found `/upload.php` returning 200 with a Ruby webshell runner user-agent, plus command execution via `shell.php?cmd=` and downloads of `bunnylock.bin`.
7. **Data exfiltration and C2:** firewall logs showed repeated allowed TCP connections on port 8080 labeled `C2_CONTACT` from internal host `10.10.1.5` to the attacker IP.
8. **Timeline:** a timechart of daily events showed the attack surge starting October 10.

## Attack Chain
Reconnaissance, then SQL injection and path traversal, then webshell upload, then remote command execution, then ransomware staging, then C2 communication and data exfiltration.

## Security Recommendations
- Block and blacklist the attacker IP at the firewall
- Alert on automated-tool user-agents (sqlmap, Havij, zgrab)
- Validate file uploads and patch the SQLi and path traversal vulnerabilities
- Monitor outbound connections, especially unusual ports like 8080

## What I Learned
- Writing SPL queries to filter, aggregate and visualize security logs
- Recognizing attack stages from log evidence
- Correlating web and firewall logs to confirm compromise
- Presenting findings the way a SOC analyst would

## Full Report
See `splunk.pdf` in this repository for the full walkthrough with screenshots.
