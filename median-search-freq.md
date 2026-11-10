# [Задача про медиану от Google](https://datalemur.com/questions/median-search-freq)

Отдел маркетинга Google готовит рекламный ролик и хочет добавить в него простую статистику: медианное количество поисковых запросов, сделанных одним пользователем за прошлый год.

Однако в масштабах Google выполнение запросов по всем 2 триллионам поисков обходится слишком дорого. К счастью, у вас есть доступ к агрегированной сводной таблице, которая показывает количество поисковых запросов за прошлый год и сколько пользователей попало в каждую категорию.

Напишите SQL-запрос для вычисления медианного количества поисков на одного пользователя. Округлите медиану до одного знака после запятой.

Задача на самом деле интересна тем, какое решение ей предложили на datalemur.com. Я не буду его копировать, можно посмотреть на сайте. Но суть такая, что там товарищи предложили развернуть категории вот так: 

```sql 
WITH searches_expanded AS (
  SELECT searches
  FROM search_frequency
  GROUP BY 
    searches, 
    GENERATE_SERIES(1, num_users))
```

Разворачивание нужно, чтобы применить PERCENTILE_CONT(0.50). НО! Нам в условии задачи сообщили, что там 2 триллиона поисков, и разворачивать, возможно, не очень хорошая идея. 

*Что тут стоит запомнить:*

- PERCENTILE_CONT() нужно применять к "сырым данным". На категориях он даст неверный результат.
- Для небольших значений можно воспользоваться таким unpivoting / unaggregating. 

Еще один вариант:

```sql 
SELECT 
    f.searches
FROM search_frequency f
CROSS JOIN LATERAL GENERATE_SERIES(1, f.num_users);
```

А так решаем задачу простым и надежным способом.

```sql
WITH cte as(
  SELECT
    *,
    SUM(num_users) OVER(ORDER BY searches) as running_sum_users,
    SUM(num_users) OVER() as total_users
  FROM search_frequency
)

SELECT
  ROUND(AVG(searches), 1) as median
FROM cte
WHERE total_users <= running_sum_users * 2 AND
  total_users >= (running_sum_users - num_users) * 2
  ```