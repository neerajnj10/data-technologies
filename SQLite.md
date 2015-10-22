SQLite


You will be constructing a `coffee.db` containing tables for `coffees`, `salespeople`, `customers`, `orders`, and `order_items`. Once constructed, you will query this database using R.

1 Create the `coffee.db` from the `coffee.sql` file. List the tables.  
```
1.1   
Open shell
bash-4.1$ sqlite3
sqlite> .read coffee.sql
sqlite> .database    
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main         
sqlite> .tables
coffees  customers    order_items  orders       salespeople 
sqlite> .backup coffee.db
sqlite> .restore  coffee.db    
sqlite> .tables     
coffees  customers    order_items  orders       salespeople 

```



2 Display and explain the schema.  

#schema obtained from shell
```   
sqlite> .restore  coffee.db     
sqlite> .schema   

```
CREATE TABLE coffees (
  id INTEGER PRIMARY KEY,
  coffee_name TEXT NOT NULL,
  price REAL NOT NULL
);
CREATE TABLE customers (
  id INTEGER PRIMARY KEY,
  company_name TEXT NOT NULL,
  street_address TEXT NOT NULL,
  city TEXT NOT NULL,
  state TEXT NOT NULL,
  zip TEXT NOT NULL
);
CREATE TABLE order_items (
  id INTEGER PRIMARY KEY,
  order_id INTEGER,
  coffee_id INTEGER,
  coffee_quantity INTEGER,
  FOREIGN KEY(order_id) REFERENCES orders(id),
  FOREIGN KEY(coffee_id) REFERENCES coffees(id)
);
CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  customer_id INTEGER,
  salesperson_id INTEGER,
  FOREIGN KEY(customer_id) REFERENCES customers(id),
  FOREIGN KEY(salesperson_id) REFERENCES salespeople(id)
);
CREATE TABLE salespeople (
  id INTEGER PRIMARY KEY,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  commission_rate REAL NOT NULL
);    
            
  
following are the five tables in the database:      
coffees (id[PK],coffee_name,price)       
customers(id[PK], company_name, street_address,city,state,zip)      
order_items(id[PK],order_id[FK orders.id ], coffee_id[FK coffees.id],  coffee_quantity )        
orders(id[PK],customer_id[FK customer.id], salesperson_id[FK salespeople.id])        
salespeople(id[PK],first_name, last_name, commission_rate)    
    
Table `order_items`  connect to table `orders` and table `coffees`  with order_id[FK orders.id ]  and product_id[FK coffees.id]         
Table `orders ` connect to table `customers `and table  `salespeople` with customer_id[FK], salesperson_id[FK salespeople.id])        
   

3 Discuss the extent to which the normal forms are satified.     
```{r}  
library(RSQLite)
drvr<-dbDriver("SQLite")
con<-dbConnect(drvr,dbname="coffee.db")
(coffees.df<-dbGetQuery(con,"SELECT * FROM coffees"))
(customers.df<-dbGetQuery(con,"SELECT * FROM customers"))
(items.df<-dbGetQuery(con,"SELECT * FROM order_items"))
(orders.df<-dbGetQuery(con,"SELECT * FROM orders"))
(sale.df<-dbGetQuery(con,"SELECT * FROM salespeople"))

```
1NF.
The database satisfies the three normal form where the columns in five tables are atomic. No redundanctable and has a primary key.     

2NF.        
Each table is in first normal form and all columns in each table relate to the the primary key in the table.  

3NF.   
All tables are in second normal form and all columns in each table are not related to to each other, only related to the primary key in the table.  
 
```````



4 What are the total sales and total commissions for each salesperson? The rows of the resulting table should start with the `last_name` of the salespeople.
```{r}
library(RSQLite)
drvr<-dbDriver("SQLite")
con<-dbConnect(drvr,dbname="coffee.db")
total_sales_comm<-dbGetQuery(con,"SELECT last_name, commission_rate/100*sales total FROM ( SELECT order_id, sum(coffee_quantity*price) sales FROM coffees c , order_items o where c.id=o.coffee_id  GROUP BY order_id) t, salespeople s, orders where s.id=orders.salesperson_id and t.order_id=orders.id;")
total_sales_comm

```



5 What are the total amounts bought by each customer? The rows of the table should start with the `company_name`.
```{r}
total_amt<-dbGetQuery(con,"SELECT company_name, sales FROM ( SELECT order_id, sum(coffee_quantity*price) sales FROM coffees c , order_items o where c.id=o.coffee_id  GROUP BY order_id) t, customers c, orders where c.id=orders.customer_id and t.order_id=orders.id;")
total_amt

```

6 What are the sales numbers and the dollar amounts of each type of coffee sold? The rows of the table should start with the `coffee_name`.
```{r}
coffee_sales<-dbGetQuery(con,"SELECT coffee_name,sum(coffee_quantity) quantity, sum(coffee_quantity*price) sales  FROM coffees c , order_items o where c.id=o.coffee_id  GROUP BY coffee_id;")
coffee_sales


```

7 create an R data frame for each order item (in this case 4 rows). The columns of the table should be: `company_name`, `last_name`, `commission_rate`, `coffee_name`, `coffee_quantity`, and `price`. More descriptive names of the columns can be used.
```{r}

des<-dbGetQuery(con," select company_name, last_name, commission_rate, coffee_name, coffee_quantity,  price from (select * from (select coffee_name,  coffee_quantity,price, order_id aid from coffees c,order_items o where o.coffee_id=c.id)  cross join (select company_name, orders.id bid from customers, orders where customers.id=orders.customer_id ) where aid=bid)  cross join (select last_name, commission_rate, orders.id from salespeople s, orders where orders.salesperson_id=s.id) where aid=id;") 
des
               
```

8 Redo questions 4---6 in R based on the data in the data frame created in question 7.
```{r}
##question no.4
sales<-des$coffee_quantity*des$price
total<-sales*des$commission_rate/100
destotal<-cbind(des,total)
total_sale_redo<-aggregate(total~last_name, data=destotal, FUN=sum)
total_sale_redo

##question no.5
total_buy_redo<-aggregate(sales~company_name, data=destotal, FUN=sum)
total_buy_redo

#question no.6
q<-aggregate(coffee_quantity~coffee_name, data=destotal, FUN=sum)
q
quantity<-q[,2]
coffee_redo<-aggregate(sales~coffee_name, data=destotal, FUN=sum)
coffee_sales_redo<-cbind(coffee_redo,quantity)
coffee_sales_redo

```


