# 🛒 Case Study #5 - Data Mart

## 🧼 Solution - C. Before & After Analysis

Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
- `region`
- `platform`
- `age_band`
- `demographic`
- `customer_type`

Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?

**Answer:**

#### Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

Let's start from the absolute values first and then compare performance for the 12 week before and after period.

- **region:**

```sql
SET
  search_path = data_mart;
SELECT
  region,
  week_number,
  SUM(sales) AS total_sales
FROM
  clean_weekly_sales
WHERE
  calendar_year = 2020
  AND week_number - 11 <= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
  AND week_number + 12 >= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
GROUP BY
  1,
  2
ORDER BY
  2,
  1
```

![New query (16)](https://user-images.githubusercontent.com/98699089/153703749-325eafdc-122f-48db-a410-9a5298ac35c8.png)

Africa and USA showed drop in sales on the week 17, and Oceania - on the week 25.

Let's check the total sales for 12 weeks before and after 2020-06-15 by region:

```sql
SET
  search_path = data_mart;
WITH sales_before AS (
    SELECT
      region,
      SUM(sales) AS total_sales_before
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week - 12)
      AND (base_week - 1)
    GROUP BY
      1
  ),
  sales_after AS (
    SELECT
      region,
      SUM(sales) AS total_sales_after
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week)
      AND (base_week + 11)
   GROUP BY
      1
  )
SELECT
  sb.region,
  total_sales_before,
  total_sales_after,
  total_sales_after - total_sales_before AS change_in_sales,
  ROUND(
    100 * (total_sales_after - total_sales_before) :: numeric / total_sales_before,
    2
  ) AS percentage_of_change
FROM
  sales_before AS sb
  JOIN sales_after AS sa ON sb.region = sa.region
GROUP BY
  1,
  2,
  3,
  4
ORDER BY
  5
```

| region        | total_sales_before | total_sales_after | change_in_sales | percentage_of_change  |
|---------------|--------------------|-------------------|-----------------|-----------------------|
| ASIA          | 1637244466         | 1583807621        | -53436845       | -3.26                 |
| OCEANIA       | 2354116790         | 2282795690        | -71321100       | -3.03                 |
| SOUTH AMERICA | 213036207          | 208452033         | -4584174        | -2.15                 |
| CANADA        | 426438454          | 418264441         | -8174013        | -1.92                 |
| USA           | 677013558          | 666198715         | -10814843       | -1.60                 |
| AFRICA        | 1709537105         | 1700390294        | -9146811        | -0.54                 |
| EUROPE        | 108886567          | 114038959         | 5152392         | 4.73                  |

Before and after analysis shows that sales drop 3.26% in Asia and $71,321,100 in Oceania had the highest negative impact in sales metrics performance in 2020.


- **platform:**

```sql
SET
  search_path = data_mart;
SELECT
  platform,
  week_number,
  SUM(sales) AS total_sales
FROM
  clean_weekly_sales
WHERE
  calendar_year = 2020
  AND week_number - 11 <= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
  AND week_number + 12 >= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
GROUP BY
  1,
  2
ORDER BY
  2,
  1
```

![New query (17)](https://user-images.githubusercontent.com/98699089/153703877-bf7c784c-7a36-478b-a6b7-a8d58b7aef2e.png)

We can see sale drops on Retail platform on the weeks 15, 17, 25 and 32. 

Let's check the total sales for 12 weeks before and after 2020-06-15 by platform:

```sql
SET
  search_path = data_mart;
WITH sales_before AS (
    SELECT
      platform,
      SUM(sales) AS total_sales_before
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week - 12)
      AND (base_week - 1)
    GROUP BY
      1
  ),
  sales_after AS (
    SELECT
      platform,
      SUM(sales) AS total_sales_after
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week)
      AND (base_week + 11)
    GROUP BY
      1
  )
SELECT
  sb.platform,
  total_sales_before,
  total_sales_after,
  total_sales_after - total_sales_before AS change_in_sales,
  ROUND(
    100 * (total_sales_after - total_sales_before) :: numeric / total_sales_before,
    2
  ) AS percentage_of_change
FROM
  sales_before AS sb
  JOIN sales_after AS sa ON sb.platform = sa.platform
GROUP BY
  1,
  2,
  3,
  4
ORDER BY
  5
```

| platform | total_sales_before | total_sales_after | change_in_sales | percentage_of_change  |
|----------|--------------------|-------------------|-----------------|-----------------------|
| Retail   | 6906861113         | 6738777279        | -168083834      | -2.43                 |
| Shopify  | 219412034          | 235170474         | 15758440        | 7.18                  |

Sales drop in Retail had the highest negative impact in sales metrics performance in 2020. And we can see that the Shopify platform has showed 7.18% growth. However, the growth of the Shopify platform did not compensate for the drop in Retail platform.

- **age_band:**

```sql
SET
  search_path = data_mart;
SELECT
  age_band,
  week_number,
  SUM(sales) AS total_sales
FROM
  clean_weekly_sales
WHERE
  calendar_year = 2020
  AND week_number - 11 <= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
  AND week_number + 12 >= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
GROUP BY
  1,
  2
ORDER BY
  2,
  1
```

![New query (18)](https://user-images.githubusercontent.com/98699089/153703951-018c16d6-51bc-4d73-ba8b-8fc4a3d836c6.png)

There is sale drop in the 'unknown' age group on the weeks 17, 25 and 32, and in the Retirees age group on the weeks 15 and 17.

Let's check the total sales for 12 weeks before and after 2020-06-15 by age:

```sql
SET
  search_path = data_mart;
WITH sales_before AS (
    SELECT
      age_band,
      SUM(sales) AS total_sales_before
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week - 12)
      AND (base_week - 1)
    GROUP BY
      1
  ),
  sales_after AS (
    SELECT
      age_band,
      SUM(sales) AS total_sales_after
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week)
      AND (base_week + 11)
   GROUP BY
      1
  )
SELECT
  sb.age_band,
  total_sales_before,
  total_sales_after,
  total_sales_after - total_sales_before AS change_in_sales,
  ROUND(
    100 * (total_sales_after - total_sales_before) :: numeric / total_sales_before,
    2
  ) AS percentage_of_change
FROM
  sales_before AS sb
  JOIN sales_after AS sa ON sb.age_band = sa.age_band
GROUP BY
  1,
  2,
  3,
  4
ORDER BY
  5
```

| age_band     | total_sales_before | total_sales_after | change_in_sales | percentage_of_change  |
|--------------|--------------------|-------------------|-----------------|-----------------------|
| unknown      | 2764354464         | 2671961443        | -92393021       | -3.34                 |
| Middle Aged  | 1164847640         | 1141853348        | -22994292       | -1.97                 |
| Retirees     | 2395264515         | 2365714994        | -29549521       | -1.23                 |
| Young Adults | 801806528          | 794417968         | -7388560        | -0.92                 |

Before and after analysis shows that sales drop 3.34% in unknown age group had the highest negative impact in sales metrics performance in 2020.

- **demographic**

```sql
SET
  search_path = data_mart;
SELECT
  demographic,
  week_number,
  SUM(sales) AS total_sales
FROM
  clean_weekly_sales
WHERE
  calendar_year = 2020
  AND week_number - 11 <= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
  AND week_number + 12 >= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
GROUP BY
  1,
  2
ORDER BY
  2,
  1
```

![New query (19)](https://user-images.githubusercontent.com/98699089/153704066-75fb742e-21a1-49dd-9f2e-c762f37eb5eb.png)

All three demographic groups: 'unknown', Families and Couples showed sales drop on the weeks 17 and 32. Couples and Families also showed a drop on the week 15, and the 'unknown' group - on the week 25.

Let's check the total sales for 12 weeks before and after 2020-06-15 by demographic groups:

```sql
SET
  search_path = data_mart;
WITH sales_before AS (
    SELECT
      demographic,
      SUM(sales) AS total_sales_before
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week - 12)
      AND (base_week - 1)
    GROUP BY
      1
  ),
  sales_after AS (
    SELECT
      demographic,
      SUM(sales) AS total_sales_after
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week)
      AND (base_week + 11)
   GROUP BY
      1
  )
SELECT
  sb.demographic,
  total_sales_before,
  total_sales_after,
  total_sales_after - total_sales_before AS change_in_sales,
  ROUND(
    100 * (total_sales_after - total_sales_before) :: numeric / total_sales_before,
    2
  ) AS percentage_of_change
FROM
  sales_before AS sb
  JOIN sales_after AS sa ON sb.demographic = sa.demographic
GROUP BY
  1,
  2,
  3,
  4
ORDER BY
  5
```

| age_band    | total_sales_before | total_sales_after | change_in_sales | percentage_of_change  |
|-------------|--------------------|-------------------|-----------------|-----------------------|
| unknown     | 2764354464         | 2671961443        | -92393021       | -3.34                 |
| Families    | 2328329040         | 2286009025        | -42320015       | -1.82                 |
| Couples     | 2033589643         | 2015977285        | -17612358       | -0.87                 |

Sales drop 3.34% in unknown demographic group had the highest negative impact in sales metrics performance in 2020.

- **customer_type:**

```sql
SET
  search_path = data_mart;
SELECT
  customer_type,
  week_number,
  SUM(sales) AS total_sales
FROM
  clean_weekly_sales
WHERE
  calendar_year = 2020
  AND week_number - 11 <= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
  AND week_number + 12 >= EXTRACT(
    WEEK
    FROM
      '2020-06-15' :: date
  )
GROUP BY
  1,
  2
ORDER BY
  2,
  1
```

![New query (20)](https://user-images.githubusercontent.com/98699089/153704680-84f658d3-022d-48af-a8d3-351935412056.png)

We can see that Guests and Existing customers had a sales drop on the weeks 17 and 32, and guest customers also showed sales drop on the week 25.

Let's check the total sales for 12 weeks before and after 2020-06-15 by customer type:

```sql
SET
  search_path = data_mart;
WITH sales_before AS (
    SELECT
      customer_type,
      SUM(sales) AS total_sales_before
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week - 12)
      AND (base_week - 1)
    GROUP BY
      1
  ),
  sales_after AS (
    SELECT
      customer_type,
      SUM(sales) AS total_sales_after
    FROM
      clean_weekly_sales,
      LATERAL(
        SELECT
          EXTRACT(
            WEEK
            FROM
              '2020-06-15' :: date
          ) AS base_week
      ) bw
    WHERE
      calendar_year = 2020
      AND week_number between (base_week)
      AND (base_week + 11)
   GROUP BY
      1
  )
SELECT
  sb.customer_type,
  total_sales_before,
  total_sales_after,
  total_sales_after - total_sales_before AS change_in_sales,
  ROUND(
    100 * (total_sales_after - total_sales_before) :: numeric / total_sales_before,
    2
  ) AS percentage_of_change
FROM
  sales_before AS sb
  JOIN sales_after AS sa ON sb.customer_type = sa.customer_type
GROUP BY
  1,
  2,
  3,
  4
ORDER BY
  5
```

| customer_type | total_sales_before | total_sales_after | change_in_sales | percentage_of_change  |
|---------------|--------------------|-------------------|-----------------|-----------------------|
| Guest         | 2573436301         | 2496233635        | -77202666       | -3.00                 |
| Existing      | 3690116427         | 3606243454        | -83872973       | -2.27                 |
| New           | 862720419          | 871470664         | 8750245         | 1.01                  |


Sales drop 3% in the guest customer group had the highest negative impact in sales metrics performance in 2020.

We can conclude that these areas of business had the highest negative impact in sales metrics performance in 2020: sales in Asia and Oceania, retail platform, unknown age and demographic group and guest customers.

In general, packaging issues had a negative sales impact but it was not the only reason for decreasing sales in 2020. 

### Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?

<details><summary> Click to expand :arrow_down: </summary>
  
Retail sales is the biggest channel and the retired customers is the biggest group of customers. It is important to keep their loyality and try to meet their expectations. 

Shopify sales channel is increasing its share year over year and developing of this channel might be profitable from the future prospective. 

The analysis shows that the loyal customers (existing customers and guest customers) generate the most sales. The sales trend is slopping down. Focusing on new customer acquisition and retention could be another growth point for the company. Middle aged adults is the most lucrative age group to appeal to. They show their interest in the company products and there is a room for further market penetration.

The share of 'unknown' customers in age and demographic groups takes a considerable part, which can make the analysis of age and demographic characteristics incorrect. To make the analysis more accurate we need to reduce the number of 'unknown' records. 
</details>
