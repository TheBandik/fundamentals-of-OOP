# Введение в SQL

## Подключение к удаленной СУБД

```
mysql -h [имя хоста] -u [имя пользователя] -p
```

Реквизиты для подключения к БД:
+ Имя хоста: 95.154.68.63
+ Имя пользователя: student
+ Пароль: Hn523cv-

```
USE [имя БД] - установить БД с указанным именем в качесте текущей
SHOW TABLES - показать список таблиц в текущей БД
SHOW TABLES FROM [имя БД] - показать список таблиц в указанной БД
SHOW FIELDS FROM [имя таблицы] - показать список полей из указанной таблицы
DESC [имя таблицы] - альтернативный вариант для SHOW TABLES FROM
SELECT COUNT(*) FROM [имя таблицы] - показать количество строк в указанной таблице
```

## Выборка данных

### Вывод данных таблицы

```sql
select * from <имя таблицы>;
select * from <имя таблицы> limit <количество строк>;
select * from <имя таблицы> limit <с какой записи>, <количество строк>;

/* Примеры */

select * from companies;
select * from companies limit 10;
select * from companies limit 5, 10;
```

### Определение списка полей результата

```sql
select <список полей> from <имя таблицы>;

/* Пример */

select id, name, prefix from regions limit 2;
```

### Сортировка

```sql
select ... order by <поле [порядок]>;

/* Порядок: asc или desc, по умолчанию asc */

/* Примеры */
select * from games order by year_release;
select * from games order by year_release desc;
```

### Фильтрация

```sql
select ... from ... where <логическое выражение>;

/* Под логическим выражением может быть: */
/* <поле> in <значения> - проверка вхождения значения поля во множество */
/* значений */
/* <поле> is null - проверка на значение null */
/* <поле> like <значение> - проверка на соответствие строки маске */
/* <поле> between <начало> and <конец> - проверка значения на вхождение */
/* в диапазон */

/* Примеры */

select id, name, rating, year_release from games where id = 3;
select id, name, rating, year_release from games where rating = 100;
select id, name, rating, year_release from games where rating = 94 
and year_release = 2014;
select id, name, rating, year_release from games where rating 
between 60 and 70;

select id, name from games where name = 'Quantum Break';
select id, name from games where name like '%halo%';
```

## Подзапросы

```sql
<Запрос> in (<запрос>)

-- Пример

select id, name, sales_worldwide, developer_id from games where 
sales_worldwide > 50000000 and developer_id in (select id 
from companies where name = 'Valve');
```

## Агрегаторы

```sql
-- avg() - среднее арифметическое
-- max() - максимальное значение
-- min() - минимальное значение
-- sum() - сумма значений
-- count() - количество значений

-- Пример

select min(rating) from games where year_release = 2008;
```

## Группировка данных

```sql
-- Групповые функции count, max, min применяются только к не NULL значениям

select <список полей> ... group by <список полей группировки> 
having <логическое выражение>

-- Пример

select year_release, count(*) from games group by year_release having 
year_release < 2010

-- Можно переименовать столбцы результата

select <список полей> as <новое название>

-- Пример

select year_release, count(*) as count_of_games from games group by 
year_release having year_release < 2010
```