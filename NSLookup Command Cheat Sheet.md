# 🖥️ NSLookup Command Cheat Sheet
*Network administration tool for querying Domain Name System (DNS)*

Web Based : https://www.nslookup.io/

```markdown
# 🖥️ NSLookup Command Cheat Sheet
*Network administration tool for querying Domain Name System (DNS)*

---

## 📋 Table of Contents
- [[#Basic Usage|Basic Usage]]
- [[#Command Syntax|Command Syntax]]
- [[#Interactive Mode|Interactive Mode]]
- [[#Common Queries|Common Queries]]
- [[#DNS Record Types|DNS Record Types]]
- [[#Troubleshooting Examples|Troubleshooting Examples]]
- [[#Advanced Usage|Advanced Usage]]
- [[#Alternative Tools|Alternative Tools]]

---

## 🚀 Basic Usage

### Purpose:
NSLookup is used to query DNS servers to obtain domain name or IP address mapping information.

### Basic Command:
```bash
# Quick lookup
nslookup domain.com

# Lookup specific DNS server
nslookup domain.com dns-server.com

# Interactive mode
nslookup
```

---

## 🖥️ Command Syntax

### Basic Syntax:
```bash
nslookup [options] [domain] [dns-server]
```

### Common Options:
| Option | Description |
|--------|-------------|
| `-type=X` | Query specific record type |
| `-debug` | Display detailed response |
| `-timeout=X` | Set timeout in seconds |
| `-port=X` | Use specific port (default 53) |
| `-query=X` | Query type (same as -type) |
| `-retry=X` | Number of retries |

---

## 🔄 Interactive Mode

### Starting Interactive Mode:
```bash
nslookup
```

### Interactive Commands:
| Command | Description |
|---------|-------------|
| `server DNS` | Change default DNS server |
| `set type=X` | Set query record type |
| `set debug` | Turn debugging on/off |
| `set timeout=X` | Set timeout value |
| `set retry=X` | Set number of retries |
| `domain` | Set default domain name |
| `exit` or `Ctrl+D` | Exit nslookup |

### Interactive Example:
```bash
nslookup
> server 8.8.8.8
> set type=MX
> google.com
> set type=A
> wikipedia.org
> exit
```

---

## 📊 Common Queries

### Basic Lookups:
```bash
# Standard forward lookup (A record)
nslookup example.com

# Reverse lookup (PTR record)
nslookup 8.8.8.8

# Query specific DNS server
nslookup example.com ns1.example.com
nslookup example.com 8.8.8.8
```

### Specific Record Types:
```bash
# MX records (Mail servers)
nslookup -type=MX google.com

# NS records (Name servers)
nslookup -type=NS example.com

# CNAME records (Aliases)
nslookup -type=CNAME www.example.com

# TXT records (Text records)
nslookup -type=TXT example.com

# SOA records (Zone authority)
nslookup -type=SOA example.com

# ALL records
nslookup -type=ANY example.com
```

---

## 📋 DNS Record Types

| Type | Description | Example |
|------|-------------|---------|
| `A` | IPv4 Address | `nslookup -type=A example.com` |
| `AAAA` | IPv6 Address | `nslookup -type=AAAA example.com` |
| `MX` | Mail Exchange | `nslookup -type=MX gmail.com` |
| `NS` | Name Server | `nslookup -type=NS example.com` |
| `CNAME` | Canonical Name | `nslookup -type=CNAME www.example.com` |
| `TXT` | Text Record | `nslookup -type=TXT example.com` |
| `PTR` | Pointer (Reverse DNS) | `nslookup -type=PTR 8.8.8.8` |
| `SOA` | Start of Authority | `nslookup -type=SOA example.com` |
| `SRV` | Service Record | `nslookup -type=SRV _sip._tcp.example.com` |

---

## 🐛 Troubleshooting Examples

### DNS Resolution Issues:
```bash
# Check if domain resolves
nslookup example.com

# Compare different DNS servers
nslookup example.com 8.8.8.8
nslookup example.com 1.1.1.1

# Check reverse DNS
nslookup 8.8.8.8
```

### Email Server Configuration:
```bash
# Check MX records for email delivery
nslookup -type=MX gmail.com

# Verify SPF records
nslookup -type=TXT gmail.com | grep spf

# Check DKIM records (typically specific selector)
nslookup -type=TXT selector._domainkey.example.com
```

### Domain Delegation Issues:
```bash
# Check name servers
nslookup -type=NS example.com

# Verify SOA record
nslookup -type=SOA example.com

# Check glue records
nslookup -type=A ns1.example.com
```

---

## ⚡ Advanced Usage

### Debug Mode:
```bash
# Get detailed query information
nslookup -debug example.com

# Interactive debug mode
nslookup
> set debug
> example.com
```

### Batch Queries:
```bash
# Multiple queries from command line
nslookup -type=A example.com google.com cloudflare.com

# Using a list of domains from file
for domain in $(cat domains.txt); do
    echo "=== $domain ==="
    nslookup $domain
    echo ""
done
```

### Specific Port Queries:
```bash
# Query DNS server on non-standard port
nslookup -port=5353 example.com localhost
```

### Reverse Lookup Patterns:
```bash
# Reverse lookup for IP range
for ip in {1..10}; do
    nslookup 192.168.1.$ip
done
```

---

## 🔄 Alternative Tools

### dig (Domain Information Groper):
```bash
# More detailed DNS information
dig example.com
dig example.com ANY
dig +short example.com
```

### host:
```bash
# Simpler DNS lookup
host example.com
host -t MX example.com
host 8.8.8.8
```

### whois:
```bash
# Domain registration information
whois example.com
```

---

## 💡 Pro Tips

1. **Use specific DNS servers** to test if resolution issues are server-specific
2. **Always check both forward and reverse DNS** for completeness
3. **Use debug mode** when troubleshooting complex DNS issues
4. **Remember TTL values** when making DNS changes
5. **Test different record types** to understand domain configuration
6. **Compare results from multiple DNS servers** (Google DNS, Cloudflare, OpenDNS)

### Common Troubleshooting Flow:
1. `nslookup domain.com` - Basic resolution check
2. `nslookup -type=NS domain.com` - Check name servers
3. `nslookup domain.com 8.8.8.8` - Test with public DNS
4. `nslookup -type=SOA domain.com` - Check zone authority
5. `nslookup -debug domain.com` - Detailed query information

---

## 🛡️ Security Considerations

- DNS queries are generally unencrypted (consider DNS-over-HTTPS or DNS-over-TLS for sensitive queries)
- Be aware of DNS cache poisoning attacks
- DNSSEC provides authentication but isn't universally implemented
- DNS information can reveal network structure

---

**Cheat Sheet Version**: 1.2  
**Last Updated**: 2024-09-24
```
