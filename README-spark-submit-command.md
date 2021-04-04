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
#### 2.1 Deployment Modes (–deploy-mode)
Using ```--deploy-mode```, you specify where to run the Spark application driver program. Spark support cluster and client deployment modes.

| VALUE        | DESCRIPTION |
| -------------|:-----------:|
| cluster      |In cluster mode, the driver runs on one of the worker nodes, and this node shows as a driver on the Spark Web UI of your application. cluster mode is used to run production jobs.|
| client       | In client mode, the driver runs locally where you are submitting your application from. client mode is majorly used for interactive and debugging purposes. Note that in client mode only the driver runs locally and all other executors run on different nodes on the cluster.|        

#### 2.2 Cluster Managers (–master)

Using ```--master option```, you specify what cluster manager to use to run your application. Spark currently supports Yarn, Mesos, Kubernetes, Stand-alone, and local. The uses of these are explained below.

| CLUSTER MANAGER |  VALUE   | DESCRIPTION |
| ----------------|:--------:|:-----------:|
| Yarn            | yarn|Use yarn if your cluster resources are managed by Hadoop Yarn.|
| Mesos           | mesos://HOST:PORT|use mesos://HOST:PORT for Mesos cluster manager, replace the host and port of Mesos cluster manager.|
| Standalone      | spark://HOST:PORT|Use spark://HOST:PORT for Standalone cluster, replace the host and port of stand-alone cluster.|
| Kubernetes      | k8s://HOST:PORT k8s://https://HOST:PORT| Use k8s://HOST:PORT for Kubernetes, replace the host and port of Kubernetes. This by default connects with https, but if you wanted to use unsecured use k8s://https://HOST:PORT|                    
| local           |local local[k] local[K,F] | Use local to run locally with a one worker thread. Use local[k] and specify k with the number of cores you have locally, this runs application with k worker threads. Use local[k,F] and specify F with number of attempts it should run when failed.|
                    
**Example**: Below submits applications to yarn managed cluster.

While submitting an application, you can also specify how much memory and cores you wanted to give for driver and executors.
```
./bin/spark-submit \
    --deploy-mode cluster \
    --master yarn \
    --class org.apache.spark.examples.SparkPi \
    /spark-home/examples/jars/spark-examples_versionxx.jar 80
```

#### 2.3 Driver and Executor Resources (Cores & Memory)

While submitting an application, you can also specify how much memory and cores you wanted to give for driver and executors.

| OPTION | DESCRIPTION |
| ------:|:-----------:|
| –driver-memory|Memory to be used by the Spark driver.|
| –driver-cores|CPU cores to be used by the Spark driver|
| –num-executors|The total number of executors to use.|            
| –executor-memory|Amount of memory to use for the executor process.|             
| –executor-cores|Number of CPU cores to use for the executor process.|             
| –total-executor-cores|The total number of executor cores to use.|

**Example:**
```
./bin/spark2-submit \
   --master yarn \
   --deploy-mode cluster \
   --driver-memory 8g \
   --executor-memory 16g \
   --executor-cores 2  \
   --class org.apache.spark.examples.SparkPi \
   /spark-home/examples/jars/spark-examples_versionxx.jar 80```
   
   
