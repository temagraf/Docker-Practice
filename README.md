# Домашнее задание к занятию "`«Практическое применение Docker»`"   

---

### Задание 1

Задача 1
1) Сделайте в своем github пространстве fork репозитория https://github.com/netology-code/shvirtd-example-python/blob/main/README.md.  
2) Создайте файл с именем Dockerfile.python для сборки данного проекта. Используйте базовый образ python:3.9-slim. Протестируйте корректность сборки. Не забудьте dockerignore.  
3) (Необязательная часть, *) Изучите инструкцию в проекте и запустите web-приложение без использования docker в venv.
4) (Необязательная часть, *) По образцу предоставленного python кода внесите в него исправление для управления названием используемой таблицы через ENV переменную.  

### Выполнения задания 1

1) Сделайте в своем github пространстве fork репозитория [https://github.com/temagraf/shvirtd-example-python ] 
2) Создайте файл с именем Dockerfile.python для сборки данного проекта. Используйте базовый образ python:3.9-slim. Протестируйте корректность сборки. Не забудьте dockerignore.

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/1-1.png)

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/1-2.png)

----

### Задание 3

1) Изучите файл "proxy.yaml"  
2) Создайте в репозитории с проектом файл compose.yaml. С помощью директивы "include" подключите к нему файл "proxy.yaml".  
3) Опишите в файле compose.yaml следующие сервисы:  
- web. Образ приложения должен ИЛИ собираться при запуске compose из файла Dockerfile.python ИЛИ скачиваться из yandex cloud container registry(из задание №2 со *). Контейнер должен работать в bridge-сети с названием backend и иметь фиксированный ipv4-адрес 172.20.0.5. Сервис должен всегда перезапускаться в случае ошибок. Передайте необходимые ENV-переменные для подключения к Mysql базе данных по сетевому имени сервиса web
- db. image=mysql:8. Контейнер должен работать в bridge-сети с названием backend и иметь фиксированный ipv4-адрес 172.20.0.10. Явно перезапуск сервиса в случае ошибок. Передайте необходимые ENV-переменные для создания: пароля root пользователя, создания базы данных, пользователя и пароля для web-приложения.Обязательно используйте уже существующий .env file для назначения секретных ENV-переменных!
2) Запустите проект локально с помощью docker compose , добейтесь его стабильной работы: команда curl -L http://127.0.0.1:8090 должна возвращать в качестве ответа время и локальный IP-адрес. Если сервисы не стартуют воспользуйтесь командами: docker ps -a  и docker logs <container_name>
3) Подключитесь к БД mysql с помощью команды docker exec <имя_контейнера> mysql -uroot -p<пароль root-пользователя>(обратите внимание что между ключем -u и логином root нет пробела. это важно!!! тоже самое с паролем) . Введите последовательно команды (не забываем в конце символ ; ): show databases; use <имя вашей базы данных(по-умолчанию example)>; show tables; SELECT * from requests LIMIT 10;.
4) Остановите проект. В качестве ответа приложите скриншот sql-запроса.

### Выполнения задания 3

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/3-1.png)


![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/3-2.png) 


----

### Задание 4

1) Запустите в Yandex Cloud ВМ (вам хватит 2 Гб Ram).
2) Подключитесь к Вм по ssh и установите docker.
3) Напишите bash-скрипт, который скачает ваш fork-репозиторий в каталог /opt и запустит проект целиком.
4) Зайдите на сайт проверки http подключений, например(или аналогичный): https://check-host.net/check-http и запустите проверку вашего сервиса http://<внешний_IP-адрес_вашей_ВМ>:8090. Таким образом трафик будет направлен в ingress-proxy.
5) (Необязательная часть) Дополнительно настройте remote ssh context к вашему серверу. Отобразите список контекстов и результат удаленного выполнения docker ps -a
6) В качестве ответа повторите sql-запрос и приложите скриншот с данного сервера, bash-скрипт и ссылку на fork-репозиторий.

### Выполнения задания 4

1) Запустил в Yandex Cloud ВМ.
2) Подключитесь к Вм по ssh и установил docker.
   
![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/4-2.png)

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/4-2-1.png)


Написание bash-скрипта
Создал скрипт:

```bash
nano /opt/start_project.sh
```
Добавил следующий код в скрипт:

```bash
#!/bin/bash

# Обновил систему и установил git
sudo apt update
sudo apt install git -y

# Переход в каталог /opt
cd /opt

# Скачать наш fork-репозиторий
git clone https://github.com/temagraf/shvirtd-example-python.git

# Перешел в каталог репозитория
cd shvirtd-example-python

# Создал файл .env
cat <<EOF > .env
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=example
MYSQL_USER=your_db_user
MYSQL_PASSWORD=your_db_password
EOF

# Создал файл Dockerfile.python
cat <<EOF > Dockerfile.python
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
EOF

# Создал файл compose.yaml
cat <<EOF > compose.yaml
version: "3"
services:
  proxy:
    image: nginx:latest
    ports:
      - "5000:80"
    networks:
      backend:
        ipv4_address: 172.20.0.2

  web:
    build:
      context: .
      dockerfile: Dockerfile.python
    networks:
      backend:
        ipv4_address: 172.20.0.5
    restart: always
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: \${MYSQL_DATABASE}
      MYSQL_USER: \${MYSQL_USER}
      MYSQL_PASSWORD: \${MYSQL_PASSWORD}

  db:
    image: mysql:8
    networks:
      backend:
        ipv4_address: 172.20.0.10
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: \${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: \${MYSQL_DATABASE}
      MYSQL_USER: \${MYSQL_USER}
      MYSQL_PASSWORD: \${MYSQL_PASSWORD}

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
EOF

# Запуск Docker Compose
sudo docker-compose up -d

```

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/4-3.png)


4) Проверил http подключений, например(или аналогичный): https://check-host.net/check-http и запустите проверку вашего сервиса http://<внешний_IP-адрес_вашей_ВМ>:5000.

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/4-4.png)

6) В качестве ответа повторил sql-запрос и прикладываю скриншот с данного сервера, bash-скрипт.

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/4-6правка.png)


----

### Задание 6

Скачайте docker образ hashicorp/terraform:latest и скопируйте бинарный файл /bin/terraform на свою локальную машину, используя dive и docker save. Предоставьте скриншоты действий.

6.1 Добейтесь аналогичного результата, используя docker cp.  
Предоставьте скриншоты действий.  


### Выполнения задания 6


![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/6-1.png) 


![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/6-1-1.png)



### Доработка задания 6

Скачайте docker образ hashicorp/terraform:latest и скопируйте бинарный файл /bin/terraform на свою локальную машину, используя dive и docker save. Предоставьте скриншоты действий.   

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/6%20bin%201.png)

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/6%20bin%202.png)

![image.jpg](https://github.com/temagraf/Docker-Practice/blob/main/6%20bin%203.png) 
