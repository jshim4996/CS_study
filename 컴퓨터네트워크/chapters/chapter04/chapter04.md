# Chapter 04. Other Application Protocols (기타 응용 프로토콜)

> Exploring essential application-layer protocols for file transfer, email, and secure remote access.

---

## FTP (File Transfer Protocol)

### Concept (개념)
**FTP (File Transfer Protocol)** is a standard protocol for transferring files between a client and server over a network. It operates on the application layer and uses TCP for reliable data transfer.

**Key Characteristics:**
- **Two Connections**: Control (port 21) and Data (port 20 or dynamic)
- **Modes**: Active mode and Passive mode
- **Authentication**: Username/password or anonymous access
- **Not Secure**: Credentials sent in plain text (use FTPS or SFTP instead)

**FTP Modes:**
| Mode | Description | Firewall Friendly |
|------|-------------|-------------------|
| Active | Server initiates data connection to client | No |
| Passive | Client initiates both connections | Yes |

### Diagram / Example
```
FTP Active Mode:
[Client]                                    [Server]
   │                                           │
   │ ── Control Connection (Port 21) ────────> │
   │    USER username                          │
   │    PASS password                          │
   │    PORT 192.168.1.10,45,67                │
   │                                           │
   │ <───── Data Connection (Port 20) ──────── │
   │        File data transfer                 │


FTP Passive Mode (Firewall Friendly):
[Client]                                    [Server]
   │                                           │
   │ ── Control Connection (Port 21) ────────> │
   │    USER username                          │
   │    PASS password                          │
   │    PASV                                   │
   │                                           │
   │ <── "227 Entering Passive Mode           │
   │      (192.168.1.100,195,149)"             │
   │                                           │
   │ ── Data Connection (Port 50069) ────────> │
   │    File data transfer                     │


Common FTP Commands:
USER    - Username for authentication
PASS    - Password for authentication
LIST    - List files in directory
RETR    - Retrieve (download) file
STOR    - Store (upload) file
PWD     - Print working directory
CWD     - Change working directory
QUIT    - End session
```

---

## FTPS and SFTP (보안 파일 전송)

### Concept (개념)
Since FTP is insecure, two secure alternatives exist:

**FTPS (FTP Secure):**
- FTP over SSL/TLS
- Encrypts both control and data channels
- Uses ports 990 (implicit) or 21 (explicit)
- Certificate-based authentication

**SFTP (SSH File Transfer Protocol):**
- Completely different protocol (not FTP over SSH)
- Uses SSH for encryption
- Single connection on port 22
- Supports key-based authentication

| Feature | FTP | FTPS | SFTP |
|---------|-----|------|------|
| Port | 21, 20 | 990 or 21 | 22 |
| Encryption | None | SSL/TLS | SSH |
| Firewall | Problematic | Problematic | Simple |
| Authentication | Password | Certificate/Password | Key/Password |

### Diagram / Example
```
FTPS Connection (Explicit):
[Client]                                    [Server]
   │                                           │
   │ ── TCP Connection (Port 21) ────────────> │
   │                                           │
   │ ── AUTH TLS ─────────────────────────────>│
   │ <── 234 AUTH TLS OK ─────────────────────  │
   │                                           │
   │ ══ TLS Handshake ═══════════════════════> │
   │ <═════════════════════════════════════════ │
   │                                           │
   │ [Encrypted FTP commands/data]             │


SFTP Connection:
[Client]                                    [Server]
   │                                           │
   │ ══ SSH Connection (Port 22) ════════════> │
   │    [Key exchange, encryption setup]       │
   │                                           │
   │ ══ SFTP Subsystem Request ══════════════> │
   │                                           │
   │ [All file operations over single          │
   │  encrypted SSH connection]                │
```

---

## SMTP (Simple Mail Transfer Protocol)

### Concept (개념)
**SMTP (Simple Mail Transfer Protocol)** is used for sending emails between mail servers and from email clients to mail servers.

**Key Characteristics:**
- **Push Protocol**: Sends email to next server
- **Port**: 25 (server-to-server), 587 (client submission), 465 (SMTPS)
- **Text-based**: Human-readable commands
- **Store-and-Forward**: Servers store and relay messages

**SMTP Commands:**
| Command | Purpose |
|---------|---------|
| HELO/EHLO | Identify client to server |
| MAIL FROM | Specify sender address |
| RCPT TO | Specify recipient address |
| DATA | Begin message body |
| QUIT | End session |

### Diagram / Example
```
Email Sending Process:

[Your Computer]                     [Your Mail Server]           [Recipient's Mail Server]
  (Outlook)                         (smtp.gmail.com)             (smtp.company.com)
      │                                   │                             │
      │ ── SMTP (Port 587) ─────────────> │                             │
      │    "Send email to                 │                             │
      │     user@company.com"             │                             │
      │                                   │                             │
      │                                   │ ── SMTP (Port 25) ─────────>│
      │                                   │    Forward email             │
      │                                   │                             │
      │                                   │                     [Email stored in
      │                                   │                      recipient's mailbox]


SMTP Session Example:
S: 220 smtp.example.com ESMTP ready
C: EHLO client.example.com
S: 250-smtp.example.com Hello
S: 250-SIZE 35882577
S: 250-STARTTLS
S: 250 OK
C: MAIL FROM:<sender@example.com>
S: 250 OK
C: RCPT TO:<recipient@example.com>
S: 250 OK
C: DATA
S: 354 Start mail input
C: From: sender@example.com
C: To: recipient@example.com
C: Subject: Hello!
C:
C: This is the email body.
C: .
S: 250 OK: Message queued
C: QUIT
S: 221 Bye
```

---

## POP3 (Post Office Protocol version 3)

### Concept (개념)
**POP3 (Post Office Protocol version 3)** is used by email clients to retrieve emails from a mail server. It downloads emails to the local device.

**Key Characteristics:**
- **Pull Protocol**: Downloads email from server
- **Port**: 110 (plain), 995 (POP3S with SSL)
- **Download and Delete**: Typically removes email from server after download
- **Single Device**: Not ideal for multiple devices

**POP3 Commands:**
| Command | Purpose |
|---------|---------|
| USER | Specify username |
| PASS | Specify password |
| STAT | Get mailbox status |
| LIST | List messages |
| RETR | Retrieve message |
| DELE | Mark message for deletion |
| QUIT | End session (apply deletions) |

### Diagram / Example
```
POP3 Email Retrieval:

[Mail Server]                              [Email Client]
(pop.gmail.com)                            (Outlook)
      │                                         │
      │ <── TCP Connection (Port 995/SSL) ───── │
      │                                         │
      │     USER john@gmail.com                 │
      │     PASS secretpassword                 │
      │                                         │
      │     STAT                                │
      │ ──> +OK 3 messages (4500 bytes) ──────> │
      │                                         │
      │     RETR 1                              │
      │ ──> +OK [email content] ──────────────> │
      │                                         │
      │     DELE 1                              │
      │ ──> +OK message deleted ──────────────> │
      │                                         │
      │     QUIT                                │


POP3 Session:
S: +OK POP3 server ready
C: USER john@example.com
S: +OK User accepted
C: PASS mypassword
S: +OK Pass accepted
C: STAT
S: +OK 2 320
C: LIST
S: +OK 2 messages
S: 1 120
S: 2 200
S: .
C: RETR 1
S: +OK 120 octets
S: [Message content...]
S: .
C: QUIT
S: +OK Bye
```

---

## IMAP (Internet Message Access Protocol)

### Concept (개념)
**IMAP (Internet Message Access Protocol)** is a more advanced protocol for accessing email that keeps messages on the server.

**Key Characteristics:**
- **Server-side Storage**: Emails remain on server
- **Port**: 143 (plain), 993 (IMAPS with SSL)
- **Folder Management**: Create, delete, rename folders on server
- **Multiple Devices**: Sync across all devices
- **Partial Download**: Can fetch headers only, then body

**POP3 vs IMAP:**
| Feature | POP3 | IMAP |
|---------|------|------|
| Storage | Local | Server |
| Multi-device | Poor | Excellent |
| Offline Access | Full (downloaded) | Limited |
| Server Space | Freed after download | Required |
| Folder Sync | No | Yes |

### Diagram / Example
```
IMAP Multi-Device Sync:

              [Mail Server (IMAP)]
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
   [Desktop]     [Laptop]      [Phone]

   All devices see the same:
   - Inbox (3 unread)
   - Sent
   - Drafts
   - Custom Folders

   Action on any device syncs to all:
   - Read email on phone → Shows as read on desktop
   - Delete on laptop → Deleted everywhere
   - Move to folder → Reflected on all devices


IMAP Commands:
a001 LOGIN username password
a002 SELECT INBOX
a003 FETCH 1:* (FLAGS ENVELOPE)
a004 FETCH 1 BODY[TEXT]
a005 STORE 1 +FLAGS (\Seen)
a006 CREATE "Work Projects"
a007 COPY 1 "Work Projects"
a008 LOGOUT
```

---

## Email System Overview (이메일 시스템 개요)

### Concept (개념)
The complete email system uses multiple protocols working together.

**Components:**
- **MUA (Mail User Agent)**: Email client (Outlook, Gmail app)
- **MSA (Mail Submission Agent)**: Accepts email from MUA
- **MTA (Mail Transfer Agent)**: Routes email between servers
- **MDA (Mail Delivery Agent)**: Delivers to mailbox
- **Mailbox**: Storage for user's emails

### Diagram / Example
```
Complete Email Flow:

Sender                                                           Recipient
┌─────────┐    SMTP     ┌─────────┐    SMTP     ┌─────────┐    POP3/IMAP   ┌─────────┐
│   MUA   │ ──────────> │   MTA   │ ──────────> │   MTA   │ ──────────────>│   MUA   │
│(Outlook)│   (587)     │(Gmail)  │   (25)      │(Yahoo)  │    (993/995)   │(Yahoo)  │
└─────────┘             └─────────┘             └─────────┘                └─────────┘
                              │                       │
                              ▼                       ▼
                        [DNS: MX Record]        [Mailbox Storage]


Protocol Usage Summary:
┌────────────────────────────────────────────────────────────────┐
│ Sending Email (SMTP):                                          │
│   User → Mail Server → Recipient's Mail Server                 │
│                                                                │
│ Receiving Email (POP3 or IMAP):                                │
│   Mail Server → User's Email Client                            │
└────────────────────────────────────────────────────────────────┘
```

---

## SSH (Secure Shell)

### Concept (개념)
**SSH (Secure Shell)** is a cryptographic network protocol for secure remote login and command execution over an unsecured network.

**Key Characteristics:**
- **Encrypted**: All traffic is encrypted
- **Port**: 22
- **Authentication**: Password or public key
- **Tunneling**: Can create secure tunnels for other protocols
- **Replaces**: Telnet, rlogin, rsh (insecure protocols)

**SSH Components:**
- **SSH Client**: Initiates connection (ssh command)
- **SSH Server**: Accepts connections (sshd daemon)
- **SSH Keys**: Public/private key pair for authentication

### Diagram / Example
```
SSH Connection Process:

[Client]                                    [Server]
   │                                           │
   │ ── TCP Connection (Port 22) ────────────> │
   │                                           │
   │ <── SSH Version Exchange ──────────────── │
   │                                           │
   │ ══ Key Exchange (Diffie-Hellman) ═══════> │
   │ <═══════════════════════════════════════  │
   │    [Session key established]              │
   │                                           │
   │ ══ User Authentication ═════════════════> │
   │    (Password or Public Key)               │
   │ <══ Authentication Success ═════════════  │
   │                                           │
   │ [Encrypted shell session begins]          │


SSH Key-Based Authentication:
┌─────────────────────────────────────────────────────────────┐
│ Setup (one-time):                                           │
│   1. Generate key pair: ssh-keygen -t ed25519               │
│   2. Copy public key to server: ssh-copy-id user@server     │
│                                                             │
│ Connection:                                                 │
│   1. Client sends public key                                │
│   2. Server encrypts challenge with public key              │
│   3. Client decrypts with private key                       │
│   4. Server verifies → Access granted                       │
└─────────────────────────────────────────────────────────────┘


Common SSH Commands:
ssh user@hostname              # Basic connection
ssh -p 2222 user@hostname      # Custom port
ssh -i ~/.ssh/mykey user@host  # Specific key
ssh -L 8080:localhost:80 user@host  # Local port forwarding
scp file.txt user@host:/path/  # Secure copy
sftp user@hostname             # SFTP session
```

---

## SSH Tunneling (SSH 터널링)

### Concept (개념)
SSH tunneling (port forwarding) creates encrypted tunnels to securely access services.

**Types:**
- **Local Forwarding (-L)**: Access remote service through local port
- **Remote Forwarding (-R)**: Expose local service to remote network
- **Dynamic Forwarding (-D)**: SOCKS proxy for any connection

### Diagram / Example
```
Local Port Forwarding:
Access remote database through SSH tunnel

ssh -L 3306:db.internal:3306 user@jumphost

[Your Computer]       [Jump Host]           [Database Server]
    :3306    ═══SSH═══>  :22    ──────────>   db.internal:3306
      │                                              │
      └──────── Encrypted Tunnel ───────────────────┘

Connect to localhost:3306 → Actually connects to db.internal:3306


Remote Port Forwarding:
Expose local web server to internet

ssh -R 8080:localhost:3000 user@public-server

[Your Computer]       [Public Server]        [Internet Users]
   localhost:3000 <═══SSH═══  :8080  <────────  :8080
                                                  │
                   [Users access public-server:8080]
                   [Traffic forwarded to your localhost:3000]


Dynamic Port Forwarding (SOCKS Proxy):
ssh -D 1080 user@server

[Your Computer]                    [SSH Server]
Browser ──> SOCKS Proxy :1080 ═══> Internet Access
   │                                    │
   └── All traffic through encrypted SSH tunnel
```

---

## Summary (요약)

| Protocol | Purpose | Port | Security |
|----------|---------|------|----------|
| FTP | File transfer | 21, 20 | None (use FTPS/SFTP) |
| FTPS | Secure file transfer | 990/21 | SSL/TLS |
| SFTP | Secure file transfer | 22 | SSH |
| SMTP | Send email | 25, 587, 465 | STARTTLS/SSL |
| POP3 | Download email | 110, 995 | SSL |
| IMAP | Access email | 143, 993 | SSL |
| SSH | Secure remote access | 22 | Built-in encryption |

---

## Key Terms (주요 용어)

- **FTP (File Transfer Protocol)**: 파일 전송 프로토콜
- **SMTP (Simple Mail Transfer Protocol)**: 이메일 전송 프로토콜
- **POP3 (Post Office Protocol)**: 이메일 수신 프로토콜 (다운로드)
- **IMAP (Internet Message Access Protocol)**: 이메일 접근 프로토콜 (서버 동기화)
- **SSH (Secure Shell)**: 보안 원격 접속 프로토콜
- **Port Forwarding (포트 포워딩)**: 포트를 통한 트래픽 전달
- **Tunneling (터널링)**: 암호화된 통신 경로 생성
