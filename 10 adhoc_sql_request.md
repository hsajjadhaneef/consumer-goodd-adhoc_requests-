# consumer-goodd-adhoc_requests-  

Requests:
1. Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region.

select  
market from dim_customer 
where customer = "Atliq Exclusive" 
and region = "APAC" group by market;


2. What is the percentage of unique product increase in 2021 vs. 2020? The
final output contains these fields,
unique_products_2020
unique_products_2021
percentage_chg


with x as(
select *,
 count(distinct product_code) as un_count_2020  from fact_sales_monthly 
where fiscal_year = 2020),
y as (select *, count(distinct  product_code) as un_count_2021 
from fact_sales_monthly 
where fiscal_year =2021) 
select x.un_count_2020,y.un_count_2021, concat(round((y.un_count_2021-x.un_count_2020)*100/ 
un_count_2020,2),"%") as pct_change 
from x 
join y on 
x.product_code=y.product_code





3. Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts. The final output contains
2 fields,
segment
product_count

select segment,count( distinct product_code) as unique_count  from 
dim_product 
group by segment 
order by  unique_count desc;



4. Follow-up: Which segment had the most increase in unique products in
2021 vs 2020? The final output contains these fields,
segment
product_count_2020
product_count_2021
difference


with cte1 as
(
select p.segment as A,count(distinct s.product_code) as
unique_product_2021 from fact_sales_monthly 
s 
join dim_product p 
on s.product_code =p.product_code 
where s.fiscal_year =2021 
group by p.segment,s.fiscal_year 
order by unique_product_2021 desc),

cte2 AS
 (
select p.segment AS B,count(distinct s.product_code) as
unique_product_2020 from fact_sales_monthly 
s 
join dim_product p 
on s.product_code =p.product_code 
where s.fiscal_year =2020 
group by p.segment,s.fiscal_year 
order by unique_product_2020 desc)
 
 SELECT cte1.A as segment, cte2.unique_product_2020 as unique_product_2020,
 cte1.unique_product_2021 as 
 unique_product_2021, 
 (cte1.unique_product_2021-cte2.unique_product_2020)
 as difference 
 from cte1,cte2
where 
cte1.A=cte2.B;




5. Get the products that have the highest and lowest manufacturing costs.
The final output should contain these fields,
product_code
product
manufacturing_cost




SELECT c.product_code,p.product,min(c.manufacturing_cost)
as lowest_cost FROM  
fact_manufacturing_cost c 
join dim_product p 
on c.product_code = p.product_code 
group by product_code;

select c.product_code,p.product,c.manufacturing_cost from fact_manufacturing_cost c 
join dim_product 
p 
on c.product_code = p.product_code 
where c.manufacturing_cost in(
select max(manufacturing_cost) as min from fact_manufacturing_cost 
union
select min(manufacturing_cost) as min from fact_manufacturing_cost )
order by manufacturing_cost desc;






6. Generate a report which contains the top 5 customers who received an
average high pre_invoice_discount_pct for the fiscal year 2021 and in the
Indian market. The final output contains these fields,
customer_code
customer
average_discount_percentage


select d.customer_code,c.customer, avg(pre_invoice_discount_pct) 
as avg_discount_pct from fact_pre_invoice_deductions d 
join dim_customer c 
on d.customer_code=c.customer_code 
where d.fiscal_year =2021 
and c.market = "india"
group by d.customer_code 
order by avg_discount_pct desc
limit 5;




7. Get the complete report of the Gross sales amount for the customer “Atliq
Exclusive” for each month. This analysis helps to get an idea of low and
high-performing months and take strategic decisions.
The final report contains these columns:
Month
Year
Gross sales Amount




select CONCAT(MONTHNAME(s.date), ' (', YEAR(s.date), ')') AS 'Month',s.fiscal_year,
concat(round(sum(s.sold_quantity*g.gross_price)/1000000,2),"M") as
gross_sales_amount_mln from 
fact_sales_monthly s 
join  fact_gross_price g 
on s.product_code = g.product_code

join dim_customer c 
on s.customer_code =c.customer_code 
where c.customer = "Atliq Exclusive"
group by month,s.fiscal_year order by 
s.fiscal_year;



8. In which quarter of 2020, got the maximum total_sold_quantity? The final
output contains these fields sorted by the total_sold_quantity,
Quarter
total_sold_quantity


select 
get_fiscal_quarter(s.date) as Quarter ,round(sum(s.sold_quantity),2)
 AS total_sold_qty
 from fact_sales_monthly s 
 where s.fiscal_year =2020 
 group by Quarter 
 order by total_sold_qty DESC;
 



9. Which channel helped to bring more gross sales in the fiscal year 2021
and the percentage of contribution? The final output contains these fields,
channel
gross_sales_mln
percentage


 with cte1 as(
 SELECT 
  c.channel, concat(round(sum(s.sold_quantity*g.gross_price)/1000000,2),"M")  as 
  gross_sales_mln
  from fact_sales_monthly s 
  join fact_gross_price g 
  on s.product_code = 
  g.product_code 

  join dim_customer c 
  on s.customer_code  = c.customer_code 
  where s.fiscal_year =2021
group by c.channel
    order by gross_sales_mln desc
 ) 
 select * ,  round((gross_sales_mln/sum(gross_sales_mln) over())*100,2) as percentage 
 
from cte1  
ORDER BY percentage  DESC;










10. Get the Top 3 products in each division that have a high
total_sold_quantity in the fiscal_year 2021? The final output contains these
fields,
division
product_code

product
total_sold_quantity
rank_order



with cte1 as 
(select p.division,s.product_code,p.product,sum(s.sold_quantity) as total_qty
 from fact_sales_monthly s 
join dim_product 
p 
on s.product_code = p.product_code 
where s.fiscal_year = 2021 
group by s.product_code
order by total_qty desc),cte2 as 
(select * , rank() over(partition by division order by total_qty desc) as 
rank_order from cte1)

select * from cte2 where rank_order in(1,2,3)
;



