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
