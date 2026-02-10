# [Command & Control - level 3](https://www.root-me.org/en/Challenges/Forensic/Command-Control-level-3)

***Statement
Berthier, the antivirus software didn’t find anything. It’s up to you now. Try to find the malware in the memory dump. The validation flag is the md5 checksum of the full path of the executable.
The uncompressed memory dump md5 hash is e3a902d4d44e0f7bd9cb29865e0a15de***

## Solution

- Let's download the attachment and unzip it.
```
  ~/Root_me/Forensics ❯ tar -xjf ch2.tbz2    
  ~/Root_me/Forensics ❯ ls                                                                                                                                                                                                             25s
ch2.dmp  ch2.tbz2
```
- What is a memory dump? **A memory dump is a snapshot of RAM at a moment in time.**
- The core questions we should ask ourselves when we analyze a memory dump for finding malware.

	- What OS is this memory from?
	- What processes were running?	
	- Do any processes look abnormal?
	- Is any code hidden or injected?
	- Is there evidence of execution or control?
	- Can we prove malicious behavior?
- Let's go one by one.
- Indentifying os and profile of the memory dump file 
```
 vol -f ch2.dmp windows.info                                                                                                                                                                                 py_venv
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
- From the above output, we can see the operating system is  **Windows 7 RTM (Build 7600) 32-bit (x86)**
- Well, malware always runs as a process, so I want to list all the active processes 
```
  ~/Root_me/Forensics ❯vol -f ch2.dmp windows.pslist                                                                                                                                                                               py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
PID	PPID	ImageFileName	Offset(V)	Threads	Handles	SessionId	Wow64	CreateTime	ExitTime	File output

4	0	System	0x87978b78	103	3257	N/A	False	2013-01-12 16:38:09.000000 UTC	N/A	Disabled
308	4	smss.exe	0x88c3ed40	2	29	N/A	False	2013-01-12 16:38:09.000000 UTC	N/A	Disabled
404	396	csrss.exe	0x8929fd40	9	469	0	False	2013-01-12 16:38:14.000000 UTC	N/A	Disabled
456	396	wininit.exe	0x892ac2b8	3	77	0	False	2013-01-12 16:38:14.000000 UTC	N/A	Disabled
468	448	csrss.exe	0x88d03a00	10	471	1	False	2013-01-12 16:38:14.000000 UTC	N/A	Disabled
500	448	winlogon.exe	0x892ced40	3	111	1	False	2013-01-12 16:38:14.000000 UTC	N/A	Disabled
560	456	services.exe	0x896294c0	6	205	0	False	2013-01-12 16:38:16.000000 UTC	N/A	Disabled
576	456	lsass.exe	0x896427b8	6	566	0	False	2013-01-12 16:38:16.000000 UTC	N/A	Disabled
584	456	lsm.exe	0x8962f7e8	10	142	0	False	2013-01-12 16:38:16.000000 UTC	N/A	Disabled
692	560	svchost.exe	0x8962f030	10	353	0	False	2013-01-12 16:38:21.000000 UTC	N/A	Disabled
764	560	svchost.exe	0x897b5c20	7	263	0	False	2013-01-12 16:38:23.000000 UTC	N/A	Disabled
832	560	svchost.exe	0x89805420	19	435	0	False	2013-01-12 16:38:23.000000 UTC	N/A	Disabled
904	560	svchost.exe	0x89852918	17	409	0	False	2013-01-12 16:38:24.000000 UTC	N/A	Disabled
928	560	svchost.exe	0x8986b030	26	869	0	False	2013-01-12 16:38:24.000000 UTC	N/A	Disabled
1084	560	svchost.exe	0x898911a8	10	257	0	False	2013-01-12 16:38:26.000000 UTC	N/A	Disabled
1172	560	svchost.exe	0x898b2790	15	475	0	False	2013-01-12 16:38:27.000000 UTC	N/A	Disabled
1220	560	AvastSvc.exe	0x898a7868	66	1180	0	False	2013-01-12 16:38:28.000000 UTC	N/A	Disabled
1712	560	spoolsv.exe	0x8a0f9c40	14	338	0	False	2013-01-12 16:38:58.000000 UTC	N/A	Disabled
1748	560	svchost.exe	0x8a102748	18	310	0	False	2013-01-12 16:38:58.000000 UTC	N/A	Disabled
1872	560	sppsvc.exe	0x88cded40	4	143	0	False	2013-01-12 16:39:02.000000 UTC	N/A	Disabled
1968	560	vmtoolsd.exe	0x8a1d84e0	6	220	0	False	2013-01-12 16:39:14.000000 UTC	N/A	Disabled
336	560	wlms.exe	0x9541c7e0	4	45	0	False	2013-01-12 16:39:21.000000 UTC	N/A	Disabled
448	560	VMUpgradeHelpe	0x8a1f5030	4	89	0	False	2013-01-12 16:39:21.000000 UTC	N/A	Disabled
1612	560	TPAutoConnSvc.	0x9542a030	9	135	0	False	2013-01-12 16:39:23.000000 UTC	N/A	Disabled
2352	560	taskhost.exe	0x87ac0620	8	149	1	False	2013-01-12 16:40:24.000000 UTC	N/A	Disabled
2496	904	dwm.exe	0x87ad44d0	5	77	1	False	2013-01-12 16:40:25.000000 UTC	N/A	Disabled
2548	2484	explorer.exe	0x87ac6030	24	766	1	False	2013-01-12 16:40:27.000000 UTC	N/A	Disabled
2568	1612	TPAutoConnect.	0x87ae2880	5	146	1	False	2013-01-12 16:40:28.000000 UTC	N/A	Disabled
2600	468	conhost.exe	0x87a9c288	1	35	1	False	2013-01-12 16:40:28.000000 UTC	N/A	Disabled
2660	2548	VMwareTray.exe	0x87b82438	5	80	1	False	2013-01-12 16:40:29.000000 UTC	N/A	Disabled
2676	2548	VMwareUser.exe	0x87aa9220	8	190	1	False	2013-01-12 16:40:30.000000 UTC	N/A	Disabled
2720	2548	AvastUI.exe	0x87b784b0	14	220	1	False	2013-01-12 16:40:31.000000 UTC	N/A	Disabled
2744	2548	StikyNot.exe	0x898fe8c0	8	135	1	False	2013-01-12 16:40:32.000000 UTC	N/A	Disabled
2772	2548	iexplore.exe	0x87b6b030	2	74	1	False	2013-01-12 16:40:34.000000 UTC	N/A	Disabled
2900	560	SearchIndexer.	0x898fbb18	13	636	0	False	2013-01-12 16:40:38.000000 UTC	N/A	Disabled
3176	560	wmpnetwk.exe	0x87bd35b8	9	240	0	False	2013-01-12 16:40:48.000000 UTC	N/A	Disabled
3352	560	svchost.exe	0x89f3d2c0	9	141	0	False	2013-01-12 16:40:58.000000 UTC	N/A	Disabled
3452	2548	swriter.exe	0x87c6a2a0	1	19	1	False	2013-01-12 16:41:01.000000 UTC	N/A	Disabled
3512	3452	soffice.exe	0x87ba4030	1	28	1	False	2013-01-12 16:41:03.000000 UTC	N/A	Disabled
3556	3544	soffice.bin	0x95483d18	0	-	1	False	2013-01-12 16:41:05.000000 UTC	2013-01-12 16:41:39.000000 UTC	Disabled
3564	3512	soffice.bin	0x87b8ca58	12	400	1	False	2013-01-12 16:41:05.000000 UTC	N/A	Disabled
3624	560	svchost.exe	0x89f1d3e8	14	348	0	False	2013-01-12 16:41:22.000000 UTC	N/A	Disabled
1232	2548	taskmgr.exe	0x95495c18	6	116	1	False	2013-01-12 16:42:29.000000 UTC	N/A	Disabled
3152	2548	cmd.exe	0x87bf7030	1	23	1	False	2013-01-12 16:44:50.000000 UTC	N/A	Disabled
3228	468	conhost.exe	0x87c595b0	2	54	1	False	2013-01-12 16:44:50.000000 UTC	N/A	Disabled
1616	2772	cmd.exe	0x89898030	2	101	1	False	2013-01-12 16:55:49.000000 UTC	N/A	Disabled
2168	468	conhost.exe	0x954826b0	2	49	1	False	2013-01-12 16:55:50.000000 UTC	N/A	Disabled
1136	2548	iexplore.exe	0x9549f678	18	454	1	False	2013-01-12 16:57:44.000000 UTC	N/A	Disabled
3044	1136	iexplore.exe	0x87d4d338	37	937	1	False	2013-01-12 16:57:46.000000 UTC	N/A	Disabled
1720	832	audiodg.exe	0x87c90d40	5	117	0	False	2013-01-12 16:58:11.000000 UTC	N/A	Disabled
3144	3152	winpmem-1.3.1.	0x87cbfd40	1	23	1	False	2013-01-12 16:59:17.000000 UTC	N/A	Disabled
```
- Well, if you observe thoroughly, you can notice a couple of iexplore.exe processes running on the system.  with pid **2772,1136,3044**
- See the process ID with **1616**. cmd.exe is initiated by iexplore.exe. Looks a little odd, right? Let's dig deep.
```
1616	2772	cmd.exe	0x89898030	2	101	1	False	2013-01-12 16:55:49.000000 UTC	N/A	Disabled
```
- lets check the cmdline of that iexplore processes.

```
vol -f ch2.dmp windows.cmdline --pid 2772                                                                                                                                                              6s  py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
PID	Process	Args

2772	iexplore.exe	"C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe" 
```
```
vol -f ch2.dmp windows.cmdline --pid 1136                                                                                                                                                              6s  py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
PID	Process	Args

1136	iexplore.exe	"C:\Program Files\Internet Explorer\iexplore.exe"
```

```
vol -f ch2.dmp windows.cmdline --pid 3044                                                                                                                                                              6s  py_venv

Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
PID	Process	Args

3044	iexplore.exe	"C:\Program Files\Internet Explorer\iexplore.exe" SCODEF:1136 CREDAT:71937
```

- The iexplore.exe with pid 2772 and (1136,3044) are initiated from different locations. 
- lets confirm our suspicion using the **console** plugin.
- The **Windows.consoles** plugin extracts data from Windows Console Host memory.
	- That includes:
	- Commands typed in cmd.exe
	- Commands typed in PowerShell.exe
	- Output shown in the console window
	- Command history even if the window was closed
```
vol -f ch2.dmp windows.consoles 


ConsoleProcess: conhost.exe Pid: 2168
Console: 0x1081c0 CommandHistorySize: 50
HistoryBufferCount: 3 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\system32\cmd.exe
Title: C:\Windows\system32\cmd.exe
AttachedProcess: cmd.exe Pid: 1616 Handle: 0x64
 — — 
CommandHistory: 0x427a60 Application: tcprelay.exe Flags:
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x0
```
- The conhost.exe is associated with cmd.exe and tries to execute tcprelay.exe(a networking utility, often used as a command-line tool, designed to act as a TCP connection forwarder, load balancer, or to proxy/log network traffic between a client and a server).

- So this suspicious behaviour tells that the iexplorer running form user directory John Doe is malware.

- So the answer is md5 sum of the path **C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe**

**49979149632639432397b3a1df8cb43d**
