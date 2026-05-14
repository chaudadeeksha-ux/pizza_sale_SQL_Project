create table orders(
   order_id	Int not null,
   order_date	date,	
   order_time	time	
);
Select * from orders;
SELECT COUNT(*) 
FROM orders;

create table pizzas(

 pizza_id	Varchar(50)	not null,
 pizza_type_id	Varchar(50)	not null,
 size	char(10)	not null,
 price	Numeric(10,2)	
);
Select * from pizzas;

create table pizza_type(
pizza_type_id varchar(100),
name varchar(100),
category varchar(50),
ingredients varchar(300)
);

Select * from pizza_type;

SELECT COUNT(*) 
FROM pizza_type;

create table order_details(
 order_details_id int ,
 order_id int,
 pizza_id varchar(100),
 quantity int
);
Select * from order_details;
SELECT COUNT(*) 
FROM order_details;

Select * from orders;
Select * from order_details;
Select * from pizzas;
Select * from pizza_type;
--Basic:
--Retrieve the total number of orders placed.
select count(order_id) 
from orders;

--Calculate the total revenue generated from pizza sales.

select sum(o.quantity * p.price) as total_sales
from  order_details o
join pizzas p
on p.pizza_id = o.pizza_id;


--Identify the highest-priced pizza.
Select o.order_id, p.price
from pizzas p
join orders o
on p.pizza_type_id = pt.pizza_type_id
order by p.price DESc limit 1;

--Identify the most common pizza size ordered.

Select p.size , count(od.order_details_id)
from pizza_type p
join order_details od
on p.pizza_id = od.pizza_id
group by p.size
order by count(od.order_details_id) DESc limit 1;

--List the top 5 most ordered pizza types along with their quantities.
Select pt.name , sum(od.quantity) as quantity
from pizza_type pt join pizzas p
on pt.pizza_type_id = p.pizza_type_id
join order_details od
on p.pizza_id = od.pizza_id
group by pt.name
order by quantity desc limit 5;


--Intermediate:

--Join the necessary tables to find the total quantity of each pizza category ordered.
Select pt.category , sum(od.quantity) as quantity
from pizza_type pt join pizzas p
on pt.pizza_type_id = p.pizza_type_id
join order_details od
on p.pizza_id = od.pizza_id
group by pt.category order by quantity desc;


--Determine the distribution of orders by hour of the day.

select EXTRACT(HOUR FROM order_time) , count(order_id)
from orders
group by EXTRACT(HOUR FROM order_time);

--Join relevant tables to find the category-wise distribution of pizzas.
select category , count(name) from pizza_type
group by category;

--Group the orders by date and calculate the average number of pizzas ordered per day.
select round(avg(sum_quantity),0) from (select orders.order_date ,  sum(order_details.quantity) as sum_quantity
from orders join order_details
on orders.order_id = order_details.order_id
group by orders.order_date
order by orders.order_date) as order_quantity;


--Determine the top 3 most ordered pizza types based on revenue.
Select pt.name , sum(order_details.quantity * p.price) as revenue
from pizza_type pt join pizzas p
on pt.pizza_type_id = p.pizza_type_id
join order_details 
on p.pizza_id = order_details .pizza_id
group by pt.name order by revenue desc limit 3;

--Advanced:

--Calculate the percentage contribution of each pizza type to total revenue.
Select pt.category , (sum(order_details.quantity * p.price) / (select sum(o.quantity * p.price) as total_sales
from  order_details o
join pizzas p
on p.pizza_id = o.pizza_id))*100 as total_revenue
from pizza_type pt join pizzas p
on pt.pizza_type_id = p.pizza_type_id
join order_details 
on p.pizza_id = order_details .pizza_id
group by pt.category order by total_revenue desc ;

--Analyze the cumulative revenue generated over time.
select order_date , 
sum(revenue) over (order by order_date) as cum_revenue
from
(select orders.order_date, sum(order_details.quantity * pizzas.price) as revenue
from order_details join pizzas
on order_details.pizza_id = pizzas.pizza_id
join orders
on orders.order_id = order_details.order_id
group by order_date) as sales ;

--Determine the top 3 most ordered pizza types based on revenue for each pizza category.

select name , revenue from(select category , name, revenue ,
rank() over (partition by category order by revenue desc ) as rn
from (select pizza_type.category , pizza_type.name,
Sum((order_details.quantity) * pizzas.price) as revenue
from pizza_type join pizzas
on pizza_type.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_type.category , pizza_type.name) as a)as b
where rn<=3;







