
# Hashcat Complete Reference 
## 1. Introduction to Hashcat
Hashcat is a popular password-recovery tool used to test how strong passwords are by trying different password combinations. It does not directly reverse a hash to reveal the original password. Instead, it creates possible passwords, converts each one into a hash using the same algorithm as the target, and compares the result with the original hash. If the hashes match, the password candidate is considered a match.

The success and speed of Hashcat depend on several factors:

**Hash type:** Different hashing algorithms take different amounts of time to process. For example, MD5 is generally fast, while bcrypt and Argon2 are designed to make password guessing slower.

**Attack mode:** Hashcat supports different methods of generating password guesses, such as trying words from a dictionary, testing every possible character combination, or modifying existing words.

**Hardware:** Hashcat can use CPUs and GPUs to perform password guesses. GPUs can process many suitable hash calculations in parallel, making them particularly effective for many common hash types. However, memory-intensive algorithms such as Argon2 and scrypt require significant memory resources, which can affect performance.

```bash
hashcat [options]... hash|hashfile|hccapxfile [dictionary|mask|directory]...
```

---

## 2. Attack Modes (`-a`)

|#|Mode|What it does|
|---|---|---|
|0|Straight|Tries each wordlist entry as-is (optionally mutated by rules)|
|1|Combination|Concatenates word1+word2 from two wordlists|
|3|Brute-force (mask)|Generates every combination matching a character pattern|
|6|Hybrid Wordlist + Mask|Wordlist word, then mask appended|
|7|Hybrid Mask + Wordlist|Mask prepended, then wordlist word|
|9|Association|Hash:word pairs tied 1:1 (niche, e.g. known-plaintext scenarios)|

### Basic Examples (from `--help`, annotated)

```bash
# Wordlist attack — phpass ($P$, WordPress/phpBB)
hashcat -a 0 -m 400 example400.hash example.dict

# Wordlist + Rules — MD5
hashcat -a 0 -m 0 example0.hash example.dict -r rules/best64.rule

# Brute-Force — MD5, 6 lowercase+upper+digit+symbol chars
hashcat -a 3 -m 0 example0.hash ?a?a?a?a?a?a

# Combinator — MD5, word1+word2
hashcat -a 1 -m 0 example0.hash example.dict example.dict

# Association — md5crypt ($1$), one rule applied per hash:word pair
hashcat -a 9 -m 500 example500.hash 1word.dict -r rules/best64.rule
```

---

## 3. Hash Modes (`-m`) — Reference Table

Full enumeration: `hashcat -hh` (shows every supported mode). Detail on one mode: `hashcat -H -m <mode>` or `--example-hashes` (alias of `-H`).

```bash
hashcat -h              # short help
hashcat -hh             # full list of all hash modes
hashcat -H -m 1000      # detailed info + example hash for mode 1000 (NTLM)
hashcat -I              # backend/device info
hashcat --identify my.hash   # guess possible hash-modes for a hash file
```

More : https://hashcat.net/wiki/doku.php?id=example_hashes

|Mode|Type|
|---|---|
|0|MD5|
|10|md5($pass.$salt)|
|100|SHA1|
|1400|SHA256|
|1700|SHA512|
|1000|NTLM|
|3000|LM|
|500|md5crypt / $1$ (Unix)|
|1800|sha512crypt / $6$ (Unix)|
|7400|sha256crypt / $5$ (Unix)|
|400|phpass / $P$ (WordPress, phpBB)|
|3200|bcrypt / $2*$|
|7500|Kerberos 5 AS-REQ Pre-Auth (etype 23)|
|13100|Kerberos 5 TGS-REP (Kerberoasting)|
|18200|Kerberos 5 AS-REP (AS-REP Roasting)|
|5500|NetNTLMv1|
|5600|NetNTLMv2|
|2500|WPA/WPA2 (legacy, deprecated)|
|22000|WPA-PBKDF2-PMKID+EAPOL (current)|
|10900|PBKDF2-HMAC-SHA256 (Django)|
|1450|HMAC-SHA256|
|11600|7-Zip|
|13400|KeePass 1/2|
|13600|WinZip|

---

## 4. Straight / Dictionary Attacks (Mode 0)

```bash
hashcat -m 1000 -a 0 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt
```

### With Rules (`-r`, `-j`, `-k`)

```bash
# Full rule FILE — many transformations per word
hashcat -m 0 -a 0 hashes.txt rockyou.txt -r rules/best64.rule

# -j: single rule applied to every word from the LEFT wordlist
hashcat -a 1 -m 0 hashes.txt dict1.txt dict2.txt -j 'c'

# -k: single rule applied to every word from the RIGHT wordlist
hashcat -a 1 -m 0 hashes.txt dict1.txt dict2.txt -k '^-'
```

Common built-in rule files (sizes approximate — bigger = slower, more thorough):

- `best64.rule` — ~64 rules, fast, high hit-rate starting point
- `d3ad0ne.rule` — larger, very effective generalist set
- `rockyou-30000.rule` — derived from password reuse data
- `OneRuleToRuleThemAll.rule` — tens of thousands of rules, very slow, very thorough

### Random Rule Generation

```bash
hashcat -a 0 -m 0 hashes.txt rockyou.txt -g 10000 \
  --generate-rules-func-min=1 --generate-rules-func-max=4 \
  --generate-rules-func-sel=ioTlc --generate-rules-seed=1337
```

`-g` generates X random rules on the fly instead of using a static rule file — useful for fuzzing variations you haven't predicted.

### Rule Debugging

```bash
hashcat -a 0 -m 0 hashes.txt rockyou.txt -r rules/best64.rule \
  --debug-mode=4 --debug-file=good.log
```

Debug modes: `1` finding-rule only, `2` original word only, `3` word:rule, `4` word:rule:result, `5` word:rule:result:wordlist. Use this to see _which_ rule cracked which hash — valuable when tuning custom rules against your org's actual password patterns.

---

## 5. Brute-Force / Mask Attacks (Mode 3)

### Built-in charsets

|Placeholder|Set|
|---|---|
|`?l`|a-z|
|`?u`|A-Z|
|`?d`|0-9|
|`?h`|0-9a-f|
|`?H`|0-9A-F|
|`?s`|`!"#$%&'()*+,-./:;<=>?@[\]^_\`{\|}~ `|
|`?a`|?l?u?d?s (all printable)|
|`?b`|0x00–0xff (full byte range)|

```bash
# 8 chars: Upper + 5 lower + 2 digits (e.g. "Pattern" + "25")
hashcat -m 0 -a 3 hashes.txt ?u?l?l?l?l?l?d?d
```

### Custom charsets (`-1` through `-8`)

```bash
# Define ?1 = lower+digit, then use it
hashcat -m 0 -a 3 -1 ?l?d hashes.txt ?1?1?1?1?1?1

# Multiple custom sets, e.g. keyspace = upper-or-lower + digit + symbol
hashcat -m 0 -a 3 -1 ?u?l -2 ?d?s hashes.txt ?1?1?1?1?2?2
```

### Increment mode

```bash
# Try all lengths from 4 to 8 instead of one fixed length
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a?a?a --increment --increment-min=4 --increment-max=8

# Increment right-to-left instead of left-to-right
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a -i --increment-inverse
```

### Markov chains (statistical ordering of candidates)

Hashcat orders brute-force candidates by letter-frequency statistics by default (faster real-world hit rate than pure sequential brute force).

```bash
# Use a custom statistics file
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --markov-hcstat2=my.hcstat2

# Disable markov entirely — classic uniform brute-force
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --markov-disable

# Classic markov (no per-position stats) / inverse markov
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --markov-classic
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --markov-inverse

# Threshold: stop accepting new markov chains after X
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a -t 50
```

### Keyspace / candidate counting (plan before you run)

```bash
hashcat -m 0 -a 3 ?a?a?a?a?a?a?a?a --keyspace          # shows base:mod
hashcat -m 0 -a 3 ?a?a?a?a?a?a?a?a --total-candidates  # base*mod total count
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --speed-only     # estimate speed, then quit
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --progress-only  # estimate ideal chunking/time
```

Always run `--keyspace`/`--speed-only` before committing a GPU to a multi-day mask — it tells you up front whether the job is hours or years.

---

## 6. Combination Attack (Mode 1)

Concatenates one word from list A with one word from list B — good for guessing `word1word2` patterns (e.g. firstname+lastname, word+companyname).

```bash
hashcat -m 0 -a 1 hashes.txt wordlist1.txt wordlist2.txt

# Apply a rule to only the left-side word before concatenation
hashcat -m 0 -a 1 hashes.txt wordlist1.txt wordlist2.txt -j 'c'

# Apply a rule to only the right-side word
hashcat -m 0 -a 1 hashes.txt wordlist1.txt wordlist2.txt -k '$!'
```

---

## 7. Hybrid Attacks (Modes 6 / 7)

Mix a wordlist (human-chosen base word) with a mask (predictable suffix/prefix — years, digits, symbols).

```bash
# Wordlist + Mask: word then 2 digits + 1 symbol (e.g. "Summer25!")
hashcat -m 0 -a 6 hashes.txt wordlist.txt ?d?d?s

# Mask + Wordlist: digits/symbol prefix then word (e.g. "2025!Summer")
hashcat -m 0 -a 7 hashes.txt ?d?d?s wordlist.txt
```

This is usually the highest-value attack after a plain dictionary+rules pass, because real users append the current year, a symbol, or sequential digits to a base word.

---

## 8. Association Attack (Mode 9)

Ties each candidate to a _specific_ hash rather than trying every candidate against every hash — used when you have hash:word pairs from prior intel (e.g. known username → suspected base word).

```bash
hashcat -a 9 -m 500 example500.hash 1word.dict -r rules/best64.rule
```

---

## 9. Session, Restore & Output Management

```bash
# Name a session so it can be resumed later
hashcat -m 0 -a 0 hashes.txt rockyou.txt --session=audit_q4

# Resume an interrupted session
hashcat --session=audit_q4 --restore

# Skip writing a restore file at all (e.g. ephemeral container run)
hashcat -m 0 -a 0 hashes.txt rockyou.txt --restore-disable

# Custom path for the restore file
hashcat -m 0 -a 0 hashes.txt rockyou.txt --restore-file-path=/data/audit.restore

# Cap total runtime to 10 minutes then stop automatically
hashcat -m 0 -a 0 hashes.txt rockyou.txt --runtime=600

# Skip the first 1,000,000 candidates / limit to the next 1,000,000
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a -s 1000000 -l 1000000
```

### Output

```bash
-o cracked.txt                     # write cracked plaintexts here
--outfile-format=1,3               # hash[:salt] + hex_plain columns
--outfile-json                     # JSON formatted outfile
--outfile-autohex-disable          # don't wrap unprintable plains in $HEX[]
--outfile-check-timer=30           # how often to poll for 3rd-party outfiles
```

Outfile format codes: `1` hash[:salt], `2` plain, `3` hex_plain, `4` crack_pos, `5` timestamp absolute, `6` timestamp relative.

### Potfile (hashcat's running database of cracked hashes)

```bash
hashcat -m 0 hashes.txt --show              # list already-cracked hashes from potfile
hashcat -m 0 hashes.txt --left              # list hashes NOT yet cracked
hashcat -m 0 hashes.txt rockyou.txt --potfile-disable       # don't record cracks
hashcat -m 0 hashes.txt rockyou.txt --potfile-path=audit.pot
```

### Hash list parsing options

```bash
--username            # input is username:hash — ignore the username column
-p :                  # custom separator character for hash:salt / user:hash
--remove              # strip cracked hashes from the input file as they crack
--remove-timer=30     # how often (seconds) to rewrite the input file
--dynamic-x           # ignore $dynamic_X$ prefix wrapper on hashes
--hex-charset / --hex-salt / --hex-wordlist   # treat values as hex-encoded
```

---

## 10. Performance Tuning

```bash
-O                      # optimized kernels — faster, but caps max password length (usually 32)
-w 3                    # workload profile: 1=Low 2=Default 3=High 4=Nightmare
-d 1,2                  # restrict to specific backend device IDs
-T 64 -n 64 -u 256      # manual thread/kernel-accel/kernel-loops tuning (advanced)
--backend-vector-width=4
-M                      # disable multiply kernel-accel by core count
--spin-damp=10          # use CPU for device sync X% of the time (reduces GPU busy-wait power draw)
```

|Workload (`-w`)|Perf|Runtime/iter|Power|Desktop impact|
|---|---|---|---|---|
|1|Low|2ms|Low|Minimal|
|2|Default|12ms|Economic|Noticeable|
|3|High|96ms|High|Unresponsive|
|4|Nightmare|480ms|Insane|Headless only|

### Benchmarking

```bash
hashcat -b                       # benchmark all common modes
hashcat -b -m 1000               # benchmark just NTLM
hashcat -b --benchmark-all        # exhaustive benchmark of every supported mode
hashcat -b --benchmark-min=100 --benchmark-max=1800   # benchmark a mode range
```

### Thermal / hardware safety

```bash
--hwmon-disable             # disable temp/fan monitoring reads (not recommended)
--hwmon-temp-abort=90       # abort the job if GPU hits 90°C
--backend-devices-keepfree=5   # leave 5% device memory free for OS/display
```

### Distributed cracking (Brain server/client — split work across machines)

```bash
# On the coordinating machine
hashcat --brain-server --brain-password=correcthorsebatterystaple

# On each worker node
hashcat -m 0 -a 0 hashes.txt rockyou.txt -z \
  --brain-host=10.0.0.5 --brain-password=correcthorsebatterystaple \
  --brain-client-features=3
```

`--brain-client-features`: `1` send hashed passwords only, `2` send attack positions only, `3` send both (avoids duplicate work across nodes).

---

## 11. Specialty Input Types

```bash
# TrueCrypt volume with keyfile
hashcat -m 6211 -a 0 tc_header.txt rockyou.txt --truecrypt-keyfiles=key.png

# VeraCrypt volume, PIM range scan
hashcat -m 13711 -a 0 vc_header.txt rockyou.txt \
  --veracrypt-keyfiles=key.txt --veracrypt-pim-start=450 --veracrypt-pim-stop=500

# WPA/WPA2 handshake capture (hccapx message-pair filtering)
hashcat -m 22000 -a 0 capture.hc22000 rockyou.txt --hccapx-message-pair=2

# Print candidate words only — no cracking, useful to preview/pipe a mask or rule set
hashcat -a 3 ?u?l?l?l?l?d?d --stdout | head -20
hashcat -a 0 rockyou.txt -r rules/best64.rule --stdout | wc -l
```

---

## 12. Common SOC Password-Audit Workflow

1. **Extract** hashes (e.g., `secretsdump.py` against NTDS.dit, `/etc/shadow`, Kerberos ticket dumps via Rubeus/impacket).
2. **Identify** hash type: `hashcat --identify dump.hash`, or `hashid`/`hash-identifier` as a cross-check.
3. **Fast pass** — dictionary + best64 rules against rockyou (and any org-specific wordlist built from breach data, company name variants, product names). Minutes to an hour; catches the worst offenders.
    
    ```bash
    hashcat -m 1000 -a 0 ntlm.txt rockyou.txt -r rules/best64.rule -O -w 3
    ```
    
4. **Hybrid pass** — base word + year/symbol suffixes, based on known org password policy.
    
    ```bash
    hashcat -m 1000 -a 6 ntlm.txt --left wordlist_company.txt ?d?d?d?d?s
    ```
    
5. **Targeted mask pass** — patterns matching the org's documented complexity rule (e.g. `Capital + 6 lower + 2 digit`).
6. **Long-tail brute-force / heavy rule set** — run `OneRuleToRuleThemAll.rule` or incremented masks overnight on remaining hashes.
7. **Benchmark + keyspace check** before any brute-force job you plan to leave running — know the ETA before committing hardware time.
8. **Report** — aggregate by _category_ of weakness, never by plaintext in a widely shared doc:
    - cracked in dictionary pass (reused / breach-known password)
    - cracked in hybrid pass (predictable suffix pattern)
    - cracked via brute-force (short password, still weak even if "random-looking")
    - not cracked within audit window (does not mean "strong" — note elapsed compute time and coverage)
9. **Remediate**: MFA rollout priority list, banned-password list seeded from this audit's hits, legacy hash algorithm migration plan, Kerberoasting/AS-REP exposure list for service accounts.

---

## 13. Target-Specific Quick Reference

### Active Directory

```bash
# Kerberoasting (TGS-REP) — service account passwords, offline-crackable once ticket obtained
hashcat -m 13100 -a 0 kerberoast.txt rockyou.txt -r rules/best64.rule

# AS-REP Roasting — accounts with "Do not require Kerberos preauth" set
hashcat -m 18200 -a 0 asrep.txt rockyou.txt -r rules/best64.rule

# NTLM (local SAM / NTDS dump)
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt

# NetNTLMv2 (captured via Responder/relay in authorized internal testing)
hashcat -m 5600 -a 0 netntlmv2.txt rockyou.txt
```

### Linux

```bash
hashcat -m 1800 -a 0 shadow_sha512.txt rockyou.txt    # sha512crypt ($6$)
hashcat -m 500  -a 0 shadow_md5.txt rockyou.txt        # md5crypt ($1$)
hashcat -m 7400 -a 0 shadow_sha256.txt rockyou.txt     # sha256crypt ($5$)
```

### Wireless

```bash
hashcat -m 22000 -a 0 capture.hc22000 rockyou.txt
```

### Web application databases

```bash
hashcat -m 400  -a 0 wp_hashes.txt rockyou.txt    # WordPress/phpBB phpass
hashcat -m 3200 -a 0 bcrypt_hashes.txt rockyou.txt -w 3   # bcrypt — slow, GPU gain is modest
```

### Archives / containers

```bash
hashcat -m 13600 -a 0 zip_hash.txt rockyou.txt   # WinZip
hashcat -m 11600 -a 0 7z_hash.txt rockyou.txt    # 7-Zip
hashcat -m 13400 -a 0 kdbx.txt rockyou.txt       # KeePass DB
```

---

## 14. Rule-Writing Quick Syntax

|Function|Effect|
|---|---|
|`:`|No-op|
|`l`|Lowercase entire word|
|`u`|Uppercase entire word|
|`c`|Capitalize first letter, lowercase rest|
|`C`|Lowercase first letter, uppercase rest|
|`t`|Toggle case of all characters|
|`r`|Reverse the word|
|`d`|Duplicate the word (word→wordword)|
|`$X`|Append character X|
|`^X`|Prepend character X|
|`sXY`|Substitute all X with Y|
|`@X`|Purge/remove all instances of X|
|`[` / `]`|Delete first / last character|
|`{` / `}`|Rotate word left / right|

Example rule line:

```
$1 $2 $3 c
```

→ appends "123" then capitalizes the first letter: `password` → `Password123`

Example small custom rule file for an org audit (`custom.rule`):

```
c $2025 $!
c $2026 $!
u $123
$1 $2 $3 $4
```

---

## 15. Defensive Takeaways (for SOC reporting, not the attacker's checklist)

- Anything cracked by `best64` + rockyou in minutes = **critical finding**: password reuse or breach-known password in active use.
- Unsalted fast hashes (plain MD5/SHA1/NTLM) in any system of record should be flagged for migration to a salted, slow/memory-hard KDF (bcrypt, scrypt, Argon2id, or PBKDF2 with a high iteration count).
- Favor **long passphrases (12–14+ chars) + MFA** over complexity rules — complexity rules produce _predictable_ patterns (`Word` + digit + symbol) that hybrid-mode attacks target directly.
- **Kerberoastable / AS-REP-roastable** service accounts are a standing offline-crackable risk the moment a ticket is obtained — require long random (25+ char) passwords or managed service accounts (gMSA) instead.
- Treat "not cracked within the audit window" as _unknown_, not _secure_ — report elapsed compute time and ruleset/mask coverage alongside any "uncracked" count so the result isn't over-trusted.

---

_For internal, authorized testing and SOC training use only. Always operate within the scope of an approved engagement or isolated test environment, with results handled per your org's data-handling policy._
