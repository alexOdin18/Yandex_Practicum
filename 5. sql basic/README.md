##Раздел "Базовый SQL". Типичные задачи по базе данных.
[Схема базы данных](/database_schema.png)

1. Посчитайте, сколько компаний закрылось.
```
SELECT COUNT(id)
FROM company
WHERE status = 'closed'
```
2. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .
```
WITH
i AS (SELECT id
FROM company
WHERE country_code = 'USA'
      AND category_code = 'news'),
w AS (SELECT id,
       funding_total
FROM company)

SELECT w.funding_total
FROM i LEFT JOIN w ON i.id = w.id
ORDER BY w.funding_total DESC;

```
3. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.
```
SELECT SUM(price_amount)
FROM acquisition
WHERE EXTRACT(year FROM CAST(acquired_at AS date)) BETWEEN 2011 AND 2013
      AND term_code = 'cash'

```
4. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.
```
SELECT first_name,
       last_name,
       twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%'

```
5. Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.
```
SELECT *
FROM people
WHERE twitter_username LIKE '%money%'
      AND last_name LIKE 'K%'
```
6. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.
```

SELECT country_code,
       SUM(funding_total)
FROM company
GROUP BY country_code
ORDER BY SUM(funding_total) DESC;
```
7. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.
```
WITH
i AS (SELECT CAST(funded_at AS date) AS date_round,
       MIN(raised_amount) AS min_raised
FROM funding_round
GROUP BY date_round),
w AS (SELECT CAST(funded_at AS date) AS date_round,
       MAX(raised_amount) AS max_raised
FROM funding_round
GROUP BY date_round)

SELECT i.date_round,
       i.min_raised,
       w.max_raised
FROM i INNER JOIN w ON i.date_round = w.date_round
WHERE i.min_raised != 0 AND i.min_raised != w.max_raised

```
8. Создайте поле с категориями:
Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
Отобразите все поля таблицы fund и новое поле с категориями.
```
SELECT *,
        CASE
            WHEN invested_companies >=100 THEN 'high_activity'
            WHEN invested_companies >=20 AND invested_companies <100 THEN 'middle_activity'
            WHEN invested_companies <20 THEN 'low_activity'
        END
FROM fund
```
9. Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.
```
SELECT 
       CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies>=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity,
       ROUND(AVG(investment_rounds))
FROM fund
GROUP BY activity 
ORDER BY ROUND(AVG(investment_rounds));
```
10. Выгрузите таблицу с десятью самыми активными инвестирующими странами. Активность страны определите по среднему количеству компаний, в которые инвестируют фонды этой страны.
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды, основанные с 2010 по 2012 год включительно.
Исключите из таблицы страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Отсортируйте таблицу по среднему количеству компаний от большего к меньшему.
Для фильтрации диапазона по годам используйте оператор BETWEEN.
```
SELECT country_code,
       MIN(invested_companies),
       MAX(invested_companies),
       AVG(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) BETWEEN 2010 AND 2012
GROUP BY country_code
HAVING MIN(invested_companies) != 0
ORDER BY AVG(invested_companies) DESC
LIMIT 10;
```
11. Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.
```
SELECT first_name,
       last_name,
       ed.instituition
FROM people AS p 
LEFT JOIN education AS ed ON p.id = ed.person_id
```
12. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.
```
WITH
i AS (SELECT id,
       name
FROM company
WHERE name IS NOT NULL),
w AS (SELECT id,
       company_id
FROM people),
e AS (SELECT person_id,
             instituition
      FROM education)

SELECT i.name,
       COUNT(DISTINCT(e.instituition))
FROM i 
LEFT JOIN w ON i.id = w.company_id
LEFT JOIN e ON w.id = e.person_id
GROUP BY i.name
ORDER BY COUNT(DISTINCT(e.instituition)) DESC
LIMIT 5;
```
13. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.
```
WITH
i AS (SELECT DISTINCT(company_id) AS id
FROM funding_round
WHERE is_first_round = 1
  AND is_last_round = 1
ORDER BY company_id)

SELECT c.name
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed'
```
14. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.
```
WITH
i AS (SELECT DISTINCT(company_id) AS id
FROM funding_round
WHERE is_first_round = 1
  AND is_last_round = 1
ORDER BY company_id),
w AS (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed')



SELECT id
FROM people
WHERE company_id IN (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed')
```
15. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.
```
WITH
i AS (SELECT DISTINCT(company_id) AS id
FROM funding_round
WHERE is_first_round = 1
  AND is_last_round = 1
ORDER BY company_id),
w AS (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed'),
f AS (SELECT id
FROM people
WHERE company_id IN (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed'))

SELECT f.id,
       ed.instituition
FROM education AS ed
INNER JOIN f ON ed.person_id = f.id
```
16. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания.
```
WITH
i AS (SELECT DISTINCT(company_id) AS id
FROM funding_round
WHERE is_first_round = 1
  AND is_last_round = 1
ORDER BY company_id),
w AS (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed'),
f AS (SELECT id
FROM people
WHERE company_id IN (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed'))

SELECT f.id,
       COUNT(ed.instituition)
FROM education AS ed
INNER JOIN f ON ed.person_id = f.id
GROUP BY f.id
```
17. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.
```
WITH
i AS (SELECT DISTINCT(company_id) AS id
FROM funding_round
WHERE is_first_round = 1
  AND is_last_round = 1
ORDER BY company_id),
w AS (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed'),
f AS (SELECT id
FROM people
WHERE company_id IN (SELECT c.id
FROM i
LEFT JOIN company AS c ON i.id=c.id
WHERE c.status = 'closed')),
v AS (SELECT f.id,
       COUNT(ed.instituition) AS amount_inst
FROM education AS ed
INNER JOIN f ON ed.person_id = f.id
GROUP BY f.id)

SELECT AVG(amount_inst)
FROM v
```
18. Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники компании Facebook.
```
WITH
i AS (SELECT id
FROM people
WHERE company_id = 5),
w AS (SELECT person_id,
       COUNT(instituition) AS amount_inst
FROM education
WHERE person_id IN (SELECT id
FROM people
WHERE company_id = 5)
GROUP BY person_id)

SELECT AVG(w.amount_inst)
FROM w
```
19. Составьте таблицу из полей:
*name_of_fund* — название фонда;
*name_of_company* — название компании;
*amount* — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.
```
WITH
com AS (SELECT id,
       name AS name_of_company
FROM company
WHERE milestones > 6),
fundround AS (SELECT id,
       raised_amount AS amount
FROM funding_round
WHERE EXTRACT(YEAR FROM CAST(funded_at AS date)) BETWEEN 2012 AND 2013),
name_fund AS (SELECT id AS id_fund,
       name AS name_of_fund
    FROM fund)
    
SELECT name_fund.name_of_fund,
       com.name_of_company,
       fundround.amount
FROM investment AS inv
INNER JOIN com ON inv.company_id = com.id
INNER JOIN name_fund ON inv.fund_id = name_fund.id_fund
INNER JOIN fundround ON inv.funding_round_id = fundround.id
```
20. Выгрузите таблицу, в которой будут такие поля:
* название компании-покупателя;
* сумма сделки;
* название компании, которую купили;
* сумма инвестиций, вложенных в купленную компанию;
* доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в алфавитном порядке. Ограничьте таблицу первыми десятью записями.
```
WITH 
comp AS (SELECT id,
       name AS company_name,
       funding_total
FROM company
WHERE funding_total != 0),
comp2 AS (SELECT id,
       name AS company_name
FROM company)

SELECT comp2.company_name,
       ac.price_amount,
       comp.company_name,
       comp.funding_total,
       ROUND(ac.price_amount / comp.funding_total)
FROM acquisition AS ac
INNER JOIN comp ON comp.id = ac.acquired_company_id
INNER JOIN comp2 ON comp2.id = ac.acquiring_company_id
WHERE ac.price_amount != 0
ORDER BY ac.price_amount DESC
LIMIT 10;
```
21. Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Выведите также номер месяца, в котором проходил раунд финансирования.
```
WITH
com AS (SELECT id,
       name
FROM company
WHERE category_code = 'social')

SELECT com.name,
       EXTRACT(MONTH FROM CAST(fr.funded_at AS date))
FROM funding_round AS fr
INNER JOIN com ON fr.company_id = com.id
WHERE EXTRACT(YEAR FROM CAST(fr.funded_at AS date)) BETWEEN 2010 AND 2013
```
22. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
* номер месяца, в котором проходили раунды;
* количество уникальных названий фондов из США, которые инвестировали в этом месяце;
* количество компаний, купленных за этот месяц;
* общая сумма сделок по покупкам в этом месяце.
```
WITH
--Раунды финансирования с 2010 по 2013 по месяцам 
fr AS (SELECT id,
       EXTRACT(month FROM CAST(funded_at AS date)) AS month_round
FROM funding_round
WHERE EXTRACT(YEAR FROM CAST(funded_at AS date)) BETWEEN 2010 AND 2013),
--Фонды из США 
fund AS (SELECT id,
       name
FROM fund
WHERE country_code = 'USA'),
-- количество компаний, купленных по месяцам с 2010 по 2013 и сумма покупок 
pur_month AS (SELECT EXTRACT(month FROM CAST(acquired_at AS date)) AS month_pur,
       COUNT(acquired_company_id) AS amount_pur,
       SUM(price_amount) AS total_pur_month
       FROM acquisition
       WHERE EXTRACT(YEAR FROM CAST(acquired_at AS date)) BETWEEN 2010 AND 2013
       GROUP BY month_pur),
count_fund AS (SELECT fr.month_round AS month_round,
       COUNT(DISTINCT(fund.name)) AS count_fund
FROM investment AS inv
INNER JOIN fr ON fr.id = inv.funding_round_id
INNER JOIN fund ON fund.id = inv.fund_id
GROUP BY fr.month_round)

SELECT cf.month_round,
       cf.count_fund,
       pur_month.amount_pur,
       pur_month.total_pur_month
FROM count_fund AS cf
INNER JOIN pur_month ON pur_month.month_pur = cf.month_round
```
23. Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.
```
WITH
y11 AS (SELECT country_code,
       AVG(funding_total) AS year_2011 
FROM company
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date))  = 2011
GROUP BY country_code),
y12 AS (SELECT country_code,
       AVG(funding_total) AS year_2012
FROM company
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date))  = 2012
GROUP BY country_code),
y13 AS (SELECT country_code,
       AVG(funding_total) AS year_2013
FROM company
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date))  = 2013
GROUP BY country_code)

SELECT y11.country_code,
       y11.year_2011,
       y12.year_2012,
       y13.year_2013
FROM y11
INNER JOIN y12 ON y11.country_code = y12.country_code
INNER JOIN y13 ON y11.country_code = y13.country_code
ORDER BY year_2011 DESC;
```
