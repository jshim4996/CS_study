# Chapter 03. DNS (Domain Name System)

> Understanding how domain names are translated to IP addresses through the DNS hierarchy.

---

## DNS Concept (DNS 개념)

### Concept (개념)
**DNS (Domain Name System)** is a hierarchical, distributed naming system that translates human-readable domain names into IP addresses. It's often called the "phonebook of the Internet."

**Why DNS?**
- Humans remember names better than numbers
- IP addresses can change, but domain names stay the same
- Load balancing: one domain can map to multiple IPs
- Email routing and other services

**Key Components:**
- **Domain Name (도메인 이름)**: Human-readable address (e.g., www.google.com)
- **IP Address (IP 주소)**: Numerical address (e.g., 142.250.190.78)
- **DNS Server (DNS 서버)**: Server that stores and provides DNS records
- **Resolver (리졸버)**: Client-side DNS lookup service

### Diagram / Example
```
DNS Translation:

www.google.com  ─────►  142.250.190.78

User's Perspective:
┌─────────────────────────────────────────────────────┐
│ Browser Address Bar: https://www.google.com         │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│ Behind the scenes: Connect to 142.250.190.78:443    │
└─────────────────────────────────────────────────────┘
```

---

## Domain Hierarchy (도메인 계층 구조)

### Concept (개념)
DNS uses a hierarchical tree structure. Domain names are read from right to left, with each level separated by a dot.

**Hierarchy Levels:**
1. **Root Domain (루트 도메인)**: Represented by a dot (.) at the end
2. **TLD (Top-Level Domain / 최상위 도메인)**: .com, .org, .kr, .edu
3. **SLD (Second-Level Domain / 2차 도메인)**: google, naver, amazon
4. **Subdomain (서브도메인)**: www, mail, api, blog

**TLD Types:**
| Type | Examples | Description |
|------|----------|-------------|
| gTLD (Generic) | .com, .org, .net | General purpose |
| ccTLD (Country Code) | .kr, .jp, .uk | Country-specific |
| sTLD (Sponsored) | .edu, .gov, .mil | Restricted purpose |
| New gTLD | .app, .dev, .io | Recently added |

### Diagram / Example
```
Domain Name Hierarchy:

                        . (Root)
                        │
        ┌───────────────┼───────────────┐
        │               │               │
      .com            .org            .kr
        │               │               │
    ┌───┴───┐       ┌───┴───┐       ┌───┴───┐
  google  amazon   wikipedia       naver  daum
    │       │           │            │
┌───┼───┐   │           │        ┌───┼───┐
www mail  www          www      www  mail


FQDN (Fully Qualified Domain Name):
www.google.com.
│   │      │  │
│   │      │  └── Root (usually omitted)
│   │      └───── TLD (Top-Level Domain)
│   └──────────── SLD (Second-Level Domain)
└──────────────── Subdomain
```

---

## DNS Query Process (DNS 쿼리 과정)

### Concept (개념)
When you enter a URL, a series of DNS queries occur to resolve the domain name to an IP address.

**Query Types:**
- **Recursive Query (재귀적 쿼리)**: Resolver does all the work, returns final answer
- **Iterative Query (반복적 쿼리)**: Server returns referral to next server

**DNS Resolution Steps:**
1. Check browser cache
2. Check OS cache
3. Check local DNS resolver cache
4. Query Root DNS server
5. Query TLD DNS server
6. Query Authoritative DNS server

### Diagram / Example
```
DNS Resolution Process for "www.example.com":

[User's Computer]
       │
       │ 1. www.example.com?
       ▼
[Local DNS Resolver]  ← Check cache first
       │
       │ 2. www.example.com?
       ▼
[Root DNS Server]
       │
       │ 3. "Ask .com TLD server at 192.5.6.30"
       ▼
[.com TLD Server]
       │
       │ 4. "Ask example.com NS at 198.51.100.1"
       ▼
[example.com Authoritative Server]
       │
       │ 5. "www.example.com = 93.184.216.34"
       ▼
[Local DNS Resolver]  ← Cache the result
       │
       │ 6. "93.184.216.34"
       ▼
[User's Computer]  ← Connect to IP


Timeline:
Browser Cache → OS Cache → Router Cache → ISP DNS → Root → TLD → Authoritative
   ~0ms          ~1ms        ~5ms          ~10ms    ~50ms  ~100ms    ~150ms
```

---

## DNS Record Types (DNS 레코드 유형)

### Concept (개념)
DNS records are stored in zone files on DNS servers. Each record type serves a specific purpose.

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps domain to IPv4 address | example.com → 93.184.216.34 |
| **AAAA** | Maps domain to IPv6 address | example.com → 2606:2800:220:1:... |
| **CNAME** | Alias to another domain | www.example.com → example.com |
| **MX** | Mail server for domain | example.com → mail.example.com |
| **NS** | Authoritative name server | example.com → ns1.example.com |
| **TXT** | Text information | SPF, DKIM, domain verification |
| **SOA** | Start of Authority | Zone metadata |
| **PTR** | Reverse DNS (IP to domain) | 34.216.184.93 → example.com |
| **SRV** | Service location | _sip._tcp.example.com |

### Diagram / Example
```
DNS Zone File Example (example.com):

; SOA Record (Start of Authority)
example.com.    IN  SOA   ns1.example.com. admin.example.com. (
                          2024011901  ; Serial
                          3600        ; Refresh
                          600         ; Retry
                          86400       ; Expire
                          3600 )      ; Minimum TTL

; NS Records (Name Servers)
example.com.    IN  NS    ns1.example.com.
example.com.    IN  NS    ns2.example.com.

; A Records (IPv4 Addresses)
example.com.    IN  A     93.184.216.34
www             IN  A     93.184.216.34
api             IN  A     93.184.216.35

; AAAA Record (IPv6 Address)
example.com.    IN  AAAA  2606:2800:220:1:248:1893:25c8:1946

; CNAME Records (Aliases)
blog            IN  CNAME www.example.com.
shop            IN  CNAME www.example.com.

; MX Records (Mail Servers)
example.com.    IN  MX    10 mail1.example.com.
example.com.    IN  MX    20 mail2.example.com.

; TXT Records
example.com.    IN  TXT   "v=spf1 include:_spf.google.com ~all"
```

---

## DNS Caching (DNS 캐싱)

### Concept (개념)
DNS caching stores DNS query results temporarily to reduce lookup time and network traffic.

**Cache Locations:**
1. **Browser Cache**: Shortest TTL, per-browser
2. **OS Cache**: System-wide DNS cache
3. **Router Cache**: Local network level
4. **ISP DNS Cache**: Service provider level

**TTL (Time To Live / 생존 시간):**
- Specifies how long a record can be cached
- Measured in seconds
- Lower TTL = faster propagation, more queries
- Higher TTL = fewer queries, slower updates

### Diagram / Example
```
DNS Caching Hierarchy:

Request: www.google.com

[Browser Cache]        TTL: ~1 minute
    │ Miss
    ▼
[OS DNS Cache]         TTL: varies by OS
    │ Miss
    ▼
[Router Cache]         TTL: ~30 minutes
    │ Miss
    ▼
[ISP DNS Server]       TTL: based on record
    │ Miss
    ▼
[Authoritative DNS]    Source of truth
    │
    │ Response with TTL=300 (5 minutes)
    │
    ▼
[All caches store result for 300 seconds]


Clearing DNS Cache:
- Browser: chrome://net-internals/#dns (Clear host cache)
- Windows: ipconfig /flushdns
- macOS: sudo dscacheutil -flushcache
- Linux: sudo systemd-resolve --flush-caches
```

---

## hosts File (hosts 파일)

### Concept (개념)
The **hosts file** is a local text file that maps hostnames to IP addresses. It's checked before DNS queries and can override DNS lookups.

**Location:**
- **Windows**: `C:\Windows\System32\drivers\etc\hosts`
- **macOS/Linux**: `/etc/hosts`

**Use Cases:**
- Local development (mapping localhost domains)
- Blocking websites
- Testing before DNS propagation
- Bypassing DNS for specific hosts

### Diagram / Example
```
hosts File Example:

# /etc/hosts (macOS/Linux) or C:\Windows\System32\drivers\etc\hosts (Windows)

# Localhost
127.0.0.1       localhost
::1             localhost

# Local Development
127.0.0.1       myapp.local
127.0.0.1       api.myapp.local
192.168.1.100   dev-server.local

# Block unwanted sites
0.0.0.0         ads.example.com
0.0.0.0         tracking.example.com

# Testing before DNS propagation
203.0.113.50    newserver.example.com


Resolution Order:
┌─────────────────────────────────────────────────┐
│ 1. hosts file                                   │
│    └── If found, use this IP (skip DNS)         │
│                                                 │
│ 2. DNS Cache                                    │
│    └── If found in cache, use cached IP         │
│                                                 │
│ 3. DNS Query                                    │
│    └── Query DNS servers for IP                 │
└─────────────────────────────────────────────────┘
```

---

## DNS Security (DNS 보안)

### Concept (개념)
DNS was designed without security, making it vulnerable to attacks. Several mechanisms have been developed to secure DNS.

**Common DNS Attacks:**
| Attack | Description |
|--------|-------------|
| DNS Spoofing/Cache Poisoning | Inject false DNS records into cache |
| DNS Hijacking | Redirect DNS queries to malicious server |
| DDoS on DNS | Overwhelm DNS servers |
| DNS Tunneling | Exfiltrate data through DNS queries |

**Security Solutions:**
- **DNSSEC**: Cryptographic signatures for DNS records
- **DNS over HTTPS (DoH)**: Encrypt DNS queries via HTTPS
- **DNS over TLS (DoT)**: Encrypt DNS queries via TLS

### Diagram / Example
```
DNS Cache Poisoning Attack:

Normal Flow:
[Client] → [DNS Resolver] → [Authoritative Server]
              ↓
         "bank.com = 1.2.3.4" (legitimate)

Attack:
[Client] → [DNS Resolver] ← [Attacker]
              ↓
         "bank.com = 6.6.6.6" (malicious)

[Client connects to 6.6.6.6 thinking it's bank.com]


DNSSEC Protection:
┌────────────────────────────────────────────────────┐
│ DNS Response:                                      │
│   bank.com.  A     1.2.3.4                        │
│   bank.com.  RRSIG  A  [digital signature]        │
│                                                    │
│ Resolver verifies signature using public key       │
│ If signature invalid → reject response             │
└────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Key Points |
|---------|------------|
| DNS | Translates domain names to IP addresses |
| Hierarchy | Root → TLD → SLD → Subdomain |
| Query Process | Recursive/Iterative queries through hierarchy |
| Record Types | A, AAAA, CNAME, MX, NS, TXT, etc. |
| Caching | TTL-based caching at multiple levels |
| hosts File | Local override for DNS resolution |

---

## Key Terms (주요 용어)

- **DNS (Domain Name System)**: 도메인 이름 시스템
- **Domain (도메인)**: 인터넷 주소 이름
- **TLD (Top-Level Domain)**: 최상위 도메인 (.com, .kr 등)
- **Resolver (리졸버)**: DNS 조회를 수행하는 클라이언트
- **Authoritative Server (권한 있는 서버)**: 도메인의 공식 DNS 서버
- **TTL (Time To Live)**: 캐시 유효 시간
- **Record (레코드)**: DNS 데이터베이스의 항목
- **FQDN (Fully Qualified Domain Name)**: 전체 주소 도메인 이름
