# Orchestrate an Amazon EMR on Amazon EKS Spark job with AWS Step Functions

Re:Invent 2020 has announced the general availability of Amazon EMR on Amazon EKS, a new deployment option for Amazon EMR that allows you to automate the provisioning and management of open-source big data frameworks on Amazon Elastic Kubernetes Service (Amazon EKS). With Amazon EMR on EKS, you can now run Spark applications alongside other types of applications on the same EKS cluster to improve resource utilization and simplify infrastructure management. If you currently self-manage Apache Spark on Amazon EKS, you can use Amazon EMR to automate provisioning and take advantage of the Amazon EMR optimized runtime for Apache Spark to run your workloads up to three times faster.

AWS Step Functions now enables developers to assemble Amazon EMR on EKS into a serverless workflow in minutes. Step Functions ensures that the steps in the serverless workflow are followed reliably, that the information is passed between stages, and errors are handled automatically. Amazon States Language is the JSON-based language used to write declarative state machines to define durable and event-driven workflows in Step Functions.

In this page, we explain how you can orchestrate an Amazon EMR on EKS job with Step Functions. We also integrate Amazon EventBridge to invoke the Step Functions state machine.

## Agenda

- Use case overview
- Architecture overview
- Prerequisites
- Create a Step Functions state machine
- Create an EventBridge scheduling rule
- Validate the output with an Athena query
- Create reports in QuickSight
- Conclusion

## Use case overview

In this post, we step through an example using the [New York City Taxi Records](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page) dataset. For the Spark job, we use only one month of data, but you could easily do this for the entire dataset.
We use the ```yellow_tripdata_<date>``` and ```green_tripdata_<date>``` files, which we uploaded to our [Amazon Simple Storage Service](https://aws.amazon.com/s3/)(Amazon S3) bucket in the following structure. The code year=2020 represents the partition folder. As you upload your entire dataset, you can upload them into respective year or partition folders.
- ```s3://<bucket-name>/nyc-taxi-records/input/yellow-taxi/year=2020/```
- ```s3://<bucket-name>/nyc-taxi-records/input/green-taxi/year=2020/```

Our objective is to find the average distance and average cost per mile for both green and yellow taxis in 2020.
To implement the solution, we read the input data from the S3 input path, apply aggregations with PySpark code, and write the summarized output to the S3 output path ```s3://<bucket-name>/nyc-taxi-records/output/```.

When our Spark job is ready, we can deploy it through Step Functions, which does the following:
- Creates and registers the Amazon EMR on EKS virtual cluster with the ```createVirtualCluster``` API. This allows you to register your EKS cluster with Chicago by creating a virtual cluster with the specified name. This API step waits for cluster creation.
- Submits the PySpark job and waits for its completion with the ```startJobRun (.sync)```API. This allows you to submit a job to an Amazon EMR on EKS virtual cluster and waits until the job reaches a terminal state.
- Stops the cluster with the ```deleteVirtualCluster (.sync)``` API. This allows you to delete the Chicago virtual cluster registered with your EKS cluster. The connector waits until the virtual cluster deletion is complete.

When the summarized output is available in the S3 output directory, we can run an [Amazon Athena](https://aws.amazon.com/athena/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc) query to create a virtual table of the output data and integrate [Amazon QuickSight](https://aws.amazon.com/quicksight/) for reporting.

## Architecture overview
To automate the complete process, we use the following architecture, which integrates Step Functions for orchestration, Amazon EMR on EKS for data transformations, Athena for data analysis with standard SQL, QuickSight for business intelligence (BI) reporting, and EventBridge for scheduling the Step Functions workflow.

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/bdb1345-emr-eks-step-functions-1.jpg)

The architecture includes the following steps:
- **Step 1** – User uploads input CSV files to the defined S3 input bucket.
- **Step 2** – An EventBridge rule is scheduled to trigger the Step Functions state machine.
- **Steps 3, 4, and 5** – Step Functions submits a Spark job to the Amazon EMR on EKS cluster, which reads input data from S3 input bucket. After it applies transformations, it     writes the summarized output to the S3 output bucket.
- **Steps 6, 7.1, and 7.2** – After the Amazon EMR on EKS job, Step Functions invokes the Athena query, which creates a virtual table in the [AWS Glue](https://aws.amazon.com/glue/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc) Data Catalog on top of the S3 output data.
- **Steps 8 and 9** – After we create the virtual table created in the Data Catalog through Athena, data analysts can use Athena to query the data with standard SQL statements or can build QuickSight reports by connecting to Athena tables.

When we talk about Amazon EMR virtual cluster integration with Amazon EKS, the architecture looks like the following diagram, in which an EKS cluster can support multitenancy with different Spark versions and configurations backed by Amazon Elastic Compute Cloud (Amazon EC2) or AWS Fargate computing resources.

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/bdb1345-emr-eks-step-functions-2.jpg)

## Prerequisites

Before beginning this tutorial, make sure you have the required permissions to create the necessary resources as part of the solution. You should also have the EKS cluster available already.
For instructions on creating the EKS cluster and enabling access for Amazon EMR on EKS, see [Setting up](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up.html).
Additionally, create the S3 input and output buckets with the required subfolders to capture the input bucket and output buckets.

## Create a Step Functions state machine

To create a Step Functions state machine, complete the following steps:
1. On the Step Functions console, choose **Create state machine.**
2. For Define state machine, select **Author with code snippets.**
3. For **Type**, select **Standard.**

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/bdb1345-emr-eks-step-functions-3.jpg)

In the **Definition** section, Step Functions provides a list of service actions that you can use to automatically generate a code snippet for your state machine’s state. The following screenshot shows that we have options to create an EMR virtual cluster, submit a job to it, and delete the cluster.

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/bdb1345-emr-eks-step-functions-4.jpg)

4. For **Generate code snippet**, **choose Amazon EMR on EKS: Create a virtual cluster**.
5. For **Enter virtual cluster name**, enter a name for your cluster.
6. For **Enter Container provider**, enter the following code:
```
{
  "Id": "<existing-eks-cluster-id>",
  "Type": "EKS",
  "Info": {
    "EksInfo": {
      "Namespace": "<eks-cluster-namespace>"
    }
  }
}
```
The JSON snippet appears in the **Preview** pane.
6. Choose **Copy to clipboard**.

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/bdb1345-emr-eks-step-functions-5.jpg)

7. Follow the user interface to generate the state machine JSON for Start a job run and Delete a virtual cluster, then integrate them into the final state machine JSON that also has an Athena query.
The following code is the final Step Functions state machine JSON, which you can refer to. Make sure to replace the variables related to your EKS cluster ID, AWS Identity and Access Management (IAM) role name, S3 bucket paths, and other placeholders.
```
{
  "Comment": "An example of the Amazon States Language for creating a virtual cluster, running jobs and terminating the cluster using EMR on EKS",
  "StartAt": "Create EMR-EKS Virtual Cluster",
  "States": {
    "Create EMR-EKS Virtual Cluster": {
      "Type": "Task",
      "Resource": "arn:aws:states:::emr-containers:createVirtualCluster",
      "Parameters": {
        "Name": "<emr-virtual-cluster-name>",
        "ContainerProvider": {
          "Id": "<eks-cluster-id>",
          "Type": "EKS",
          "Info": {
            "EksInfo": {
              "Namespace": "<eks-namespace>"
            }
          }
        }
      },
      "ResultPath": "$.cluster",
      "Next": "Submit PySpark Job"
    },
    "Submit PySpark Job": {
      "Type": "Task",
      "Resource": "arn:aws:states:::emr-containers:startJobRun.sync",
      "Parameters": {
        "Name": "<pyspark-job-name>",
        "VirtualClusterId.$": "$.cluster.Id",
        "ExecutionRoleArn": "arn:aws:iam::<aws-account-id>:role/<role-name>",
        "ReleaseLabel": "emr-6.2.0-latest",
        "JobDriver": {
          "SparkSubmitJobDriver": {
            "EntryPoint": "s3://<bucket-name-path>/<script-name>.py",
            "EntryPointArguments": [
              "60"
            ],
            "SparkSubmitParameters": "--conf spark.driver.cores=1 --conf spark.executor.instances=1 --conf spark.kubernetes.pyspark.pythonVersion=3 --conf spark.executor.memory=1G --conf spark.driver.memory=1G --conf spark.executor.cores=1 --conf spark.dynamicAllocation.enabled=false"
          }
        },
        "ConfigurationOverrides": {
          "ApplicationConfiguration": [
            {
              "Classification": "spark-defaults",
              "Properties": {
                "spark.executor.instances": "1",
                "spark.executor.memory": "1G"
              }
            }
          ],
          "MonitoringConfiguration": {
            "PersistentAppUI": "ENABLED",
            "CloudWatchMonitoringConfiguration": {
              "LogGroupName": "<log-group-name>",
              "LogStreamNamePrefix": "<log-stream-prefix>"
            },
            "S3MonitoringConfiguration": {
              "LogUri": "s3://<log-bucket-path>"
            }
          }
        }
      },
      "ResultPath": "$.job",
      "Next": "Delete EMR-EKS Virtual Cluster"
    },
    "Delete EMR-EKS Virtual Cluster": {
      "Type": "Task",
      "Resource": "arn:aws:states:::emr-containers:deleteVirtualCluster.sync",
      "Parameters": {
        "Id.$": "$.job.VirtualClusterId"
      },
      "Next": "Create Athena Summarized Output Table"
    },
    "Create Athena Summarized Output Table": {
      "Type": "Task",
      "Resource": "arn:aws:states:::athena:startQueryExecution.sync",
      "Parameters": {
        "QueryString": "CREATE EXTERNAL TABLE IF NOT EXISTS default.nyc_taxi_avg_summary(`type` string, `avgDist` double, `avgCostPerMile` double, `avgCost` double) ROW FORMAT SERDE   'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' STORED AS INPUTFORMAT   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' OUTPUTFORMAT   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' LOCATION  's3://<output-bucket-path>/nyc-taxi-records/output/' TBLPROPERTIES ('classification'='parquet', 'compressionType'='none', 'typeOfData'='file')",
        "WorkGroup": "primary",
        "ResultConfiguration": {
           "OutputLocation": "s3://<athena-query-results-bucket-path>/"
        }
      },
      "End": true
    }
  }
}
```
The following diagram is the visual representation of the state machine flow.

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/bdb1345-emr-eks-step-functions-6.jpg)

The Step Functions definition shows that it invokes createVirtualCluster for Amazon EMR on EKS, it invokes StartJobRun with the PySpark script as the parameter, and triggers the deleteVirtualCluster step. You can embed the cluster creation and deletion step within the Step Functions or have the cluster created beforehand and just invoke the startJobRun within Step Functions. We recommend you create the Amazon EMR on EKS cluster once and keep it active for multiple job runs, because keeping it active in idle state doesn’t consume any resources or add anything to the overall cost.
The following code is the PySpark script available in Amazon S3, which reads the yellow taxi and green taxi datasets from Amazon S3 as Spark DataFrames, creates an aggregated summary output through SparkSQL transformations, and writes the final output to Amazon S3 in Parquet format:
```
import sys
import time
from pyspark.sql import SparkSession
from pyspark.sql.functions import * 
from pyspark.sql.types import *
 
spark =  SparkSession.builder.appName("nyc-taxi-trip-summary").getOrCreate()
 
YellowTaxiDF =  spark.read.option("header",True).csv("s3://<input-bucket-path>/nyc-taxi-records/input/yellow-taxi/")
GreenTaxiDF =  spark.read.option("header",True).csv("s3://<input-bucket-path>/nyc-taxi-records/input/green-taxi/")
 
YellowTaxiDF.registerTempTable("yellow_taxi")
GreenTaxiDF.registerTempTable("green_taxi")
 
TaxiSummaryDF = spark.sql("SELECT 'yellow' as type,  round(avg(trip_distance),2) AS avgDist,  round(avg(total_amount/trip_distance),2) AS avgCostPerMile,  round(avg(total_amount),2) avgCost from yellow_taxi WHERE trip_distance >  0 AND total_amount > 0 UNION SELECT 'green' as type, round(avg(trip_distance),2)  AS avgDist, round(avg(total_amount/trip_distance),2) AS avgCostPerMile,  round(avg(total_amount),2) avgCost from green_taxi WHERE trip_distance > 0  AND total_amount > 0")
 
TaxiSummaryDF.write.mode(SaveMode.Overwrite).parquet("s3://<output-bucket-path>/nyc-taxi-records/output/")
 
spark.stop()
```
When the PySpark job is complete, Step Functions invokes the following Athena query to create a virtual table on top of the output Parquet files:
```
CREATE EXTERNAL TABLE `nyc_taxi_avg_summary`(`type` string, `avgDist` double, `avgCostPerMile` double, `avgCost` double) ROW FORMAT SERDE   'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' STORED AS INPUTFORMAT   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' OUTPUTFORMAT   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' LOCATION  's3://<output-bucket-path>/nyc-taxi-records/output/' TBLPROPERTIES ('classification'='parquet', 'compressionType'='none', 'typeOfData'='file')
```
