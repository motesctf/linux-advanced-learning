1. `SMTP` - протокол, используемый для отправки эл. почты от клиента к серверу (и от сервера к серверу)
2. Авторизация - процесс, в ходе которого проверяется, разрешено ли пользователю выполнять действия (например отправлять письма по почте)
3. MAIL FROM - команда SMTP, используемая для указания адреса отправителя письма
4. RCPT TO - команда SMTP, используемая для указания адреса получателя письма
5. DNS-запись MX (Mail Exchanger) - указывает почтовому серверу, куда отправить письмо. SMTP-сервер выполнит запрос DNS MX для домена адреса получателя, чтобы узнать, куда отправить письмо 
6. TLS-порт 465 - стандартный порт для SMTPS (защищенный SMTP) - исп. TLS для шифрования данных 
7. Порт 25 - стандартный порт для SMTP 

`Промежуточная выжимка`

SMTP - это протокол, который используется для передачи электронной почты от отправителя к получателю. Этот процесс включает в себя ряд этапов, таких как авторизация, определение адресов отправителя и получателя, поиск подходящего почтового сервера с использованием записей MX, отправка письма и выполнение проверок безопасности и фильтрации спама на получающем сервере.


### Как работает электронная почта на техническом уровне 
1. При отправлении письма, оно передается с устройства на SMTP-сервер
2. Сервер авторизует ваши данные, после чего письмо помещается в очередь отправки
3. Когда наступает очередь вашего письма - сервер проверяет отправляемые данные и определяет, на какой сервер нужно отправить письмо
4. Сервер получается домен из адреса получается и получает DNS-запись MX этого домена
5. Письмо отправляется на соотв. домен по SMTP
6. Все остальные действия происходят на стороне удаленного сервера, который принимает письмо, проводит проверки безопасности и помещает письмо в почтовый ящик получателя.

### 2 компонента сервера электронной почты
`MTA` (Mail Transger Agent) - агент пересылки почты, который отвечает за прием и передачу почты. Работает по SMTP
Если говорить простым языком, то `MTA` - почтальон. Написали вы письмо другу и хотите его отправить, вы передаете его почтальону (`MTA`), который отнесет его по адресу друга

`MDA` (Mail Delivery Agent) - агент доставки почты, который передает почту из ящика на сервере по запросу почтового клиента.
MDA - что-то типа секретаря, который принимает почту от почтальона и кладет ее в ящик

### Postfix 

1. `Это MTA` - соотв. он обеспечивает прием и отправку почты
2. `Postfix` - модульный `MTA` - сервисы подключаются для выполнения конкретной логики обработки почты
3. Конфигурация Postfix состоит из двух основных файлов
   1. `/etc/postfix/master.cf` - используется для конфигурации сервисов 
   2. `/etc/posfix/main.cf` - конфиг MTA, общие настройки (имя хоста, типы поддерживаемой аутентификации и тд.)
4. Все сервисы Postfix запускаются как подпроцессы от главного. Postfix в данном случае выступает в роли супервизора этих процессов 
5. Существуют три основных точки входа для писем в Postfix:

    `smtpd` используется для принятия входящих запросов по протоколу SMTP.
    `qmqpd` используется для работы с протоколом QMQP (Quick Mail Queuing Protocol), который предназначен для замены SMTP как более быстрого решения, но SMTP по-прежнему является стандартом.
    `maildrop` является сервисом для принятия сообщений на самом хосте. 


### Разбор конфига master.cf (конфигурация сервисов)

```bash
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (no)    (never) (100)
# ==========================================================================
smtp      inet  n       -       y       -       -       smtpd
pickup    unix  n       -       y       60      1       pickup
cleanup   unix  n       -       y       -       0       cleanup
qmgr      unix  n       -       n       300     1       qmgr
tlsmgr    unix  -       -       y       1000?   1       tlsmgr
rewrite   unix  -       -       y       -       -       trivial-rewrite
bounce    unix  -       -       y       -       0       bounce
defer     unix  -       -       y       -       0       bounce
trace     unix  -       -       y       -       0       bounce
verify    unix  -       -       y       -       1       verify
flush     unix  n       -       y       1000?   0       flush
proxymap  unix  -       -       n       -       -       proxymap
proxywrite unix -       -       n       -       1       proxymap
smtp      unix  -       -       y       -       -       smtp
relay     unix  -       -       y       -       -       smtp
        -o syslog_name=postfix/$service_name
showq     unix  n       -       y       -       -       showq
error     unix  -       -       y       -       -       error
retry     unix  -       -       y       -       -       error
discard   unix  -       -       y       -       -       discard
local     unix  -       n       n       -       -       local
virtual   unix  -       n       n       -       -       virtual
lmtp      unix  -       -       y       -       -       lmtp
anvil     unix  -       -       y       -       1       anvil
scache    unix  -       -       y       -       1       scache
maildrop  unix  -       n       n       -       -       pipe
  flags=DRhu user=vmail argv=/usr/bin/maildrop -d ${recipient}
uucp      unix  -       n       n       -       -       pipe
  flags=Fqhu user=uucp argv=uux -r -n -z -a$sender - $nexthop!rmail ($recipient)
ifmail    unix  -       n       n       -       -       pipe
  flags=F user=ftn argv=/usr/lib/ifmail/ifmail -r $nexthop ($recipient)
bsmtp     unix  -       n       n       -       -       pipe
  flags=Fq. user=bsmtp argv=/usr/lib/bsmtp/bsmtp -t$nexthop -f$sender $recipient
scalemail-backend unix	-	n	n	-	2	pipe
  flags=R user=scalemail argv=/usr/lib/scalemail/bin/scalemail-store ${nexthop} ${user} ${extension}
mailman   unix  -       n       n       -       -       pipe
  flags=FR user=list argv=/usr/lib/mailman/bin/postfix-to-mailman.py
  ${nexthop} ${user}
```

#### Стобцы
1. service - имя сервиса (уникальное)
2. type - тип сокета (inet=TCP, unix=UNIX socket - то есть, доступный только в системе)
3. private (yes) - должен ли этот сервис быть доступен извне (по умолчанию - yes, если стоит -, то значение по умолчанию)
4. unpriv (yes) - запускать сервис от имени непривелегированного пользователя
5. chroot (no) - запускать сервис в chroot (в отдельном окружении)
   Например, если вы делаете chroot в каталог /var/mychroot, то для процессов, запущенных в этом окружении, самым верхним уровнем директории будет /var/mychroot, а не реальный корневой каталог /.

6. wakeup (never) - как часто требуется отправлять запросы на запуск сервиса. Что-то типа cron внутри postfix
7. maxproc (100) - количество процессов - которое может быть запущено. Значение 0 = можно выделять неограниченное количество
8. command + args определяет, какую компоненту нужно запускать для этого сервиса (могут повторяться). Кроме самой команды, также передаются параметры запуска. Так, к примеру, у сервиса relay передается параметр -o syslog_name=postfix/$service_name, который перезаписывает (override) глобальный параметр запуска с именем сервиса, который будет передаваться в syslog.

#### Основные сервисы
1. smtp - принятие входных сообшение по SMTP
2. pickup - сервис обрабатывает почту, которая была сгенерирована непосредственно на самом сервере. Это включает в себя, например, почту от системных демонов или локальных пользователей.
3. cleanup - cервис очистки, который отвечает за предварительную обработку и "очистку" всех входящих сообщений, важно для обеспечения целостности и безопасности системы почты.
4. qmgr - служба управления очередью (управляет доставкой писем)
5. tlsmgr - менеджер TLS сеансов и кэша постфикс
6. bounce - служба управляет созданием и доставкой сообщений о возврате и отказе в доставке.
7. local - сервис обрабатывает почту, предназначенную для доставки локально.


#### main.cf 

Пример минимального конфига по умолчанию

```bash
# egrep -v "(^#.*|^$)" main.cf
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu) # что будет написано при установлении соединения
biff = no 
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = localhost # определяет имя хоста
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, localhost, localhost.localdomain, , localhost # список доменов, для которых сервер принимает почту
relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
```