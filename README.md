# Active Directory Attack Path – Detection & Mitigation Lab

A structured **Active Directory (AD) attack simulation lab** covering the full offensive lifecycle from **initial domain foothold to Kerberos ticket abuse**.

The project documents **attack methodology, tooling, detection indicators, and mitigation strategies** for each stage.

The objective is to understand **how attackers move through an AD environment** and how defenders can **detect and stop them**.

---

# Project Scope

Currently implemented modules:

| Module | Phase | Focus |
| --- | --- | --- |
| **M1** | Initial AD Exploitation | LLMNR / NBT-NS poisoning |
| **M2** | AD Enumeration | Domain discovery and privilege mapping |
| **M3** | DACL Abuse | Active Directory permission abuse |
| **M4** | Kerberos Abuse | Ticket extraction and password cracking |
| **M5** | Credential Dumping | NTDS, SAM, DCSync attacks |
| **M6** | Kerberos Ticket Attacks | Golden / Silver / Diamond tickets |

Future modules planned:

- Privilege Escalation
- Group-based attacks
- Persistence mechanisms
- ADCS abuse

---

# Attack Path Overview

Active Directory attacks generally follow a predictable progression:

```
Initial Access
     ↓
Enumeration
     ↓
Privilege Escalation
     ↓
Credential Dumping
     ↓
Kerberos Abuse
     ↓
Persistence
```

This repository walks through that chain step-by-step using a **controlled lab environment**.

---

# Lab Environment

Typical setup used in this project:

| Component | Technology |
| --- | --- |
| Attacker Machine | Kali Linux |
| Domain Controller | Windows Server |
| Domain | Active Directory |
| Tools | BloodHound, Responder, Impacket, PowerView |

Network model:

```
Kali (Attacker)
      |
      |  Internal Network
      |
Windows Domain Controller
      |
Domain Users / Services
```

---

# Tools Used

| Tool | Purpose |
| --- | --- |
| Responder | LLMNR / NBT-NS poisoning |
| BloodHound | AD privilege graph analysis |
| PowerView | Domain enumeration |
| RPCClient | SMB and AD queries |
| Impacket | Credential dumping and Kerberos abuse |
| NetExec / CrackMapExec | Lateral movement and enumeration |
| ADRecon | Domain reconnaissance |

---

# Module Breakdown

---

# M1 — Initial AD Exploitation

### Concept

Windows uses **LLMNR and NBT-NS** when DNS resolution fails.

An attacker can **poison these broadcasts** to impersonate legitimate hosts and capture authentication hashes.

### Attack

1. Start network poisoning listener

```
responder -I eth0 -dv
```

1. Victim attempts name resolution.
2. Attacker intercepts request.
3. **NTLM hash is captured.**

### Detection

Indicators:

- LLMNR traffic spikes
- Unusual responder services
- NTLM authentication to unknown hosts

Monitoring tools:

- Windows Event Logs
- Network IDS
- SIEM

### Mitigation

- Disable **LLMNR**
- Disable **NBT-NS**
- Enforce **SMB signing**
- Use **DNS properly configured**

---

# M2 — AD Post Enumeration

Once inside the domain network, the attacker maps:

- Users
- Groups
- Trust relationships
- Privilege escalation paths

### Enumeration Tools

| Tool | Purpose |
| --- | --- |
| BloodHound | Graph analysis of AD permissions |
| PowerView | PowerShell AD enumeration |
| RPCClient | SMB-based queries |
| NetExec | SMB / LDAP enumeration |
| ADRecon | Automated domain audit |

### Example Enumeration

User discovery:

```
rpcclient -U "" <DC_IP>
enumdomusers
```

LDAP enumeration:

```
nxc ldap <target>
```

### Detection

Indicators:

- Large LDAP queries
- High volume SMB enumeration
- BloodHound data collection

### Mitigation

- Monitor LDAP queries
- Restrict anonymous enumeration
- Harden service accounts

---

# M3 — DACL Abuse

Active Directory objects are controlled through **Access Control Lists (ACL)**.

If misconfigured, an attacker can gain control of privileged accounts.

### Common Abuses

| Permission | Impact |
| --- | --- |
| GenericAll | Full control over object |
| GenericWrite | Modify attributes |
| WriteDACL | Change object permissions |
| WriteOwner | Take ownership |
| ForceChangePassword | Reset user password |

### Example Attack

If attacker has **GenericWrite** on a user:

```
Change user password
Add user to privileged group
Modify attributes
```

### Detection

- Changes to ACLs
- Unexpected privilege assignments
- Event ID monitoring

### Mitigation

- Review ACL permissions
- Monitor sensitive object changes
- Use **least privilege model**

---

# M4 — Abusing Kerberos

Kerberos is the primary authentication protocol in Active Directory.

Misconfigurations allow attackers to extract **service ticket hashes** and crack them offline.

### Key Attacks

| Attack | Description |
| --- | --- |
| AS-REP Roasting | Extract hash from users without pre-auth |
| Kerberoasting | Crack service account passwords |
| Timeroasting | Exploit time-based authentication |
| Kerberos brute force | Guess weak passwords |

### Kerberoasting Example

1. Request service ticket.
2. Extract encrypted hash.
3. Crack offline.

Example tool usage:

```
GetUserSPNs.py
```

### Detection

Indicators:

- Large number of Kerberos service requests
- Ticket requests from unusual hosts

### Mitigation

- Strong service account passwords
- Use **Group Managed Service Accounts (gMSA)**
- Monitor Kerberos anomalies

---

# M5 — Credential Dumping

After privilege escalation, attackers extract credentials from system databases.

### Targets

| Source | Data |
| --- | --- |
| SAM | Local account hashes |
| LSASS | Memory credentials |
| NTDS.dit | Domain password database |
| Registry Hives | System keys |
| Domain Cache | Cached credentials |

### Major Attacks

- NTDS.dit extraction
- SAM dumping
- DCSync
- LAPS password retrieval
- gMSA credential extraction

### DCSync

Attackers impersonate a Domain Controller and request password hashes.

Example:

```
secretsdump.py
```

### Detection

- Replication requests from non-DC hosts
- Suspicious LSASS access
- NTDS database reads

### Mitigation

- Restrict replication privileges
- Monitor DC replication
- Protect LSASS memory

---

# M6 — Kerberos Ticket Attacks

Kerberos tickets can be forged or reused to maintain unauthorized access.

### Major Techniques

| Attack | Description |
| --- | --- |
| Golden Ticket | Forged TGT using KRBTGT hash |
| Silver Ticket | Forged service ticket |
| Diamond Ticket | Modified ticket from valid TGT |
| Sapphire Ticket | Advanced forged ticket |
| Pass-the-Ticket | Reuse stolen Kerberos ticket |

### Golden Ticket

Requires:

- **KRBTGT hash**
- Domain SID
- Domain name

Attackers generate a fake **TGT** granting full domain access.

### Detection

- Long ticket lifetimes
- Abnormal Kerberos activity
- Ticket usage without authentication

### Mitigation

- Reset **KRBTGT account twice**
- Monitor Kerberos ticket anomalies
- Implement strict logging

---

# Project Structure

```
AD-Attack-Lab
│
├── M1-Initial-Access
│
├── M2-Enumeration
│
├── M3-DACL-Abuse
│
├── M4-Kerberos-Abuse
│
├── M5-Credential-Dumping
│
├── M6-Kerberos-Tickets
│
└── README.md
```

---

# Learning Outcomes

This project demonstrates practical understanding of:

- Active Directory internals
- Attack path modeling
- Kerberos authentication
- Credential theft techniques
- Detection engineering
- Blue team mitigation strategies

---

# Disclaimer

This repository is intended **strictly for educational and defensive security research**.

All demonstrations were performed in a **controlled lab environment**.

Do not perform these techniques on systems without explicit authorization.
