# AI Usage and Critique

**Group:** UnityCrew  
**Members:** Aeint Kyi Pyar Soe (6705140003), Khun Maung Aye (6705140019), Htet Soe Lin (6705140023), Thida Khaing (6705140024), Kaung Khant Htoo (6705140027), Lapatrada Tusitanond - Mingie (6705140060)
**Company / domain:** novapeak.lab

## Part 3a: Declaration (required)

| AI tool | What we used it for                                                                                                                                                            |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ChatGPT | Helped us understand the assignment, explained the DNS timeout, suggested commands for checking BIND and firewall rules, and helped organize our incident log and AI critique. |
| Gemini  | Planned second opinion on the zone file, API A record, firewall, and final test results. Replace this line with the actual uses and dates once team members have used it.      |

**Team responsibilities and example prompts for the AI comparison:**

| Member                                | Responsibility                      | Tools assigned for comparison | Natural prompt to use (not a recorded past prompt)                                                                                            |
| ------------------------------------- | ----------------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Aeint Kyi Pyar Soe (@AeintKyiPyarSoe) | Setup, tests, missing DNS zone file | ChatGPT + Gemini              | “My `named-checkconf -z` says `/etc/bind/db.MISSING` cannot be found, but `db.novapeak.lab` exists. What should I check and how do I fix it?” |
| Khun Maung Aye (@KhunMaungAye)        | API A record and DNS settings       | Gemini                        | “`api.novapeak.lab` resolves to `192.168.19.1`, but it should be `192.168.19.40`. Which DNS file do I edit and how can I test the change?”    |
| Htet Soe Lin (@Tyzon22) | DHCP DNS and network settings | ChatGPT | “The client gets an IP address, but the checker says DHCP gives it DNS `8.8.8.8`. How do I make it use our DNS server `192.168.19.10`?” |
| Kaung Khant Htoo (@kxkhantx) | Final tests, documentation, submission | ChatGPT + Gemini | “Our checker is now 25/25. What does that prove about DNS and DHCP, and what should we still include in our GitHub submission?” |

