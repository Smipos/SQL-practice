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
* Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.
  ```
  SELECT country_code,
         SUM(funding_total) total_invested
  FROM company
  GROUP BY country_code
  ORDER BY total_invested DESC
  ```
* Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.
  ```
  SELECT funded_at,
         MIN(raised_amount) min_invest,
         MAX(raised_amount) max_invest
  FROM funding_round
  GROUP BY funded_at
  HAVING MIN(raised_amount) <> 0
         AND
         MIN(raised_amount) <> MAX(raised_amount)
  ```
* Создайте поле с категориями:
  * Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
  * Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
  * Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.

  Отобразите все поля таблицы fund и новое поле с категориями.
  ```
  SELECT *,
         CASE
             WHEN invested_companies >= 100 THEN 'high_activity'
             WHEN  invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'
             WHEN invested_companies < 20 THEN 'low_activity'
         END fund_category
  FROM fund
  ```
