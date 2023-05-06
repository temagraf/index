# Домашнее задание к занятию «`Индексы`» - `Барановский Станислав`

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
![Скриншот выполнения запроса](https://github.com/StanislavBaranovskii/12-5-hw/blob/main/img/12-5-1.png "Скриншот выполнения запроса")

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


В простом случае, индекс необходимо создавать для тех столбцов, которые присутствуют в условии отбора where или по которым выполняется сотрировка order


При анализе запроса с использованием оператора explain видим, что в запросе к талицам film и payment не используется ни один индекс (стобец key), так же отсутсвуют какие либо индексы, которые могут быть использованы для этого запроса (стобец possibly_keys). Столбец rows показывае число записей, которые пришлось прочитать базе данных для выполнения данного запроса.
![Скриншот выполнения запроса с explain](https://github.com/StanislavBaranovskii/12-5-hw/blob/main/img/12-5-2-1.png "Скриншот выполнения запроса с explain")

SELECT *
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_NAME='payment';

CREATE INDEX idx_payment_date ON payment(payment_date);

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