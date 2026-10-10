# [XMPP - Authentication](https://www.root-me.org/en/Challenges/Network/XMPP-authentication-197)

***Statement***

***Spying on the user during an authentication phase, it seems they reused part of their login as part of their password. Find the user's password in this XMPP session capture. The flag is the SHA-1 hash of the password.***

- This is a network challenge from Root-Me. The challenge description says we have to analyse a PCAP file to recover the user's password.
- Download the attachment, and we're ready to go.

- Let's open the PCAP file in Wireshark.
- XMPP stands for Extensible Messaging and Presence Protocol. It is an open protocol for exchanging messages, presence information, and other structured data. It was historically associated with Jabber and is used for instant messaging, collaboration, and machine-to-machine communication.
- **There are three main communication patterns:**
	- Client-to-server (C2S): A client connects to its XMPP server to authenticate and send messages.
	- Server-to-server (S2S): Servers communicate with each other when users belong to different domains.
	- Client-to-client messaging: Users exchange messages logically, but the traffic is normally routed through servers rather than directly between their devices.

- SASL is the framework XMPP uses to authenticate a connection.
	- PLAIN — transmits credentials in a reversible encoding; it requires a properly protected channel, such as TLS.
	- SCRAM-SHA-256 — a challenge-response password authentication mechanism, where supported.
	- SCRAM-SHA-1 — an older SCRAM mechanism that uses SHA-1 and is still found in some deployments.
	- EXTERNAL — can use an identity established by an external authentication mechanism, such as a TLS client certificate.

![screenshot](../Data/xmpp1.png)

- From the screenshot above, we can confirm that the client is using the `SCRAM-SHA-1` mechanism for authentication.

- **SCRAM stands for Salted Challenge Response Authentication Mechanism. SHA-1 is the hash function used by this particular variant.**
- SCRAM-SHA-1 lets a client prove that it knows a password without sending the password itself to the server during authentication.
- The client and server perform a challenge-response exchange.

![screenshot](../data/xmpp2.png)

- Let's try to understand the authentication flow using our PCAP file.

![screenshot](../data/xmpp3.png)

- The authentication exchange begins when the client at `192.168.0.10` sends an initial authentication message to the server. If we expand the packet under the XMPP protocol in Wireshark, we can find a Base64-encoded string in the CDATA section.

![screenshot](../data/xmpp4.png)

- Decoding the Base64 string gives us:
```
n,,n=koma_test,r=hydra
```
- `n,,` — The GS2 header, indicating that no channel binding is being used and no authorization identity has been provided.
- `n=koma_test` — The username, encoded using SCRAM's escaping rules.
- `r=...` — The client-generated nonce.

- The server then responds with a `CHALLENGE` packet.

![screenshot](../data/xmpp4.png)

- The server generates its own nonce contribution, appends it to the client's nonce, and sends the combined nonce, a salt, and an iteration count.
```
r=hydraFe3A1scL7C0jtKsm+kcg96MWg769FuRu,s=kM6lTjjnZW4F8WLboyagcA==,i=4096
```
- `r` — The combined nonce, which must begin with the client's original nonce.
- `s` — The Base64-encoded salt used in password derivation.
- `i` — The number of iterations used by the password-based key derivation function (PBKDF2).

- Now let's look at the client-to-server response.

```
c=biws,r=hydraFe3A1scL7C0jtKsm+kcg96MWg769FuRu,p=mZU2Qekd8JR7ybCtb3hnJMGEfIg=
```
- `c=biws` — The Base64-encoded GS2 header `n,,`, indicating that no channel binding is being used.
- `r=...` — The combined nonce from the server's challenge. It must match the combined nonce exactly.
- `p=...` — The client's proof, calculated using password-derived key material and the authentication transcript.

- The server verifies the client's proof. If it is valid, the server sends its own proof in the server-final message. The client must verify that proof as well before the authentication exchange can be considered successful.

```
v=YQlegvbEwDo2o60YiK2iAkYyPKE=
```
- `v` — The server's proof (server signature), which the client verifies using the expected signature calculated from its derived server key and the authentication transcript.

- You can see the visual diagram below to understand the complete flow.
![screenshot](../data/xmpp5.png)

- Now that we have gathered the information we need, we can write a Python script to perform an offline password-cracking attempt.
```
#!/usr/bin/env python3

import base64
import hashlib
import hmac
import itertools
import sys

charset = "_abcdefghijklmnopqrstuvwxyz"

initial_message = "n=koma_test,r=hydra"

server_first_message = (
    "r=hydraFe3A1scL7C0jtKsm+kcg96MWg769FuRu,"
    "s=kM6lTjjnZW4F8WLboyagcA==,i=4096"
)

server_final_message_compare = "v=YQlegvbEwDo2o60YiK2iAkYyPKE="

# Extract the server nonce, salt, and iteration count
parts = dict(
    field.split("=", 1)
    for field in server_first_message.split(",")
)

r = parts["r"]
s = parts["s"]
iterations = int(parts["i"])

salt = base64.b64decode(s)

for passlen in range(1, 3):
    print(f"Testing password suffix length {passlen}")

    for k in itertools.permutations(charset, passlen):
        password = "koma" + "".join(k)

        # Derive the salted password using PBKDF2-HMAC-SHA-1
        salt_password = hashlib.pbkdf2_hmac(
            "sha1",
            password.encode(),
            salt,
            iterations
        )

        # Construct the SCRAM authentication transcript
        client_final_message_bare = f"c=biws,r={r}"

        auth_message = (
            initial_message + ","
            + server_first_message + ","
            + client_final_message_bare
        ).encode()

        # Calculate the expected server signature
        server_key = hmac.new(
            salt_password,
            b"Server Key",
            hashlib.sha1
        ).digest()

        server_signature = hmac.new(
            server_key,
            auth_message,
            hashlib.sha1
        ).digest()

        calculated_server_proof = (
            "v=" + base64.b64encode(server_signature).decode()
        )

        # Compare the calculated proof with the captured proof
        if hmac.compare_digest(
            calculated_server_proof,
            server_final_message_compare
        ):
            print("[+] Password found:", password)

            password_hash = hashlib.sha1(
                password.encode()
            ).hexdigest()

            print("[+] SHA-1 hash:", password_hash)
            sys.exit(0)

print("[-] No matching password found.")
```

```
python3 xmpp_crack.py
Testing password suffix length 1
Testing password suffix length 2
[+] Password found: komato
[+] SHA-1 hash: 8f653e36780d5dd7fe7b07e64ff6fde48c3199cb
```

**password: 8f653e36780d5dd7fe7b07e64ff6fde48c3199cb**