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
