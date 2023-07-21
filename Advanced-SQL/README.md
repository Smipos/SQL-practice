Задания + код решения

<details>
      <summary><strong>Первая часть.</strong></summary>

* Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».
``` sql
SELECT COUNT(p.id)
FROM stackoverflow.posts p
JOIN stackoverflow.post_types pt ON p.post_type_id = pt.id
WHERE (p.favorites_count >= 100
      OR
      p.score > 300)
      AND
      pt.type = 'Question'
```

* Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлите до целого числа.
``` sql
WITH total_q AS
(
    SELECT COUNT(p.id) count_questions,
       EXTRACT(DAY FROM p.creation_date::date) date_of_question
    FROM stackoverflow.posts p
    JOIN stackoverflow.post_types pt ON p.post_type_id = pt.id
    WHERE pt.type = 'Question'
          AND
          p.creation_date::date BETWEEN '2008-11-01' AND '2008-11-18'
    GROUP BY 2
)

SELECT ROUND(AVG(count_questions))
FROM total_q
```

* Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.
``` sql
SELECT COUNT(DISTINCT u.id)
FROM stackoverflow.users u
JOIN stackoverflow.badges b ON u.id = b.user_id
WHERE u.creation_date::date = b.creation_date::date
```

* Сколько уникальных постов пользователя с именем `Joel Coehoorn` получили хотя бы один голос?
``` sql
SELECT COUNT(DISTINCT p.id)
FROM stackoverflow.users u
JOIN stackoverflow.posts p ON u.id = p.user_id
JOIN stackoverflow.votes v ON p.id = v.post_id
WHERE u.display_name = 'Joel Coehoorn'
HAVING COUNT(v.id) >= 1
```

* Выгрузите все поля таблицы `vote_types`. Добавьте к таблице поле `rank`, в которое войдут номера записей в обратном порядке. Таблица должна быть отсортирована по полю `id`.
``` sql
SELECT *,
       ROW_NUMBER() OVER(ORDER BY id DESC) rank
FROM stackoverflow.vote_types 
ORDER BY id
```

* Отберите 10 пользователей, которые поставили больше всего голосов типа `Close`. Отобразите таблицу из двух полей: идентификатором пользователя и количеством голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.
``` sql
SELECT DISTINCT u.id,
       COUNT(v.id) OVER (PARTITION BY u.id) close_count
FROM stackoverflow.users u
JOIN stackoverflow.votes v ON u.id = v.user_id
JOIN stackoverflow.vote_types vt ON v.vote_type_id = vt.id
WHERE vt.name = 'Close'
ORDER BY close_count DESC, u.id DESC
LIMIT 10
```

* Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
  Отобразите несколько полей:
  * идентификатор пользователя;
  * число значков;
  * место в рейтинге — чем больше значков, тем выше рейтинг.

Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.
Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.
``` sql
SELECT DISTINCT u.id,
       COUNT(b.id) total_badges, 
       DENSE_RANK() OVER(ORDER BY COUNT(b.id) DESC) rating
FROM stackoverflow.users u
JOIN stackoverflow.badges b ON u.id = b.user_id
WHERE b.creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'
GROUP BY u.id
ORDER BY total_badges DESC, u.id
LIMIT 10
```

* Сколько в среднем очков получает пост каждого пользователя?
  
  Сформируйте таблицу из следующих полей:
  * заголовок поста;
  * идентификатор пользователя;
  * число очков поста;
  * среднее число очков пользователя за пост, округлённое до целого числа.
  
 Не учитывайте посты без заголовка, а также те, что набрали ноль очков.
``` sql
SELECT title,
       user_id,
       score,
       ROUND(AVG(score) OVER (PARTITION BY user_id))
FROM stackoverflow.posts
WHERE title IS NOT NULL
      and
      score <> 0
```

* Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.
``` sql
SELECT p.title
FROM stackoverflow.posts p
JOIN stackoverflow.users u ON p.user_id = u.id
JOIN stackoverflow.badges b ON u.id = b.user_id
WHERE p.title IS NOT NULL
GROUP BY p.title
HAVING COUNT(b.id) > 1000
```

* Напишите запрос, который выгрузит данные о пользователях из Канады.
  
  Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
  * пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
  * пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
  * пользователям с числом просмотров меньше 100 — группу 3.
    
  Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.
``` sql
SELECT u.id,
       u.views,
       CASE
           WHEN u.views >= 350 THEN 1
           WHEN u.views >= 100 AND u.views < 350 THEN 2
           WHEN u.views < 100 THEN 3
       END group_user
FROM stackoverflow.users u
WHERE u.location LIKE '%Canada%'
      AND
      u.views <> 0 
```

* Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.
``` sql
WITH grouped AS
(
    SELECT u.id,
       u.views,
       CASE
           WHEN u.views >= 350 THEN 1
           WHEN u.views >= 100 AND u.views < 350 THEN 2
           WHEN u.views < 100 THEN 3
       END group_user
    FROM stackoverflow.users u
    WHERE u.location LIKE '%Canada%'
          AND
          u.views <> 0 
),
sorted AS
(
    SELECT *,
           MAX(views) OVER (PARTITION BY group_user) max_views
    FROM grouped
)

SELECT id,
       group_user,
       max_views
FROM sorted
WHERE views = max_views
ORDER BY views DESC, id 
```

* Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года.
  
  Сформируйте таблицу с полями:
  * номер дня;
  * число пользователей, зарегистрированных в этот день;
  * сумму пользователей с накоплением.
``` sql
SELECT DISTINCT EXTRACT(DAY FROM creation_date::date) day_number,
       COUNT(id) OVER (ORDER BY EXTRACT(DAY FROM creation_date)),
       COUNT(id) OVER (PARTITION BY EXTRACT(DAY FROM creation_date))
FROM stackoverflow.users
WHERE creation_date::Date BETWEEN '2008-11-01' AND '2008-11-30'
```

* Для каждого пользователя, который написал хотя бы один пост, найдите интервал между регистрацией и временем создания первого поста.

  Отобразите:
  * идентификатор пользователя;
  * разницу во времени между регистрацией и первым постом.
``` sql
SELECT DISTINCT u.id,
       (MIN(p.creation_date) OVER (PARTITION BY u.id)) - u.creation_date
FROM stackoverflow.users u 
JOIN stackoverflow.posts p ON u.id = p.user_id
```
</details>

<details>
      <summary><strong>Вторая часть.</strong></summary>

* Выведите общую сумму просмотров у постов, опубликованных в каждый месяц 2008 года. Если данных за какой-либо месяц в базе нет, такой месяц можно пропустить. Результат отсортируйте по убыванию общего количества просмотров.
``` sql
SELECT DISTINCT SUM(views_count) OVER (PARTITION BY DATE_TRUNC('month', creation_date)) total_views,
        DATE_TRUNC('month', creation_date)::date
FROM stackoverflow.posts
ORDER BY total_views DESC
```

* Выведите имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов. Вопросы, которые задавали пользователи, не учитывайте. Для каждого имени пользователя выведите количество уникальных значений `user_id`. Отсортируйте результат по полю с именами в лексикографическом порядке.
``` sql
SELECT u.display_name,
       COUNT(DISTINCT p.user_id)
FROM stackoverflow.posts p
JOIN stackoverflow.users u ON p.user_id = u.id
JOIN stackoverflow.post_types pt ON p.post_type_id = pt.id
WHERE pt.type = 'Answer'
      AND
      DATE_TRUNC('day', p.creation_date::date) <= DATE_TRUNC('day', u.creation_date::date) + INTERVAL '1 month'
GROUP BY u.display_name
HAVING COUNT(p.id) > 100
ORDER BY u.display_name
```

* Выведите количество постов за 2008 год по месяцам. Отберите посты от пользователей, которые зарегистрировались в сентябре 2008 года и сделали хотя бы один пост в декабре того же года. Отсортируйте таблицу по значению месяца по убыванию.
``` sql
WITH users AS
(
    SELECT user_id
    FROM stackoverflow.posts
    WHERE DATE_TRUNC('month',creation_date)::DATE BETWEEN '2008-12-01' AND '2008-12-31'
    GROUP BY user_id
    HAVING COUNT(id) >= 1
),
pp AS
(
SELECT u.id,
       u.creation_date dt
FROM stackoverflow.users u
JOIN users ON u.id = users.user_id
WHERE DATE_TRUNC('month',creation_date)::DATE BETWEEN '2008-09-01' AND '2008-09-30'
    
)

SELECT DISTINCT COUNT(pp.id) OVER (PARTITION BY DATE_TRUNC('month',po.creation_date)::DATE),
       DATE_TRUNC('month',po.creation_date)::DATE mt_date
FROM stackoverflow.posts po
JOIN pp ON po.user_id = pp.id
ORDER BY mt_date DESC
```

* Используя данные о постах, выведите несколько полей:
  
      * идентификатор пользователя, который написал пост;
      * дата создания поста;
      * количество просмотров у текущего поста;
      * сумма просмотров постов автора с накоплением.
  
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, а данные об одном и том же пользователе — по возрастанию даты создания поста.
``` sql
SELECT DISTINCT user_id,
       creation_date,
       views_count,
       SUM(views_count) OVER (PARTITION BY user_id ORDER BY creation_date)
FROM stackoverflow.posts
ORDER BY user_id
```

* Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой? Для каждого пользователя отберите дни, в которые он или она опубликовали хотя бы один пост. Нужно получить одно целое число — не забудьте округлить результат.
``` sql
WITH p AS
(
    SELECT DISTINCT COUNT(id) OVER (PARTITION BY user_id, creation_date::date) count_actions,
           creation_date::date,
           user_id
    FROM stackoverflow.posts
    WHERE creation_date::date BETWEEN '2008-12-01' AND '2008-12-07'
)

SELECT ROUND(AVG(count_actions))
FROM p
```

* На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года?

  Отобразите таблицу со следующими полями:
  * Номер месяца.
  * Количество постов за месяц.
  * Процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим. Не забудьте сравнить количество постов в сентябре со значением предыдущего месяца.

Если постов стало меньше, значение процента должно быть отрицательным, если больше — положительным. Округлите значение процента до двух знаков после запятой.
``` sql
WITH count_p AS
(
    SELECT DISTINCT EXTRACT(MONTH FROM creation_date::DATE) month_number,
       COUNT(id) OVER (PARTITION BY EXTRACT(MONTH FROM creation_date::DATE)) total_posts
    FROM stackoverflow.posts
    WHERE creation_date::date BETWEEN '2008-09-01' AND '2008-12-31'
)

SELECT month_number,
       total_posts,
       ROUND((total_posts::numeric * 100 / LAG (total_posts) OVER (ORDER BY total_posts DESC) - 100), 2)
FROM count_p
```

* Найдите пользователя, который опубликовал больше всего постов за всё время с момента регистрации.
  
  Выведите данные его активности за октябрь 2008 года в таком виде:
  * номер недели;
  * дата и время последнего поста, опубликованного на этой неделе.
``` sql
WITH top AS
(
    SELECT DISTINCT user_id,
       COUNT(id) OVER (PARTITION BY user_id) count_posts
    FROM stackoverflow.posts 
    ORDER BY count_posts DESC
    LIMIT 1
)

SELECT DISTINCT EXTRACT(WEEK FROM p.creation_date::date),
       MAX(creation_date) OVER (PARTITION BY EXTRACT(WEEK FROM p.creation_date::date))
FROM stackoverflow.posts p
JOIN top ON p.user_id = top.user_id
WHERE p.creation_date::date BETWEEN '2008-10-01' AND '2008-10-31'
```
</details>
