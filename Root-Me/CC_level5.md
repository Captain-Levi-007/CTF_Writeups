# [Command & Control - level 5](https://www.root-me.org/en/Challenges/Forensic/Command-Control-level-5)

***Statement
Berthier, the malware seems to be manually maintained on the workstations. Therefore, it’s likely that the hackers have found all of the computers’ passwords.
Since ACME’s computer fleet seems to be up to date, it’s probably only due to password weakness. John, the system administrator, doesn’t believe you. Prove him wrong!
Find john password.
The uncompressed memory dump md5 hash is e3a902d4d44e0f7bd9cb29865e0a15de***

- Download the attachment and unzip it
```
 tar -xvjf ch2.tbz2 
ch2.dmp
```

- We are going to use the volatility framework to analyze this memory dump. For more information, read my other write-ups on Command & Control.
- To solve this, we have to dump the password  hashes from the memory dump. 
- We are going to use two plugins, **hashdump & lsadump**, both of which are used to extract password hashes from the memory.

```
vol -f ch2.dmp windows.hashdump                                                     py_venv

Volatility 3 Framework 2.27.0

User	rid	lmhash	nthash
Administrator	500	aad3b435b51404eeaad3b435b51404ee	31d6cfe0d16ae931b73c59d7e0c089c0
Guest	501	aad3b435b51404eeaad3b435b51404ee	31d6cfe0d16ae931b73c59d7e0c089c0
John Doe	1000	aad3b435b51404eeaad3b435b51404ee	b9f917853e3dbf6e6831ecce60725930
```
- **hashdump** plugin successfully dumped the NTLM hashes from the memory. 
- Lsadump failed, we were still able to obtain the required information.
- To solve the lab, we need John's password 
- I am going to use [crackstation](https://crackstation.net/) to crack john's NTLM hash **b9f917853e3dbf6e6831ecce60725930**

**passw0rd**
