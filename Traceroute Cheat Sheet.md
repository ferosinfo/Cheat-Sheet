# Traceroute Cheat Sheet

```markdown
# 🌐 Traceroute Command Cheat Sheet
*Network diagnostic tool for tracing the path of packets across networks*

---

## 📋 Table of Contents
- [[#Basic Usage|Basic Usage]]
- [[#Command Syntax|Command Syntax]]
- [[#Unix/Linux Traceroute|Unix/Linux Traceroute]]
- [[#Windows Tracert|Windows Tracert]]
- [[#Advanced Options|Advanced Options]]
- [[#Common Output Interpretation|Common Output Interpretation]]
- [[#Troubleshooting Common Issues|Troubleshooting Common Issues]]
- [[#Alternative Tools|Alternative Tools]]
- [[#Security Considerations|Security Considerations]]

---

## 🚀 Basic Usage

### Purpose:
Traceroute maps the path packets take from your computer to a destination host, showing:
- Each hop (router) along the path
- Response times at each hop
- Network bottlenecks and routing issues

### Basic Command:
```bash
# Unix/Linux/macOS
traceroute example.com

# Windows
tracert example.com
```

---

## 🖥️ Command Syntax

### Unix/Linux/macOS:
```bash
traceroute [-options] host [packet_length]
```

### Windows:
```cmd
tracert [-options] host
```

---

## 🐧 Unix/Linux Traceroute Options

| Option        | Description                                          |
| ------------- | ---------------------------------------------------- |
| `-4`          | Use IPv4 only                                        |
| `-6`          | Use IPv6 only                                        |
| `-I`          | Use ICMP ECHO instead of UDP                         |
| `-T`          | Use TCP SYN (default port 80)                        |
| `-U`          | Use UDP (default port 33434)                         |
| `-p port`     | Use specific destination port                        |
| `-f ttl`      | Set initial TTL value                                |
| `-m max_ttl`  | Set maximum number of hops (default 30)              |
| `-q nqueries` | Set number of probes per hop (default 3)             |
| `-w waittime` | Set timeout in seconds                               |
| `-n`          | Display numerical addresses only (no DNS resolution) |
| `-g gateway`  | Specify loose source route gateway                   |

**Examples:**
```bash
# Basic traceroute with ICMP
traceroute -I google.com

# Traceroute with specific port and max hops
traceroute -T -p 443 -m 20 example.com

# Fast traceroute without DNS resolution
traceroute -n -q 1 target.com
```

---

## ⊞ Windows Tracert Options

| Option | Description |
|--------|-------------|
| `-4` | Force IPv4 |
| `-6` | Force IPv6 |
| `-d` | Don't resolve addresses to hostnames |
| `-h maximum_hops` | Maximum number of hops |
| `-w timeout` | Timeout in milliseconds for each reply |
| `-j host-list` | Loose source route along host-list |
| `-R` | Trace round-trip path (IPv6 only) |
| `-S srcaddr` | Source address to use (IPv6 only) |

**Examples:**
```cmd
# Basic tracert
tracert google.com

# Tracert without DNS resolution
tracert -d example.com

# Tracert with custom hop limit and timeout
tracert -h 15 -w 1000 target.com
```

---

## 🔧 Advanced Options

### Protocol-Specific Tracing:
```bash
# ICMP Echo tracing (often bypasses firewalls better)
traceroute -I example.com

# TCP SYN tracing (useful for web services)
traceroute -T -p 80 example.com

# UDP tracing (traditional method)
traceroute -U example.com
```

### Advanced Diagnostic Commands:
```bash
# Trace path and capture packets with tcpdump
traceroute -I example.com & sudo tcpdump -i any -n host example.com

# Continuous tracing for monitoring
while true; do traceroute -n example.com; echo "---"; sleep 5; done

# Trace to multiple destinations
for host in google.com yahoo.com bing.com; do echo "=== $host ==="; traceroute -n $host; done
```

---

## 🔍 Common Output Interpretation

### Sample Output:
```
traceroute to google.com (172.217.164.110), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  1.234 ms  1.456 ms  1.678 ms
 2  10.10.10.1 (10.10.10.1)  10.123 ms  10.456 ms  10.789 ms
 3  72.14.208.1 (72.14.208.1)  15.111 ms  15.222 ms  15.333 ms
 4  * * *
 5  216.239.46.248 (216.239.46.248)  20.555 ms  20.666 ms  20.777 ms
 6  google.com (172.217.164.110)  21.888 ms  21.999 ms  22.111 ms
```

### Understanding Output:
- **Hop number**: Sequential number of the router
- **IP address/hostname**: Router identification
- **Response times**: Three round-trip times in milliseconds
- `* * *`: No response from that hop (timeout, firewall, etc.)
- **Sudden latency increase**: May indicate network congestion
- **Consistent timeouts**: May indicate firewall blocking

---

## 🐛 Troubleshooting Common Issues

### Timeouts (`* * *`):
- **Cause**: Firewalls blocking ICMP/UDP, network congestion
- **Solution**: Try different protocols (`-I`, `-T`)

### High Latency:
- **Diagnosis**: Compare response times between hops
- **Solution**: Identify where latency increases significantly

### Asymmetric Routing:
- **Symptom**: Different paths for request vs response
- **Diagnosis**: Use `mtr` (My TraceRoute) for continuous monitoring

### Destination Unreachable:
- **Message**: `!H` (Host unreachable), `!N` (Network unreachable)
- **Solution**: Check destination availability and firewall rules

---

## 🔄 Alternative Tools

### mtr (My TraceRoute):
```bash
# Combines traceroute and ping functionality
mtr example.com

# Report mode (output after 10 cycles)
mtr --report --report-cycles 10 example.com
```

### tcptraceroute:
```bash
# TCP-based traceroute (bypasses UDP filters)
tcptraceroute example.com 80
```

### hping3:
```bash
# Advanced traceroute with custom packets
hping3 --traceroute -S -p 80 example.com
```

### pathping (Windows):
```cmd
# Combines traceroute and ping statistics
pathping example.com
```

---

## 🛡️ Security Considerations

### Privacy Implications:
- Traceroute reveals internal network structure
- Consider firewall rules to limit external tracing

### Detection:
- Traceroute activity may be logged by network devices
- Some organizations monitor for traceroute attempts

### Ethical Use:
- Only trace to systems you own or have permission to test
- Be aware of organizational policies regarding network probing

### Defensive Measures:
```bash
# Limit ICMP responses on Linux
sysctl -w net.ipv4.icmp_echo_ignore_all=1

# Rate limit ICMP responses
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
```

---

## 💡 Pro Tips

1. **Use multiple protocols** when troubleshooting - if UDP fails, try ICMP or TCP
2. **Combine with other tools** like ping, dig, and mtr for comprehensive diagnosis
3. **Remember timeouts aren't always failures** - many routers rate-limit or block traceroute packets
4. **Use numerical IP addresses** (`-n` flag) to avoid DNS resolution delays
5. **Consider time of day** - network paths and performance can vary throughout the day
6. **Save output for comparison** when troubleshooting intermittent issues

---

