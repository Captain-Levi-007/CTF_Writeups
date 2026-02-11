# [Command & Control - level 6](https://www.root-me.org/en/Challenges/Forensic/Command-Control-level-6)

***Statement
Berthier, before blocking any of the malware’s traffic on our firewalls, we need to make sure we found all its C&C. This will let us know if there are other infected hosts on our network and be certain we’ve locked the attackers out. That’s it Berthier, we’re almost there, reverse this malware!
The validation password is a fully qualified domain name : hote.domaine.tld
The uncompressed memory dump md5 hash is e3a902d4d44e0f7bd9cb29865e0a15de
NB : This challenge require the clearance of the level 3.***

- Lets Download the attachment . and unzip it

```
tar -xvjf ch2.tbz2                                                                               py_venv
ch2.dmp
```
- Well from the [Comand & Control level3](https://github.com/Captain-Levi-007/CTF_Writeups/blob/main/Root-Me/CC_level3.md) We find that the iexplore.exe is the malisious process. 
- And the pid od te precess is 2772.  
- First i want to check what dlls and executable are being utilized by the process 
- To list all dlls and main executable i am goinf to use tool called  **dlllist**
- **dlllist** Lists all DLLs (and the main executable) loaded inside a specific process.

```
vol -f ch2.dmp windows.dlllist --pid 2772                                                        py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
PID	Process	Base	Size	Name	Path	LoadCount	LoadTime	File output

2772	iexplore.exe	0x400000	0x6000	iexplore.exe	C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe	-1	N/A	Disabled
2772	iexplore.exe	0x77660000	0x13c000	ntdll.dll	C:\Windows\SYSTEM32\ntdll.dll	-1	N/A	Disabled
2772	iexplore.exe	0x70e70000	0x3c000	snxhk.dll	C:\Program Files\AVAST Software\Avast\snxhk.dll	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x77480000	0xd4000	KERNEL32.dll	C:\Windows\system32\KERNEL32.dll	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x75920000	0x4a000	KERNELBASE.dll	C:\Windows\system32\KERNELBASE.dll	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x76a60000	0xac000	msvcrt.dll	C:\Windows\system32\msvcrt.dll	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x777a0000	0x35000	WS2_32.DLL	C:\Windows\system32\WS2_32.DLL	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x76c10000	0xa1000	RPCRT4.dll	C:\Windows\system32\RPCRT4.dll	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x77880000	0x6000	NSI.dll	C:\Windows\system32\NSI.dll	-1	2013-01-12 16:40:34.000000 UTC	Disabled
2772	iexplore.exe	0x751f0000	0x3c000	mswsock.dll	C:\Windows\system32\mswsock.dll	4	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x76990000	0xc9000	user32.dll	C:\Windows\system32\user32.dll	24	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x75ab0000	0x4e000	GDI32.dll	C:\Windows\system32\GDI32.dll	21	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x76980000	0xa000	LPK.dll	C:\Windows\system32\LPK.dll	6	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x777e0000	0x9d000	USP10.dll	C:\Windows\system32\USP10.dll	6	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x75b00000	0x1f000	IMM32.DLL	C:\Windows\system32\IMM32.DLL	2	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x77210000	0xcc000	MSCTF.dll	C:\Windows\system32\MSCTF.dll	1	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x750b0000	0x44000	DNSAPI.dll	C:\Windows\system32\DNSAPI.dll	2	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x76ec0000	0x19000	sechost.dll	C:\Windows\SYSTEM32\sechost.dll	2	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x73fa0000	0x1c000	IPHLPAPI.DLL	C:\Windows\system32\IPHLPAPI.DLL	1	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x73f80000	0x7000	WINNSI.DLL	C:\Windows\system32\WINNSI.DLL	1	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x727d0000	0x6000	rasadhlp.dll	C:\Windows\system32\rasadhlp.dll	1	2013-01-12 16:55:34.000000 UTC	Disabled
2772	iexplore.exe	0x74d40000	0x5000	wshtcpip.dll	C:\Windows\System32\wshtcpip.dll	1	2013-01-12 16:55:49.000000 UTC	Disabled
2772	iexplore.exe	0x747b0000	0x10000	NLAapi.dll	C:\Windows\system32\NLAapi.dll	1	2013-01-12 16:55:49.000000 UTC	Disabled
2772	iexplore.exe	0x72810000	0x8000	winrnr.dll	C:\Windows\System32\winrnr.dll	1	2013-01-12 16:55:49.000000 UTC	Disabled
2772	iexplore.exe	0x72800000	0x10000	napinsp.dll	C:\Windows\system32\napinsp.dll	1	2013-01-12 16:55:49.000000 UTC	Disabled
2772	iexplore.exe	0x727e0000	0x12000	pnrpnsp.dll	C:\Windows\system32\pnrpnsp.dll	2	2013-01-12 16:55:49.000000 UTC	Disabled
2772	iexplore.exe	0x73d80000	0x38000	fwpuclnt.dll	C:\Windows\System32\fwpuclnt.dll	1	2013-01-12 16:55:49.000000 UTC	Disabled
2772	iexplore.exe	0x756b0000	0x4b000	apphelp.dll	C:\Windows\system32\apphelp.dll	-1	2013-01-12 16:55:49.000000 UTC	Disabled
```
- See the main executable is iexplore.exe and remeber the base **0x400000** we need this for later.
- Now we are going to use another plugin called **pedump**
- **pedump** Reconstructs a PE (Portable Executable) image directly from process memory.

```
vol -f ch2.dmp -o dumps windows.pedump --pid 2772 --base 0x400000   py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
PID	Process	File output

2772	iexplore.exe	PE.0x87b6b030.2772.0x400000.dmp
```
- now in dumps directory we have "PE.0x87b6b030.2772.0x400000.dmp" file .
```
file PE.0x87b6b030.2772.0x400000.dmp                                                                                                                      py_venv
PE.0x87b6b030.2772.0x400000.dmp: PE32 executable for MS Windows 4.00 (GUI), Intel i386 (stripped to external PDB), 5 sections
```
- I am goinf to analyze this file using virustotal . upload the file and in the behaviour ection we can find different domain at Network Communication section.
```
furious.devilslife.com
ns2.wrauzfevvo.com
th1sis.l1k3aK3y.org
whereare.sexy-serbian
y0ug.itisjustluck.com
```

- According to the lab specified format we got top 3 domain . i tried on by one . and found the third one is the answer.
**th1sis.l1k3aK3y.org**