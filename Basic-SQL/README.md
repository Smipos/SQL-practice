Задания + код решения.

* Посчитайте, сколько компаний закрылось.
  ```
  SELECT COUNT(id)
  FROM company
  WHERE status = 'closed'
  ```
* Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы `company`. Отсортируйте таблицу по убыванию значений в поле `funding_total`.
  ```
  SELECT funding_total
  FROM company
  WHERE category_code = 'news'
        AND
        country_code = 'USA'
  ORDER BY funding_total DESC
  ```
* Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.
  ```
  SELECT SUM(price_amount)
  FROM acquisition
  WHERE term_code = 'cash'
        AND
        EXTRACT(YEAR FROM acquired_at) BETWEEN 2011 AND 2013
  ```
* Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на `'Silver'`.
  ```
  SELECT first_name,
         last_name,
         twitter_username
  FROM people
  WHERE twitter_username LIKE 'Silver%'
  ```
* Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку `'money'`, а фамилия начинается на `'K'`.
  ```
  SELECT *
  FROM people
  WHERE twitter_username LIKE '%money%'
        AND
        last_name LIKE 'K%'
  ```
