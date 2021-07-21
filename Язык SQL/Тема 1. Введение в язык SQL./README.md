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
### Вставка строк в таблицу (1)
Упрощенный синтаксис:
```SQL
INSERT INTO имя_таблицы
    [(имя_аттрибута, имя_аттрибута, ...)]
    VALUES (значение_аттрибута, значени_аттрибута, ...);
```

- В начале команды перечисляются атрибуты таблицы. При этом можно указывать их не в том порядке, 
в котором они были указаны при её создании.
  
- Если вы не привели список атрибутов, тогда вы обязаны в предложении VALUES задавать значения атрибутов 
с учетом того порядка, в котором они следуют в определении таблицы.
  
***
### Вставка строк в таблицу (2)
```SQL
INSERT INTO aircrafts (aircraft_code, model, range) 
VALUES ('SU9', 'Sukhoi SuperJet-100', 3000); 
```
В ответе мы получим сообщение об успешном добавлении этой строки:
`INSERT 0 1`

0 - имеет отношение к внутреннему устройству PostgreSQL.\
1 - кол-во добавленых строк.
***
### Выборка строк из таблицы
Синтаксис упрощенный до предела таков:
```SQL
SELECT имя_атрибута, имя_атрибута, ...
FROM имя_таблицы;
```
Часто бывает так что требуется вывести значение из всех стообцов таблицы. В таком случае можно
не перечислять имена аттрибутов, а просто ввести символ `*`. Давайте выберем всю информацию из таблицы 
aircrafts:
```SQL
SELECT * FROM aircrafts;
```
СУБД ответит таким образом:

| aircraft_code | model | range |
--- | --- | --- |
SU9 | Sukhoi SuperJet-100 | 3000 |
(1 строка) |
***
### Вставка нескольких строк в таблицу
```SQL
INSERT INTO aircrafts (aircraft_code, model, range) 
VALUES ('SU9', 'Sukhoi SuperJet-100', 2000)
       ('SU9', 'Sukhoi SuperJet-100', 3000)
       ('SU9', 'Sukhoi SuperJet-100', 1000)
       ('SU9', 'Sukhoi SuperJet-100', 4000)
       ('SU9', 'Sukhoi SuperJet-100', 5000)
       ('SU9', 'Sukhoi SuperJet-100', 6000); 
```
СУБД сообщит об успешном вводе 6 строк в таблицу aircrafts:
`INSERT 0 6`
***
### Что получилось в таблице?
```SQL
SELECT * FROM aircrafts;
```
Теперь в ней уже 7 строк.

| aircraft_code | model | range |
--- | --- | --- |
SU9 | Sukhoi SuperJet-100 | 3000 |
SU9 | Sukhoi SuperJet-100 | 2000 |
SU9 | Sukhoi SuperJet-100 | 3000 |
SU9 | Sukhoi SuperJet-100 | 1000 |
SU9 | Sukhoi SuperJet-100 | 4000 |
SU9 | Sukhoi SuperJet-100 | 5000 |
SU9 | Sukhoi SuperJet-100 | 6000 |
(7 строк) |

**ВАЖНО!** При выполнении простой выборки из таблицы СУБД не гарантирует никакого
конкретного порядка вывода строк.
***
### Упорядочивани строк в выборке
```SQL
SELECT aircraft_code, model, range
FROM aircrafts
ORDER BY range;
```
Упорядочим строки по значению атрибута range.

| aircraft_code | model | range |
--- | --- | --- |
SU9 | Sukhoi SuperJet-100 | 1000 |
SU9 | Sukhoi SuperJet-100 | 2000 |
SU9 | Sukhoi SuperJet-100 | 3000 |
SU9 | Sukhoi SuperJet-100 | 3000 |
SU9 | Sukhoi SuperJet-100 | 4000 |
SU9 | Sukhoi SuperJet-100 | 5000 |
SU9 | Sukhoi SuperJet-100 | 6000 |
(7 строк) |
***
### Ограничение множества строк
Выберем модели самолетов, у которых максимальная дальность полетов находится в пределах
от 4 до 6 тысяч км включительно.
```SQL
SELECT aircraft_code, model, range
FROM aircrafts
WHERE range >= 4000 AND range <= 6000;
```
| aircraft_code | model | range |
--- | --- | --- |
SU9 | Sukhoi SuperJet-100 | 4000 |
SU9 | Sukhoi SuperJet-100 | 5000 |
SU9 | Sukhoi SuperJet-100 | 6000 |
(3 строки) |
***
### Обновление строк в таблице (1)
Команда `UPDATE` предназначена для обновления данных в таблицах. Её упрощенный синтаксис таков:
```SQL
UPDATE имя_таблицы
SET имя_аттрибута1 = значение_аттрибута1,
    имя_аттрибута2 = значение_аттрибута2
WHERE условие;
```
**ВАЖНО!** Если это условие не задать, то будут обновлены все встроки в таблице.
***
### Обновление строк в таблице (2)
Предположим, что нужно увеличить дальность у самолета с дальностью 1000 км на 100 км.
```SQL
UPDATE aircrafts
SET range = 1100 
WHERE range = 1000;
```
СУБД выведет сообщение, подтверждающее успешное обновление одной строки:
`UPDATE 1`
```SQL
SELECT * FROM aircrafts 
WHERE range = 1100;
```
| aircraft_code | model | range |
--- | --- | --- |
SU9 | Sukhoi SuperJet-100 | 1100 |
(1 строка) |
***
### Удаление строк из таблицы
Для этого используется команда `DELETE`. Она похожа на команду `SELECT`:
```SQL
DELETE FROM имя_таблицы WHERE условие;
```
Удалим одну строку из таблицы aircrafts:
```SQL
DELETE FROM aircrafts WHERE range > 10000 OR range < 3000;
```
СУБД сообщит об успешном удалении одной строки:
`DELETE 1`

При необходимости удаления всех строк из таблицы, команда будет совсем простой:
```SQL
DELETE FROM aircrafts;
```
***
### Восстановлени данных в таблице
Теперь в таблице aircrafts нет ни одной строки. Для продолжения нужно эти данные восстановить.
Можно использовать несколько способов.

1. Ввести заново команды `INSERT` из текста пособия, которые вы ранее уже вводили.
2. Используя клавиши "стрелка вверх" и "стрелка вниз", найти команды `INSERT` в истории команд и повторно
выполнить их.
3. C помощью специальной команды, предусмотренной в утилите psql, сохранить все историю 
выполненых команд в текстовом файле.
   
`\s имя_файла_для_сохранения_истории_команд`

Затем нужно открыть его в текстовом редакторе, найти в файле нужные вам команды `INSERT`
и, компируя команды в буфер обмена, вставить их в командную строку утилиты psql и выполнить.
***
### Таблица "Места" (1)
| Описание атрибута | Имя атрибута | Тип данных | Тип PostgreSQL | Ограничения |
--- | --- | --- | --- | --- |
Код самолета IATA | aircraft_code | Символьный | char(3) | NOT NULL
Номер места | seat_no | Символьный | varchar(4) | NOT NULL
Класс обслуживания | fare_conditions | Символьный | varchar(10) | NOT NULL <br> значения из списка: <br> Economy, Comfort, Business
***
### Таблица "Места" (2)
```SQL
CREATE TABLE seats
(
  aircraft_code char(3) NOT NULL,
  seat_no varchar(4) NOT NULL,
  fare_conditions varchar(10) NOT NULL,
  CHECK (fare_conditions IN ('Economy', 'Conform', 'Business')),
  PRIMARY KEY (aircraft_code, seat_no),  
  FOREIGN KEY (aircraft_code) REFERENCES aircrafts (aircraft_code) ON DELETE CASCADE
);
```
- Первичный ключ - это ***составной***: комбинация атрибутов "Код самолета IATA" и "Номер места".Этот
первичный ключ будет ***ествественным****.
- Предложение `FOREIGN KEY` создает ограничени ссылочной целостности - ***внешний ключ***.
В качестве внешнего ключа служит аттрибут "Код самолета" (aircrft_code). Он ссылается на одноименный
  атрибут в таблице aircrafts.
- Таблица "Места" называется ссылающейся (referencing), а таблица aircrafts - ссылочной (referenced).
- Каскадное удаление - `ON DELETE CASCADE`.
***
### Что получилось?
`\d seats`

Таблица "public.seats"

| Колонка | Тип | Модификаторы
| --- | --- | --- |
aircraft_code | character(3) | NOT NULL
seat_no | character varying(4) | NOT NULL
fare_conditions | character varying(10) | NOT NULL

Индексы: `"seats_pkey" PRIMARY KEY, btree (aircraft_code, seat_no)`

Ограничения-проверки: `"seats_fare_conditions_check" CHECK (fare_conditions::text
= ANY (ARRAY['Economy'::character varying,
'Comfort'::character varying,
'Business'::character varying]::text[]))`


Ограничения внешнего ключа: `"seats_aircraft_code_fkey" FOREIGN KEY (aircraft_code)
REFERENCES aircrafts(aircraft_code) ON DELETE CASCADE`
***
### Какие таблицы есть в базе данных?
`\d`

Список отношений

| Схема | Имя | Тип | Владелец
| --- | --- | --- | --- |
public | aircrafts | таблица | postgres
public | seats | таблица | postgres
(2 строки) |
***
### Как работает внешний ключ
Выполните следующую команду для ввода данных в таблицу «Места»:

```SQL
INSERT INTO seats VALUES ( '123', '1A', 'Business' );
```

СУБД ответит так:

`ОШИБКА: INSERT или UPDATE в таблице "seats" нарушает
ограничение внешнего ключа "seats_aircraft_code_fkey"
ПОДРОБНОСТИ: Ключ (aircraft_code)=(123) отсутствует в
таблице "aircrafts".`

Это совершенно логично: если в таблице «Самолеты», на которую
ссылается таблица «Места», нет описания самолета с кодом самолета,
равным «123», то добавлять информацию о номерах кресел для такого —
несуществующего — самолета не имеет смысла. Так действует
поддержка правил ссылочной целостности со стороны СУБД.
***
### Заполнение данными таблицы «Места»

Для каждой модели самолетов введите только несколько строк, при этом
предусмотрите записи для классов обслуживания «Business» и
«Economy». С помощью одной команды `INSERT` можно ввести сразу
несколько строк:

```SQL
INSERT INTO seats
VALUES ( 'SU9', '1A', 'Business' ),
       ( 'SU9', '1B', 'Business' ),
       ( 'SU9', '10A', 'Economy' ),
       ( 'SU9', '10B', 'Economy' ),
       ( 'SU9', '10F', 'Economy' ),
       ( 'SU9', '20F', 'Economy' );
```

Затем измените значение атрибута aircraft_code на другое, например,
«773», и повторите команду `INSERT`. Так придется поступить со всеми
моделями самолетов
***
### Группирование строк
Предположим, что нам нужно получить информацию о количестве мест в
салонах для всех типов самолетов.
Нерациональное решение:
```SQL
SELECT count( * ) FROM seats WHERE aircraft_code = 'SU9';
```
```SQL
SELECT count( * ) FROM seats WHERE aircraft_code = 'CN1';
```
... ... ... ...

Рациональное решение:
```SQL
SELECT aircraft_code, count( * ) FROM seats
GROUP BY aircraft_code;
```

|aircraft_code | count
| --- | ---
773 | 402
733 | 130
CN1 | 12
CR2 | 50
319 | 116
SU9 | 97
321 | 170
763 | 222
320 | 140
(9 строк) |
*** 
### Сортировка полученной выборки
Если мы захотим отсортировать выборку по числу мест в самолетах, то
нужно будет дополнить команду предложением `ORDER BY`:
```SQL
SELECT aircraft_code, count( * ) FROM seats
GROUP BY aircraft_code
ORDER BY count;
```

| aircraft_code | count
| --- | ---
CN1 | 12
CR2 | 50
SU9 | 97
319 | 116
733 | 130
320 | 140
321 | 170
763 | 222
773 | 402
(9 строк) | 
***
### Более сложная задача
Подсчитать количество мест в салонах для всех моделей самолетов, но
теперь уже с учетом класса обслуживания (бизнес-класс и
экономический класс). В этом случае группировка выполняется уже по
двум атрибутам: aircraft_code и fare_conditions. 
```SQL
SELECT aircraft_code, fare_conditions, count( * )
FROM seats
GROUP BY aircraft_code, fare_conditions
ORDER BY aircraft_code, fare_conditions;
```

| aircraft_code | fare_conditions | count |
| --- | --- | --- |
319 | Business | 20
319 | Economy | 96
320 | Business | 20
320 | Economy | 120
(17 строк) |