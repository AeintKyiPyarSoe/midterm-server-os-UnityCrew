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

| Member                          | GitHub username  | Documentation responsibility                         |
| ------------------------------- | ---------------- | ---------------------------------------------------- |
| Aeint Kyi Pyar Soe - 6705140003 | @AeintKyiPyarSoe | Setup overview, test results & DNS zone file problem |
| Khun Maung Aye - 6705140019     | @KhunMaungAye    | API A record problem and DNS settings                |
| Htet Soe Lin - 6705140023 | @Tyzon22 | DHCP DNS problem and DHCP settings |
| Thida Khaing - 6705140024 | @Thida-Khaing-may | Firewall problem and troubleshooting |
| Kaung Khant Htoo - 6705140027 | @kxkhantx | Final testing, AI critique and submission review |

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