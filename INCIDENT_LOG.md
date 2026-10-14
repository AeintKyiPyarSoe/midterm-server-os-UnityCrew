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