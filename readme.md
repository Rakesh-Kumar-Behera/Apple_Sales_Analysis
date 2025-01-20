
## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, you'll demonstrate your ability to write sophisticated SQL queries that extract valuable insights from large datasets.

The project is ideal for data analysts looking to enhance their SQL skills by working with a large-scale dataset and solving real-world business questions.

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)



---

### What’s Included:

- **20 Advanced SQL Queries**: Step-by-step solutions for complex queries, enhancing your skills in performance tuning and optimization.
- **5 Detailed Tables**: Comprehensive datasets with over 1 million rows, including sales, stores, product categories, products, and warranties.
- **Hands-on Learning**: Practical experience with complex datasets and advanced business problem-solving.
- **Comprehensive Coverage**: Each table provides new opportunities to explore SQL concepts.


## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.
```sql
select country,count(1) as no_of_stores 
from stores
group by country
order by 1;
```
2. Calculate the total number of units sold by each store.
```sql
select s.store_id,st.store_name, 
		sum(s.quantity) as total_units_sold
from sales s
join stores st
on s.store_id = st.store_id
group by 1,2
order by 3 desc;
```
3. Identify how many sales occurred in December 2023.
```sql
select count(1) as sales_count
from sales
where extract(month from sale_date) = 12 and
		extract(year from sale_date) = 2023;
		
-- or

select count(1) as total_sales
from sales
where to_char(sale_date, 'mm-yyyy') = '12-2023';
```
4. Determine how many stores have never had a warranty claim filed.
```sql
select count(1)
from stores 
where store_id not in (select distinct s.store_id
					  from warranty w
					  left join sales s
					  on s.sale_id = w.sale_id)
```
5. Calculate the percentage of warranty claims marked as "Rejected".
```sql
with cte as(
	select *, case
					when repair_status = 'Rejected' then 1
					else 0
					end as rejected_count
	from warranty)
select round(sum(rejected_count)/count(1)::numeric*100,2) as pct_rejected from cte;

-- or

select 
	round(count(1)/(select count(1) from warranty)::numeric*100,2) as pct_rejected
from warranty
where repair_status = 'Rejected';
```
6. Identify which store had the highest total units sold in the last year(2023).
```sql
select s.store_id,st.store_name, sum(quantity) as total_qty_sold
from sales s
left join stores st
on s.store_id = st.store_id
where extract(year from s.sale_date) = 2023
group by 1,2
order by sum(quantity) desc
limit 1;
```
7. Count the number of unique products sold in the last year(2023).
```sql
select count(distinct product_id)
from sales
where extract(year from s.sale_date) = 2023
-- sale_date >= current_date - interval '1 year'
```
8. Find the average price of products in each category.
```sql
select p.category_id,c.category_name,
		avg(p.price) as avg_product_price
from products p
left join category c
on p.category_id = c.category_id
group by 1,2
order by 3 desc;

```
9. How many warranty claims were filed in 2020?
```sql
select 
	count(1) as claim_count
from warranty
where extract(year from claim_date) = 2020;
```
10. For each store, identify the best-selling day based on highest quantity sold.
```sql
select store_id,day_name
from
	(select store_id,
				extract(dow from sale_date) as day_of_week,
		 		to_char(sale_date , 'Day') as day_name,
				sum(quantity) as qty_sold,
		 		rank() over(partition by store_id order by sum(quantity) desc) as rnk
		from sales
		group by 1,2,3)
where rnk = 1;
```

### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.
```sql
with product_rank as
	(select st.country , 
			to_char(sale_date, 'yyyy') as year,
			s.product_id,
			p.product_name,
			sum(quantity) as total_units_sold,
			rank() over(partition by st.country,to_char(sale_date, 'yyyy') order by sum(quantity)) as rnk
	from sales s
	left join stores st
	on s.store_id = st.store_id
	join products p
	on s.product_id = p.product_id
	group by 1,2,3,4)
select country,year,product_name
from product_rank
where rnk = 1;
```
12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
select count(1) 
from warranty w 
left join sales s
on s.sale_id = w.sale_id
where w.claim_date - s.sale_date <= 180;
-- s.sale_date + interval '180 day' >= w.claim_date
```
13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
select p.product_name ,
		count(w.claim_id) as no_of_claim,
		count(s.sale_id) as no_of_sales
from warranty w
right join sales s
on w.sale_id = s.sale_id
join products p
on p.product_id = s.product_id
where p.launch_date >= (current_date - interval '2 years')
group by 1;
```
14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
select to_char(sale_date, 'mm-yyyy') as month,
		sum(quantity) as total_units_sold
from sales s
join stores st
on s.store_id = st.store_id
where st.country = 'USA' 
		and sale_date >= current_date - interval '3 years' 
group by 1
having sum(quantity) > 5000;
```
15. Identify the product category with the most warranty claims filed in the last two years.
```sql
select c.category_name ,count(1) as num_of_claims
from warranty w
join sales s
on w.sale_id = s.sale_id
join products p
on s.product_id = p.product_id
join category c
on c.category_id = p.category_id
where sale_date >= current_date - interval '2 years'
group by 1
order by count(1) desc;
-- limit 1
```

### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
select 
	country,
	total_qty_sold,
	total_claims,
	coalesce(total_claims::numeric *100/ total_qty_sold:: numeric, 0) as risk
from
	(select
	st.country,
	sum(s.quantity) total_qty_sold,
	count(w.claim_id) as total_claims
	from sales s
	join stores st
	on s.store_id = st.store_id
	left join warranty w
	on w.sale_id = s.sale_id
	group by 1)
order by 4 desc;
```
17. Analyze the year-by-year growth ratio for each store.
```sql
with yearly_sales as(
		select s.store_id,st.store_name,
				extract(year from s.sale_date) as year ,
				sum(s.quantity * p.price) as  current_year_sales
		from sales s
		join stores st
		on s.store_id = st.store_id
		join products p
		on p.product_id = s.product_id
		group by 1,2,3
		order by 2,3),
	growth_ratio as (
		select store_id,store_name ,year,
				current_year_sales ,
				lag(current_year_sales) over(partition by store_id,store_name ) as prev_year_sales
		from yearly_sales)
select store_name,
		year,
		prev_year_sales,
		current_year_sales,
		(current_year_sales - prev_year_sales)::numeric / prev_year_sales * 100 yoy_growth
from growth_ratio
where prev_year_sales is not null
		and year <> extract(year from current_date);
```
18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
-- to check warranty claim for products of different price category 
-- (to check if lower price product category has more claim or higher price category has more)

select 
		case when p.price < 500 then 'Low budget product'
			 when p.price between 500 and 1000 then 'mid range product'
			 else 'flagship product'
			 end as price_segemnt,
		count(claim_id) as total_claim
from warranty w
left join sales s
on w.sale_id = s.sale_id
join products p
on p.product_id = s.product_id
where w.claim_date >= current_date - interval '5 years'
group by 1;
```
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
with total_completed_claim as (
		select s.store_id,
				count(w.claim_id) as total_repaired_claims
		from sales s
		right join warranty w
		on w.sale_id = s.sale_id
		where repair_status = 'Completed'
		group by 1),
		
	total_repaired as(
		select s.store_id,
				count(w.claim_id) as total_claims
		from sales s
		right join warranty w
		on w.sale_id = s.sale_id
		group by 1)
select 
	tr.store_id,
	st.store_name
	total_repaired_claims,
	total_claims,
	round(total_repaired_claims*100/
	total_claims,2) as pct_completed_claim
from total_completed_claim tc
join total_repaired tr
on tc.store_id = tr.store_id
join stores st
on tr.store_id = st.store_id
order by 4 desc;
```
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
with monthly_sales as(
		select 
			s.store_id,
			st.store_name,
			extract(year from sale_date) as year,
			extract(month from sale_date) as month,
			sum(p.price*s.quantity) as total_sales_amount
		from sales s
		join stores st
		on s.store_id = st.store_id
		join products p
		on s.product_id = p.product_id
		where s.sale_date > current_date - interval '4 years'
		group by 1,2,3,4
		order by 1,3,4)
select
	store_id,
	store_name,
	year,
	month,
	total_sales_amount,
	sum(total_sales_amount) over(partition by store_id,store_name order by year,month) as running_total
from monthly_sales
```

### Bonus Question

- Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
select 
	p.product_name,
	case when s.sale_date between p.launch_date and p.launch_date + interval '6 month' then '0-6 months'
		 when s.sale_date between p.launch_date + interval '6 month' and p.launch_date + interval '12 month' then '6-12 months'
		 when s.sale_date between p.launch_date + interval '12 month' and p.launch_date + interval '18 month' then '12-18 months'
		 else '18+ months'
	end as time_period,
	sum(s.quantity) total_sales
	
from sales s
join products p
on s.product_id = p.product_id
group by 1,2;
```

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

---
