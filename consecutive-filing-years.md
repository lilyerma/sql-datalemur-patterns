# [Последовательные годы подачи отчетности](https://datalemur.com/questions/consecutive-filing-years)

Компания Intuit, известная своими продуктами для подачи налоговых деклараций, такими как TurboTax и QuickBooks, предлагает несколько версий этих продуктов.

Напишите SQL-запрос, который определяет идентификаторы пользователей (user ID), подававших налоговые декларации с использованием любой версии TurboTax в течение трех или более лет подряд. Каждый пользователь может подать декларацию с помощью конкретного продукта только один раз в год. Выведите результат в порядке возрастания user ID.

Приведу 2 решения на базе Gaps and Islands. 

## Решение 1

Это решение мне нравится больше. Суть такая что при вычитании из последовательно  равномерно пронумерованного списка другого последовательно равномерно пронумерованного списка (из последовательности годов, номера рядов) - разность будет одинаковая. Т.е. ее можно использовать как номер "острова".

```SQL
WITH ranked AS (
  SELECT 
    user_id,
    EXTRACT(YEAR FROM filing_date) - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY EXTRACT(YEAR FROM filing_date)) as strike
  FROM filed_taxes
  WHERE product ilike 'TurboTax%')
SELECT DISTINCT user_id
  FROM ranked 
  GROUP BY user_id, streak
HAVING COUNT(*) >=3
ORDER BY user_id ASC
```
## Решение 2

Второй вариант - поставить флаг начала периода, всем событиям внутри периода проставить 0, сделать running сумму по порядку и таким образом всем событиям проставится порядковый номер. 

```SQL
WITH filtered AS (
    SELECT
        user_id,
        EXTRACT(YEAR FROM filing_date) as year
    FROM filed_taxes
    WHERE product ilike 'TurboTax%'
),
flagged AS (
  SELECT
    user_id,
    year, 
    CASE WHEN year - LAG(year) OVER w > 1
         OR LAG(year) OVER w IS NULL
             THEN 1 ELSE 0 END AS is_start
    FROM filtered
    WINDOW w AS (PARTITION BY user_id ORDER BY year)
    ),
numbered AS (    
SELECT
    user_id,
    year,
    SUM(is_start) OVER (PARTITION BY user_id ORDER BY year) AS streak_no
FROM flagged
  )
SELECT 
  distinct user_id
FROM numbered 
GROUP BY user_id, strike_no
HAVING COUNT(*) >=3
ORDER BY user_id ASC
```

