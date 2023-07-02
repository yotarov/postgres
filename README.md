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

Задание было выполнено на PostgreSQL v.15.3. с использованием DBeaver Community v.23.1.1. 

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

В итоге будут созданы таблицы 

