# [Logs analysis - web attack](https://www.root-me.org/en/Challenges/Forensic/Logs-analysis-web-attack)

**Statement**

Our website appears to have been attacked, but our system administrator does not understand web server logs. Can you find out if any data has been stolen ?

- Downlaod the log file .ch13.txt

```
head ch13.txt

192.168.1.23 - - [18/Jun/2015:12:12:54 +0200] "GET /admin/?action=membres&order=QVNDLChzZWxlY3QgKGNhc2UgZmllbGQoY29uY2F0KHN1YnN0cmluZyhiaW4oYXNjaWkoc3Vic3RyaW5nKHBhc3N3b3JkLDEsMSkpKSwxLDEpLHN1YnN0cmluZyhiaW4oYXNjaWkoc3Vic3RyaW5nKHBhc3N3b3JkLDEsMSkpKSwyLDEpKSxjb25jYXQoY2hhcig0OCksY2hhcig0OCkpLGNvbmNhdChjaGFyKDQ4KSxjaGFyKDQ5KSksY29uY2F0KGNoYXIoNDkpLGNoYXIoNDgpKSxjb25jYXQoY2hhcig0OSksY2hhcig0OSkpKXdoZW4gMSB0aGVuIFRSVUUgd2hlbiAyIHRoZW4gc2xlZXAoMikgd2hlbiAzIHRoZW4gc2xlZXAoNCkgd2hlbiA0IHRoZW4gc2xlZXAoNikgZW5kKSBmcm9tIG1lbWJyZXMgd2hlcmUgaWQ9MSk%3D HTTP/1.1" 200 1005 "-" "-"
192.168.1.23 - - [18/Jun/2015:12:13:00 +0200] "GET /admin/?action=membres&order=QVNDLChzZWxlY3QgKGNhc2UgZmllbGQoY29uY2F0KHN1YnN0cmluZyhiaW4oYXNjaWkoc3Vic3RyaW5nKHBhc3N3b3JkLDEsMSkpKSwzLDEpLHN1YnN0cmluZyhiaW4oYXNjaWkoc3Vic3RyaW5nKHBhc3N3b3JkLDEsMSkpKSw0LDEpKSxjb25jYXQoY2hhcig0OCksY2hhcig0OCkpLGNvbmNhdChjaGFyKDQ4KSxjaGFyKDQ5KSksY29uY2F0KGNoYXIoNDkpLGNoYXIoNDgpKSxjb25jYXQoY2hhcig0OSksY2hhcig0OSkpKXdoZW4gMSB0aGVuIFRSVUUgd2hlbiAyIHRoZW4gc2xlZXAoMikgd2hlbiAzIHRoZW4gc2xlZXAoNCkgd2hlbiA0IHRoZW4gc2xlZXAoNikgZW5kKSBmcm9tIG1lbWJyZXMgd2hlcmUgaWQ9MSk%3D HTTP/1.1" 200 1005 "-" "-"

```

- We can see that from the above sample . ther is a base64 payload served via the order perameter . 
- lets try to decode the base64 payload .
![screenshot](/Data/loganlysis.png)

```
ASC,
(
  select
  (
    case field(
      concat(
        substring(
          bin(
            ascii(
              substring(password,1,1)
            )
          ),
        1,1),

        substring(
          bin(
            ascii(
              substring(password,1,1)
            )
          ),
        2,1)
      ),

      concat(char(48),char(48)),
      concat(char(48),char(49)),
      concat(char(49),char(48)),
      concat(char(49),char(49))

    )

    when 1 then TRUE
    when 2 then sleep(2)
    when 3 then sleep(4)
    when 4 then sleep(6)

  end)

from membres where id=1)
``` 
- It is a time base blind sql injection attack it is trying to extract password of userid=1 from members table.
- it extract each char from password field and convert it into ascii numeric value using ascii() function.
- then convert it infot binary form using bin() function.
- After obtaining the binary representation , the payload extracts the first and second bits using the substring() and combines them using concat() function .
- The exracted two-bit value can produce one of four possible combinations:00,01,10,11.
- FIELD() function is then used to compare the extracted bits against these four possibilities and return a corresponding numeric position
- a CASE statement maps each possible result to a different server behavior: immediate response for 00, a 2-second delay for 01, a 4-second delay for 10, and a 6-second delay for 11

- Now our best shot is timestamp
```
 head ch13.txt | awk '{print $4}'
[18/Jun/2015:12:12:54
[18/Jun/2015:12:13:00
[18/Jun/2015:12:13:00
[18/Jun/2015:12:13:06
[18/Jun/2015:12:13:10
[18/Jun/2015:12:13:16
[18/Jun/2015:12:13:20
[18/Jun/2015:12:13:22
[18/Jun/2015:12:13:22
[18/Jun/2015:12:13:26
```
- If you observe closely the time gap between successive requests is wither 0,2,4,6 . so based on the time gap we can construct the binary extracted by the attacker.

- so to automate this is wrote a python script .
```
from datetime import datetime

f = open("ch13.txt", "r")

sec = []

for line in f:

    try:
        t = line.split(" ")[3]
        s = int(t.split(":")[-1])
        sec.append(s)

    except:
        continue

f.close()

delays = []

for i in range(len(sec)-1):

    d = sec[i+1] - sec[i]

    if d < 0:
        d += 60

    delays.append(d)

groups = []
tmp = []

for d in delays:

    tmp.append(d)

    if len(tmp) == 4:
        groups.append(tmp)
        tmp = []

result = ""

print("\n[+] Reconstruction\n")

for i, g in enumerate(groups):

    bits = ""

    for x in g[:3]:

        if x == 0:
            bits += "00"

        elif x == 2:
            bits += "01"

        elif x == 4:
            bits += "10"

        elif x == 6:
            bits += "11"

    last = g[3]

    if last == 2:
        bits += "0"

    elif last == 4:
        bits += "1"

    try:

        val = int(bits, 2)
        ch = chr(val)

        result += ch

        print(
            f"{i+1:02d} | "
            f"{g} | "
            f"{bits} | "
            f"{val} | "
            f"{ch}"
        )

    except:

        print(f"failed -> {bits}")

print("\n[+] Result:\n")
print(result)
```
- the above script extract the time stamp from the log file and calculate delay between them and then construct the binary to ascii char . 

```
 python3 stolen_data.py

[+] Reconstruction

01 | [6, 0, 6, 4] | 1100111 | 103 | g
02 | [6, 4, 2, 0] | 111001 | 57 | 9
03 | [4, 4, 4, 4] | 1010101 | 85 | U
04 | [4, 4, 6, 4] | 1010111 | 87 | W
05 | [4, 0, 4, 2] | 1000100 | 68 | D
06 | [6, 4, 0, 0] | 111000 | 56 | 8
07 | [4, 0, 4, 4] | 1000101 | 69 | E
08 | [4, 6, 2, 2] | 1011010 | 90 | Z
09 | [6, 0, 6, 4] | 1100111 | 103 | g
10 | [4, 0, 2, 2] | 1000010 | 66 | B
11 | [6, 2, 0, 2] | 1101000 | 104 | h
12 | [4, 0, 2, 2] | 1000010 | 66 | B
13 | [6, 4, 0, 2] | 1110000 | 112 | p
14 | [6, 0, 2, 4] | 1100011 | 99 | c
15 | [6, 2, 0, 0] | 110100 | 52 | 4
16 | [6, 2, 6, 2] | 1101110 | 110 | n
17 | [4, 4, 4, 2] | 1010100 | 84 | T
18 | [4, 4, 2, 4] | 1010011 | 83 | S
19 | [4, 0, 0, 4] | 1000001 | 65 | A
20 | [4, 4, 2, 4] | 1010011 | 83 | S

[+] Result:

g9UWD8EZgBhBpc4nTSAS
```