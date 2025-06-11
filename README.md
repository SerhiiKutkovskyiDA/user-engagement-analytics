# User Engagement and Email Marketing Analysis


This project features a comprehensive SQL query designed to analyze user behavior and email marketing performance. The query aggregates data from multiple tables to generate a consolidated report on key metrics, segmented by country and user characteristics.
Description

The primary goal of this query is to merge user session data with email campaign data to track key performance indicators. It calculates the number of active accounts and the counts of emails sent, opened, and clicked through.

Ultimately, a query ranks countries based on two primary metrics: the total number of user accounts and the total number of emails sent. It then filters the output to display only the top 10 countries for each of these metrics.

## Query Structure


The query is built using Common Table Expressions (CTEs) to process the data in a logical, step-by-step manner.

```account_data```: Gathers daily statistics on the number of unique accounts, grouped by country, send interval, and user status (verified, unsubscribed).

    WITH account_data AS (
      SELECT
        s.date as date,
        sp.country as country,
        send_interval,
        is_verified,
        is_unsubscribed,
        COUNT(DISTINCT a.id) as acc_cnt
      FROM `DA.account_session` acs
      JOIN `DA.session` s
      ON acs.ga_session_id = s.ga_session_id
      JOIN `DA.session_params` sp
      ON s.ga_session_id = sp.ga_session_id
      JOIN `DA.account` a
      ON acs.account_id = a.id
      GROUP BY 1, 2, 3, 4, 5
    )

```email_data```: Aggregates data from email campaigns, including:

```sent_cnt```: The number of emails sent.

```open_cnt```: The number of emails opened.

```visit_cnt```: The number of clicks on links within emails.
        This data is also grouped by date, country, and account attributes.

    email_data AS (
      SELECT
        DATE_ADD(s.date, INTERVAL es.sent_date DAY) as date,
        sp.country as country,
        a.send_interval as send_interval,
        a.is_verified as is_verified,
        a.is_unsubscribed as is_unsubscribed,
        COUNT(DISTINCT es.id_message) as sent_cnt,
        COUNT(DISTINCT eo.id_message) as open_cnt,
        COUNT(DISTINCT ev.id_message) as visit_cnt
      FROM `DA.email_sent` es
      LEFT JOIN `DA.email_open` eo
      ON es.id_message = eo.id_message
      LEFT JOIN `DA.email_visit` ev
      ON es.id_message = ev.id_message
      JOIN `DA.account_session` acs
      ON es.id_account = acs.account_id
      JOIN `DA.session` s
      ON acs.ga_session_id = s.ga_session_id
      JOIN `DA.session_params` sp
      ON s.ga_session_id = sp.ga_session_id
      JOIN `DA.account` a
      ON es.id_account = a.id
      GROUP BY 1, 2, 3, 4, 5
    )

```union_data```: Combines the results from account_data and email_data into a single dataset for further aggregation.

    union_data AS (
      SELECT
        date,
        country,
        send_interval,
        is_verified,
        is_unsubscribed,
        acc_cnt,
        0 as sent_cnt,
        0 as open_cnt,
        0 as visit_cnt
      FROM account_data

      UNION ALL

      SELECT
        date,
        country,
        send_interval,
        is_verified,
        is_unsubscribed,
        0 as acc_cnt,
        sent_cnt,
        open_cnt,
        visit_cnt
      FROM email_data
    )
```daily_stats```: Sums the daily metrics and uses window functions to calculate the total number of accounts and emails sent per country over the entire period.

    daily_stats AS (
      SELECT
        date,
        country,
        send_interval,
        is_verified,
        is_unsubscribed,
        SUM(acc_cnt) as acc_cnt,
        SUM(sent_cnt) as sent_cnt,
        SUM(open_cnt) as open_cnt,
        SUM(visit_cnt) as visit_cnt,
        SUM(SUM(acc_cnt)) OVER (PARTITION BY country) as total_country_account_cnt,
        SUM(SUM(sent_cnt)) OVER (PARTITION BY country) as total_country_sent_cnt
      FROM union_data
      GROUP BY 1, 2, 3, 4, 5
    )

```rank_data```: Ranks each country using DENSE_RANK() based on its total account count and total sent email count.

    rank_data AS (
      SELECT
        *,
        DENSE_RANK() OVER (ORDER BY total_country_account_cnt DESC) as rank_total_country_account_cnt,
        DENSE_RANK() OVER (ORDER BY total_country_sent_cnt DESC) as rank_total_country_sent_cnt
      FROM daily_stats
    )
 ```Final SELECT```: Filters the final result set to include only countries that rank in the top 10 for either total accounts or total emails sent, ordered for clear presentation.

    SELECT
      *
    FROM rank_data
    WHERE rank_total_country_account_cnt <= 10 OR rank_total_country_sent_cnt <= 10
    ORDER BY date, country, send_interval, is_verified, is_unsubscribed

## Key Metrics

 ```acc_cnt```: Daily active accounts.

 ```sent_cnt```: Daily emails sent.

 ```open_cnt```: Daily emails opened.

 ```visit_cnt```: Daily clicks from emails.

```total_country_account_cnt```: Total unique accounts per country.

```total_country_sent_cnt```: Total emails sent per country.

## Output


The final output provides a daily breakdown of the above metrics for the top-performing countries, allowing for a detailed analysis of user engagement and email effectiveness in key markets.

![image](https://github.com/user-attachments/assets/140f82b9-4e48-40fd-b40c-da36b637a6ba)
