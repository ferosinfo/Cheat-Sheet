
# 🕵️‍♂️ Google Dorking Cheat Sheet

> Use this cheat sheet responsibly. Only for ethical research, OSINT, bug bounty, and authorized testing.

---

## 🔍 Basic Search Operators

### 🔸 `"Search Term"` – _Exact Phrase Search_

```
"sensitive information"
"confidential document"
"internal use only"
```

---

### 🔸 `site:` – _Site-Specific Search_

```
site:example.com
site:*.gov
site:github.com "password"
site:pastebin.com "api key"
```

---

### 🔸 `filetype:` – _File Type Search_

```
filetype:pdf "confidential"
filetype:xls "employee list"
filetype:doc "budget 2024"
filetype:sql "insert into"
filetype:log "error"
```

---

### 🔸 `intitle:` – _Title Search_

```
intitle:"index of" phpmyadmin
intitle:"admin panel"
intitle:"login page"
intitle:"dashboard"
```

---

### 🔸 `inurl:` – _URL Search_

```
inurl:admin
inurl:login
inurl:config
inurl:backup
inurl:test
```

---

### 🔸 `intext:` – _Body Text Search_

```
intext:"password"
intext:"username"
intext:"database connection"
```

---

## ⚙️ Advanced Search Operators

### 🔹 `cache:` – _Cached Pages_

```
cache:example.com
cache:example.com/admin
```

---

### 🔹 `related:` – _Related Sites_

```
related:example.com
related:github.com
```

---

### 🔹 `info:` – _Page Information_

```
info:example.com
```

---

### 🔹 `define:` – _Definitions_

```
define:phishing
define:SQL injection
```

---

### 🔹 `link:` – _Backlinks_

```
link:example.com
```

---

## 🔗 Combination Operators

- `AND` – Combine terms
    
    ```
    security AND audit  
    "user manual" AND filetype:pdf
    ```
    
- `OR` – Either term
    
    ```
    admin OR administrator  
    login OR signin
    ```
    
- `-` – Exclude terms
    
    ```
    security -course  
    site:github.com -site:gist.github.com
    ```
    
- `+` – Include exact term
    
    ```
    +security +audit
    ```
    
- `*` – Wildcard
    
    ```
    "index of" * "parent directory"  
    "welcome to" * "server"
    ```
    

---

## 🔐 Practical Examples for Security Research

### 🗂️ Finding Configuration Files

```
filetype:conf "password"
filetype:config "database"
site:github.com "database.yml"
```

### 🧩 Finding Database Files

```
filetype:sql "INSERT INTO"
filetype:db
"mysql_connect" filetype:php
```

### 📜 Finding Log Files

```
filetype:log "error"
intitle:"index of" "access.log"
site:*.edu filetype:log
```

### 🔐 Finding Admin Panels

```
intitle:"admin panel"
inurl:admin intitle:login
site:example.com inurl:wp-admin
```

### 📁 Finding Directory Listings

```
intitle:"index of" "parent directory"
intitle:"index of" site:example.com
"Index of /" +passwd
```

### 💾 Finding Backup Files

```
filetype:bak site:example.com
filetype:backup
intitle:"index of" "backup"
```

### ❗ Finding Error Messages

```
"Warning: mysql_connect()"
"Fatal error:" "Call to undefined function"
"ORA-00936: missing expression"
```

### 🔍 Finding Version Information

```
"powered by" "version"
intitle:"Apache" "server status"
"Server: Apache" filetype:txt
```

---

## 📄 Common File Extensions

### 📚 Documents

- `filetype:pdf` – PDF documents
    
- `filetype:doc` – Word (older)
    
- `filetype:docx` – Word (new)
    
- `filetype:xls` – Excel
    
- `filetype:xlsx` – Excel (new)
    
- `filetype:ppt` – PowerPoint
    

### ⚙️ Configuration Files

- `filetype:conf`
    
- `filetype:config`
    
- `filetype:ini`
    
- `filetype:cfg`
    

### 🗃️ Database Files

- `filetype:sql`
    
- `filetype:db`
    
- `filetype:dbf`
    
- `filetype:mdb`
    

### 💻 Code Files

- `filetype:php`
    
- `filetype:asp`
    
- `filetype:jsp`
    
- `filetype:py`
    

### 💾 Backup Files

- `filetype:bak`
    
- `filetype:backup`
    
- `filetype:old`
    

---

## 🌐 URL Patterns to Search

### 🔐 Admin Areas

```
inurl:admin
inurl:administrator
inurl:wp-admin
inurl:cpanel
inurl:webmail
```

### 🔑 Login Pages

```
inurl:login
inurl:signin
inurl:auth
inurl:session
```

### ⚙️ Configuration Areas

```
inurl:config
inurl:settings
inurl:setup
inurl:install
```

### 🧪 Test/Development Areas

```
inurl:test
inurl:dev
inurl:staging
inurl:demo
```

---

## ✅ Tips for Effective Google Dorking

1. Combine multiple operators for specific results
    
2. Use `"` for exact phrases
    
3. Be patient—some results take time
    
4. Respect `robots.txt` and terms of service
    
5. Use only for ethical and legal purposes
    
6. Document findings clearly
    
7. Report security issues via responsible channels
    

---

## ⚠️ Ethical Considerations

- ✅ Use only for legitimate, authorized testing
    
- ✅ Follow **responsible disclosure** guidelines
    
- ❌ Do **not** access sensitive data without permission
    
- ❌ Never use for malicious or illegal purposes
    
- ✅ Respect site policies and legal boundaries
    
- ✅ Use findings to **improve security**, not exploit it
    

---

## 🧠 Rate Limiting & Best Practices

- 🔄 Google may limit excessive searches
    
- 🌐 Use VPNs or rotate IPs if rate-limited
    
- ⏳ Delay between queries to avoid detection
    
- 🕵️ Try alternative engines: `Bing`, `DuckDuckGo`, `Yandex`
    

---

## 📌 Example Queries

```
1. username site:website.com  
2. intitle:"index of" phpmyadmin  
3. site:stackoverflow.com/users "JaSON" "Australia"
```

---

## 📚 References

- [AT&T Cybersecurity Cheat Sheet (PDF)](https://cdn-cybersecurity.att.com/blog-content/GoogleHackingCheatSheet.pdf)
    
- [Compass OSINT Cheat Sheet (PDF)](https://www.compass-security.com/fileadmin/Research/White_Papers/2017-01_osint_cheat_sheet.pdf)
    
- [Semrush Dorking Guide (PDF)](https://static.semrush.com/blog/uploads/files/39/12/39121580a18160d3587274faed6323e2.pdf)
    
- [Scadahacker Web Hacking Cheat Sheet](https://scadahacker.com/library/Documents/Cheat_Sheets/Web%20-%20Google%20Hacking%20and%20Defense.pdf)
    
- [BlackHat Europe Presentation (PDF)](https://www.blackhat.com/presentations/bh-europe-05/BH_EU_05-Long.pdf)
    

---
