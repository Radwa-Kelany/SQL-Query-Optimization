## Functions to inset bulk data to e-commerce tables
### 1- Write database function to insert data in  categories table around 100k row 
<img width="502" height="582" alt="image" src="https://github.com/user-attachments/assets/e5335241-d76c-45d6-a2d2-aa89accb3c38" />

### 2- Write database functions to insert data in the product table around 1M rows

<img width="387" height="195" alt="product_table_1" src="https://github.com/user-attachments/assets/72212d1c-bb0a-40e3-b4aa-fbe4a98a922b" />

<img width="716" height="836" alt="product_table_2" src="https://github.com/user-attachments/assets/03b6b3ce-59c1-4b94-8a8b-ea1ecd68c1d3" />


### 3- Write database functions to insert data in customer table around 5M  rows 

<img width="645" height="703" alt="customer_table" src="https://github.com/user-attachments/assets/dbc73d17-45b0-479c-9271-555da0e4baaa" />

### 4- Write database functions to insert data in orders table around 20M rows based on the inserted data in customers.
<img width="526" height="172" alt="orders_table_1" src="https://github.com/user-attachments/assets/7ea78764-e1c7-4a58-9b85-fe6f9685277d" />

<img width="840" height="837" alt="orders_table_2" src="https://github.com/user-attachments/assets/eab06019-cfc8-4377-bf46-56776e37ae80" />

<img width="558" height="722" alt="orders_table_3" src="https://github.com/user-attachments/assets/5e0b8052-7dec-47c9-bacd-e8d35e0932be" />

### 5- Write database functions to insert data in order_details table around 20M rows based on the inserted data in product and orders tables.
<img width="652" height="786" alt="order_details_table_1" src="https://github.com/user-attachments/assets/4b67c964-6755-4fe4-8489-832221c70e3d" />


### update total_amont of orders table based on insert data to order_details table 
<img width="631" height="183" alt="orders_table_4" src="https://github.com/user-attachments/assets/35a8065e-719b-44e9-976b-64bf069d197e" />

### update unit price in order_details table to be equal price column in product table

<img width="626" height="130" alt="order_details_table_2" src="https://github.com/user-attachments/assets/f2156575-30cb-4b49-a980-da123ee9dd54" />


## Query Optimization Examples 
### 1- Write an SQL query to generate a daily report of the total revenue for a specific date.  
<img width="633" height="507" alt="query_example_1_1" src="https://github.com/user-attachments/assets/0c5ddb01-4986-4fd6-9b76-8593c3f8d47a" />

<img width="637" height="777" alt="query_example_1_2" src="https://github.com/user-attachments/assets/be01c824-d170-4f27-8159-f2f36c38728d" />

### 2- Write an SQL query to generate a monthly report of the top-selling products in a given month.

<img width="607" height="476" alt="query_example_2_1" src="https://github.com/user-attachments/assets/be2bccec-0b00-41d8-8ad1-a451501bf04e" />

<img width="630" height="772" alt="query_example_2_2png" src="https://github.com/user-attachments/assets/c747428b-9796-43c2-86d9-dc867df02282" />

### 3- Write SQL Query to Retrieve the total number of products in each category.

<img width="586" height="457" alt="query_example_3_1" src="https://github.com/user-attachments/assets/59fe5bbd-33af-476e-a2d9-396bfd5fbc36" />

<img width="608" height="726" alt="query_example_3_2" src="https://github.com/user-attachments/assets/2da7a588-172c-4049-ba69-ea847ad3501a" />

### 4- Write SQL Query to Find the top customers by total spending.

<img width="565" height="476" alt="query_example_4_1" src="https://github.com/user-attachments/assets/800a4821-d849-436c-a66c-c6ee8c99df30" />

<img width="596" height="783" alt="query_example_4_2_1" src="https://github.com/user-attachments/assets/7c691ec1-d004-488b-acbd-3c456c8bfc9e" />

<img width="598" height="468" alt="query_example_4_2_2" src="https://github.com/user-attachments/assets/a53b70e2-9419-4459-9629-6e477e3d4a5a" />

### 5- Write SQL Query to List products that have low stock quantities of less than 10 quantities.

<img width="581" height="372" alt="query_example_5_1" src="https://github.com/user-attachments/assets/e7eadbad-77ff-4add-b14e-cbca81fa9d99" />

<img width="631" height="668" alt="query_example_5_2" src="https://github.com/user-attachments/assets/d170e805-d10c-4c11-b232-b0e54d053fa3" />

<img width="593" height="416" alt="query_example_5_2_2" src="https://github.com/user-attachments/assets/5c4dad1c-85b2-4694-aa11-535ca7fc253e" />

### *********************************************************************************************
## SQL Query Optimization Using Denormalization and Materlialized View:
## Example 1
### Write SQL Query to Retrieve the total number of products in each category. 
\```sql
-- Write SQL Query to Retrieve the total number of products in each category. 
select category_name, count(*) 
from category c join product p
on c.category_id = p.category_id
and p.is_deleted = false
group by c.category_id
order by c.category_id limit 2;

-- Optimization techniques:

-- 1- create index on category_id of product table 

create index idx_category_id on product(category_id);

-- ************************************************************************************************************
-- 2- Create Denormalized column "products_count" in category table

-- update category table with new column "products_count"
alter TABLE category add COLUMN products_count INT;


-- Test if column added 
select * from category order by category_id limit 1;
-- ------------------------------------------------------------------------------------------------------------

-- A- For Existing Data:
-- fill 'products_count' column with existing data 
UPDATE category c
SET products_count = p.product_count
FROM (
    SELECT category_id, COUNT(*) as product_count
    FROM product 
    where product.is_deleted = false
    GROUP BY category_id
) as p
WHERE c.category_id = p.category_id;

-- -------------------------------------------------------------------------------------------------------------

-- B- For Future Data:
-- create trigger when insert new product to products table, increment products_count column in category table
create or replace function increment_products_count ()
returns trigger as $$ 
begin
UPDATE category c
SET products_count = products_count + 1
WHERE c.category_id = new.category_id;
return new;
end;
$$
language plpgsql;

CREATE TRIGGER trigger_increment_products_count
after insert on product
FOR EACH ROW EXECUTE FUNCTION increment_products_count();

-- test the trigger 
select * from category order by category_id limit 1;

--  for example: products_count = 11;

insert  into product (product_id, category_id, product_name,description, price, stock_quantity)
values (1000002, 1, 'secondt_prod', 'descri', 100.00, 12);


select * from category order by category_id limit 1;

-- becomes: products_count = 12;
-- if products_count of category_id = 1 increase by 1; then the trigger is worked.
-- -------------------------------------------------------------------------------------------------------------
 
-- C- For Decrementation 
-- create trigger when delete product from products table, decrement products_count column in category table

-- Deletion here is soft delete, as we will add new column " is_deleted" to product table with default value of "false"

alter TABLE  product add COLUMN  is_deleted boolean  not null default false ;

--  The following function locks the row from any update if is_deleted = true,
--  but if is_deleted = false, we add another check if the comming update includes [is_deleted = true]
--  so we will decrement the products_count and allow update of soft delete
--  but if the coming update inclused any other update rather than soft delete
--  we do not change products_count in category table and allow update of product table for other columns.
create or replace function decrement_products_count()
returns trigger as $$ 
begin
    IF OLD.is_deleted THEN
        RAISE EXCEPTION 'Row is permanently locked';

    ELSEIF  OLD.is_deleted = false then 
         if new.is_deleted  then 
            UPDATE category c
            SET products_count = products_count - 1
            WHERE c.category_id = new.category_id;
         END IF;
    END IF;
   return new;
end;
$$
language plpgsql;

CREATE TRIGGER trigger_decrement_products_count
before update ON  product
FOR EACH ROW EXECUTE FUNCTION decrement_products_count();


-- --------------------------------------------------------------------------------------------------
-- Test update of other columns rather than is_deleted column:
update product p
set price = 200
where product_id = 1000003;

select * from product  where product_id = 1000003;

select * from category order by category_id limit 1;

-- The result should include 'is_deleted = false', and 'products_count' not change.
-- --------------------------------------------------------------------------------------------------

-- Test update is_deleted column to be true:
update product p
set is_deleted = true
where product_id = 1000003;

select * from product  where product_id = 1000003;

select * from category order by category_id limit 1;

-- The result should include 'is_deleted = true', and 'products_count' decrease by 1.

-- if we try to make any further update on that row "product_id = 1000003", the DBMS will not allow update and 
-- give error message "Row is permanently locked"


-- SQL query after denormalization will be 

select category_name, products_count from category limit 2;


-- ********************************************************************************************************************
  -- 3- Create Materlized View

CREATE MATERIALIZED VIEW category_products_count as
select c.category_id, category_name, count(*) as products_count
from category c join product p
on c.category_id = p.category_id
and p.is_deleted = false
group by c.category_id
order by c.category_id;

select * from category_products_count limit 10;


-- Notes: 
-- if query is heavily used in dashborad, we can use Denormalized table or column
-- but if it is less frequently used, for example " every month or week", we can use Materlialized View with cron job for refresh.


\```
## Example 2
### Write SQL Query to Find the top customers by total spending. 
\```sql
-- Write SQL Query to Find the top customers by total spending. 

  select concat('first_name',' ', 'last_name') as customer_name , sum(total_amount) as total_spending
  from customer c join orders o
  on c.customer_id = o.customer_id 
  group by c.customer_id 
  order by total_spending desc
  limit 10;
  
  -- ------------------------
  -- Optimization techniques:
  -- ------------------------
  
  -- 1- Create index on customer_id table on orders table 
  create index idx_customer_id on orders(customer_id);
  
  -- ************************************************************************************************************
  -- 2- Create Denormalized column "total_spending" in customer table
  
  alter table customer add column total_spending INT;
  
-- Test if column added 
  select * from customer order by customer_id limit 1;
  
-- A- For Existing Data:
-- fill 'total_spending' column with existing data 
  update customer c 
  set total_spending = o.total_spending
  from 
  (select customer_id, sum(total_amount) as total_spending
  from orders 
  group by customer_id 
  ) as o
  where c.customer_id = o.customer_id;


-- -------------------------------------------------------------------------------------------------------------

-- B- For Future Data:
-- create trigger when insert new order to orders table, update total_spending column in customer table

create or replace function increment_customer_total_spending()
returns trigger as $$
begin
update customer c
set total_spending = total_spending +  new.total_amount 
where c.customer_id = new.customer_id;
return new;
end;
$$
language plpgsql;

CREATE TRIGGER trigger_increment_customer_total_spending
after insert on orders
FOR EACH ROW EXECUTE FUNCTION increment_customer_total_spending();


-- Test the trigger 
select * from customer where customer_id = 1;
-- Result is total_spending = 3,500;

insert into orders (order_id, customer_id, order_date, total_amount)
values (20000001, 1 , '2026-01-02', 1000);

select * from customer where customer_id = 1;
-- Result is total_spending = 4,500;
-- Trigger is working


-- What if customer cancel the order totally or partially and make refund to his money?
-- This means that the total_amount will decrease... 
-- No possible cases of increase total_amount of exixting order... right! so the update will be decrement.
-- First we must add 'status' column to orders table for register last state of orders and its date.
-- Second we need to make log table for orders to track all its states [confirmed, refunded...]  with timestamp
-- for updating 'total spending' column on customer table we depend on 'total_amount' column and 'status' column  of orders table;
   
   alter table orders add column status varchar(15) not null default 'confirmed';
   
   create or replace function decrement_customer_total_spending()
   returns trigger as $$
   DECLARE
     difference_amount INTEGER;
   begin
    if new.status = 'refunded' then
   	difference_amount := old.total_amount - new.total_amount;
    update customer c
    set total_spending = total_spending - difference_amount
    where c.customer_id = new.customer_id;
    end if;
    return new;
    end;
   $$
   language plpgsql;
   
   create trigger trigger_decremrnt_customer_total_spending
   after update on orders
   for each row execute function decrement_customer_total_spending();
   
   -- Test Trigger
   -- before update
   select  total_spending from customer where customer_id = 1;
   -- Result is total_spending = 8000;
   select total_amount from orders where order_id =20000001;
   -- Result is total_amount = 1000;
   
   update orders o
   set total_amount = 600.
       status = 'refunded'
   where order_id = 20000001;
   
   select  total_spending from customer where customer_id = 1;
   -- Result is total_spending = 7600;  -- the difference is 1000 - 600 = 400
   
   select total_amount from orders where order_id =20000001;
   -- Result is total_amount = 600;

   -- SQL query after denormalization will be 

   select concat('first_name' , ' ' , 'last_name') as customer_name, total_spending from customer order by total_spending desc limit 2;
   
   -- ********************************************************************************************************************
  -- 3- Create Materlized View

CREATE MATERIALIZED VIEW customer_total_spending as
  select c.customer_id, concat('first_name',' ', 'last_name') as customer_name , sum(total_amount) as total_spending
  from customer c join orders o
  on c.customer_id = o.customer_id 
  group by c.customer_id 
  order by total_spending desc;

select * from customer_total_spending order by total_spending desc limit 2;
\```
