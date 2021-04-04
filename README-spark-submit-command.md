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
| :-------------|:-----------|
| cluster      |In cluster mode, the driver runs on one of the worker nodes, and this node shows as a driver on the Spark Web UI of your application. cluster mode is used to run production jobs.|
| client       | In client mode, the driver runs locally where you are submitting your application from. client mode is majorly used for interactive and debugging purposes. Note that in client mode only the driver runs locally and all other executors run on different nodes on the cluster.|        

#### 2.2 Cluster Managers (–master)

Using ```--master option```, you specify what cluster manager to use to run your application. Spark currently supports Yarn, Mesos, Kubernetes, Stand-alone, and local. The uses of these are explained below.

| CLUSTER MANAGER |  VALUE   | DESCRIPTION |
|:----------------|:---------|:------------|
| Yarn            | yarn|Use yarn if your cluster resources are managed by Hadoop Yarn.|
| Mesos           | mesos://HOST:PORT|use mesos://HOST:PORT for Mesos cluster manager, replace the host and port of Mesos cluster manager.|
| Standalone      | spark://HOST:PORT|Use spark://HOST:PORT for Standalone cluster, replace the host and port of stand-alone cluster.|
| Kubernetes      | k8s://HOST:PORT<br />k8s://https://HOST:PORT| Use k8s://HOST:PORT for Kubernetes, replace the host and port of Kubernetes. This by default connects with https, but if you wanted to use unsecured use k8s://https://HOST:PORT|                    
| local           |local<br />local[k]<br />local[K,F] | Use local to run locally with a one worker thread.<br />Use local[k] and specify k with the number of cores you have locally, this runs application with k worker threads.<br />Use local[k,F] and specify F with number of attempts it should run when failed.|
                    
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
| :------|:------------|
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
   /spark-home/examples/jars/spark-examples_versionxx.jar 80
   ```
   
   #### 2.4 Other Options
   
| OPTION | DESCRIPTION |
| :------|:------------|
|–files|Use comma-separated files you wanted to use. Usually, these can be files from your resource folder. Using this option, Spark submits all these files to cluster.|
|–verbose|Displays the verbose information. For example, writes all configurations spark application uses to the log file.|

**Note:** Files specified with ```--files``` are uploaded to the cluster.
**Example:** Below example submits the application to yarn cluster manager by using cluster deployment mode and with 8g driver memory, 16g and 2 cores for each executor.

```
./bin/spark2-submit \
   --verbose
   --master yarn \
   --deploy-mode cluster \
   --driver-memory 8g \
   --executor-memory 16g \
   --executor-cores 2  \
   --files /path/log4j.properties,/path/file2.conf,/path/file3.json
   --class org.apache.spark.examples.SparkPi \
   /spark-home/examples/jars/spark-examples_versionxx.jar 80
 ```
 
 ## 3. Spark Submit Configurations
 
Spark submit supports several configurations using ```--config```, these configurations are used to specify Application configurations, shuffle parameters, runtime configurations.
Most of these configurations are the same for Spark applications written in Java, Scala, and Python(PySpark)

| CONFIGURATION KEY | CONFIGURATION DESCRIPTION |
| :------|:------------|
| spark.executor.memoryOverhead|Number of partitions to create for wider shuffle transformations (joins and aggregations).
| spark.serializer|org.apache.spark.serializer.<br />JavaSerializer (default)<br />org.apache.spark.serializer.KryoSerializer|
| spark.sql.files.maxPartitionBytes|The maximum number of bytes to be used for every partition when reading files. Default 128MB.|
| spark.dynamicAllocation.enabled	|Specifies whether to dynamically increase or decrease the number of executors based on the workload. Default true.|
| spark.dynamicAllocation.minExecutors|A minimum number of executors to use when dynamic allocation is enabled.|
|spark.dynamicAllocation.maxExecutors|A maximum number of executors to use when dynamic allocation is enabled.|
|spark.executor.extraJavaOptions|Specify JVM options (see example below)|

Besides these, Spark also supports [many more configurations](https://spark.apache.org/docs/latest/configuration.html).

**Example :**
```
./bin/spark2-submit \
--master yarn \
--deploy-mode cluster \
--conf "spark.sql.shuffle.partitions=20000" \
--conf "spark.executor.memoryOverhead=5244" \
--conf "spark.memory.fraction=0.8" \
--conf "spark.memory.storageFraction=0.2" \
--conf "spark.serializer=org.apache.spark.serializer.KryoSerializer" \
--conf "spark.sql.files.maxPartitionBytes=168435456" \
--conf "spark.dynamicAllocation.minExecutors=1" \
--conf "spark.dynamicAllocation.maxExecutors=200" \
--conf "spark.dynamicAllocation.enabled=true" \
--conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" \ 
--files /path/log4j.properties,/path/file2.conf,/path/file3.json \
--class org.apache.spark.examples.SparkPi \
/spark-home/examples/jars/spark-examples_repace-spark-version.jar 80
```
Alternatively, you can also set these globally @ ```$SPARK_HOME/conf/spark-defaults.conf``` to apply for every Spark application. And you can also set using ```SparkConf``` programmatically.
```
val config = new SparkConf()
config.set("spark.sql.shuffle.partitions",300)
val spark=SparkSession.builder().config(config)
```
First preference goes to SparkConf, then spark-submit –config and then configs mentioned in spark-defaults.conf

## 4. Submit Scala or Java Application

Regardless of which language you use, most of the options are same however, there are few options that are specific to a language, for example, to run a Spark application written in Scala or Java, you need to use the additionally following options.

| OPTION | DESCRIPTION |
| :------|:------------|
|–jars|If you have all dependency jar’s in a folder, you can pass all these jars using this spark submit –jars option. All your jar files should be comma-separated.
for example –jars jar1.jar,jar2.jar, jar3.jar.|
|–packages|All transitive dependencies will be handled when using this command.|
|–class|Scala or Java class you wanted to run.This should be a fully qualified name with the package.For example org.apache.spark.examples.SparkPi.|

**Note:** Files specified with --jars and --packages are uploaded to the cluster.
**Example :**
```
./bin/spark-submit \
--master yarn \
--deploy-mode cluster \
--conf "spark.sql.shuffle.partitions=20000" \
--jars "dependency1.jar,dependency2.jar"
--class com.sparkbyexamples.WordCountExample \
spark-by-examples.jar 
```

## 5. Spark Submit PySpark (Python) Application

When you wanted to submit a PySpark application, you need to specify the .py file you wanted to run or specify the .egg file.
Below are some of the options & configurations specific to PySpark application.

| PYSPARK SPECIFIC CONFIGURATIONS | DESCRIPTION |
| :------|:------------|
|–py-files|Use --py-files to add .py, .zip or .egg files.|
|–config spark.executor.pyspark.memory|The amount of memory to be used by PySpark for each executor.|
|–config spark.pyspark.driver.python|Python binary executable to use for PySpark in driver.|
|–config spark.pyspark.python|Python binary executable to use for PySpark in both driver and executors.|

**Note:** Files specified with --py-files are also uploaded to the cluster.
**Example 1 :**
```
./bin/spark-submit \
   --master yarn \
   --deploy-mode cluster \
   wordByExample.py
```
**Example 2 :**
```
./bin/spark-submit \
   --master yarn \
   --deploy-mode cluster \
   --py-files file1.py,file2.py
   wordByExample.py
```

