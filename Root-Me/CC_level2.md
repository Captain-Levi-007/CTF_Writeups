# [Command & Control - level 2](https://www.root-me.org/en/Challenges/Forensic/Command-Control-level-2)

***Congratulations Berthier, thanks to your help the computer has been identified. You have requested a memory dump but before starting your analysis you wanted to take a look at the antivirus’ logs. Unfortunately, you forgot to write down the workstation’s hostname. But since you have its memory dump you should be able to get it back!
The validation flag is the workstation’s hostname.***

- Download the attachment
```
ls                                                                                                                                                                                                          py_venv
ch2.tbz2

file ch2.tbz2                                                                                                                                                                                               py_venv
ch2.tbz2: bzip2 compressed data, block size = 900k
```
- It is a bzip compressed data let unzip it 
```
tar -xvjf ch2.tbz2                                                                                                                                                                                          py_venv
ch2.dmp
```
- Now we have memory dump file.
- A memory dump is a snapshot of a system’s RAM at a specific moment.
- RAM contains:
	- Running processes
	- Passwords
	- Encryption keys
	- Network connections
	- Injected malware
	- Command history
	- Decrypted data that NEVER touches disk
- Lets analyze it. I am going to use volatility tool, which is a awsome tool analyze volatile memory. 

***The Volatility Framework is a completely open collection of tools,
implemented in Python under the GNU General Public License, for the
extraction of digital artifacts from volatile memory (RAM) samples.***

- Installation 
```
pip install volatility3
```
- Now all set lets go!

- Well first i tried to identify the OS
```
vol -f ch2.dmp banner                                                                                                                                                                                       py_venv
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                  
Offset	Banner
```
- Used banner to extract banner but we got nothing . which means the momoey dump is likely form windos os. 
- lets try to find info about the memory dump.

```
vol -f ch2.dmp windows.info                                                                                                                                                                            7s  py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                                                                                              
Variable	Value

Kernel Base	0x82801000
DTB	0x185000
Symbols	file:///home/light/py_venv/lib/python3.13/site-packages/volatility3/symbols/windows/ntkrpamp.pdb/5B308B4ED6464159B87117C711E7340C-2.json.xz
Is64Bit	False
IsPAE	True
layer_name	0 WindowsIntelPAE
memory_layer	1 FileLayer
KdDebuggerDataBlock	0x82929be8
NTBuildLab	7600.16385.x86fre.win7_rtm.09071
CSDVersion	0
KdVersionBlock	0x82929bc0
Major/Minor	15.7600
MachineType	332
KeNumberProcessors	1
SystemTime	2013-01-12 16:59:18+00:00
NtSystemRoot	C:\Windows
NtProductType	NtProductWinNt
NtMajorVersion	6
NtMinorVersion	1
PE MajorOperatingSystemVersion	6
PE MinorOperatingSystemVersion	1
PE Machine	332
PE TimeDateStamp	Mon Jul 13 23:15:19 2009
```

- This line clearly tells that the os is Windows7 **NTBuildLab	7600.16385.x86fre.win7_rtm.09071** 
- wll to solve our lab we need to know the hostname . for that i am going to read windows registry
```
vol -f ch2.dmp windows.registry.hivelist                                                                                                                                                                    py_venv


Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
Offset	FileFullPath	File output

0x8b20c008		Disabled
0x8b21c008	\REGISTRY\MACHINE\SYSTEM	Disabled
0x8b23c008	\REGISTRY\MACHINE\HARDWARE	Disabled
0x8ee66008	\Device\HarddiskVolume1\Boot\BCD	Disabled
0x8ee66740	\SystemRoot\System32\Config\SOFTWARE	Disabled
0x90cab9d0	\SystemRoot\System32\Config\DEFAULT	Disabled
0x9670e9d0	\??\C:\Users\John Doe\ntuser.dat	Disabled
0x9670f9d0	\??\C:\Users\John Doe\AppData\Local\Microsoft\Windows\UsrClass.dat	Disabled
0x9aad6148	\SystemRoot\System32\Config\SAM	Disabled
0x9ab25008	\SystemRoot\System32\Config\SECURITY	Disabled
0x9aba79d0	\??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT	Disabled
0x9abb1720	\??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT	Disabled
```

- Copy the offset value of **\REGISTRY\MACHINE\SYSTEM**
- I asked ChatGPT to give me the location in the registry hive where the hostname resides.
- It given couple of locaitons i got the hostname value in the below loaction. 
- we got the path **ControlSet001\Control\ComputerName\ActiveComputerName**

```
vol -f ch2.dmp windows.registry.printkey --offset 0x8b21c008 --key "ControlSet001\Control\ComputerName\ActiveComputerName"                                                                                  py_venv
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
Last Write Time	Hive Offset	Type	Key	Name	Data	Volatile

2013-01-12 16:38:14.000000 UTC	0x8b21c008	REG_SZ	\REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\ComputerName\ActiveComputerName	ComputerName	WIN-ETSA91RKCFP	True
```
**Hostname: WIN-ETSA91RKCFP**