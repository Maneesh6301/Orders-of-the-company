# Orders-of-the-company
Performance of the company based on orders 

sql code 
#TOP 10 selling products
select  product_id,sum(sale_price) as sales
from orders
group by product_id
order by sales desc
limit 10

#comparison of sales between 2023 and 2022 with month
with xyz as(
SELECT MONTH(order_date) as mn ,year(order_date) as yr,sum(sale_price) as sales
from orders 
group by month(order_date),year(order_date)
)
select mn,
sum(case when yr=2023 then sales else 0 end) as sales_2023,
sum(case when yr=2022 then sales else 0 end ) as sales_2022
from xyz
group by mn
order by mn

#most selling category with month and year
#with xyz as(
select category as cat,month(order_date) as mn, year(order_date)as yr,sum(sale_price) as sales,
 ROW_NUMBER() OVER(partition by category ORDER  by sum(sale_price) desc) as ranked
from orders
group by category,month(order_date),year(order_date)
)
select * 
from xyz
where ranked=1


#percentage of profit comparision  between  2023 and 2022
with xyz as(
select sub_category as sc,year(order_date) as yr,sum(sale_price) as sales
from orders
group by year(order_date),sub_category
order by sum(sale_price) desc
)
, xyz1 as (
select sc,
sum(case when yr=2023 then sales else 0 end) as sales_2023,
sum(case when yr=2022 then sales else 0 end ) as sales_2022
from xyz 
group by sc
having sum(case when yr=2023 then sales else 0 end ) >
sum(case when yr=2022 then sales else 0 end) 
)
select sc,
sales_2023,
sales_2022,
sales_2023 *100/sales_2022  as percenatge 
from xyz1
group by sc 
order by percenatge desc















