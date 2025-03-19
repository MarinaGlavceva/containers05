# Лабораторная работа №5: Запуск сайта в контейнере

## Цель работы

В ходе выполнения данной работы я подготовила образ контейнера для запуска веб-сайта на базе Apache HTTP Server + PHP (mod_php) + MariaDB. Настроила базу данных в монтируемом томе и установила сайт WordPress, обеспечив его работоспособность.

---

## Задание

1. Создать `Dockerfile` для сборки образа контейнера с Apache, PHP и MariaDB.
2. Настроить монтируемый том для базы данных MariaDB.
3. Установить и настроить WordPress.
4. Проверить работоспособность веб-сайта.
5. Ответить на контрольные вопросы.

---

## Выполнение работы

### **1. Создание репозитория**
Я создала новый репозиторий **containers05** на GitHub и склонировала его на свой компьютер:
```sh
 git clone  https://github.com/MarinaGlavceva/containers05
 cd containers05
```

### **2. Подготовка структуры проекта**
В терминале создала необходимые папки:
```sh
C:\Users\marin\containers05>mkdir files

C:\Users\marin\containers05>mkdir files\apache2

C:\Users\marin\containers05>mkdir files\php

C:\Users\marin\containers05>mkdir files\mariadb

C:\Users\marin\containers05>mkdir files\supervisor
```

Создала пустые файлы для конфигурации:
```sh
C:\Users\marin\containers05>echo. > Dockerfile

C:\Users\marin\containers05>echo. > files\supervisor\supervisord.conf
```

---

### **3. Создание `Dockerfile`**
В файл `Dockerfile` добавила следующий код:
```dockerfile
# Используем базовый образ Debian
FROM debian:latest

# Устанавливаем Apache, PHP и MariaDB
RUN apt-get update && \
    apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
    apt-get clean

# Открываем порт 80
EXPOSE 80

# Запускаем Apache при старте контейнера
CMD ["apachectl", "-D", "FOREGROUND"]
```

Собрала образ:
```sh
 docker build -t apache2-php-mariadb .
```
![Image](https://github.com/user-attachments/assets/dc5c911e-4754-423c-9991-e3522e876fad)

Запустила контейнер:
```sh
 docker run -d --name apache2-php-mariadb -p 8000:80 apache2-php-mariadb
```
![Image](https://github.com/user-attachments/assets/8edf2d88-df06-4a4f-9eb1-8d885e89ac16)

Проверила работу через `http://localhost:8000`.
![Image](https://github.com/user-attachments/assets/db09fa94-be06-45f6-913f-0b88fcbc2191)
---

### **4. Извлечение конфигурационных файлов**
Скопировала конфигурационные файлы из контейнера:
```sh
 docker cp apache2-php-mariadb:/etc/apache2/sites-available/000-default.conf files/apache2/
 docker cp apache2-php-mariadb:/etc/apache2/apache2.conf files/apache2/
 docker cp apache2-php-mariadb:/etc/php/8.2/apache2/php.ini files/php/
 docker cp apache2-php-mariadb:/etc/mysql/mariadb.conf.d/50-server.cnf files/mariadb/
```
Остановила и удалила контейнер:
```sh
 docker stop apache2-php-mariadb
 docker rm apache2-php-mariadb
```

---

### **5. Настройка конфигурационных файлов**

#### **Apache**
В файле `files/apache2/000-default.conf` изменила:
```plaintext
ServerName localhost
ServerAdmin myemail@example.com
DocumentRoot /var/www/html
DirectoryIndex index.php index.html
```

#### **PHP**
Изменила `files/php/php.ini`:
```plaintext
error_log = /var/log/php_errors.log
memory_limit = 128M
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 120
```

#### **MariaDB**
В файле `files/mariadb/50-server.cnf` раскомментировала строку:
```plaintext
log_error = /var/log/mysql/error.log
```

---

### **6. Развертывание WordPress**
Собрала обновленный образ:
```sh
 docker build -t apache2-php-mariadb .
```

Запустила контейнер:
```sh
 docker run -d --name apache2-php-mariadb -p 8000:80 apache2-php-mariadb
```

Создала базу данных и пользователя внутри контейнера:
```sh
 docker exec -it apache2-php-mariadb mysql
```
Далее в MySQL:
```sql
CREATE DATABASE wordpress;
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'wordpress';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

В браузере открыла `http://localhost:8000`, заполнила данные подключения к базе и сохранила `wp-config.php`. Скопировала его в `files/wp-config.php`, затем добавила его в `Dockerfile`:
```dockerfile
COPY files/wp-config.php /var/www/html/wordpress/wp-config.php
```
Пересобрала и перезапустила контейнер.

---

## Контрольные вопросы

1. **Какие файлы конфигурации были изменены?**
   - `000-default.conf` (Apache)
   - `apache2.conf` (Apache)
   - `php.ini` (PHP)
   - `50-server.cnf` (MariaDB)
   - `wp-config.php` (WordPress)

2. **За что отвечает инструкция `DirectoryIndex` в конфигурации Apache?**
   Она определяет порядок загрузки файлов, указывая, что сначала должен загружаться `index.php`, а затем `index.html`.

3. **Зачем нужен файл `wp-config.php`?**
   Этот файл содержит настройки подключения к базе данных WordPress, включая имя БД, пользователя, пароль и префикс таблиц.

4. **За что отвечает параметр `post_max_size` в `php.ini`?**
   Этот параметр определяет максимальный размер данных, отправляемых через HTTP POST-запросы (например, при загрузке файлов).

5. **Какие недостатки есть в созданном образе контейнера?**
   - Отсутствует автоматическая инициализация базы данных.
   - MariaDB запускается без безопасных настроек.
   - WordPress загружается вручную, можно было бы добавить автоматическую настройку.

---

## Выводы
Проект загружен на GitHub: [ссылка на репозиторий](https://github.com/мой_логин/containers05).

---

🎯 **Лабораторная выполнена успешно!** 🚀

