# AI Usage and Critique

**Group:** UnityCrew  
**Members:** Aeint Kyi Pyar Soe (6705140003), Khun Maung Aye (6705140019), Htet Soe Lin (6705140023), Thida Khaing (6705140024), Kaung Khant Htoo (6705140027), Lapatrada Tusitanond - Mingie (6705140060)
**Company / domain:** novapeak.lab  

## Part 3a: Declaration (required)

| AI tool | What we used it for |
| --- | --- |
| ChatGPT  | Helped us understand the assignment, explained the DNS timeout, suggested commands for checking BIND and firewall rules, and helped organize our incident log and AI critique. |
| Gemini  | Planned second opinion on the zone file, API A record, firewall, and final test results. Replace this line with the actual uses and dates once team members have used it. |

**Team responsibilities and example prompts for the AI comparison:**

| Member | Responsibility | Tools assigned for comparison | Natural prompt to use (not a recorded past prompt) |
| --- | --- | --- | --- |
| Aeint Kyi Pyar Soe (@AeintKyiPyarSoe) | Setup, tests, missing DNS zone file | ChatGPT + Gemini | “My `named-checkconf -z` says `/etc/bind/db.MISSING` cannot be found, but `db.novapeak.lab` exists. What should I check and how do I fix it?” |