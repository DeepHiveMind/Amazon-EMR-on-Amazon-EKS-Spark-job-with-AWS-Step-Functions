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

## 1. Spark Submit Command

Spark binary comes with ```spark-submit.sh``` script file for Linux, Mac, and ```spark-submit.cmd```command file for windows, these scripts are available at ```$SPARK_HOME/bin``` directory.

If you are using [Cloudera distribution](https://www.cloudera.com/downloads/cdh.html), you may also find ```spark2-submit.sh``` which is used to run Spark 2.x applications. By adding this Cloudera supports both Spark 1.x and Spark 2.x applications to run in parallel.

spark-submit command internally uses ```org.apache.spark.deploy.SparkSubmit``` class with the options and command line arguments you specify.
Below is a spark-submit command with the most-used command options.
```
./bin/spark-submit \
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key<=<value> \
  --driver-memory <value>g \
  --executor-memory <value>g \
  --executor-cores <number of cores>  \
  --jars  <comma separated dependencies>
  --class <main-class> \
  <application-jar> \
  [application-arguments]
 ```
 You can also submit the application like below without using the script.
 
 ```
 ./bin/spark-class org.apache.spark.deploy.SparkSubmit <options & arguments>
 ```
 ## 2. Spark Submit Options
 
 Below I have explained some of the common options, configurations, and specific options to use with Scala and Python. You can also get all options available by running the below command.

```./bin/spark-submit --help```
###### 2.1 Deployment Modes (–deploy-mode)
Using ```--deploy-mode```, you specify where to run the Spark application driver program. Spark support cluster and client deployment modes.

