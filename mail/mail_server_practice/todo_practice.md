Для создания виртуальных машин выбрал Yandex Cloud 
Сделал 3 машины на убунте 22, для каждой машины сгенерировал ssh ключи

### Развертывание PostgreSQL 

1. Подключился по SSH к машине db01
2. `sudo apt update && sudo apt upgrade`
   Для себя хочу отметить, что `apt` более современный по сравнению `apt-get`, поэтому использовал его
3. Настроил брэндмауэр на разрешение доступа по SSH
```bash
db01@db01:~$ sudo ufw app list
Available applications:
  OpenSSH
db01@db01:~$ sudo ufw allow OpenSSH
Rules updated
Rules updated (v6)
db01@db01:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
db01@db01:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

4. Установка PostgreSQL 
`sudo apt install postgresql postgresql-contrib`
    postgresql-contrib - пакет с доп. модулями (различные индекс для поиска), утилиты для админов (скрипты), различные расширения

5. Запускаем и проверяем статус
```bash
db01@db01:~$ sudo systemctl start postgresql.service
db01@db01:~$ sudo systemctl status postgresql.service
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Wed 2023-07-26 12:55:43 UTC; 15min ago
    Process: 26940 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 26940 (code=exited, status=0/SUCCESS)
        CPU: 1ms

Jul 26 12:55:43 db01 systemd[1]: Starting PostgreSQL RDBMS...
Jul 26 12:55:43 db01 systemd[1]: Finished PostgreSQL RDBMS.
```

Все остальное настрою после развертывания приложения


### Развертывание приложения 

Нужно развернуть https://gitfront.io/r/deusops/RcVABw5Efwit/Email-Django-app/
Email-Django-App

1. Склонировал репозиторий 
2. Попытался создать и активировать виртуальное окружение с помощью `python -m venv .venv && source .venv/bin/activate`
   - Стокнулся с ошибкой, оказывается не установлена python3.10-env, установил и все заработало
3. Установил зависимости `pip3 install -r requirements.txt`
4. Создал файл .env и внес такие изменения (пока что не знаю, как буду настраивать постфикс, поэтому проставил значения на рандом)
```python
EMAIL_HOST=smtp.gmail.com
EMAIL_HOST_USER=motesctf@address
EMAIL_HOST_PASSWORD=1111
RECIPIENT_ADDRESS=none
```
5. Чтобы Django мог читать переменные окружения из .env файлика, установил python-dotenv
`pip3 install python-dotenv`
    - Увидел, что в коде в файлике settings.py используется environ, поставил его 
    - 