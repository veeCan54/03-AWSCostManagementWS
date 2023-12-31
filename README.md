# 01-AWSCostManagementWS

**First in a series of AWS hands on workshops.**

The list is available here - https://catalog.workshops.aws/general-immersionday/en-US. <br>
The first module, **Cost - Monitoring and Observability**, has exercises that help understand mechanisms to monitor cost, create reports and organize information using tags when tracking spending across various AWS services. The second section has to do with ETL and creating visualisations using a dataset. I completed selected tasks in Section 1 that I needed to refresh my memory on. Section 2 was a really good learning experience. Let's go over each section. 

**Section 1:**

1. Cost Allocation Tags. 
Cost Allocation tags are basically labels. A tag has a key and a value, tags allow us to track cost based on user defined or AWS defined fields. As an example, a user defined field could be Name which indicates Resource name. Another tag could be Environment, based on whether it is DEV, TEST or PROD. Examples of AWS defined tags are aws:createdBy, aws:cloudformation:stack-name etc. After Cost allocation tags are enabled, resources need to be properly tagged so that they can be organized by tags in the Cost and Usage Reports. I explored enabling them via the admin console. However since my compute spending was well within the free tier, my Cost & Usage reports did not have a whole lot of information. 

2. Cost Anomaly Detection. 
This is another feature I explored. AWS Cost Anomaly Detection uses machine learning models to detect and alert on anomalous spend patterns in your deployed AWS services. Using this feature has the following benefits:
    1. You can investigate the root cause of an anomaly.
    2. You can configure how to evaluate your costs. Do you want to analyze a specific member account only? Do you want to analyze AWS services independently?
    3. You can receive alerts in aggregate reports.

    In my opinion it would be a good idea to setup anomaly monitors early on during account creation and budget allocation.
    This is because it can take upto 24 hours to detect an anomaly after usage occurs, because Cost Anomaly Detection uses data from Cost Explorer, which has a delay of up to 24 hours. If you create a new monitor, it can take 24 hours to begin detecting new anomalies. For a new service subscription, 10 days of historical service usage data is needed before anomalies can be detected for that service. This way when there is an anomaly we receive the alert within the expected amount of time so we can take necessary steps. 

3. Cost Optimization - Rightsizing Recommendations.
This feature reviews historical usage to identify opportunities for greater cost and usage efficiency. By default, 14 days of usage is considered. My AWS account being in the free tier, I did not have any data for AWS to analyze my spending pattern. So there were no recommendations for me. 

**Section 2:**

**Goal: Create Cost Intelligence Dashboards** <br>
**Services used: AWS Glue, AWS Athena, AWS QuickSight**

Here we will create a Cost & Usage report. Since there is no real data, we will use fabricated data made available as part of the exercise, [here](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/12_mths_costso-training.parquet). This will be uploaded to an S3 bucket. 

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/01-reportCreated.png)

We will use AWS Glue which is a serverless ETL tool. We will create a db schema and load data in to a database table. We will then use Amazon Athena to query the data in place and create views. These views will be used by Quicksight to create visualizations. These interactive dashboards will help the decision makers to analyze trends/performance metrics and other criteria, make sound business decisions. 

**AWS Glue** is a serverless Data integration service. With this service we can discover, prepare, move and integrate data from multiple sources.
With AWS Glue, you can discover and connect to more than 70 datasources and manage data in a centralized **Data Catalog**. You can visually create, monitor and run ETL pipelines to load data into your data lakes. This catalogued data can be searched and queried using Amazon Athena, EMR and Redshift Spectrum. 

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/001-Glue.png)

Key terminologies and how Glue works: 

1. Job is a unit of work. Jobs are defined in Glue, to accomplish ETL work from a source to a target.
2. Table definitions are stored in a Glue Data Catalog. 
3. Crawlers populates the Data Catalog. Crawlers are programs that are pointed at datasources.
4. Crawlers do this using Classifiers.
5. Classifiers are what determine the schema of our data. Classifiers for common file types, CSV, JSON, AVRO, XML etc are provided by AWS Glue.
6. Crawlers process classifiers to determine the schema of your data and create table definitions in the Data Catalog.
7. Data is transformed using a script. AWS Glue can generate a script for you. Or you can provide the script in the AWS Glue console or API.
8. The job can be run on demand or schedule it.
9. When the job runs, the sript extracts data from data source, transforms data and loads into the target. 

Let's proceed with the ETL step.

Created a Crawler with a default classifier. Pointed to the S3 file location that has the uploaded (fabricated) dataset.  Ran the Crawler.

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/03-crawlerRunComplete.png)

After the Crawler finished running, in AWS Glue we can see the schema that was created.

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/04-dataBase.png)

We will now integrate this schema with Amazon Athena so we can write SQL queries on the S3 data. 

**Amazon Athena** is a serverless, interactive analytics service. It helps analyze data where it lives in other words there is no need to move/copy data anywhere.It is built on open-source Trino and Presto engines and Apache spark framework. Athena supports open-table and file formats.  
It does not need any provisioning or configuration effort. In this step we will create views in Athena leveraging the Glue Data Catalog schema. Then we will use these views as Data Source for generating interactive dsahboards using Amazon QuickSight.

>**Workgroups in Athena**: In Athena it is possible to separate users, teams, applications, or workloads, to set limits on amount of data for each query by configuring workgroups. This can also be very useful to track costs. Workgroups act as resources so we can use resource level identity based policies to control access to a specific workgroup. However, in this hands on we are only using the default primary workgroup. 

Using the queries provided in the instructions, created 6 views in Athena: 
[account_map](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/02-account_map.sql), [summary_view](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/02-summary_view.sql), [ec2_running_cost](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/02-ec2_running_cost.sql), [ri_sp_mapping](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/02-ri_sp_mapping.sql),[s3_view](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/02-s3_view.sql), [compute_savingsplan_eligible_spend](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/02-compute_savingsplan_eligible_spend.sql)

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/05-athenaViewsCreated.png)

Now moving on to QuickSight to build Visualizations from these views.

**Amazon QuickSight** is a service that can be used to can access data and prepare it for use in reporting. QuickSight gives decision makers the opportunity to explore and interpret information in an interactive visual environment. They have secure access to dashboards from any device on the network and from mobile devices. Quicksight saves your prepared data either in SPICE memory or as a direct query. **SPICE** (Super-fast, Parallel, In-memory Calculation Engine) is the robust in-memory engine that QuickSight uses. **SPICE** is engineered to rapidly perform advanced calculations and serve data. 

Since this is my first time using Quicksight I had to setup a username and link it with my AWS account. After signing up for a QuickSight username I pointed it to the datasource. 

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/06-quickSightConfigured.png)

Now we will be deploying the dashboards. For this we go to CloudShell. 
**CloudShell** is pre-authenticated with our console credentials. CloudShell makes it easy to securely manage, explore, and interact with your AWS resources. CloudShell comes with 1 GB storage free per AWS region and with AWS CLI, Python, Node.js to name a few tools. Files saved in your home directory are available in future sessions for the same AWS region. 

Uploaded the [ZIP file](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/files/STAMCostOpsSOW.zip), to Cloudshell home directory. 

After that ran the commands to unzip the file, upgrade packages followed by deployment of cid-cmd, a command line tool for managing various dashboards provided in AWS Well Architected LAB Cloud Intelligence Dashboards.

```sh
unzip STAMCostOpsSOW.zip

cd STAMCostOpsSOW

pip3 install -U .

cid-cmd deploy --recursive yes  -vvv
```
Deployed Cost Intelligence Dashboard

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/07-cashDashboard.png)

Followed by CUDOS 

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/06-cudosDeployed.png)

After deployment, the dashboards were available in QuickSight.

I have to say these interactive dashboards are extensive and very detailed. They provide several aggregations and the capability to look at various AWS services, linked accounts, tags, to name a few. 

Some screenshots: 

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/10-db01.png)

![Alt text](https://github.com/veeCan54/01-AWSCostManagementWS/blob/main/images/10-db02.png)

Now that the exercise is complete, make sure to clean up the resources!

# Summary <br>
**What did I learn?** <br>
**Amazon Glue**, it's architecture, terminologies and it's capabilities. <br>
**Amazon QuickSight** and it's functionalities out of the box. Pretty amazing. The robust in-memory **SPICE** engine. <br>
**Athena WorkGroups** and how they can be used for fine grained access control. <br>
**Mistakes?** <br>
I made a mistake while selecting the destination table name in AWS Glue which resulted in the table name and my bucket name being the same. The table name should have been ```customer_all```. I didn't realize this until I ran the queries in Athena to create the views. Thankfully this was easy to fix by changing the table name in the queries. <br>
**TODO?** <br>
Explore Cloud Intelligence Dashboards (CUDOS Framework).








