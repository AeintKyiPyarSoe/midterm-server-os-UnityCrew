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

### 2. Khun Maung Aye: Wrong API address (Gemini)

**Prompt:**

> My api.novapeak.lab is giving 192.168.19.1 when I run `dig`, but the task says it should be `192.168.19.40`. Which file should I edit and what should I do after changing it?

**Gemini response :**

> Check the forward zone file `/etc/bind/db.novapeak.lab`. Change the api A record from 192.168.19.1 to `192.168.19.40`. Increase the zone serial number, validate with `named-checkzone`, restart BIND, and query the hostname again with `dig`.

**Our evaluation:** This advice matches the issue. The actual dig @192.168.19.10 api.novapeak.lab A +short output before repair was `192.168.19.1`. After the change and service restart, the query returned `192.168.19.40`. The comparison response still needs confirmation from Khun's Gemini history.

### 3. Htet Soe Lin: Wrong DNS server given by DHCP (ChatGPT)

**Prompt:**

> The client got an IP from DHCP, but the checker says it is using 8.8.8.8 for DNS. We have our own DNS server at `192.168.19.10`. How can I fix DHCP without changing the other network settings?

**ChatGPT response :**

> Open /etc/dhcp/dhcpd.conf and change option domain-name-servers 8.8.8.8; to `option domain-name-servers 192.168.19.10;`. Keep the existing pool and gateway. Test with `dhcpd -t -cf /etc/dhcp/dhcpd.conf`, restart `isc-dhcp-server`, and run the checker again to see what DNS server the client receives.

**Our evaluation:** The suggested setting is correct. The initial checker C9 reported `DHCP advertises 8.8.8.8`; after editing the configuration, the service remained active and C9 reported `DHCP advertises the company DNS (192.168.19.10)`. This is a sample response until Htet confirms his actual ChatGPT conversation.

### 4. Thida Khaing: DNS query timeout and firewall (ChatGPT and Gemini)

**Prompt for both tools:**

> BIND says `active (running)`, but dig @192.168.19.10 ns1.novapeak.lab A +short keeps timing out. What should I check next? I don't want to delete all firewall rules.

**ChatGPT response (documented in our conversation, summarized):**

> Check iptables -L INPUT -n -v --line-numbers for a DROP rule on UDP port 53. If the matching rule exists, remove that specific rule using `iptables -D INPUT -p udp --dport 53 -j DROP`. Then test dig again from the server and the client namespace. Also check BIND and its configuration if the timeout continues.

**Gemini response :**

> Verify whether named listens on the server address with ss -lntup and whether the firewall drops port 53. If a rule blocks DNS queries, remove only the matching rule, then check the response from the client using `dig`.

**Our evaluation:** The actual terminal output showed DROP ... udp dpt:53 with 3 packets counted. After deleting the rule, iptables -S INPUT showed only `-P INPUT ACCEPT`, and dig returned `192.168.19.10`. The ChatGPT suggestion was appropriate and was verified in the lab. The Gemini text remains an example, not a recorded response.

### 5. Kaung Khant Htoo: Final checker and submission (ChatGPT and Gemini)

**Prompt for both tools:**

> Our Server OS checker finally shows `25.0 / 25`. Is that enough for the midterm? What files should we include in the group submission, and what does the checker score actually prove?

**ChatGPT response (documented in our conversation, summarized):**

> A 25/25 build-phase checker result confirms the tested DNS and DHCP functions, but the full midterm also requires the fixed configuration files, `INCIDENT_LOG.md`, `AI_CRITIQUE.md`, `session.log`, and a README in the group's GitHub repository. It does not guarantee 100/100 for the assignment.

**Gemini response :**

> Keep the successful checker result and the final configurations. Include the troubleshooting log and an explanation of any AI used, following the teacher's required folder structure. The checker proves the tested network behavior, not the quality of the written report.

**Our evaluation:** The final checker showed every check C1–C11 passing with `TOTAL: 25.0 / 25`. The instructor's handout also requires the incident log, AI critique, session recording, and GitHub submission. This confirms the substance of the advice, but the Gemini response should be replaced with Kaung's actual response if he used it.

### Comparison table

| Fault (in our words) | We found it? | ChatGPT | Gemini | Was each AI actually right? |
| --- | --- | --- | --- | --- |
| Wrong forward-zone file path | Yes | correct named.conf.local path and validate | change the missing zone path | Technically correct suggestions; actual replies not yet confirmed |
| Wrong API A record | Yes | Not assigned | edit api record and reload zone | Technically correct suggestion; Gemini interaction unverified |
| DHCP offered 8.8.8.8 | Yes | advertise 192.168.19.10 and retest | Not assigned | Technically correct suggestion; individual AI interaction unverified |
| UDP 53 firewall DROP | Yes | Documented: inspect and remove only the matching rule | inspect listening ports and firewall | ChatGPT advice verified by the before/after DNS tests; Gemini unverified |
| Final result and submission | Yes | Documented: 25/25 isn't the entire submission | include configs and documentation | ChatGPT advice agrees with rubric; Gemini unverified |

**Potentially misleading advice:** Saying “BIND is running, so DNS must work” would have been wrong here. BIND was active while DNS requests timed out because UDP port 53 was blocked. We checked the actual firewall rule and confirmed that DNS queries succeeded after its removal. Similarly, “DNS resolves now, so everything is fixed” would have missed the incorrect API IP and the DHCP-advertised resolver. The intermediate checker score was **18.25/25**, not full marks.

## Part 3d: Verdict

- **Where our human diagnosis beat the AIs:** We could inspect the real configuration and test the actual client. For example, we saw BIND was running but DNS still timed out, and the firewall check showed why. We also used the intermediate checker to see that some issues remained after DNS became reachable.
- **Where an AI genuinely helped:** ChatGPT explained the DNS timeout, suggested specific inspection commands, and helped organize the troubleshooting evidence and documents. Its firewall guidance was consistent with what we found in `iptables`.
- **What we learned about trusting AI to diagnose a live system it cannot see:** AI suggestions are a starting point, not evidence. We still need to check the files, run commands, and verify client behavior. The final checker result of **25/25** shows that the tested lab requirements passed, while the incident log shows how we got there.

**Before submitting:** Replace the “To verify” cells with each member's real AI replies and confirm Gemini use. Keep the tool declaration accurate and have members review the work attributed to them.
