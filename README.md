# Описание

У вас SQL база с таблицами:  

1) Users(userId, age)  
2) Purchases (purchaseId, userId, itemId, date)  
3) Items (itemId, price)  

<br/>


Напишите SQL запросы для расчета следующих метрик:

А) какую сумму в среднем в месяц тратит:
- пользователи в возрастном диапазоне от 18 до 25 лет включительно 
- пользователи в возрастном диапазоне от 26 до 35 лет включительно 

Б) в каком месяце года выручка от пользователей в возрастном диапазоне 35+ самая большая  
В) какой товар обеспечивает дает наибольший вклад в выручку за последний год  
Г) топ-3 товаров по выручке и их доля в общей выручке за любой год  

<br/>
<br/>


# Решение

Задание было выполнено на PostgreSQL v.15.3 с использованием DBeaver Community v.23.1.1. 

База данных была создана через графический интерфейс DBeaver. Схемы для данного задания не создавались. 

После подключения к базе данных необходимо запустить скрипт для создания таблиц в ней:

```
CREATE TABLE IF NOT EXISTS users (
userId int PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
age int NOT NULL
);

CREATE TABLE IF NOT EXISTS items (
itemId int PRIMARY KEY GENERATED ALWAYS AS IDENTITY, 
price decimal NOT NULL
);

CREATE TABLE IF NOT EXISTS purchases (
purchaseId int PRIMARY KEY GENERATED ALWAYS AS IDENTITY, 
userId int NOT NULL REFERENCES users(userId),
itemId int NOT NULL REFERENCES items(itemId),
date timestamptz NOT NULL DEFAULT now()
);
```

Модель созданной БД показана на картинке ниже:

![Image alt](https://github.com/yotarov/postgres/blob/master/images/1.png)


<br/>
<br/>

Скрипт для заполнения таблиц данными показан ниже:

```
INSERT INTO users(age)
VALUES (17),
       (15),
       (18),
       (22),
       (28),
       (24),
       (30),
       (35),
       (48),
       (63),
       (45),
       (33),
       (38),
       (55),
       (39),
       (38),
       (30),
       (35),
       (48),
       (67);
      
      
INSERT INTO items(price)
VALUES (350.00),
       (1200.95),
       (101456.00),
       (123000.00),
       (345.59),
       (34670.00),
       (3000),
       (3595.99),
       (2999),
       (134567.0),
       (4500),
       (234120.25),
       (3.99),
       (15.75),
       (23),
       (12000),
       (240000),
       (76500),
       (4834.34),
       (314.1);
      
      
INSERT INTO purchases(userid, itemid, date)
VALUES (6, 19, '2022-02-24'),
       (7, 17, '2023-01-17 16:43:17-6'),
       (1, 9, '2023-02-16 19:37:47-6'),
       (4, 20, DEFAULT),
       (1, 3, '2023-04-20 00:22:07-6'),
       (8, 6, '2022-03-16 13:05:05-6'),
       (6, 9, '2021-07-15 22:01:02-6'),
       (10, 11, '2023-04-01 15:22:10-6'),
       (19, 2, '2023-02-23 23:11:16-6'),
       (4, 4, '2022-12-1 17:03:04-6'),
       (5, 8, DEFAULT),
       (7, 19, '2023-05-17 12:03:01-6'),
       (13, 5, '2021-03-11 10:45:00-6'),
       (12, 4, '2022-04-03 11:34:09-6'),
       (11, 10, '2023-11-07 20:19:01-6'),
       (4, 3, DEFAULT),
       (4, 20, DEFAULT),
       (1, 19, '2023-04-20 00:22:07-6'),
       (8, 3, '2022-03-16 13:05:05-6'),
       (6, 3, '2021-07-15 22:01:02-6'),
       (10, 3, '2023-04-01 15:22:10-6'),
       (19, 12, '2023-02-23 23:11:16-6'),
       (4, 12, '2022-12-1 17:03:04-6'),
       (5, 8, DEFAULT),
       (7, 12, '2023-05-17 12:03:01-6'),
       (13, 5, '2021-03-11 10:45:00-6'),
       (12, 3, '2022-04-03 11:34:09-6'),
       (11, 10, '2023-11-07 20:19:01-6'),
       (4, 13, DEFAULT);
```

<br/>
<br/>

## Приступаем к решению задач

<br/>

### А) какую сумму в среднем в месяц тратит:
### - пользователи в возрастном диапазоне от 18 до 25 лет включительно

```
SELECT sum(i.price)/count(DISTINCT date_trunc('month', p."date")) AS avg_monthly_spending
FROM items i 
JOIN purchases p 
ON i.itemid = p.itemid 
JOIN users u 
ON u.userid = p.userid 
WHERE u.age >= 18 AND u.age <=25;
```

### Ответ: 142124.445

<br/>
<br/>

### - пользователи в возрастном диапазоне от 26 до 35 лет включительно

<br/>

Чтобы решить данную под задачу, нужно изменить условие ```WHERE``` на:
```
WHERE u.age >= 26 AND u.age <=35;
```

![Image alt](https://github.com/yotarov/postgres/blob/master/images/2.png)

### Ответ: 169345.714

<br/>
<br/>

### Б) в каком месяце года выручка от пользователей в возрастном диапазоне 35+ самая большая 
```
SELECT to_char(p."date", 'YYYY-MM') AS "month",
       sum(i.price) AS max_revenue
FROM items i 
JOIN purchases p 
ON i.itemid = p.itemid 
JOIN users u 
ON u.userid = p.userid 
WHERE u.age >= 35
GROUP BY month
ORDER BY max_revenue DESC
LIMIT 3;
```

Я посмотрел первые три значения на случай если будут несколько месяцев с одинаковыми или близкими значениями

![Image alt](https://github.com/yotarov/postgres/blob/master/images/3.png)

### Ответ: Ноябрь 2023 года (сори, при заполнении таблиц данными, случайно ушел в будущее :3 )

<br/>
<br/>

### В) какой товар обеспечивает дает наибольший вклад в выручку за последний год

<br/>

Не совсем понятно что имеется ввиду под формулировкой "последний год": период с начала текущего года по сегодняшний день или 12-месячный период от текущей даты. Поэтому я предлагаю на выбор 2 решения для каждого из кейсов:

- текущий год

```
SELECT i.itemid, sum(i.price) AS max_revenue
FROM items i
JOIN purchases p 
ON i.itemid = p.itemid 
WHERE to_char(p."date", 'YYYY') = '2023'
GROUP BY i.itemid
ORDER BY max_revenue DESC
LIMIT 3;
```

Также смотрю 3 товара, чтобы удостовериться наверняка
![Image alt](https://github.com/yotarov/postgres/blob/master/images/4.png)

### Ответ: товар с id = 12

<br/>

- 12 месяцеа от текущей даты

```
SELECT i.itemid, sum(i.price) AS max_revenue
FROM items i
JOIN purchases p 
ON i.itemid = p.itemid 
WHERE p."date" >= now() - INTERVAL '12 month'
GROUP BY i.itemid
ORDER BY max_revenue DESC
LIMIT 3;
```

![Image alt](https://github.com/yotarov/postgres/blob/master/images/5.png)

### Ответ: товар с id = 12

<br/>
<br/>

### Г) топ-3 товаров по выручке и их доля в общей выручке за любой год  

```
SELECT i.itemid, round(sum(i.price) * 100 / (SELECT sum(i.price) AS tot_rev
	                            FROM items i
	                            JOIN purchases p 
	                            ON i.itemid = p.itemid 
	                            WHERE to_char(p."date", 'YYYY') = '2022'), 2) AS "revenue_share, %"
FROM items i
JOIN purchases p 
ON i.itemid = p.itemid 
WHERE to_char(p."date", 'YYYY') = '2022'
GROUP BY i.itemid 
ORDER BY "revenue_share, %" DESC
LIMIT 3;
```

### Ответ:
![Image alt](https://github.com/yotarov/postgres/blob/master/images/6.png)

<br/>
<br/>

---

<br/>

Выполнил: Ерасыл Отаров  
E-mail: otarov.logigue@gmail.com  
Mob. / WA: +7 705 528 17 17  
TG: @watasinomae  









