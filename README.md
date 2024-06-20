# Домашнее задание к занятию «`Индексы`» - `Яковлев Артем`

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

---
## Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

```sql
select sum(data_length) as SUM_Data_Length, sum(index_length) as SUM_Index_Length, sum(index_length)*100.0/sum(data_length) as Persentage_ratio
from information_schema.tables
where table_schema='sakila' and data_length is not null;
```
![Скриншот выполнения запроса](https://github.com/temagraf/index/blob/main/img/12-5-1.png "Скриншот выполнения запроса")

---
## Задание 2

Выполните explain analyze следующего запроса:

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Узкие места и их устранение

1. Запрос слишком сложный и в нём используются избыточные таблицы
Из запроса убираем фрагмент `over (partition by c.customer_id, f.title)` и оператор `distinct`.
Так же убираем избыточные таблицы `film` и `inventory` :
```sql
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
group by c.last_name, c.first_name;
```
Время выполнения запроса сокращается более чем в 60 раз.

2. Можно выполнить объединение таблиц в запросе, использую оператор `join` :
```sql
select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from customer c
join rental r on r.customer_id = c.customer_id
join payment p on p.payment_date = r.rental_date and date(p.payment_date) = '2005-07-30'
group by c.last_name, c.first_name;
```

3. Для таблицы `payment` на столбец `payment_date` можно создать индекс : 
Смотрим существующие для таблицы индексы и создаём новый индекс
```sql
select *
from INFORMATION_SCHEMA.STATISTICS
where TABLE_NAME='payment';

create index idx_payment_date on payment(payment_date);

select concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where p.payment_date >= '2005-07-30 00:00:00' and p.payment_date < date_add('2005-07-30 00:00:00', interval 1 day) and p.payment_date = r.rental_date and r.customer_id = c.customer_id
group by c.last_name, c.first_name;

drop index idx_payment_date on payment;
```
### Скриншот анализа запроса без индекса

![Скриншот анализа запроса без индекса](https://github.com/temagraf/index/blob/main/img/12-5-2-1.png "Скриншот анализа запроса без индекса")

### Скриншот анализа запроса с индексом

![Скриншот анализа запроса с индексом](https://github.com/temagraf/index/blob/main/img/12-5-2-2.png "Скриншот анализа запроса с индесом")

---
## Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*

PostgreSQL поддерживает следующие типы индексов:
- B-Tree - в MySQL так же используется;
- Hash - в MySQL используется только в механизме хранения Memory;
- GiST (R-Tree) - в MySQL так же используется;
- SP-GiST
- GIN (Inverted) - в MySQL так же используется;
- BRIN
- bloom


Для разных типов индексов применяются разные алгоритмы, ориентированные на определённые типы запросов.
По умолчанию команда CREATE INDEX создаёт индексы типа B-Tree, эффективные в большинстве случаев.

---
