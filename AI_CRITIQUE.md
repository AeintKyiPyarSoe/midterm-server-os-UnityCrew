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

| Member                                | Responsibility                         | Tools assigned for comparison | Natural prompt to use (not a recorded past prompt)                                                                                                   |
| ------------------------------------- | -------------------------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Aeint Kyi Pyar Soe (@AeintKyiPyarSoe) | Setup, tests, missing DNS zone file    | ChatGPT + Gemini              | “My `named-checkconf -z` says `/etc/bind/db.MISSING` cannot be found, but `db.novapeak.lab` exists. What should I check and how do I fix it?”        |
| Khun Maung Aye (@KhunMaungAye)        | API A record and DNS settings          | Gemini                        | “`api.novapeak.lab` resolves to `192.168.19.1`, but it should be `192.168.19.40`. Which DNS file do I edit and how can I test the change?”           |
| Htet Soe Lin (@Tyzon22)               | DHCP DNS and network settings          | ChatGPT                       | “The client gets an IP address, but the checker says DHCP gives it DNS `8.8.8.8`. How do I make it use our DNS server `192.168.19.10`?”              |
| Thida Khaing (@Thida-Khaing-may)      | Firewall and troubleshooting           | ChatGPT + Gemini              | “BIND is running, but dig @192.168.19.10 times out. I found a DROP rule for UDP port 53. How do I prove it's the problem and remove only that rule?” |
| Kaung Khant Htoo (@kxkhantx)          | Final tests, documentation, submission | ChatGPT + Gemini              | “Our checker is now 25/25. What does that prove about DNS and DHCP, and what should we still include in our GitHub submission?”                      |

## Part 3b: Confirm you diagnosed first

Our incident log records how we checked the live system and verified the faults. The confirmed faults were:

1. The forward zone in `/etc/bind/named.conf.local` pointed to `/etc/bind/db.MISSING` instead of `/etc/bind/db.novapeak.lab`.
2. The api A record pointed to 192.168.19.1 instead of `192.168.19.40`.
3. DHCP advertised 8.8.8.8 instead of our internal DNS server `192.168.19.10`.
4. An iptables INPUT rule dropped UDP port 53 DNS packets.

The initial checker scored **11/25**. After repairing the zone path and removing the firewall rule, it scored **18.25/25**, showing that the incorrect A record and DHCP DNS setting still needed fixing. The final score was **25/25**. 

## Part 3c: Putting the AIs to the test

### 1. Aeint Kyi Pyar Soe: Missing DNS zone file (ChatGPT and Gemini)

**Prompt for both tools:**

> I ran `named-checkconf -z` and got `loading from master file /etc/bind/db.MISSING failed: file not found`. But I can see `db.novapeak.lab` in `/etc/bind/`. What am I supposed to change? How can I check that I fixed it?

**ChatGPT response:**

> The zone definition is pointing to the wrong file. Open `/etc/bind/named.conf.local` and change the forward zone's `file` path to `/etc/bind/db.novapeak.lab`. Run `named-checkconf -z` and `named-checkzone novapeak.lab /etc/bind/db.novapeak.lab`, then restart `named`.

**Gemini response:**

> Check `named.conf.local` for the `novapeak.lab` zone. The declared file must match the actual zone file name. Update `db.MISSING` to `db.novapeak.lab`, validate the configuration, and reload BIND.

**Our evaluation:** Both sample suggestions address the actual cause. The original `named-checkconf -z` showed the missing-file error. After fixing the path, it reported `zone novapeak.lab/IN: loaded serial 2024010101`, and `named-checkzone` returned `OK`. This validated the proposed fix, but it does not verify either sample AI response was actually given.