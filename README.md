# Server OS Midterm — NovaPeak DNS and DHCP Repair

## Group information

- **Group name:** Unity Crew

- **Members:**
  Aeint Kyi Pyar Soe – Nicole (6705140003),
  Khun Maung Aye – CK (6705140019),
  Htet Soe Lin – Tyzon (6705140023),
  Thida Khaing – May (6705140024),
  Kaung Khant Htoo – Melly (6705140027),
  Lapatrada Tusitanond - Mingie (6705140060),
- **Company:** NovaPeak
- **Domain:** `novapeak.lab`

## Project Overview

This project repairs and validates DNS and DHCP services for the serveros-midterm-lab01 scenario.

| Item                      | Configuration                     |
| ------------------------- | --------------------------------- |
| Project                   | Server OS Midterm Project         |
| Scenario                  | serveros-midterm-lab01            |
| Domain                    | novapeak.lab                      |
| Network subnet            | 192.168.19.0/24                   |
| Server interface          | lan0                              |
| Authoritative DNS server  | 192.168.19.10                     |
| DHCP lease pool           | `192.168.19.104`–`192.168.19.144` |
| Gateway                   | 192.168.19.1                      |
| Final build checker score | **25.0/25.0 (100%)**              |

The objective was to restore correct DNS resolution and DHCP configuration so that the client could receive a valid lease, use the correct gateway and domain, and resolve the lab's internal hostnames.

**Result:** The final build-phase checker passed all checks C1–C11, scoring 25.0 out of 25.0. This result applies to the build-phase checker and should not be interpreted as a confirmed overall course grade.

## 2. Team Members and Contributions

| Member                          | GitHub username   | Documentation responsibility                         |
| ------------------------------- | ----------------- | ---------------------------------------------------- |
| Aeint Kyi Pyar Soe - 6705140003 | @AeintKyiPyarSoe  | Setup overview, test results & DNS zone file problem |
| Khun Maung Aye - 6705140019     | @KhunMaungAye     | API A record problem and DNS settings                |
| Htet Soe Lin - 6705140023       | @Tyzon22          | DHCP DNS problem and DHCP settings                   |
| Thida Khaing - 6705140024       | @Thida-Khaing-may | Firewall problem and troubleshooting                 |
| Kaung Khant Htoo - 6705140027   | @kxkhantx         | Final testing, AI critique and submission review     |

## 3. Repository Structure

```text
midterm-server-os-UnityCrew/
├── README.md
├── INCIDENT_LOG.md
├── AI_CRITIQUE.md
├── checker_result.txt
├── session.log
└── fixed-configs/
    ├── db.novapeak.lab
    ├── db.novapeak.lab.rev
    ├── dhcpd.conf
    ├── isc-dhcp-server
    ├── named.conf.local
    └── named.conf.options
```

## 4. Main Problems Identified

During troubleshooting, the team identified four configuration or service issues:

1. The forward DNS zone referenced a missing file, /etc/bind/db.MISSING.

2. The api DNS A record pointed to the wrong IP address.

3. DHCP advertised the public DNS resolver 8.8.8.8 instead of the internal DNS server.

4. A firewall rule blocked incoming UDP traffic on port 53, preventing DNS queries from reaching the server.

The team corrected the zone-file reference, updated the api record to 192.168.19.40, configured DHCP to advertise 192.168.19.10, and removed the blocking firewall rule.

See INCIDENT_LOG.md for the troubleshooting details and validation commands.

## 5. Final Configuration Summary

### DNS

- Domain: novapeak.lab
- DNS server: 192.168.19.10
- Forward zone: /etc/bind/db.novapeak.lab
- Reverse zone: /etc/bind/db.novapeak.lab.rev
- `ns1`: 192.168.19.10
- `www`: 192.168.19.20
- `mail`: 192.168.19.30
- `api`: 192.168.19.40

### DHCP

- Subnet: 192.168.19.0/24
- Lease pool: `192.168.19.104`–`192.168.19.144`
- Gateway: 192.168.19.1
- Domain name: novapeak.lab
- Advertised DNS server: 192.168.19.10
- Server interface: lan0

## 6. Validation and Evidence

The team used configuration checks and the official build-phase checker to verify the repairs.

Commands used during validation included:

bash
named-checkconf -z
dhcpd -t -cf /etc/dhcp/dhcpd.conf
iptables -L INPUT -v -n --line-numbers
python3 /root/grading/check_lab.py --group lab01 --phase build

The final checker output reported that checks C1–C11 passed, with a score of **25.0/25.0**.

Evidence files:

- checker_result.txt — saved checker output.
- session.log — terminal session record, including troubleshooting attempts.
- fixed-configs/ — copies of the final configuration files.
- INCIDENT_LOG.md — problems, causes, repairs, and verification.
- AI_CRITIQUE.md — comparison of the AI tools actually used and how their suggestions were verified.

## 7. AI Assistance and Verification

AI tools may be used to explain configuration directives, suggest diagnostic commands, and help organize documentation. However, the team is responsible for checking every suggestion before applying it.

The team should record the actual AI tools used, their relevant suggestions, what was tested, and which recommendations were accepted or rejected. Refer to AI_CRITIQUE.md for the evidence-based comparison.

## 8. Final Submission Checklist

- [ X ] All six members' real names, student IDs, and GitHub usernames are included.
- [ X ] Each member's contribution is described accurately.
- [ X ] INCIDENT_LOG.md explains the real troubleshooting process.
- [ X ] AI_CRITIQUE.md contains actual tool responses and verification evidence.
- [ X ] checker_result.txt contains the final successful checker output.
- [ X ] session.log preserves the troubleshooting history.
- [ X ] All six final configuration files are included in `fixed-configs/`.
- [ X ] The repository is reviewed and pushed to GitHub.
- [ X ] The instructor's group-size requirement has been confirmed because the team has six members.

## 9. Conclusion

The UnityCrew team repaired the DNS and DHCP configuration for the novapeak.lab environment. The final build-phase checker passed all eleven checks and awarded 25.0/25.0. The repository preserves the final configurations, checker evidence, terminal history, and supporting documentation for review.
