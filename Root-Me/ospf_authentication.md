# [OSPF-AUTHENTICATION](https://www.root-me.org/en/Challenges/Network/OSPF-Authentication)

**Statement**
***You are hired to test the security of a company’s network.
You quickly manage to capture some OSPF packets.***

***Find the OSPF authentication key to continue your investigation!***

***Good luck!***

***sha256sum: 9CF709C4984B7EB6426A6B4B9B3B35604055B6040CCD46B30DF785D7D21F28AB***

- OK! Let's download the attachment. And unzip it.

```
 unzip ch21.zip 
Archive:  ch21.zip
  inflating: ospf_authentication_hash.pcapng 
```
- Now we have a pcapng file. The file name says it is an OSPF authentication hash. 
- ***Note: (PCAP Next Generation) file is the default, modern format used to store captured network traffic.***
- Open it with Wireshark. Before going any further, let's understand what OSPF authentication is.
- [OSPF](https://www.networkacademy.io/ccna/ospf/what-is-ospf) is a link-state routing protocol that uses a mathematical algorithm to determine the best path to every IP destination in the network.
- [OSPF (Open Shortest Path First) Authentication](https://www.networkacademy.io/ccna/ospf/ospf-plain-text-authentication) is a security mechanism used to protect routing infrastructure. Without authentication, an attacker could easily connect a rogue router to your network, inject false routing information, cause a Denial of Service (DoS) by creating routing loops, or redirect traffic for interception.
- By enabling authentication, OSPF routers will only form neighbor relationships (adjacencies) and accept routing updates from other routers that share the same pre-configured secret key.

```
Types of OSPF Authentication:
	- OSPF supports three primary types of authentication:
			1. Type 0: Null Authentication (No Authentication):
						
						How it works: This is the default setting. The OSPF header includes an authentication type field set to 0, and no password or security check is performed.

			2. Type 1: Plain Text (Simple) Authentication:

						How it works: A clear-text password (up to 8 characters) is configured on the routers. This password is sent directly inside the OSPF packet header.

			3. Type 2: Cryptographic (MD5 or SHA) Authentication:

						How it works: The actual password is never sent over the wire. Instead, the router uses a hashing algorithm (traditionally MD5, though modern networks use SHA) combined with a Key ID and the OSPF packet content to generate a unique hash value.
```
- Now, with the above information lets dig into the pcapng file and find our secret key. 
- Extracting the MD5 hashes from the pcapng file.
```
tshark -r ospf_authentication_hash.pcapng -T fields -e ospf.auth.crypt.data 

debe4e93b093ade8a8bc34302c192ced
5445df30fe3d20bf23ecf26c2e531387
ed964b2ac353eb6b5431d3251a1d2074
91276c153696d2929edaefc7c2131859
c0575e191ba012bd9cd7de3c6bda49c6
0844d60b1f97b377afdf26901c0eee8e
e3ff7611705e1e39017d19084efbca1f
f1c9059ed03e82547bf45b9755223ac1
bde76e1f3eddfe8c7d4f8a32c12300da
2c5764c41f15333ad5e6509a0623aeef
c0a4b500effed0bd3d537db6c3295a2f
59e5abdc9e68404d9cf6bab427d420a4
08cbaa952e00d202a796f8fa76a2982b
ca39bac632801c8857650e8a28a35515
```
- But this info is not enough. A cracking tool also needs the packet contents (or a specially formatted hash containing the packet data).
- I'm going to use ettercap to extract hashes.
```
 ettercap -Tqr ospf_authentication_hash.pcapng 

ettercap 0.8.4.1 copyright 2001-2026 Ettercap Development Team

Reading from ospf_authentication_hash.pcapng
Libnet failed IPv4 initialization. Don't send IPv4 packets.
Libnet failed IPv6 initialization. Don't send IPv6 packets.
This product includes GeoLite2 Data created by MaxMind, available from https://www.maxmind.com/.
  34 plugins
  42 protocol dissectors
  56 ports monitored
38702 mac vendor fingerprint
1766 tcp OS fingerprint
2182 known services
Lua: no scripts were specified, not starting up!

Starting Unified sniffing...

OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8a4fffffffc000a1201000000280c0000020c00000103030303$debe4e93b093ade8a8bc34302c192ced
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8a7fffffffc000a1201000000280c0000020c00000102020202$5445df30fe3d20bf23ecf26c2e531387
OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8aefffffffc000a1201000000280c0000020c00000103030303$ed964b2ac353eb6b5431d3251a1d2074
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8b0fffffffc000a1201000000280c0000020c00000102020202$91276c153696d2929edaefc7c2131859
OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8b7fffffffc000a1201000000280c0000020c00000103030303$c0575e191ba012bd9cd7de3c6bda49c6
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8b9fffffffc000a1201000000280c0000020c00000102020202$0844d60b1f97b377afdf26901c0eee8e
OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8c1fffffffc000a1201000000280c0000020c00000103030303$e3ff7611705e1e39017d19084efbca1f
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8c2fffffffc000a1201000000280c0000020c00000102020202$f1c9059ed03e82547bf45b9755223ac1
OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8cafffffffc000a1201000000280c0000020c00000103030303$bde76e1f3eddfe8c7d4f8a32c12300da
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8ccfffffffc000a1201000000280c0000020c00000102020202$2c5764c41f15333ad5e6509a0623aeef
OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8d4fffffffc000a1201000000280c0000020c00000103030303$c0a4b500effed0bd3d537db6c3295a2f
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8d5fffffffc000a1201000000280c0000020c00000102020202$59e5abdc9e68404d9cf6bab427d420a4
OSPF-224.0.0.5-0:$netmd5$0201003002020202000000000000000200000a103c7ec8ddfffffffc000a1201000000280c0000020c00000103030303$08cbaa952e00d202a796f8fa76a2982b
OSPF-224.0.0.5-0:$netmd5$0201003003030303000000000000000200000a103c7ec8defffffffc000a1201000000280c0000020c00000102020202$ca39bac632801c8857650e8a28a35515


Capture file read completely, please exit at your convenience.


End of dump file...

Terminating ettercap...
Lua cleanup complete!
Unified sniffing was stopped.
```
- T → Text mode (run in the terminal instead of the graphical interface)
- q → Quiet mode (reduce the amount of output)
- r file → Read packets from a pcap/pcapng file instead of capturing live traffic

- Remove the "OSPF-224.0.0.5-0:" part and save the rest to a file.

```
cut -d ":" -f 2 raw_hash.txt                      
$netmd5$0201003002020202000000000000000200000a103c7ec8a4fffffffc000a1201000000280c0000020c00000103030303$debe4e93b093ade8a8bc34302c192ced
$netmd5$0201003003030303000000000000000200000a103c7ec8a7fffffffc000a1201000000280c0000020c00000102020202$5445df30fe3d20bf23ecf26c2e531387
$netmd5$0201003002020202000000000000000200000a103c7ec8aefffffffc000a1201000000280c0000020c00000103030303$ed964b2ac353eb6b5431d3251a1d2074
$netmd5$0201003003030303000000000000000200000a103c7ec8b0fffffffc000a1201000000280c0000020c00000102020202$91276c153696d2929edaefc7c2131859
$netmd5$0201003002020202000000000000000200000a103c7ec8b7fffffffc000a1201000000280c0000020c00000103030303$c0575e191ba012bd9cd7de3c6bda49c6
$netmd5$0201003003030303000000000000000200000a103c7ec8b9fffffffc000a1201000000280c0000020c00000102020202$0844d60b1f97b377afdf26901c0eee8e
$netmd5$0201003002020202000000000000000200000a103c7ec8c1fffffffc000a1201000000280c0000020c00000103030303$e3ff7611705e1e39017d19084efbca1f
$netmd5$0201003003030303000000000000000200000a103c7ec8c2fffffffc000a1201000000280c0000020c00000102020202$f1c9059ed03e82547bf45b9755223ac1
$netmd5$0201003002020202000000000000000200000a103c7ec8cafffffffc000a1201000000280c0000020c00000103030303$bde76e1f3eddfe8c7d4f8a32c12300da
$netmd5$0201003003030303000000000000000200000a103c7ec8ccfffffffc000a1201000000280c0000020c00000102020202$2c5764c41f15333ad5e6509a0623aeef
$netmd5$0201003002020202000000000000000200000a103c7ec8d4fffffffc000a1201000000280c0000020c00000103030303$c0a4b500effed0bd3d537db6c3295a2f
$netmd5$0201003003030303000000000000000200000a103c7ec8d5fffffffc000a1201000000280c0000020c00000102020202$59e5abdc9e68404d9cf6bab427d420a4
$netmd5$0201003002020202000000000000000200000a103c7ec8ddfffffffc000a1201000000280c0000020c00000103030303$08cbaa952e00d202a796f8fa76a2982b
$netmd5$0201003003030303000000000000000200000a103c7ec8defffffffc000a1201000000280c0000020c00000102020202$ca39bac632801c8857650e8a28a35515
```

```
cut -d ":" -f 2 raw_hash.txt  > ospf.hash
```

```
john ospf.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 14 password hashes with 14 different salts (net-md5, "Keyed MD5" RIPv2, OSPF, BGP, SNMPv2 [MD5 32/64 or dynamic_39])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
#10pokemonmaster (?)     
14g 0:00:00:21 DONE (2026-07-06 13:36) 0.6430g/s 652861p/s 9140Kc/s 9140KC/s #18#16torito..!lstpa88!
Use the "--show --format=net-md5" options to display all of the cracked passwords reliably
Session completed. 
```
- The secret key is **#10pokemonmaster**
