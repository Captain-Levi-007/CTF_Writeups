# Capture this 

**Statement**
An employee has lost his Keepass password. He couldn’t remember it, and couldn’t find his password file. After hours of searching, it turns out that he has sent a screen of his passwords to one of his colleagues, but it’s still nowhere to be found.

He’s asking for your help to find him.
It’s up to you

sha256sum: 028c8561f087da873b08968d55141dcfc8f10a47e787f79c35b2da611a5e07ce

## Solution 

- Download the challenge file and extract it contents.
```
> ls
ch42.zip

> unzip ch42.zip 
Archive:  ch42.zip
inflating: Capture.png             
extracting: Database.kdbx   
 ```

- We are given with the image file and the keepass database.
![Screenshot](Data/Capture.png) 
- The screenshot contians several other passwords but not the keypass password .
- if you observer closely right side we can see letter **k** . that may be somthing like keepass. 
- But we can't see cause the screenshot is cropped.
- So i googled "How to extract a completed image from a cropped image" but we can find a lots of nonsence.
- But after some time i found somthing called aCropalypse.

***Note: aCropalypse (CVE-2023-21036) is a critical security vulnerability found in image editing tools, notably Google Pixel’s Markup and Windows Snipping Tool, that allows recovery of cropped or edited image data. By failing to truncate files, edited images retain hidden original data, letting attackers reconstruct the full, unredacted image. ***

- I found this [site](https://lordofpipes.github.io/acropadetect/) I tells wether a file is vulnerable or not. 
- I checked the capture.png file and it is vulnerable.	
![Acropalypse Image Trailing Data Detector](Data/Capture2.png)

- I found this [github](https://github.com/frankthetank-music/Acropalypse-Multi-Tool). which contains a [python script](https://github.com/frankthetank-music/Acropalypse-Multi-Tool/blob/main/acropalypse.py) to extract complete image from the files whic are vulnerable to this CVE.

- You can install the gui tool if you want but we dont need that, insted Download or copy the acropalypse.py script. 
- Then i creted a small python script called exploit.py
```
from acropalypse import Acropalypse

a = Acropalypse()

a.reconstruct_image(
    cropped_image_file="Capture.png",
    img_width=1920,
    img_heigth=1080,
    rgb_alpha=True
)
```
- save both scrips in same directory and then 
```
python3 exploit.py                                                                                                               
Found 419252 trailing bytes!
Extracted 419148 bytes of idat!
building bitstream...
reconstructing bit-shifted bytestreams...
Scanning for viable parses...
Found viable parse at bit offset 278910!
Generating output PNG...
Done!

```

- The restores image is saved to /tmp/restored.png
![Restored PNG](Data/Capture3.png)

-Now we can see the keepass password 
![Password](Data/Capture4.png)
- The password is -=b9w9h^+j%\x-rMPUqv9Vv`@X%*=a

- Now open the keepass database and enter the password . we get the password for the Root-me lab.
![keepass database](Data/Capture5.png)
**Root-me paword is  @cropalypse_vuln_is_impressive**