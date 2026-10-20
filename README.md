# Server OS Midterm — Group 1 (Tuesday Afternoon Class)

## Group information

- **Group name:** Unity Crew

## Team Members

| **Name** | **Student ID** | **GitHub Username** |
|---|---|---|
| Aeint Kyi Pyar Soe | 6705140003 | [AeintKyiPyarSoe](https://github.com/AeintKyiPyarSoe) |
| Khun Maung Aye | 6705140019 | [KhunMaungAye](https://github.com/KhunMaungAye) |
| Htet Soe Lin | 6705140023 | [Tyzon22](https://github.com/Tyzon22) |
| Thida Khaing | 6705140024 | [Thida-Khaing-may](https://github.com/Thida-Khaing-may) |
| Kaung Khant Htoo | 6705140027 | [kxkhantx](https://github.com/kxkhantx) |
| Lapatrada Tusitanond | 6705140060 | [Ming-200629](https://github.com/Ming-200629) |
  
- **Company:** NovaPeak
- **Domain:** `novapeak.lab`
- **Group ID used with the checker:** `lab01`

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

| Member | GitHub Username | Documentation Responsibility |
|---|---|---|
| Aeint Kyi Pyar Soe - 6705140003 | [@AeintKyiPyarSoe](https://github.com/AeintKyiPyarSoe) | Setup overview, test results & DNS zone file problem |
| Khun Maung Aye - 6705140019 | [@KhunMaungAye](https://github.com/KhunMaungAye) | API A record problem and DNS settings |
| Htet Soe Lin - 6705140023 | [@Tyzon22](https://github.com/Tyzon22) | DHCP DNS problem and DHCP settings |
| Thida Khaing - 6705140024 | [@Thida-Khaing-may](https://github.com/Thida-Khaing-may) | Firewall problem and troubleshooting |
| Kaung Khant Htoo - 6705140027 | [@kxkhantx](https://github.com/kxkhantx) | Final testing, AI critique and submission review |
| Lapatrada Tusitanond - 6705140060 | [@Ming-200629](https://github.com/Ming-200629) | Reviewing documentation |  
      
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

During troubleshooting, our team identified four configuration or service issues:

1. The forward DNS zone referenced a missing file, /etc/bind/db.MISSING.

2. The api DNS A record pointed to the wrong IP address.

3. DHCP advertised the public DNS resolver 8.8.8.8 instead of the internal DNS server.

4. A firewall rule blocked incoming UDP traffic on port 53, preventing DNS queries from reaching the server.

Our team corrected the zone-file reference, updated the api record to 192.168.19.40, configured DHCP to advertise 192.168.19.10, and removed the blocking firewall rule.

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

After fixing the problems, we checked the DNS and DHCP configurations to make sure everything was working correctly. We also ran the official checker to confirm that the client could receive an IP address and resolve the company's hostnames.

We used the following commands:

bash
named-checkconf -z
dhcpd -t -cf /etc/dhcp/dhcpd.conf
iptables -L INPUT -v -n --line-numbers
python3 /root/grading/check_lab.py --group lab01 --phase build

The final checker showed that all checks from C1 to C11 passed, giving us a score of **25.0/25.0**.

We saved the following files as evidence of our work:

- **checker_result.txt:** Shows our final checker results.
- **session.log:** Records the commands we ran and the troubleshooting process.
- **fixed-configs/:** Contains the corrected DNS and DHCP configuration files.
- **INCIDENT_LOG.md:** Explains the problems we found, how we fixed them, and how we confirmed the results.
- **AI_CRITIQUE.md:** Explains how we used AI tools and evaluated their suggestions.

## 7. AI Assistance and Verification

We used AI tools to help us understand some DNS and DHCP problems, find useful troubleshooting commands, and organize our documentation.

However, we did not rely only on AI suggestions. We checked the configuration files, tested the commands, and used the official checker to confirm that our fixes worked.

We also compared the AI responses to see which suggestions were useful, correct, or needed further checking. The details of our AI usage and comparison are included in **AI_CRITIQUE.md**.

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

Our team repaired the DNS and DHCP configuration for the novapeak.lab environment. The final build-phase checker passed all eleven checks and awarded 25.0/25.0. The repository preserves the final configurations, checker evidence, terminal history, and supporting documentation for review.
