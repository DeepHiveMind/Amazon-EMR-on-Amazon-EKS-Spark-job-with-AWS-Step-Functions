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

