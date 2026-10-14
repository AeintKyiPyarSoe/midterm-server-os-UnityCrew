# Incident Log — Server OS Midterm Project

**Team:** UnityCrew  
**Members:**
Aeint Kyi Pyar Soe – Nicole (6705140003),
Khun Maung Aye – CK (6705140019),
Htet Soe Lin – Tyzon (6705140023),
Thida Khaing – May (6705140024),
Kaung Khant Htoo – Melly (6705140027),
Lapatrada Tusitanond - Mingie (6705140060)

**Company / domain:** novapeak.lab  
**DNS server:** 192.168.19.10  
**Subnet:** 192.168.19.0/24

## Initial Checker Result

**Command:**

```bash
python3 /root/grading/check_lab.py --group lab01 --phase build
```

**Actual output (selected checks):**

```text
[   1/2 ] C4   named-checkconf -z FAILED; named-checkzone clean
[   0/2 ] C1   client cannot reach named at 192.168.19.10 (timeout/no status). Check listen-on, allow-query, firewall, and that it serves on lan0.
[ 0.0/3 ] C2   ns1 expected 192.168.19.10 got nothing; www expected 192.168.19.20 got nothing; mail expected 192.168.19.30 got nothing; api expected 192.168.19.40 got nothing
[   0/3 ] C9   DHCP advertises 8.8.8.8, expected the company DNS 192.168.19.10 -- this is the integration the whole project is about
[   0/3 ] C11  using the leased resolver 8.8.8.8, www did not resolve to 192.168.19.20 (got nothing). If the leased resolver is a public one, that is the integration bug.
--------------------------------------------------
TOTAL: 11.0 / 25
```

---

## Fault 1 — Incorrect DNS Zone File Path

- **Symptom (what we observed):** The checker reported `named-checkconf -z FAILED`. The forward DNS zone was not loading.

- **Hypothesis (what we suspected and why):** We suspected that `/etc/bind/named.conf.local` pointed to a file that did not exist.

- **Proof (the command that confirmed the cause):**

  ```bash
  cat /etc/bind/named.conf.local
  ls -l /etc/bind/db.MISSING
  ls -l /etc/bind/db.novapeak.lab
  named-checkconf -z
  ```

  **Actual output (relevant lines):**

  ```text
  zone "novapeak.lab" {
      type master;
      file "/etc/bind/db.MISSING";
  };
  ls: cannot access '/etc/bind/db.MISSING': No such file or directory
  -rw-r--r-- 1 bind bind 212 Oct 14 04:31 /etc/bind/db.novapeak.lab
  zone novapeak.lab/IN: loading from master file /etc/bind/db.MISSING failed: file not found
  zone novapeak.lab/IN: not loaded due to errors.
  _default/novapeak.lab/IN: file not found
  ```

- **Fix (what we changed):** We replaced `db.MISSING` with the real forward zone file.

  ```bash
  sed -i 's|/etc/bind/db.MISSING|/etc/bind/db.novapeak.lab|' /etc/bind/named.conf.local
  ```

- **Confirmation (output showing it now works):**

  ```bash
  named-checkconf -z
  named-checkzone novapeak.lab /etc/bind/db.novapeak.lab
  systemctl restart named
  systemctl is-active named
  ```

  **Actual output (relevant lines):**

  ```text
  zone novapeak.lab/IN: loaded serial 2024010101
  zone 19.168.192.in-addr.arpa/IN: loaded serial 2024010101
  zone novapeak.lab/IN: loaded serial 2024010101
  OK
  active
  ```

- **Found/proved by:** Aeint Kyi Pyar Soe

---

## Fault 2 — Incorrect api A Record

- **Symptom (what we observed):** After fixing the zone path and DNS connectivity, api.novapeak.lab still resolved to 192.168.19.1 instead of `192.168.19.40`.

- **Hypothesis (what we suspected and why):** The api A record was set to the wrong IP address in the forward zone file.

- **Proof (the command that confirmed the cause):**

bash
cat /etc/bind/db.novapeak.lab
dig @192.168.19.10 api.novapeak.lab A +short

**Actual output (relevant lines):**

text
api IN A 192.168.19.1
192.168.19.1

The intermediate checker showed:

text
[2.25/3 ] C2 api expected 192.168.19.40 got ['192.168.19.1']
TOTAL: 18.25 / 25

- **Fix (what we changed):** We corrected the A record and increased the zone serial.

bash
sed -i 's/api IN A 192.168.19.1$/api IN A 192.168.19.40/' /etc/bind/db.novapeak.lab
sed -i 's/2024010101/2026101401/' /etc/bind/db.novapeak.lab
named-checkzone novapeak.lab /etc/bind/db.novapeak.lab
systemctl restart named

- **Confirmation (output showing it now works):**

bash
dig @192.168.19.10 ns1.novapeak.lab A +short
dig @192.168.19.10 www.novapeak.lab A +short
dig @192.168.19.10 mail.novapeak.lab A +short
dig @192.168.19.10 api.novapeak.lab A +short

**Actual output (in order):**

text
192.168.19.10
192.168.19.20
192.168.19.30
192.168.19.40

**Zone validation output:**

text
zone novapeak.lab/IN: loaded serial 2026101401
OK

- **Found/proved by:** Khun Maung Aye

---

## Fault 3 — DHCP Advertised the Wrong DNS Server

- **Symptom (what we observed):** The client received a lease, but the checker reported that DHCP was advertising 8.8.8.8 instead of `192.168.19.10`.

- **Hypothesis (what we suspected and why):** The domain-name-servers option in the DHCP configuration was wrong. The client would not use the company's DNS server.

- **Proof (the command that confirmed the cause):**

  
bash
  cat /etc/dhcp/dhcpd.conf
  grep -n "domain-name-servers" /etc/dhcp/dhcpd.conf
  


  **Actual output (relevant lines):**

  
text
  subnet 192.168.19.0 netmask 255.255.255.0 {
      range 192.168.19.104 192.168.19.144;
      option routers 192.168.19.1;
      option domain-name-servers 8.8.8.8;
  }
  8:  option domain-name-servers 8.8.8.8;
  


  **Initial checker output:**

  
text
  [   0/3 ] C9   DHCP advertises 8.8.8.8, expected the company DNS 192.168.19.10 -- this is the integration the whole project is about
  


- **Fix (what we changed):** We changed the DHCP option to advertise the company's DNS server and restarted DHCP.

  
bash
  sed -i 's/option domain-name-servers 8.8.8.8;/option domain-name-servers 192.168.19.10;/' /etc/dhcp/dhcpd.conf
  dhcpd -t -cf /etc/dhcp/dhcpd.conf
  systemctl restart isc-dhcp-server
  systemctl is-active isc-dhcp-server
  


- **Confirmation (output showing it now works):** The DHCP syntax test completed without an error, and the service was active.

  **Actual output (relevant lines):**

  
text
  Internet Systems Consortium DHCP Server 4.4.3-P1
  Config file: /etc/dhcp/dhcpd.conf
  Database file: /var/lib/dhcp/dhcpd.leases
  PID file: /var/run/dhcpd.pid
  active
  


  The final checker confirmed that the actual client received the correct DNS server:

  
text
  [   3/3 ] C6   client obtained a lease: 192.168.19.104
  [   3/3 ] C9   DHCP advertises the company DNS (192.168.19.10)
  [   3/3 ] C11  client resolves www via the leased resolver
  


- **Found/proved by:** Htet Soe Lin

---
## Fault 4 — Firewall Blocked DNS Requests

- **Symptom (what we observed):** BIND was active, but DNS queries to the DNS server's network address timed out.

  **Actual test and output:**

  
bash
  dig @192.168.19.10 ns1.novapeak.lab A +short
  


  
text
  ;; communications error to 192.168.19.10#53: timed out
  ;; communications error to 192.168.19.10#53: timed out
  ;; communications error to 192.168.19.10#53: timed out
  ;; no servers could be reached
  


- **Hypothesis (what we suspected and why):** An incoming firewall rule was blocking UDP DNS requests on port 53.

- **Proof (the command that confirmed the cause):**

  
bash
  iptables -L INPUT -n -v --line-numbers
  


  **Actual output:**

  
text
  Chain INPUT (policy ACCEPT 646 packets, 50575 bytes)
  num   pkts bytes target     prot opt in     out     source               destination
  1        3   255 DROP       17   --  *      *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
  


- **Fix (what we changed):** We removed only the UDP port 53 DROP rule.

  
bash
  iptables -D INPUT -p udp --dport 53 -j DROP
  


- **Confirmation (output showing it now works):**

  
bash
  iptables -S INPUT
  dig @192.168.19.10 ns1.novapeak.lab A +short +time=2 +tries=1
  ip netns exec client dig @192.168.19.10 www.novapeak.lab A +short
  


  **Actual output (in order):**

  
text
  -P INPUT ACCEPT
  192.168.19.10
  192.168.19.20
  


  **Note:** Later in the session we tried to delete the same rule again and got `iptables: Bad rule (does a matching rule exist in that chain?)`. That was because the DROP rule had already been removed. The firewall listing confirmed it was gone.

- **Found/proved by:** Thida Khaing

---

## Final Validation

**Checker command:**

bash
python3 /root/grading/check_lab.py --group lab01 --phase build

**Actual final checker output:**

```text
  [   2/2 ] C4   named-checkconf -z clean; named-checkzone clean
  [   2/2 ] C5   dhcpd -t clean; service running
  [   2/2 ] C1   named is authoritative for novapeak.lab and reachable from the client.
  [ 3.0/3 ] C2   all four A records correct
  [   2/2 ] C3   reverse lookup of 192.168.19.10 returns ns1.
  [   3/3 ] C6   client obtained a lease: 192.168.19.104
  [ 2.0/2 ] C7   pool+netmask correct
  [   2/2 ] C8   gateway advertised correctly
  [   3/3 ] C9   DHCP advertises the company DNS (192.168.19.10)
  [   1/1 ] C10  company domain advertised
  [   3/3 ] C11  client resolves www via the leased resolver
  --------------------------------------------------
  TOTAL: 25.0 / 25
```

**Additional client test:**

bash
ip netns exec client dig @192.168.19.10 www.novapeak.lab A +short
ip netns exec client dig @192.168.19.10 api.novapeak.lab A +short

**Actual output:**

text
192.168.19.20
192.168.19.40

The client also had the DHCP address 192.168.19.104/24 on `cli0`. All checks in the build phase passed. This is not the same as the overall 100-mark course grade.

## Evidence Files

- checker_result.txt — final build-phase checker output.
- session.log — recorded commands, initial errors, fixes, and final results.
- fixed-configs/ — the final DNS and DHCP configuration files.

**Before submission:** Replace each Found/proved by placeholder with the actual group member who performed that work. Add the six members' real names at the top. Do not attribute commands to members who did not perform them.

**Verified By:** Kaung Khant Htoo
