# Установка и управление сервером

## Установка пакета

* Готовые пакеты - предпочтительный способ.
  * [c сайта www.postgresql.org](https://www.postgresql.org/download/)
  * [c сайта postgrespro.ru](https://postgrespro.ru/products/download)
  

* Linux (Red Hat, CentOS, Debian, Ubuntu и другие)
  * входит в дистрибутивы ОС
  * репозиторий (yum, apt) или пакеты RPM, DEB


* FreeBSD, OpenBSD
  * пакеты из Ports and Package Collection

Mac OS X

Windows

Предпочтительным вариантом является использование готовых
пакетов, так как в этом случае получается понятная, поддерживаемая и
легко обновляемая установка.
Пакеты существуют для большинства широко распространенных
систем, но в каждой из них имеются свои особенности установки,
которые здесь не рассматриваются.

## Исходные коды

* Установка из исходных кодов
  * собрать стабильную версию сервера
  * со специфичными параметрами или на специфичной архитектуре
  * [https://ftp.postgresql.org/pub/source/v10.0/postgresql-10.0.tar.gz](https://ftp.postgresql.org/pub/source/v10.0/postgresql-10.0.tar.gz)


* Установка из репозитория git
  * собрать текущую версию сервера
  * в первую очередь для разработчиков ядра, требует более широкого
  * набора инструментов
  * [git://git.postgresql.org/git/postgresql.git](git://git.postgresql.org/git/postgresql.git)

Полезно иметь представление о том, как происходит установка из
исходных кодов, чтобы в случае необходимости можно было собрать
PostgreSQL с нестандартными параметрами или на специфичной
архитектуре.
По [адресу](https://www.postgresql.org/ftp/source/v10.0/) доступен
исходный код версии 10.0 в двух вариантах (`gzip` и `bzip2`).
В виртуальной машине необходимый файл уже находится в домашнем
каталоге пользователя `postgres`.
При установке из репозитория `git` требуется более широкий набор
инструментов. Например, лексический и синтаксический анализатор
сделан с помощью Flex и Bison, и в `git` хранятся именно исходные коды
для этих инструментов. При сборке из них генерируются файлы на Си,
которые затем компилируются. При этом в архиве с исходными кодами
находится уже сгенерированные файлы Си.


## Необходимое ПО

* Обязательно
  * tar, gzip/bzip2, GNU make, компилятор Си (C89)
  * Используются, но можно отключить
  * библиотеки GNU Readline, zlib
  

* Дополнительно
  * языки программирования Perl, Python, Tcl
  * для использования PL/Perl, PL/Python, PL/Tcl
  * Kerberos, OpenSSL, OpenLDAP, PAM
  * для аутентификации и шифрования
  

* Отдельные инструменты при сборке из репозитория git


Для сборки требуется ряд программ и утилит.
Библиотека `readline` дает возможность редактировать командную
строку, пользоваться историей команд и автодополнением. В серверной
инсталляции может быть и не нужна, если не предполагается запускать
`psql` в консоли.
Библиотека `zlib` используется для сжатия архивов `pg_dump`

## Практика

### Установка postgres
```shell
user$ tar xzf ~/Загрузки/postgresql-14.0.tar.gz

user$ cd postgresql-14.0
```

### Создание конфигурации
```shell
# (!НЕОБЯЗАТЕЛЬНО)
# Если требуется повторно выполнить конфигурацию, например c другими 
# параметрами, то предварительно нужно очистить результаты предыдущего запроса:

user$ make distclean
```

```shell
# в команде configure можно указать различные параметры конфигуарции:
# --prefix -  каталог установки, по умолчанию /usr/local/pgsql
# --enable-debug - для включения отладочной информации

user$ ./configure
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking which template to use... linux
...
config.status: linking src/backend/port/sysv_shmem.c to src/backend/port/pg_shmem.c
config.status: linking src/include/port/linux.h to src/include/pg_config_os.h
config.status: linking src/makefiles/Makefile.linux to src/Makefile.port
```
### Сборка PostgreSQL
```shell
# Возможножные варианты
# make - сборка только сервера.
# make world - сборка сервера, всех расширений и документации.
# Выбираем сборку только сервера. Установка расширений посмотрим дальше.

user$ make
make -C ./src/backend generated-headers
make[1]: Entering directory '/home/user/postgresql-14.0/src/backend'
make -C catalog distprep generated-header-symlinks
...
make[1]: Entering directory '/home/user/postgresql-14.0/config'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/home/user/postgresql-14.0/config'
```
### Установка
```shell
# Теперь выполняем устаноку. Для этого потребуются права суперпользователя.
# команда make устанавлиает в тот пакет который указан в --prefix

user$ sudo make install
make -C ./src/backend generated-headers
make[1]: Entering directory '/home/user/postgresql-14.0/src/backend'
make -C catalog distprep generated-header-symlinks
...
/usr/bin/install -c -m 755 ./install-sh '/usr/local/pgsql/lib/pgxs/config/install-sh'
/usr/bin/install -c -m 755 ./missing '/usr/local/pgsql/lib/pgxs/config/missing'
make[1]: Leaving directory '/home/user/postgresql-14.0/config'

# Если бы на прошлом шаге делали make world, т.е собрали бы сервер с расширениями и документацией
# то на этом пришлось бы делать не sudo make install а sudo make install-world
```

### Пользователь postgres и каталог PGDATA

```shell
# Пользователь postgres, под которым будет работать СУБД, уже создан заранее. Нужно создать каталог 
# для данных и сделать postgres его владельцем.

user$ sudo mkdir /usr/local/pgsql/data
user$ sudo chown postgres /usr/local/pgsql/data

# /usr/local/pgsql/data часто называют PGDATA, по имени переменной окружения, которую
# удобно использовать при работе с утилитами сервера. 
```

### Окружение пользователя

```shell
# В окружениии пользователя postgres учтено, куда устанавливается СУБД, и где находится каталог
# с данными.

postgres$ echo $PGDATA
empty

postgres$ export PGDATA=/usr/local/pgsql/data
postgres$ export PATH=/usr/local/pgsql/bin:$PATH

postgres$ echo $PGDATA; echo $PATH
/usr/local/pgsql/data
/usr/local/pgsql/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

# Команда для переключения на пользователя postgres: sudo su - postgres
```

### Инициализация и запуск кластера
```shell
# Для инициализации кластера баз данных используется утилита initdb.
# Ключ -k включает подсчет контрольной суммы страниц, что позволяет своевременно обноруживать
# повреждение данных. В остальном используем настройки по умолчанию:

postgres$ initdb -k
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.
...
Success. You can now start the database server using:
pg_ctl -D /usr/local/pgsql/data -l logfile start
```

### Запуск сервера
```shell
# Теперь все готово к запуску сервера:

postgres$ ls 
12  dbal_log  postgres

postgres$ pg_ctl -w -l postgres/logfile -D /usr/local/pgsql/data start
waiting for server to start.... done
server started

postgres$ cat postgres/logfile
2021-10-08 05:07:48.948 MSK [75163] LOG:  starting PostgreSQL 14.0 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0, 64-bit
2021-10-08 05:07:48.948 MSK [75163] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2021-10-08 05:07:48.953 MSK [75163] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-10-08 05:07:48.958 MSK [75166] LOG:  database system was shut down at 2021-10-08 05:00:54 MSK
2021-10-08 05:07:48.961 MSK [75163] LOG:  database system is ready to accept connections
```

### Проверяем работу
```shell
postgres$ psql -c 'select now();'
              now              
-------------------------------
 2021-10-08 05:11:16.175839+03
(1 row)

# Все успешно подключилось и выполнилась функция возвращающая текущее время
```

### Установка расширений
```shell
# Собираем и устанавливаем расширение pgcrypto. Для сборки отдельного расширения,
# нужно перейти в его каталог и выполнить команду make.

postgres$ su - user

user$ cd ~/postgresql-14.0/contrib/pgcrypto/
user$ make
make -C ../../src/backend generated-headers
make[1]: Entering directory '/home/v1adt3r/postgresql-14.0/src/backend'
make -C catalog distprep generated-header-symlinks
...

user$ sudo make install
make -C ../../src/backend generated-headers
make[1]: Entering directory '/home/v1adt3r/postgresql-14.0/src/backend'
make -C catalog distprep generated-header-symlinks
...
```

### Проверка расширений
```shell
# Большинство установленных расширений создают новые объекты SQL 
# (функция, представления, таблицы). Перед использованием таких расширений 
# требуется выполнить SQL-команду CREATE EXTENSION

# Список доступных расширений можно посмотреть запросом:

postgres$ psql -c 'SELECT name, comment FROM pg_available_extensions ORDER BY name;'
   name   |           comment            
----------+------------------------------
 pgcrypto | cryptographic functions
 plpgsql  | PL/pgSQL procedural language
(2 rows)
```

### Остановка сервера
```shell
# Для остановки используется команда pg_ctl stop -m fast|smart|immediate
# В ключе -m можно указать один из трех режимов останова:
# fast - принудительно завершает сеансы и записывает на диск изменения из оперативной
# памяти.
# smart - ожидает завершения всех сеансов и записывает на диск изменения из оперативной
# памяти.
# immediate - принудительно завершает сеансы, при запуске потребуется восстановление.

# По умолчанию используется fast.

postgres$ pg_ctl -w -l postgres/logfile -D /usr/local/pgsql/data stop
waiting for server to shut down.... done
server stopped
```