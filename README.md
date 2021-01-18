# Pet shop project documentation
The database is designed to store and process information about clients / co-workers / orders / products.

### Tables. Schema
This database contains 9 tables. Their scheme and relationship:
![tables](/img/table_schema.png)
### Tables. Description
 - the `products` table contains information about the products that are sold at the pet shop. Fields - id, product category and name.
Creating a table:

 ```sql
    CREATE TABLE IF NOT EXISTS `products` (
    	`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    	`category_id` int(10) NOT NULL,
    	`name` varchar(50) DEFAULT NULL,
    	PRIMARY KEY (`id`)	
    );
    
```
- the table `products_categories` contains product categories.
Creating a table:

```sql
    CREATE TABLE IF NOT EXISTS `products_categories` (
    	`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    	`name` varchar(45) DEFAULT NULL,
    	PRIMARY KEY (`id`)	
    );
```
 - table `price_history`

```sql
    CREATE TABLE IF NOT EXISTS `price_history` (
    	`product_id` int(10) unsigned NOT NULL,
    	`date` date NOT NULL,
    	`time` time NOT NULL,
    	`price` float(3) NOT NULL
    );
```
 - table `co-workers`

```sql
    CREATE TABLE IF NOT EXISTS `co-workers` (
    	`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    	`firstname` varchar(45) DEFAULT NULL,
    	`lastname` varchar(45) DEFAULT NULL,
    	`dbirth` date DEFAULT NULL,
    	`email` varchar(30) DEFAULT NULL,
    	`phone` char(12) DEFAULT NULL,
    	`address` varchar(80) DEFAULT NULL,
    	`gender` enum('M','F') DEFAULT NULL,
    	PRIMARY KEY (`id`)
    );
```
 - table `positions`

```sql
    CREATE TABLE IF NOT EXISTS `positions` (
    	`id` int(3) unsigned NOT NULL AUTO_INCREMENT,
    	`name` varchar(50) DEFAULT NULL,
    	PRIMARY KEY (`id`)
    );
```
 - table `appointment_to_position`

```sql
    CREATE TABLE IF NOT EXISTS `appointment_to_position` (
    	`co_worker_id` int(10) unsigned NOT NULL,
    	`position_id` int(10) unsigned NOT NULL
    );
```
 - table `clients`

```sql
    CREATE TABLE IF NOT EXISTS `clients` (
    	`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    	`firstname` varchar(45) DEFAULT NULL,
    	`lastname` varchar(45) DEFAULT NULL,
    	`phone` char(12) DEFAULT NULL,
    	`gender` enum('M','F') DEFAULT NULL,
    	PRIMARY KEY (`id`)
    );
```
 - table `orders`

 ```sql
    CREATE TABLE IF NOT EXISTS `orders` (
    	`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    	`client_id` int(10) unsigned NOT NULL,
    	`co-worker_id` int(10) unsigned NOT NULL,
    	`date` date DEFAULT NULL, 
    	`time` time DEFAULT NULL,
    	`order_sum` float(7) NOT NULL,
    	PRIMARY KEY (`id`)	
    );
```
 - table `orders_table`

 ```sql
    CREATE TABLE IF NOT EXISTS `orders_table` (
	`order_id` int(10) unsigned NOT NULL,
	`product_id` int(10) unsigned NOT NULL,
	`product_count` int(7) unsigned NOT NULL
    );
```

### Typical operations
##### Inserting data
- Creating an order for a new client:
 
 ```sql
 INSERT INTO `clients` (`firstname`, `lastname`, `phone`, `gender`) VALUES
	('Kelly', 'Sousa', '44567895', 'F');


INSERT INTO `orders` (`client_id`, `co-worker_id`, `date`, `time`, `order_sum`) VALUES (
	(
		SELECT id FROM `clients` WHERE `phone` = '44567895' AND `lastname` = 'Sousa'),
		5, 
		CURDATE(),
		CURTIME(), 
		(SELECT `price` FROM `price_history`
		WHERE `price_history`.`product_id` = (SELECT `id` FROM `products` WHERE `name` = 'parrot')
		AND `price_history`.`date`= (SELECT MAX(`date`) FROM `price_history` WHERE `date` <= (CURDATE()) AND `product_id` = (SELECT `id` FROM `products` WHERE `name` = 'parrot')))
	);

INSERT INTO `orders_table` VALUES
	(
		(SELECT MAX(orders.id) FROM `orders` LIMIT 1),
		(SELECT `id` FROM `products` WHERE `name` = 'parrot'),
		1
	);
	
 ```

 ##### Updating data

  - Updating the сo-worker's phone number:

```sql
    UPDATE `co-workers` SET `phone` = '41234525' WHERE `lastname` = 'Miina' AND phone = '42680356' LIMIT 1;
```

##### Deleting data

 - Deleting co-worker:

```sql
    DELETE FROM `co-workers` WHERE `id` = 9  LIMIT 1;
    DELETE FROM `appointment_to_position` WHERE id = `9` LIMIT 5;
```

### Typical queries

 - Find the most popular product:

```sql
    SELECT t1.name as t, MAX(t1.amount) as tt
    FROM (
    	SELECT p.name name, SUM(ot.product_count) amount
    	FROM `products` p
    	INNER JOIN `orders_table` ot ON ot.product_id = p.id 
    	GROUP BY ot.product_id) as t1
    GROUP BY t
    ORDER BY t DESC
    LIMIT 1
```

 - Find product price at the time of sale:

 ```sql
    SELECT `price` FROM `price_history`
    WHERE `price_history`.`product_id` = 15 
    AND  `price_history`.`date`= (SELECT MAX(`date`) FROM `price_history` WHERE `date` <= (SELECT `date` FROM `orders` WHERE `orders`.id = 16))
    LIMIT 100;
```


### Views

 - Find out how many products have been sold for all the time:

 ```sql
    CREATE VIEW `sold_products` AS
    SELECT p.name, SUM(ot.product_count) amount
    FROM `products` p
    INNER JOIN `orders_table` ot ON ot.product_id = p.id 
    GROUP BY ot.product_id,
    LIMIT 100;
```
 - Find the client who bought the most:

 ```sql
    CREATE VIEW `good_client` AS
    SELECT c.id, c.firstname,```sql
    CREATE VIEW `good_client` AS
    SELECT c.id, c.firstname, c.lastname, SUM(o.order_sum) as all_sum
    FROM `clients`  c, `orders` o
    WHERE c.id= o.client_id
    GROUP BY (c.firstname)
    ORDER BY SUM(o.order_sum) DESC
    LIMIT 1
``` c.lastname, SUM(o.order_sum) as all_sum
    FROM `clients`  c, `orders` o
    WHERE c.id= o.client_id
    GROUP BY (c.firstname)
    ORDER BY SUM(o.order_sum) DESC
    LIMIT 1
```