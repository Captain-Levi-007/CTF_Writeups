
# [POP - APOP](https://www.root-me.org/en/Challenges/Network/POP-APOP)

- The lab description mentions **Secured Authentication**.
- Our objective is to find the user's password from the captured network traffic.

- Download the attached ZIP file and extract it.

```bash
unzip ch23.zip

Archive:  ch23.zip
  inflating: ch23.pcapng
```

- Open the PCAP file using Wireshark.
- As the challenge suggests, this involves POP (Post Office Protocol) authentication. I want to filter the traffic to display only packets related to port 110.

![screenshot](../data/pop2.png)

- If you look at packet number 264, you can see the client sending the following message to the server:

```text
C: APOP bsmith 4ddd4137b84ff2db7291b568289717f0
```

- This reveals that the server is using APOP authentication.
- **APOP (Authenticated Post Office Protocol)** is an extension of the POP3 email retrieval protocol that protects user passwords from being transmitted in plaintext over the network.

**Workflow of APOP**

- **Challenge-Response Mechanism:** When you connect to an APOP-enabled email server, the server sends a unique, timestamp-based challenge string (nonce) in its greeting banner.
- **MD5 Hashing:** The email client combines the challenge with the user's secret password and computes an MD5 cryptographic hash.
- **Digest Transmission:** Instead of sending the actual password, the client sends only the resulting hash digest to the server for verification.

- Now, let's understand the communication flow. The first three packets (26, 27, and 28) represent the TCP three-way handshake. The client (`192.168.0.112`) establishes a connection with the POP server (`167.114.129.140`) on port 110.

- After establishing the TCP connection, the server greets the client and sends a challenge.

```text
S: +OK Hello little hackers. <1755.1.5f403625.BcWGgpKzUPRC8vscWn0wuA==@vps-7e2f5a72>
```

- `S:` indicates that the message is coming from the server.
- `+OK` indicates that the server is ready.
- `<1755.1.5f403625.BcWGgpKzUPRC8vscWn0wuA==@vps-7e2f5a72>` is the challenge sent by the server to the client.

- The client then responds with:

```text
C: APOP bsmith 4ddd4137b84ff2db7291b568289717f0
```

- `C:` indicates that the message is coming from the client.
- `APOP` specifies the authentication method.
- `bsmith` is the username.
- `4ddd4137b84ff2db7291b568289717f0` is the MD5 digest generated using the challenge and the user's password.

- Now, let's try to crack the hash offline. I found the corresponding hash format and Hashcat mode for this authentication mechanism.

![screenshot](../data/pop3.png)

- However, I'm using Python to perform the offline password verification.

```python
import hashlib


challenge = "<1755.1.5f403625.BcWGgpKzUPRC8vscWn0wuA==@vps-7e2f5a72>"
hash = "4ddd4137b84ff2db7291b568289717f0"

wordlist = "/usr/share/wordlists/rockyou.txt"

with open(wordlist, "r", encoding="latin-1") as file:
    for line in file:
        password = line.strip()

        if not password:
            continue
        
        candidate = challenge + password
        #print(candidate)
        digest = hashlib.md5(candidate.encode()).hexdigest()

        if digest == hash:
            print(f"[+] Found password: {password}")
            break
```

```text
[+] Found password: 100%popprincess
```

**Password: `100%popprincess`**
