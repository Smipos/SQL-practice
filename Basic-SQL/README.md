Задания + код решения.

* Посчитайте, сколько компаний закрылось.
  ``` sql
  SELECT COUNT(id)
  FROM company
  WHERE status = 'closed'
  ``` 
* Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы `company`. Отсортируйте таблицу по убыванию значений в поле `funding_total`.
  ``` sql
  SELECT funding_total
  FROM company
  WHERE category_code = 'news'
        AND
        country_code = 'USA'
  ORDER BY funding_total DESC
  ```
* Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.
  ``` sql
  SELECT SUM(price_amount)
  FROM acquisition
  WHERE term_code = 'cash'
        AND
        EXTRACT(YEAR FROM acquired_at) BETWEEN 2011 AND 2013
  ```
* Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на `'Silver'`.
  ``` sql
  SELECT first_name,
         last_name,
         twitter_username
  FROM people
  WHERE twitter_username LIKE 'Silver%'
  ```
* Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку `'money'`, а фамилия начинается на `'K'`.
  ``` sql
  SELECT *
  FROM people
  WHERE twitter_username LIKE '%money%'
        AND
        last_name LIKE 'K%'
  ```
* Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.
  ``` sql
  SELECT country_code,
         SUM(funding_total) total_invested
  FROM company
  GROUP BY country_code
  ORDER BY total_invested DESC
  ```
* Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.
  ``` sql
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
  * Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию `high_activity`.
  * Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию `middle_activity`.
  * Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию `low_activity`.

  Отобразите все поля таблицы fund и новое поле с категориями.
  ``` sql
  SELECT *,
         CASE
             WHEN invested_companies >= 100 THEN 'high_activity'
             WHEN  invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'
             WHEN invested_companies < 20 THEN 'low_activity'
         END fund_category
  FROM fund
  ```
* Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.
  ``` sql
  SELECT
         CASE
             WHEN invested_companies>=100 THEN 'high_activity'
             WHEN invested_companies>=20 THEN 'middle_activity'
             ELSE 'low_activity'
         END fund_category,
         ROUND(AVG(investment_rounds)) avg_rounds
  FROM fund
  GROUP BY fund_category
  ORDER BY avg_rounds ASC
  ```
* Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. 
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. 
Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.
  ``` sql
  SELECT country_code,
         MIN(invested_companies) min_invested_companies,
         MAX(invested_companies) max_invested_companies,
         AVG(invested_companies) avg_invested_companies
  FROM fund
  WHERE EXTRACT(YEAR FROM founded_at) BETWEEN 2010 AND 2012
  GROUP BY country_code
  HAVING MIN(invested_companies) <> 0
  ORDER BY avg_invested_companies DESC, 
           country_code
  LIMIT 10
  ```
* Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.
  ``` sql
  SELECT p.first_name,
         p.last_name,
         e.instituition
  FROM people p
  LEFT JOIN education e ON p.id = e.person_id
  ```
* Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.
  ``` sql
  SELECT  c.name,
          COUNT(DISTINCT e.instituition) unique_inst_count
  FROM company c
  JOIN people p ON c.id = p.company_id
  JOIN education e ON p.id = e.person_id
  GROUP BY c.name
  ORDER BY unique_inst_count DESC
  LIMIT 5
  ```
* Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.
  ``` sql
  SELECT DISTINCT c.name
  FROM company c
  JOIN funding_round fr ON fr.company_id = c.id
  WHERE fr.is_last_round = 1
        AND
        fr.is_first_round = 1
        AND
        c.status = 'closed'
  ```
* Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.
  ```sql
  WITH
  closed_companies AS
  (
      SELECT DISTINCT c.id
      FROM company c
      JOIN funding_round fr ON fr.company_id = c.id
      WHERE fr.is_last_round = 1
            AND
            fr.is_first_round = 1
            AND
            c.status = 'closed'
  )
  
  SELECT p.id
  FROM people p
  JOIN closed_companies cc ON cc.id = p.company_id
  ```
* Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.
  ``` sql
  WITH
  closed_companies AS
  (
      SELECT DISTINCT c.id
      FROM company c
      JOIN funding_round fr ON fr.company_id = c.id
      WHERE fr.is_last_round = 1
            AND
            fr.is_first_round = 1
            AND
            c.status = 'closed'
  ),
  staff AS
  (
    SELECT p.id
    FROM people p
    JOIN closed_companies cc ON cc.id = p.company_id
  )
  
  SELECT DISTINCT s.id,
         e.instituition
  FROM education e
  JOIN staff s ON e.person_id = s.id
  ```
* Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.
  ``` sql
  WITH
  closed_companies AS
  (
      SELECT DISTINCT c.id
      FROM company c
      JOIN funding_round fr ON fr.company_id = c.id
      WHERE fr.is_last_round = 1
            AND
            fr.is_first_round = 1
            AND
            c.status = 'closed'
  ),
  staff AS
  (
    SELECT p.id
    FROM people p
    JOIN closed_companies cc ON cc.id = p.company_id
  )
  
  SELECT s.id,
         COUNT(e.instituition)
  FROM education e
  JOIN staff s ON e.person_id = s.id
  GROUP BY s.id
  ```
* Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.
  ``` sql
  WITH
  closed_companies AS
  (
      SELECT DISTINCT c.id
      FROM company c
      JOIN funding_round fr ON fr.company_id = c.id
      WHERE fr.is_last_round = 1
            AND
            fr.is_first_round = 1
            AND
            c.status = 'closed'
  ),
  staff AS
  (
      SELECT p.id
      FROM people p
      JOIN closed_companies cc ON cc.id = p.company_id
  ),
  ins_count AS
  (
      SELECT s.id,
             COUNT(e.instituition) inst_count
      FROM education e
      JOIN staff s ON e.person_id = s.id
      GROUP BY s.id
  )
  
  SELECT AVG(inst_count)
  FROM ins_count
  ```
* Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Facebook
  ``` sql
  WITH
  fb AS
  (
      SELECT DISTINCT id
      FROM company 
      WHERE name = 'Facebook'
  ),
  staff AS
  (
      SELECT p.id
      FROM people p
      JOIN fb ON fb.id = p.company_id
  ),
  ins_count AS
  (
      SELECT s.id,
             COUNT(e.instituition) inst_count
      FROM education e
      JOIN staff s ON e.person_id = s.id
      GROUP BY s.id
  )
  
  SELECT AVG(inst_count)
  FROM ins_count
  ```
* Составьте таблицу из полей:
  * `name_of_fund` — название фонда;
  * `name_of_company` — название компании;
  * `amount` — сумма инвестиций, которую привлекла компания в раунде.
  
  В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с `2012` по `2013` год     включительно.
  ``` sql
  SELECT f.name name_of_fund,
         c.name name_of_company,
         fr.raised_amount amount
  FROM company c
  JOIN investment i ON i.company_id = c.id
  JOIN fund f ON f.id = i.fund_id
  JOIN funding_round fr ON fr.id = i.funding_round_id
  WHERE c.milestones > 6
        AND
        EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2012 AND 2013
  ```
* Выгрузите таблицу, в которой будут такие поля:
  * название компании-покупателя;
  * сумма сделки;
  * название компании, которую купили;
  * сумма инвестиций, вложенных в купленную компанию;
  * доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.

  Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
   
  Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.
  ``` sql
  SELECT buyer_company.name,
         a.price_amount,
         sell_company.name,
         sell_company.funding_total,
         ROUND(a.price_amount / sell_company.funding_total) excess
  FROM acquisition a
  JOIN company buyer_company ON a.acquiring_company_id = buyer_company.id
  JOIN company sell_company ON a.acquired_company_id = sell_company.id
  WHERE a.price_amount <> 0
        AND
        sell_company.funding_total <> 0
  ORDER BY a.price_amount DESC,
           sell_company.name
  LIMIT 10
  ```
* Выгрузите таблицу, в которую войдут названия компаний из категории `social`, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.
  ``` sql
  SELECT c.name,
         EXTRACT(MONTH FROM fr.funded_at) month_invest
  FROM company c
  JOIN funding_round fr ON fr.company_id = c.id
  WHERE c.category_code = 'social'
        AND
        fr.raised_amount <> 0
        AND
        EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013
  ```
* Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
  * номер месяца, в котором проходили раунды;
  * количество уникальных названий фондов из США, которые инвестировали в этом месяце;
  * количество компаний, купленных за этот месяц;
  * общая сумма сделок по покупкам в этом месяце.
  ``` sql
  WITH 
  acq AS
  (
      SELECT EXTRACT(MONTH FROM acquired_at) month_of_deal,
             COUNT(acquired_company_id) sold_companies_amount,
             SUM(price_amount) total_amount
      FROM acquisition
      WHERE EXTRACT(YEAR FROM acquired_at) BETWEEN 2010 AND 2013
      GROUP BY month_of_deal
  ),
  info_sold AS
  (
      SELECT EXTRACT(MONTH FROM fr.funded_at) month_f,
             COUNT(DISTINCT f.id) usa_fund_count
      FROM funding_round fr
      LEFT JOIN investment i ON i.funding_round_id = fr.id
      LEFT JOIN fund f ON f.id = i.fund_id
      WHERE EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013
            AND
            f.country_code = 'USA'
      GROUP BY month_f
  )
  
  SELECT acq.month_of_deal,
         info_sold.usa_fund_count,
         acq.sold_companies_amount,
         acq.total_amount
  FROM acq
  LEFT JOIN info_sold ON info_sold.month_f = acq.month_of_deal
  GROUP BY acq.month_of_deal, 
           info_sold.usa_fund_count,
           acq.sold_companies_amount,
           acq.total_amount
  ORDER BY acq.month_of_deal ASC
  ```
* Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в `2011`, `2012` и `2013` годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за `2011` год от большего к меньшему.
  ``` sql
  WITH
  inv_2011 AS 
  (
      SELECT country_code country,
             AVG(funding_total) avg_2011,
             EXTRACT(YEAR FROM founded_at) year_2011
      FROM company
      WHERE EXTRACT(YEAR FROM founded_at) = 2011
      GROUP BY country, year_2011
  ), 
  inv_2012 AS 
  (
      SELECT country_code country,
             AVG(funding_total) avg_2012,
             EXTRACT(YEAR FROM founded_at) year_2012
      FROM company
      WHERE EXTRACT(YEAR FROM founded_at) = 2012
      GROUP BY country, year_2012
  ),
  inv_2013 AS 
  (
      SELECT country_code country,
             AVG(funding_total) avg_2013,
             EXTRACT(YEAR FROM founded_at) year_2013
      FROM company
      WHERE EXTRACT(YEAR FROM founded_at) = 2013
      GROUP BY country, year_2013
  )
  
  SELECT inv_2011.country,
         inv_2011.avg_2011,
         inv_2012.avg_2012,
         inv_2013.avg_2013
  FROM inv_2011
  JOIN inv_2012 ON inv_2012.country = inv_2011.country
  JOIN inv_2013 ON inv_2013.country = inv_2011.country
  ORDER BY  inv_2011.avg_2011 DESC
  ```
