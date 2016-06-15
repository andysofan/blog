#### 1.千里之行始于足下,规划好数据表的切分规模，建立你的数据库集群
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE ddal-config PUBLIC "-//openddal.com//DTD ddal-config//EN" "http://openddal.com/dtd/ddal-config.dtd">
<ddal-config>

	<settings>
		<property name="ddal.compatibilityMode" value="MySQL" />
		<property name="ddal.queryTimeout" value="3000" />
		<property name="ddal.validationQuery" value="select 1" />
		<property name="ddal.validationQueryTimeout" value="1000" />
	</settings>

	<schema name="SCHEMA_MAIN" force="false">
		<tableGroup>
			<tables>
				<table name="orders" />
				<table name="order_items" />
				<table name="order_status" />
			</tables>
			<nodes>
				<node shard="shard0" suffix="_01,_02,_03,_04" />
				<node shard="shard1" suffix="_01,_02,_03,_04" />
				<node shard="shard2" suffix="_01,_02,_03,_04" />
				<node shard="shard3" suffix="_01,_02,_03,_04" />
			</nodes>
			<tableRule>
				<columns>order_id</columns>
				<algorithm>order_partitioner</algorithm>
			</tableRule>

		</tableGroup>

		<tableGroup>
			<tables>
				<table name="customers" ruleColumns="id" />
				<table name="address" ruleColumns="customer_id" />
			</tables>
			<nodes>
				<node shard="shard0" suffix="_01,_02" />
				<node shard="shard1" suffix="_01,_02" />
				<node shard="shard2" suffix="_01,_02" />
				<node shard="shard3" suffix="_01,_02" />
			</nodes>
			<tableRule>
				<columns>${table.ruleColumns}</columns>
				<algorithm>order_partitioner</algorithm>
			</tableRule>
		</tableGroup>

		<table name="product">
			<broadcast>shard0,shard1,shard2,shard3</broadcast>
		</table>

		<table name="product_category">
			<broadcast>shard0,shard1,shard2,shard3</broadcast>
		</table>
		
		<table name="customer_login_log">
			<node shard="shard0" name="t_customer_login_log"/>
		</table>

	</schema>

	<cluster>
		<shard name="shard0">
			<member ref="db1m" />
		</shard>
		<shard name="shard1">
			<member ref="db2m" />
		</shard>
		<shard name="shard2">
			<member ref="db3m" />
		</shard>
		<shard name="shard3">
			<member ref="db4m" />
		</shard>
	</cluster>


	<dataNodes>
		<datasource id="db1m" class="org.apache.commons.dbcp.BasicDataSource">
			<property name="driverClassName" value="com.mysql.jdbc.Driver" />
			<property name="url"
				value="jdbc:mysql://10.199.188.136:3306/ddal_db1?connectTimeout=1000&amp;rewriteBatchedStatements=true" />
			<property name="username" value="root" />
			<property name="password" value="!Passw0rd01" />
			<property name="maxWait" value="0" />
			<property name="poolPreparedStatements" value="true" />
		</datasource>
		<datasource id="db2m" class="org.apache.commons.dbcp.BasicDataSource">
			<property name="driverClassName" value="com.mysql.jdbc.Driver" />
			<property name="url"
				value="jdbc:mysql://10.199.188.136:3306/ddal_db2?connectTimeout=1000&amp;rewriteBatchedStatements=true" />
			<property name="username" value="root" />
			<property name="password" value="!Passw0rd01" />
			<property name="maxWait" value="0" />
			<property name="poolPreparedStatements" value="true" />
		</datasource>
		<datasource id="db3m" class="org.apache.commons.dbcp.BasicDataSource">
			<property name="driverClassName" value="com.mysql.jdbc.Driver" />
			<property name="url"
				value="jdbc:mysql://10.199.188.136:3306/ddal_db3?connectTimeout=1000&amp;rewriteBatchedStatements=true" />
			<property name="username" value="root" />
			<property name="password" value="!Passw0rd01" />
			<property name="maxWait" value="0" />
			<property name="poolPreparedStatements" value="true" />
		</datasource>
		<datasource id="db4m" class="org.apache.commons.dbcp.BasicDataSource">
			<property name="driverClassName" value="com.mysql.jdbc.Driver" />
			<property name="url"
				value="jdbc:mysql://10.199.188.136:3306/ddal_db4?connectTimeout=1000&amp;rewriteBatchedStatements=true" />
			<property name="username" value="root" />
			<property name="password" value="!Passw0rd01" />
			<property name="maxWait" value="0" />
			<property name="poolPreparedStatements" value="true" />
		</datasource>
	</dataNodes>


	<algorithms>
		<ruleAlgorithm name="customer_partitioner"
			class="com.openddal.route.algorithm.HashBucketPartitioner">
			<property name="partitionCount" value="8" />
			<property name="partitionLength" value="128" />
		</ruleAlgorithm>
		<ruleAlgorithm name="order_partitioner"
			class="com.openddal.route.algorithm.HashBucketPartitioner">
			<property name="partitionCount" value="16" />
			<property name="partitionLength" value="64" />
		</ruleAlgorithm>
	</algorithms>

</ddal-config>
```
#### 2.建设你的环境,创建数据库对像
mysql_script.sql
```sql
DROP TABLE IF EXISTS customers,address,order_items,order_status,orders,product,product_category,customer_logs;

CREATE TABLE IF NOT EXISTS `customers` (
  `id` int(11) NOT NULL,
  `rand_id` int(11) DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  `customer_info` varchar(100) DEFAULT NULL,
  `birthdate` date DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY (`birthdate`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE IF NOT EXISTS `address` (
  `address_id` int(11) NOT NULL,
  `customer_id` int(11) DEFAULT NULL,
  `address_info` varchar(512) DEFAULT NULL,
  `zip_code` varchar(16) DEFAULT NULL,
  `phone_num` varchar(16) DEFAULT NULL,
  PRIMARY KEY (`address_id`),
  KEY (`customer_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;


CREATE TABLE IF NOT EXISTS `orders` (
  `order_id` int(11) NOT NULL,
  `customer_id` int(11) NOT NULL,
  `order_info` varchar(218) DEFAULT NULL,
  `create_date` datetime NOT NULL,
  PRIMARY KEY (`order_id`),
  KEY (`customer_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE IF NOT EXISTS `order_items` (
  `item_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `item_info` varchar(218) DEFAULT NULL,
  `create_date` datetime NOT NULL,
  PRIMARY KEY (`order_id`),
  KEY (`create_date`),
  FOREIGN KEY (`order_id`) REFERENCES `orders` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE IF NOT EXISTS `order_status` (
  `status_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `order_status` int(2) DEFAULT NULL,
  `create_date` datetime NOT NULL,
  PRIMARY KEY (`order_id`),
  KEY (`create_date`),
  FOREIGN KEY (`order_id`) REFERENCES `orders` (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE IF NOT EXISTS `product_category` (
  `product_category_id` int(11) NOT NULL,
  `order_id` int(11) NOT NULL,
  `category_info` int(2) DEFAULT NULL,
  `create_date` datetime NOT NULL,
  PRIMARY KEY (`product_category_id`),
  KEY (`create_date`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

CREATE TABLE IF NOT EXISTS `product` (
  `product_id` int(11) NOT NULL,
  `product_category_id` int(11) NOT NULL,
  `product_name` int(2) DEFAULT NULL,
  `create_date` datetime NOT NULL,
  PRIMARY KEY (`product_id`),
  KEY (`create_date`),
  FOREIGN KEY (`product_category_id`) REFERENCES `product_category` (`product_category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;


CREATE TABLE IF NOT EXISTS `customer_logs` (
  `id` int(11) NOT NULL,
  `customer_id` int(11) NOT NULL,
  `logintime` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

```
#### 3.在OpendDDAL执行上面的SQL，OpendDDAL会在数据库集群上创建对应的数据库对象
```java
    //创建数据源
    BasicDataSource ds = new BasicDataSource();
    ds.setDriverClassName("com.openddal.jdbc.JdbcDriver");
    ds.setUrl("jdbc:openddal:");
    ds.setDefaultAutoCommit(false);
    ds.setDefaultTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
    ds.setTestOnBorrow(true);
    
    //运行mysql_script.sql脚本
    ScriptRunner runner = new ScriptRunner(ds.getConnection());
    runner.setAutoCommit(true);
    runner.setStopOnError(true);

    String resource = "script/mysql_script.sql";
    Reader reader = new InputStreamReader(Utils.getResourceAsStream(resource));
    try {
        runner.runScript(reader);
    } catch (Exception e) {
        Assert.fail(e.getMessage());
    }
```
#### 4.怎么样执行SQL，看看执行设划
````java
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            stmt = conn.createStatement();
            rs = stmt.executeQuery("EXPLAIN PLAN FOR CREATE TABLE IF NOT EXISTS `order_items`(`item_id` int(11) NOT NULL,`order_id` int(11) NOT NULL,`item_info` varchar(218) DEFAULT NULL,`create_date` datetime NOT NULL, PRIMARY KEY (`order_id`),  KEY (`create_date`), FOREIGN KEY (`order_id`) REFERENCES `orders` (`order_id`)) ENGINE=InnoDB DEFAULT CHARSET=latin1");
            printResultSet(rs);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.closeSilently(rs);
            JdbcUtils.closeSilently(stmt);
            JdbcUtils.closeSilently(conn);
        }
````
执行结果
````sql
PLAN 
-------------------
MULTIPLE_EXECUTION
    execute on shard0: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_01(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_01(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard0: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_02(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_02(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard0: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_03(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_03(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard0: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_04(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_04(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard1: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_01(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_01(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard1: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_02(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_02(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard1: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_03(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_03(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard1: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_04(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_04(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard2: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_01(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_01(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard2: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_02(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_02(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard2: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_03(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_03(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard2: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_04(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_04(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard3: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_01(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_01(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard3: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_02(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_02(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard3: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_03(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_03(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1
    execute on shard3: CREATE TABLE IF NOT EXISTS ORDER_ITEMS_04(ITEM_ID INT NOT NULL, ORDER_ID INT NOT NULL, ITEM_INFO VARCHAR(218) DEFAULT NULL, CREATE_DATE DATETIME NOT NULL,  CONSTRAINT PRIMARY KEY(ORDER_ID),  INDEX(CREATE_DATE),  CONSTRAINT FOREIGN KEY(ORDER_ID) REFERENCES ORDERS_04(ORDER_ID)) ENGINE = InnoDb DEFAULT CHARACTER SET = latin1 

````