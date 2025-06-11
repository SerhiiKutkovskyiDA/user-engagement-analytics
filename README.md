# SQL
User Engagement and Email Marketing Analysis

This project features a comprehensive SQL query designed to analyze user behavior and email marketing performance. The query aggregates data from multiple tables to generate a consolidated report on key metrics, segmented by country and user characteristics.
Description

The primary goal of this query is to merge user session data with email campaign data to track key performance indicators. It calculates the number of active accounts and the counts of emails sent, opened, and clicked through.

Ultimately, the query ranks countries based on two primary metrics: the total number of user accounts and the total number of emails sent. It then filters the output to display only the top 10 countries for each of these metrics.
Query Structure

The query is built using Common Table Expressions (CTEs) to process the data in a logical, step-by-step manner.

    account_data: Gathers daily statistics on the number of unique accounts, grouped by country, send interval, and user status (verified, unsubscribed).

    email_data: Aggregates data from email campaigns, including:

        sent_cnt: The number of emails sent.

        open_cnt: The number of emails opened.

        visit_cnt: The number of clicks on links within emails.
        This data is also grouped by date, country, and account attributes.

    union_data: Combines the results from account_data and email_data into a single dataset for further aggregation.

    daily_stats: Sums the daily metrics and uses window functions to calculate the total number of accounts and emails sent per country over the entire period.

    rank_data: Ranks each country using DENSE_RANK() based on its total account count and total sent email count.

    Final SELECT: Filters the final result set to include only countries that rank in the top 10 for either total accounts or total emails sent, ordered for clear presentation.

Key Metrics

    acc_cnt: Daily active accounts.

    sent_cnt: Daily emails sent.

    open_cnt: Daily emails opened.

    visit_cnt: Daily clicks from emails.

    total_country_account_cnt: Total unique accounts per country.

    total_country_sent_cnt: Total emails sent per country.

Output

The final output provides a daily breakdown of the above metrics for the top-performing countries, allowing for a detailed analysis of user engagement and email effectiveness in key markets.
