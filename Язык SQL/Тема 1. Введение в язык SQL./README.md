<div style="display: flex; justify-content: center; align-items: center">
<h1>Тема 1. Введение в язык SQL.</h1> <img style="margin-left: 20px" height="63" alt="portfolio_view" src="https://www.postgresql.org/media/img/about/press/elephant.png">
</div>

### Подготовка к работе
- Войдите в систему как пользователь postgres\
`$ sudo - postgres`
  

- Должен быть запущен сервер баз данных PostgreSQL\
`$ pg_ctl start -D /usr/local/pgsql/data`
  

- Для проверки запуска сервера выполнгите команду\
  `$ pg_ctl status -D /usr/local/pgsql/data` или `$ ps -ax | grep postgres | grep -v grep`
  

- Теперь запустите утилиту psql и подключитесь к этой базе данных с учетной записью пользователя\
  postgres\
  `$ psql -d demo -U posgres`

***
### Создание таблиц
- Упрощенный синтаксис таков

```Sql
CREATE TABLE "имя_таблицы"
(
    имя_поля тип_данных [ограничения_целостности],
    имя_поля тип_данных [ограничения_целостности],
    ...
    имя_поля тип_данных [ограничения_целостности],
    [ограничения_целостности]
    [первичный_ключ]
    [внешний_ключ]
);
```
- Для получения справки о синтаксисе SQL-команды\
```SQL
\h CREATE TABLE
```
***
### Таблица "Самолеты"

| Описание атрибута | Имя атрибута | Тип данных | Тип PostgreSQL | Ограничения |
--- | --- | --- | --- | --- |
Код самолета | aircraft_code | Символьный | char(3) | NOT NULL
Модель самолета | model | Символьный | text | NOT NULL
Масимальная дальность <br> полета | range | Числовой | integer | NOT NULL <br> range > 0
```Sql
CREATE TABLE aircrafts
(
    aircraft_code char(3) NOT_NULL,
    model text NOT NULL, 
    range integer NOT NULL,
    CHECK (range > 0),
    PRIMARY KEY (aircraft_code)
);
```
***
### Получение описания таблицы
```SQL
\d aircrafts
```
Таблица "public.aircrafts"

| Колонка | Тип | Модификаторы |
--- | --- | --- |
aircraft_code | character(3) | NOT NULL |
model | text | NOT NLL
range | integer | NOT NULL |


Индексы:\
`"aircrafts_pkey" PRIMARY KEY, btree (aircraft_code)` \
Ограничения-проверки\
`"aircrafts_range_check" CHECK (range > 0)`

- **public** означает имя так называемой **схемы**
- Для реализации **первичного ключа** (PRIMARY KEY) всегда автоматически создается **индекс**.
В данном случае тип индекса - btree, т.е B-дерево.
- Можно задать собственные имена для всех ограничений.
***
### Удаление таблицы
```SQL
DROP TABLE aircrafts;
```
***
