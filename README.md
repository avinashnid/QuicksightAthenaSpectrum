# Serverless Analytics Workshop
Amazon QuickSight, Amazon Athena and Amazon Spectrum workshop. Workshop will focus on ingesting data into Athena & Spectrum, combining it with other data sources, and visualizing it in QuickSight.

Hands on workshop is broken up into 6 different sections to get you familiar with the Quicksight, Athena & Spectrum services:</br>
- [10 min  - Sign Up for AWS](#sign-up-for-aws)</br>
- [10 min - Architecture and Permissions](#architecture-and-permissions)</br>
- [10 min - Query a file on S3 using Athena](#query-a-file-on-s3-with-athena)</br>
- [20 min - Query a file on S3 using Redshift Spectrum](#query-a-file-on-s3-with-redshift-spectrum)</br>
- [15 min - Integrating Glue with Athena and Redshift Spectrum](#integrating-glue-with-athena-and-spectrum)</br>
- [20 min - Breakout Exercises](#breakout-exercises)</br>
- [50 min - Visualizing and Dashboarding with QuickSight](#visualizing-and-dashboarding-with-quicksight)</br>

# Sign Up for AWS

### Create your AWS Account
Navigate to [Amazon AWS Free Tier](https://aws.amazon.com/free).
There are a variety of services that offer free tier to start building your solutions. Choose basic support plan.


# Architecture and Permissions
Purpose of serverless components is to reduce the overhead of maintaining, provisioning, and managing servers to serve applications. AWS provides three compelling serverless services through AWS to store large amounts of data, manipulate data at scale, query data at scale and speed, and easily visualize it - namely **AWS Glue, Amazon Athena, Amazon QuickSight.**
<br/>
![alt text](/images/architecture.png)
<br/> To get these services working we need to allow these services to talk to one another. Following we will set up permissions for to accomplish this through AWS IAM.


## Build Permissions and S3 Bucket
AWS provides a service to build resources out of predefined templates using CloudFormation. We will use a CloudFormation script to automate the creation of permissions, roles, and other elements we may require.

To create this we need to run a cloud formation template:
1. Make sure you are in the N. Virginia Region
1. Under services, click **CloudFormation** under Management Tools. </br>
![alt text](/images/cloudFormation.PNG)</br></br>
2. Click **Create Stack**
3. Under "Choose a template", select the **"Specify an Amazon S3 template URL"** radio button option and enter this template url: 
```
https://s3-us-west-2.amazonaws.com/slalom-seattle-ima/scripts/cloudformation/cf_QuickSightAthena_Workshop.template
```
4. Click **Next**
5. Enter the a name for your stack, like **QuicksightAthena-Workshop**
5. Provide a unique name for your bucket to store your data - **It needs to be globally unique name and the bucket name must contain only lowercase letters, numbers, periods (.), and dashes (-). No spaces!**
5. Hit **Next**
5. Hit **Next**
5. There is an acknowledge checkbox for you to review, and hit **Create**
6. We will wait a couple minutes until the progress says CREATE_COMPLETE</br>
![alt text](/images/cloudformationStatus.PNG)
<hr/></br>

# Query a file on S3 With Athena
To get started with Athena and QuickSight, we need to provide data to query. This data may originate from a variety of sources into S3, but for this example we will upload a file into S3 manually.
1. **Open the S3 Console** from the Services drop down menu
2. Click your newly created bucket, by you or by our CloudFormation script.
3. Hit **Create folder** and name it "B2B"
4. Create another folder within B2B called "orders"
5. Download sample dataset [B2B Orders](https://slalom-seattle-ima.s3-us-west-2.amazonaws.com/docs/B2B%20Dataset.zip). Unzip the dataset files into a folder. Click on new folder and **Upload** the **orders.csv**.
6. Make note of the folders you saved this file under.
7. Open the **Athena** console from the Services dropdown.
8. Create a table manually via DDL in the query window.
9. Replace the location value to the folder location of your dataset. s3://**your bucket name**/B2B/orders/
```sql
CREATE DATABASE labs
```
```sql 
CREATE EXTERNAL TABLE IF NOT EXISTS labs.orders (
  `row_id` int,
  `order_id` string,
  `order_date` date,
  `ship_date` date,
  `ship_mode_id` int,
  `customer_id` string,
  `segment` int,
  `product_id` string,
  `sales` double,
  `company_id` int,
  `quantity` int,
  `discount_pct` double,
  `profit_amt` double 
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = ',',
  'field.delim' = ','
) LOCATION 's3://<your bucket here>/B2B/orders/'
TBLPROPERTIES ('has_encrypted_data'='false')
```
10. Hit **Run Query**

11. Run the following SQL statement and make sure that your table is reading correctly:
```sql
SELECT * 
FROM labs.orders LIMIT 100
```
12. Show Create Table statement helps you better understand what it going on behind the scenes when creating a table.
```sql
SHOW CREATE TABLE labs.orders
```

Alternate definitions, *schema on read*:
```sql
DROP TABLE labs.orders;
CREATE EXTERNAL TABLE IF NOT EXISTS labs.orders (
  `row_id` string,
  `order_id` string,
  `order_date` string,
  `ship_date` string,
  `ship_mode_id` string,
  `customer_id` string,
  `segment_id` string,
  `product_id` string,
  `sale` string,
  `company_id` string,
  `quantity` string,
  `discount_pct` string,
  `profit_amt` string 
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
   "separatorChar" = ",",
   "quoteChar"     = "\"",
   "escapeChar"    = "\\"
)  
LOCATION "s3://marioj-bucket-02/B2B/orders/"
TBLPROPERTIES ("skip.header.line.count"="1")
```
More resources:
- [Athena Supported Formats](http://docs.aws.amazon.com/athena/latest/ug/supported-formats.html)
- [Athena Language Reference](http://docs.aws.amazon.com/athena/latest/ug/language-reference.html)

Congratulations, you queried your first S3 file through Amazon Athena!
<hr/></br>

# Query a file on S3 With Redshift Spectrum
Amazon Redshift Spectrum enables you to run Amazon Redshift SQL queries against exabytes of data in Amazon S3. With Redshift Spectrum, you can extend the analytic power of Amazon Redshift beyond data stored on local disks in your 
data warehouse to query vast amounts of unstructured data in your Amazon S3 “data lake”

To use the spectrum service, you need to instantiate a Redshift Cluster.Before you begin setting up an Amazon Redshift cluster, pls make sure that you complete the following pre-req steps
1. [Set Up Prerequisites](http://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-prereq.html)
2. Create an IAM role for Sprectum
    - Open the IAM Console.
    - In the navigation pane, choose Roles.
    - Choose Create New Role.
    - Choose AWS Service Role, and then scroll to Amazon Redshift. Choose Select.
    - The Attach Policy page appears. Choose AmazonS3FullAccess and AmazonAthenaFullAccess. Choose Next Step.
    - For Role Name, type a name for your role, for example mySpectrumRole.
    - Review the information, and then choose Create Role.
    - Copy the Role ARN to your clipboard—this value is the Amazon Resource Name (ARN) for the role that you just created. You use that value when you create external tables to reference your data files on Amazon S3.
 </br>

Now that you have the prerequisites completed, you can launch your Amazon Redshift cluster. 

3. Click on the **Amazon Redshift** link from the Services dropdown.
4. On the Amazon Redshift Dashboard, choose Launch Cluster. 
5. On the Cluster Details page, enter the following values and then choose Continue:
   - Cluster Identifier: type examplecluster.
   - Database Name: leave this box blank. Amazon Redshift will create a default database named dev.
   - Database Port: type the port number on which the database will accept connections. You should have determined the port number in the prerequisite step of this tutorial. You cannot change the port after launching the cluster, so make sure that you have an open port number in your firewall so that you can connect from SQL client tools to the database in the cluster.
   - Master User Name: type masteruser. You will use this username and password to connect to your database after the cluster is available.
   - Master User Password and Confirm Password: type a password for the master user account.
6. On the Node Configuration page, select the following values and then choose Continue:
   - Node Type: dc2.large
   - Cluster Type: Single Node
7. On the Additional Configuration page, you will see different options depending on your AWS account, which determines the type of platform the cluster uses. To keep things simple select the EC2-VPC to launch your cluster. Use the following
   values to populate the various fields in the screen -
   - Cluster Parameter Group: select the default parameter group.
   - Encrypt Database: None.
   - Choose a VPC: Default VPC (vpc-xxxxxxxx)
   - Cluster Subnet Group: default
   - Publicly Accessible: Yes
   - Choose a Public IP Address: No
   - Enhanced VPC Routing: No
   - Availability Zone: No Preference
   - VPC Security Groups: default (sg-xxxxxxxx)
   - Create CloudWatch Alarm: No
8. Associate an IAM role with the cluster.
   For AvailableRoles, choose mySpectrumRole (defined in step 2) and then choose Continue.  
9. On the Review page, review the selections that you’ve made and then choose Launch Cluster
10. A confirmation page appears and the cluster will take a few minutes to finish. Choose Close to return to the list of clusters.
11. On the Clusters page, choose the cluster that you just launched and review the Cluster Status information. 
    Make sure that the Cluster Status is available and the Database Health is healthy before you try to connect to the database.  
12. Before you can connect to the cluster, you need to configure a security group to authorize access.
13. To Configure the VPC Security Group (EC2-VPC Platform)
   - In the Amazon Redshift console, in the navigation pane, choose Clusters.
   - Choose examplecluster to open it, and make sure you are on the Configuration tab.
   - Under Cluster Properties, for VPC Security Groups, choose your security group.
   - After your security group opens in the Amazon EC2 console, choose the Inbound tab.
   - Choose Edit, and enter the following, then choose Save:
     - Type: Custom TCP Rule.
     - Protocol: TCP.
     - Port Range: type the same port number that you used when you launched the cluster. The default port for Amazon Redshift is 5439, but your port might be different.
     - Source: select Custom IP, then type 0.0.0.0/0.
     **Note**
     Using 0.0.0.0/0 is not recommended for anything other than demonstration purposes because it allows access from any computer on the internet. In a real environment, you would create inbound rules based on your own network settings.
14. Connect to the cluster using the SQL Workbench/J client(installed in the prerequisites section in step 1). To connect from SQL Workbench perform the following steps:
	- Open SQL Workbench/J.
	- Choose File, and then choose Connect window.
	- Choose Create a new connection profile.
	- In the New profile text box, type a name for the profile.
	- Choose Manage Drivers. The Manage Drivers dialog opens.
	- Choose the Create a new entry button. In the Name text box, type a name for the driver.
	  ![alt text](/images/managedrivers.png)  
    - Choose the folder icon next to the Library box, navigate to the location of the driver, select it, and then choose Open.  
      ![alt text](/images/selectdrivers.png)
	  If the Please select one driver dialog box displays, select com.amazon.redshift.jdbc4.Driver or com.amazon.redshift.jdbc41.Driver and choose OK. SQL Workbench/J automatically completes the Classname box. 
	  Leave the Sample URL box blank, and then choose OK.
	- In the Driver box, choose the driver you just added.
	- In URL, copy the JDBC URL from the Amazon Redshift (as listed below) console and paste it.
	  - In the Amazon Redshift console, in the navigation pane, choose Clusters. 
	  - Choose examplecluster to open it, and make sure you are on the Configuration tab.
	  - On the Configuration tab, under Cluster Database Properties, copy the JDBC URL of the cluster.
	  ![alt text](/images/jdbcconnection.png)  
	- In Username, type masteruser.
	- In Password, type the password associated with the master user account.
	- Choose the Autocommit box.
	- Save the profile using Save profile list icon</br>  

At this point you have a database called dev in the redshift cluster and connected to it using the SQL Workbench/J client. 
To get started with Spectrum, we need to provide data to query. This data may originate from a variety of sources into S3, but for this example we will upload a file into S3 manually.
  - **Open the S3 Console** from the Services drop down menu
  - Hit **Create folder** and name it "B2B"
  - Create folders within B2B called "sales" and "Events"
  - Download sample dataset [Sales](https://slalom-seattle-ima.s3-us-west-2.amazonaws.com/docs/sales_ts.zip). Unzip the dataset files into a folder. In the sales folder Click on  **Upload** and select the **sales_ts.000** .
    In the Events folder, click **Upload** and select the **allevents_pipe.txt** file.
Make note of the folders you saved this file under.

To start querying Sales data in S3 using Spectrum, we need to create an external table (Sales) and 
an external schema (labs) for spectrum to access. This can be achieved by executing the following SQL statements on the SQL Workbench Client 

15. Create external schema and table
```sql
	create external schema labs 
	from data catalog 
	database 'labs' 
	iam_role 'yourIAMrole/mySpectrumRole'
	create external database if not exists;
```

16. 
```sql 
	create external table labs.sales(
	salesid integer,
	listid integer,
	sellerid integer,
	buyerid integer,
	eventid integer,
	dateid smallint,
	qtysold smallint,
	pricepaid decimal(8,2),
	commission decimal(8,2),
	saletime timestamp)
	row format delimited
	fields terminated by '\t'
	stored as textfile
	location 's3://<yourbucket>/B2B/sales/'
	table properties ('numRows'='172000');
``` 
17. Run the following SQL statement and make sure that your table is reading correctly:
```sql
	SELECT * 
	FROM labs.sales LIMIT 100
```
Congratulations, you queried your first S3 file through Amazon Spectrum!

You can view the details of the queries executed by Spectrum by querying the SVL_S3QUERY system view.
```sql
	select query, segment, slice, elapsed, s3_scanned_rows, s3_scanned_bytes, s3query_returned_rows, s3query_returned_bytes, files 
	from svl_s3query 
	where query = pg_last_query_id() 
	order by query,segment,slice; 
```
One of the common use cases for spectrum is to keep the larger fact tables in Amazon S3 and your smaller dimension tables in Amazon Redshift and then perform a join across the data sets in S3 and in Redshift Cluster. The steps listed
below depicts the use case -

18. Create an EVENT table (dimension table) in Redshift. Note that this table is internal to Redshift. 
```sql
	create table event(
	eventid integer not null distkey,
	venueid smallint not null,
	catid smallint not null,
	dateid smallint not null sortkey,
	eventname varchar(200),
	starttime timestamp); 
```
19. Load the EVENT table in Redshift using the COPY command with the data being pulled from Events folder in B2B. Replace the IAM role ARN in the following COPY command with the role ARN you created in step 2.
```sql
	copy event from 's3://yourbucket/B2B/Events/allevents_pipe.txt' 
	iam_role 'yourarn/mySpectrumRole'
	delimiter '|' timeformat 'YYYY-MM-DD HH:MI:SS' region 'us-east-1';
```
20. The following query joins the external table SPECTRUM.SALES with the local table EVENT to find the total sales for the top ten events
```sql
	select top 10 labs.sales.eventid, sum(labs.sales.pricepaid) from labs.sales, event
	where labs.sales.eventid = event.eventid
	and labs.sales.pricepaid > 30
	group by labs.sales.eventid
	order by 2 desc;
```
<hr/></br>

# Integrating Glue with Athena and Spectrum
One of the many benefits of Glue, is its ability to discover and profile data from S3 Objects. This become handy in quickly creating a catalog of new and incoming data.
To get started:
1. In Athena, from the **Database** pane on the left hand side, click **Create Table** drop down and select **Automatically**
<br />![alt text](/images/CreateAutomaticTable.png)<br/>
1. If this is your first time using Glue, you may be asked to upgrade your catalog, and get redirected. Make sure you are in the Crawlers section of Glue. On the left hand side there is a Crawlers link and hit *Add Crawler*
1. Enter name your crawler **"Taxi Crawler"** and select the IAM role "GlueServiceRole".  Click Next.
2. Select the **Specify path in another account** radio button and enter **s3://serverless-analytics/canonical/NY-Pub/** for the S3 path.  Click Next.
3. Do **not** add another data source and click Next.
4. For frequency leave as **Run on Demand** and click Next.
5. Select our **labs** database as a target
6. In order to avoid table name collision Glue generates a unique table name so we'll need to provide a prefix, say **taxi_** (include the underscore)
7. Click **Next**
8. Review the information is correct, specifically the "Include Path" field. Hit **Finish** when complete.
8. Check the box next to your newly created crawler and click **Run Crawler**.  It should take about a minute to run and create our table.

## Exploring Glue Data Catalog

1. On the left hand side, click **Databases**
2. Find the **labs** database and click on it
3. Click **Tables in labs** to view our newly created table
![alt text](http://amazonathenahandson.s3-website-us-east-1.amazonaws.com/images/show_taxi_table.png) <br />
1. Click the table name and explore

## Querying Taxi Data using Athena

1. Switch back to the Athena console
  - You may need to replace the database and/or table names with ones shown in the Data Catalog.
2. Enter `SHOW PARTITIONS labs.taxi_ny_pub` to verify all partitions were automatically added
3. Try the SQL statement below to explore the data.

```sql
SELECT *
FROM labs.taxi_ny_pub
WHERE year BETWEEN '2013' AND '2016' AND type='yellow'
ORDER BY pickup_datetime desc
LIMIT 10;
```
<br/>![alt text](http://amazonathenahandson.s3-website-us-east-1.amazonaws.com/images/taxis_2013_2016.png) <br/>

```sql
SELECT 
  type,
  year, 
  count(*) fare_count, 
  avg(fare_amount) avg_fare, 
  lag(avg(fare_amount)) over (partition by type order by year asc) last_year_avg_fare
FROM labs.taxi_ny_pub
WHERE year is not null
GROUP BY year, type
ORDER BY year DESC, type DESC
```
- Remember, you have the ability to **Save a query** for future re-use and reference.

## Querying Taxi Data using Spectrum
1. Switch to the SQL Editor using SQL Workbench Client
2. As the the Glue Crawler (Taxi Crawler) has already created the taxi_ny_pub in the labs database, we can start querying these tables using the spectrum service.
3. Try the following SQL statements. Each of the queries would take around 10 mins to complete on a single node redshift configuration.
 ```sql
SELECT *
FROM labs.taxi_ny_pub
WHERE year BETWEEN '2013' AND '2016' AND type='yellow'
ORDER BY pickup_datetime desc
LIMIT 10;
```
You get the following results from the Spectrum query
![alt text](/images/taxi_ny_1.PNG) </br>
```sql
SELECT 
  	type,
  	year, 
  	count(*) fare_count, 
  	avg(fare_amount) avg_fare, 
  	lag(avg(fare_amount)) over (partition by type order by year asc) last_year_avg_fare
FROM labs.taxi_ny_pub
WHERE year is not null
GROUP BY year, type
ORDER BY year DESC, type DESC
```
You get the following results from the Spectrum query</br>
![alt text](/images/taxi_ny_2.PNG)

You can also view the details of the last query executed by Spectrum by querying the SVL_S3QUERY system view.
```sql
select query, segment, slice, elapsed, s3_scanned_rows, s3_scanned_bytes, s3query_returned_rows, s3query_returned_bytes, files 
from svl_s3query 
where query = pg_last_query_id() 
order by query,segment,slice; 
```
<hr/></br>

# Breakout Exercises

## Breakout 1 - Load B2B Dataset

Now that we have learned about crawlers, lets put it to use to load the rest of our [B2B Orders](https://slalom-seattle-ima.s3-us-west-2.amazonaws.com/docs/B2B%20Dataset.zip) dataset.

- Unzip the data, and upload it to your S3 Bucket **remember, one folder represents one table.**
- Run a crawler through your bucket to discovery the dataset.
- Add new tables to the **labs** database with prefix "**b2b_**"

Make sure to check fields, and how Glue is parsing your data. Correct any mistakes. Once complete, you should be able to run this query: 
```sql
SELECT
  year(date_parse(Order_Date,'%c/%e/%Y')) Order_Year,
  Company_Name,
  SUM(quantity) Quantity,
  SUM(sales) Total_Sales,
  SUM(sales)/revenue_billion Sales_to_Revenue_Ratio
FROM labs.b2b_orders o
  JOIN labs.b2b_company co on  co.company_id = o.company_id
  JOIN labs.b2b_customer cu on cu.customer_id = o.customer_id
  JOIN labs.b2b_product p on p.product_id = o.product_id
  JOIN labs.b2b_segment s on s.segment_id = o.segment_id
  JOIN labs.b2b_ship_mode sm on sm.ship_mode_id = o.ship_mode_id
  JOIN labs.b2b_company_financials cp on cp.company_id = co.company_id
  JOIN labs.b2b_industry i on i.industry_id = co.industry_id
GROUP BY
  year(date_parse(Order_Date,'%c/%e/%Y')),
  Company_Name,
  revenue_billion
ORDER BY
  SUM(sales)/revenue_billion DESC
LIMIT 100
```
<br/>![alt text](/images/TopCustomersResults.PNG)<br/>

## Breakout 2 - Discover Instacart Data
In this section, we will break out and follow the same instructions, but while loading data from another public source, Instacart. Instacart is a company that operates as a same-day grocery delivery service. Customers select groceries through a web application from various retailers and delivered by a personal shopper. 
Instacart has published a public datasource to provide insight into consumer shopping trends for over 200,000 users. Data [Instacart in May 2017](https://tech.instacart.com/3-million-instacart-orders-open-sourced-d40d29ead6f2) to look at Instacart's customers' shopping pattern.  You can find the data dictionary for the data set [here](https://gist.github.com/jeremystan/c3b39d947d9b88b3ccff3147dbcf6c6b)

- Source s3 bucket: **s3://royon-customer-public/instacart/**
- Database: **labs**
- Prefix: **instacart_**
### Expected output
![alt text](/images/instacartResults.PNG "Expected Results")


## Notes on best practices
- Partition your data
- Compress your data!
- With large datasets, split your files into ~100MB files
- Convert data to a columnar format, with large datasets. 

For more great tips view [this post](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/) on AWS Big Data blog.
<hr/></br>

# Visualizing and Dashboarding with QuickSight

## Exercise 1

### 1. Setting up your QuickSight Account

Go to your AWS console and search for QuickSight.  You will first be presented with a screen to sign up:
 <br />![alt text](/images/signup.png)<br/>

You can choose either Standard or Enterprise Edition (the main difference today is that Enterprise edition can hook up to Active Directory, though there will be more functionality in the future added to Enterprise Edition).  For purposes of our lab today Standard Edition is fine.  With both editions you get one free user, forever.
 <br />![alt text](/images/editions.png)<br/>

Next you will create and name for you account (you can name the account whatever you'd like) and a notification email address (set it to be your own email address). You will also see some prompts about enabling autodiscovery of your data in AWS sources, as well as access to Athena, S3 buckets, and S3 Storage analytics.  Check all the boxes. 
<br />![alt text](/images/signup_options.png)<br/>

**Note - Make sure you launch QuickSight in the same region you have chosen for Athena.**
 <br />![alt text](/images/signup_region.png)<br/><br/>

Once you are finished your account will load with some sample datasets and dashboards.  
 <br />![alt text](/images/singup_complete.png)<br/><br/>

Alright, now we are ready to roll!

Here is some documentation on getting familiar with the UI:  [Navigating the UI](http://docs.aws.amazon.com/quicksight/latest/user/navigating-the-quicksight-ui.html)

### 2. Connecting to the Data

Documentation:  [Data Preparation](http://docs.aws.amazon.com/quicksight/latest/user/example-prepared-data-set.html), [Table Joins](http://docs.aws.amazon.com/quicksight/latest/user/joining-tables.html)

Open QuickSight and **choose 'Manage Data'** in the upper right hand corner:
 <br />![alt text](/images/manage_data.png)<br/>

**Choose 'New Dataset'** and then select **Athena**.
 <br />![alt text](/images/new_dataset.png)<br/>
 <br />![alt text](/images/athena.png)<br/>

Give it a name and **choose 'Create Data Source'**. Find the database you created earlier which contains the B2B tables and select the b2b_orders table. Try to make sure you are choosing the orders table that was created automatically by Glue instead of the table that we created using the SQL statement (if you happen to pick the wrong one, no problem, you just won't need to do the step where we create a calculated field to change the order_date to a date field).  **Choose 'Edit/Preview Data'**.  (If you clicked 'Select' instead, it's OK, just choose 'Edit/Preview Data' on the next screen and leave it on 'Import to SPICE for quicker analytics'.)
 <br />![alt text](/images/athena_tables.png)<br/><br/>

Now we will join all the tables we had created in Athena by using the Glue data crawler. Some tables join directly to the Orders table and some join to the Company table. To join a table to something other than the first one we selected (Orders) drag and drop it on top of the table you want to join it to.  You will then need to define the join clauses - to do this, click on the little venn diagrams in-between the tables you see on the screen.  They will all be based on the key which is named after the dimension table you are trying to join.  Set them all to 'Inner' joins and click 'Apply' after you finish each table.
![alt text](/images/join1.png)
![alt text](/images/join2.png)

When you are finished it should look something like this (we will skip the Segment and Product tables as the crawler didn't pick up the headers correctly - we can correct this using a Glue ETL job, but for purposes of this lab we can just leave these two tables out of our new dataset):

![alt text](/images/b2b%20joins.png)

Before we start visualizing, let's also add a couple calculated fields to convert the date fields, order_date and ship_date to date fields rather than strings (normally we could just change the datatype in QuickSight in the data preview window, but Athena does not support this today.  It will be supported soon, and you could do this for any other type of data source, but for Athena we will need to make calculated fields).  On the left side choose 'New Field' and then use the parseDate() function to convert the string field to a date field.  Use these formulas for each calculated field:
```python
parseDate({order_date},'MM/dd/yyyy')
parseDate({ship_date},'MM/dd/yyyy')
```
 <br />![alt text](/images/calculated_dates.png)<br/><br/>
 
Once you are finished preparing the dataset, **choose Save & Visualize** on the top of your screen.
 <br />![alt text](/images/save_visualize.png)<br/><br/>
 
### 3. Creating Our Dashboard

Documentation:  [Creating Your First Analysis](http://docs.aws.amazon.com/quicksight/latest/user/example-create-an-analysis.html), [Modifying Visuals](http://docs.aws.amazon.com/quicksight/latest/user/example-modify-visuals.html)

Great, now we are ready to begin visualizing our data.  By default AutoGraph is chosen as the visual type, which will pick an appropriate visual type depending on the types of fields choose to visualize.  We can leave it like that for now, and later we will specify certain visual types.

First click on 'sales' and we will get a KPI visual type.  Then click on the Field Wells on the top and use the pull down menu to choose 'Show As->Currency':
<br />![alt text](/images/format_sales.png)<br/><br/>

Now click on the 'Order Date' field.  Notice how our visual type automatically is changed to a line chart.  It will default to the Year level, but use the pull down menu on the Order Date field to choose 'Aggregate->Month', or you can do the same thing by clicking on the Order Date label on the x-axis:
<br />![alt text](/images/month.png)<br/><br/>

Next click the pull down menu on the segment field in the list of measures and choose 'Convert to dimension'.  Then find it in the list of dimensions and select it.  Now we will have 3 lines in our line chart, one per segment.  Expand the axis range on the bottom of the visual to see the whole trend:
<br />![alt text](/images/convert_segment.png)<br/><br/>
<br />![alt text](/images/line_chart.png)<br/><br/>

Great, we have our first visual!  Now let's add another visual using the '+' button in the upper left and selecting 'Add visual':
<br />![alt text](/images/add_visual.png)<br/><br/>

For our next visual, let's start by clicking 'industry_name' and 'sales'.  We will get a bar chart sorted in descending order by sales:
<br />![alt text](/images/industry_chart.png)<br/><br/>

Let's add a drill down capability for our end users by dragging the 'company_name' field just below the 'industry_name' field on the  Y axis field well.  You should see a notification that says 'Add drill-down layer':
<br />![alt text](/images/add_drilldown.png)<br/><br/>

Cool, now our end users will be able to drill down from Industry to the actual Companies in that industry.  You can see how this works by clicking on one of the bars and selecting 'Drill down to company_name':
<br />![alt text](/images/drilldown_on_bar.png)<br/><br/>

If you want to drill back up, you can either click the bars again or you can use the icons in the upper right to either drill one level back up or all the way back to the top (if you have more than one drill down built in):
<br />![alt text](/images/drill_up.png)<br/><br/>

Next let's change the visual type to a Treemap using the Visual Types selector in the bottom left:
<br />![alt text](/images/treemap.png)<br/><br/>

Now add another visual to the dashboard.  This one will be a very granular table of all the order details.  First select the 'Pivot Table' visual type.  Then click on the company_name dimension.  Expand the Field Wells on top and drag the order_id to the Rows underneath the company_name:
<br />![alt text](/images/add_pivot_field.png)<br/><br/>

Also click on the product_id, ship_mode, sales, profit, and quantity fields to add more detail to our visual.  It should look something like this:
<br />![alt text](/images/table.png)<br/><br/>

Great, our dashboard is starting to shape up.  We can now add some KPI visuals across the top to provide some high-level summary information for our users.  Add another visual and select the 'sales' field.  Expand the Field Wells and drag the Order Date to the 'Trend group' field well.  Let's also resize the visual by dragging the bottom right corner of the visual to make it smaller.  Drag it to the top of the dashboard by grabbing the dotted area on the top of the visual.  Once you have it on the top it should look like this:
<br />![alt text](/images/sales_kpi.png)<br/><br/>

Let's repeat this last step to add two more KPI's to the top of the dashboard.  After you add another visual, select the KPI visual type in the lower left of the screen:
<br />![alt text](/images/kpi_visual_type.png)<br/><br/>

The second one will be a KPI for the number of unique orders YoY.  To do this, select the KPI visual type and drag 'order_id' to the 'Value' field well and 'Order Date' to the 'Trend group' field well.  Change the aggregation on 'order_id' from Count to Count Distinct:
<br />![alt text](/images/orders_kpi.png)<br/><br/>

For the third KPI, let's show a YoY trend of the average order size.  Click 'sales' and then use the pull down menu on the field to change the aggregation to Average.  Add the 'Order Date' to the 'Trend group' field well like we did for the first KPI:
<br />![alt text](/images/avg_sales.png)<br/><br/>

You can optionally play around with the KPI formatting options.  You can change the primary number that is displayed and the comparison type.  You can also choose if you would like to show the trend arrows as well as the progress bar (which is displayed as a bullet chart on the bottom of the KPI).
<br />![alt text](/images/kpi_formatting.png)<br/><br/>

Lastly let's edit the titles of the KPI's to be more user friendly.  I chose 'Sales YoY', 'Avg Order Size YoY', and '# of Orders YoY' for my titles:
<br />![alt text](/images/kpi_complete.png)<br/><br/>

Awesome!  Our dashboard is looking really good.  We are almost ready to share it with the rest of our end users.  Just before we do that, let's add a filter (or many) for our users to leverage.  On the left, choose 'Filter' and then either click 'Create one' or the little filter icon on the top and choose 'Order Date'.  I like to use the 'Relative dates' type of UI for my date filters.  Set it to the 'Last 5 years' and hit 'Apply'.  Lastly click on the top where it says 'Only this visual' and change it to 'All visuals' so that it applies to the entire dashboard:
<br />![alt text](/images/date_filter.png)<br/><br/>
<br />![alt text](/images/filter_all_visuals.png)<br/><br/>

### 4. Sharing

Documentation:  [Creating and Sharing Your First Dashboard](http://docs.aws.amazon.com/quicksight/latest/user/creating-a-dashboard.html)

We are ready to share our dashboard with the rest of our users now!  Click the 'Share' button in the upper right of the screen and select 'Create Dashboard'. Give it a name like 'Sales Dashboard' and choose 'Create Dashboard'.  
<br />![alt text](/images/create_dash.png)<br/><br/>

On the next screen you will be able to share it with other users in your QuickSight account.  
<br />![alt text](/images/share.png)<br/><br/>

Once you add them you can click 'Share' and it will send them an email saying a dashboard has been shared with them.  Also the next time they log into QuickSight they will see it in the list of dashboards they have access to.

Great job!  You have just created your first dashboard to be shared with the rest of your team!

<br />![alt text](/images/dash.png)<br/><br/>


## Exercise 2 - Visualizing NY Taxi Data

One of the most compelling reasons for using Athena to query data on S3 is that you can query some really really BIG datasets.  In our next exercise we will use QuickSight and Athena to visualize 2.7 Billion records.  That's right, billion.

### 1. Connect to the Dataset

Open QuickSight and **choose 'Manage Data'** in the upper right hand corner.

**Choose 'New Dataset'** and then select **Athena**.

Give it a name and **choose 'Create Data Source'**. Find the database you created earlier which contains the NY taxi data and select the appropriate table.  **Choose 'Edit/Preview Data'**.

Before we start visualizing, let's add a calculated field to convert the date field.  The date field in this dataset is in Epoch date format.  Therefore we will use a function to convert it to a more usable format.  On the left side choose 'New Field' and then use the epochDate() function to convert pickup_datetime field to a date field.  It is measured down to the millisecond, so we will also divide the integer by 1000 to get it into seconds before converting.  Use this formula:
```python
epochDate({pickup_datetime}/1000)
```
![alt text](/images/epoch.png)<br/><br/>

Make sure we keep it set to 'Query' rather than SPICE, which is different from what we did in the first exercise (actually when doing table joins QuickSight forces you to use SPICE, but when connecting to individual tables we get this choice).  Since we are going to be working with nearly 3 billion records, we will want to query the data directly in S3 using Athena.
<br />![alt text](/images/query.png)<br/><br/>

### 2. Creating Our Dashboard

Great, now we are ready to begin visualizing our data.  By default AutoGraph is chosen as the visual type, which will pick an appropriate visual type depending on the types of fields choose to visualize.  We can leave it like that for now, and later we will specify certain visual types.

Select 'passenger_count' and then use the pull down menu to change the aggregation to Count.  Then use the pull down menu again and choose 'Format->1234.57' to round to two decimal places.  The KPI will show that we have 2.67 billion records in the dataset.  Pretty impressive performance on a dataset of that size!
<br />![alt text](/images/count.png)<br/>
<br />![alt text](/images/count2.png)<br/><br/>

Let's add another visual.  This time select 'Pickup Date' (the calculated field you created).  You should get a line chart.  Use the pull down menu and change the aggregation to Week.  Then expand the axis range on the bottom of the visual.
<br />![alt text](/images/lines_taxi.png)<br/><br/>

Select the 'type' field and you should get three lines, one for each type of taxi:
<br />![alt text](/images/type_lines.png)<br/><br/>

Let's add another visual.  This one will also be a time trend but we will look at the data YoY.  First change the visual type to a Line Chart.  Then drag the 'month' field to the X axis field well and the 'year' field to the Color field well.

Notice the months on the bottom are out of order.  Since the field is a string data type the months are sorted in alphabetical order.  To fix this we must edit the dataset and change the data types for these columns.  Use the dropdown menu for the name of your dataset and choose 'Edit analysis data sets' and then click 'Edit' on the next screen:
<br />![alt text](/images/edit_dataset.png)<br/><br/>
<br />![alt text](/images/edit_dataset2.png)<br/><br/>

Click on the 'a' icon underneath both of these fields in the data preview window and change them both to 'Int':
<br />![alt text](/images/int.png)<br/><br/>

Choose 'Save & visualize'.  Now the months on our line chart should be sorted in the correct order:
<br />![alt text](/images/months_correct.png)<br/><br/>

One of the first things you will notice is that there is a huge drop in Feb on the 2010 line.  A quick google search for 'nyc feb 2010' will reveal that there was a huge blizzard in Feb 2010!  Makes sense why there were less rides for that month.

Feel free to continue exploring this data. There aren't a ton more dimensions to play with - the dataset was meant to highlight the scale of how many records Athena + S3 can handle rather than analytical depth - but go wild with it!

### 3. (Optional) Create A Story

[How to createa a Story](http://docs.aws.amazon.com/quicksight/latest/user/working-with-stories.html)

In addiition to creating and sharing dashboards, you can also create and share 'stories'. They are great if you have found something interesting in the data and you would like to lead you users to that particular finding.  For instance, using our last finding you could capture a 'scene' of the YoY trend visual, then filter to 2010 and capture another 'scene' to highlight the drop in Feb 2010 due to the blizzard.

### Conclusion

Congratulations on creating your first Glue Crawlers, Athena Databases & Tables, and QuickSight Analyses and Dashboards!  You are now well versed in Serverless Analytics!

For more tips and information about what's new in QuickSight, check out the [blog](https://quicksight.aws/resources/blog/) as well as the other [resources](https://quicksight.aws/resources/) on the website!

### The End
"# QuicksightAthenaSpectrum" 
