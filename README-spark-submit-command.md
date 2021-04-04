# Spark Submit Command Explained with Examples

The **spark-submit** command is a utility to run or submit a Spark or PySpark application program (or job) to the cluster by specifying options and configurations, the application you are submitting can be written in Scala, Java, or Python (PySpark). spark-submit command supports the following.

1. Submitting Spark application on different cluster managers like [Yarn](https://sparkbyexamples.com/spark/spark-setup-on-hadoop-yarn/), Kubernetes, Mesos, and Stand-alone.
2. Submitting Spark application on **client** or **cluster** deployment modes.

In this page, We will explain different spark-submit command options and configurations along with how to use a uber jar or zip file for Scala and Java, using Python .py file, and finally how to submit the application on Yarn. Mesos, Kubernetes, and standalone cluster managers.

## Table of contents
- Spark Submit Command
- Spark Submit Options
  - Deployment Modes
  - Cluster Managers
  - Driver & Executors resources
- Spark Submit Configurations
- Spark Submit PySpark (Python) Application
- Submitting Application to Mesos
- Submitting Application to Kubernetes
- Submitting Application to Standalone
