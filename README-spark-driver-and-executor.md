# Understanding the working of Spark Driver and Executor

In this page we will understand how Spark’s Driver and Executors communicate with each other to process a given job 
First, let’s see what Apache Spark is. The official definition of Apache Spark says that **“Apache Spark™ is a unified analytics engine for large-scale data processing.”** It is an in-memory computation processing engine where the data is kept in random access memory (RAM) instead of some slow disk drives and is processed in parallel.

Let’s look into the architecture of Apache Spark.

![alt text](https://github.com/DeepHiveMind/Learn-spark/blob/main/images/Image-1-1.png)

## Spark Architecture
As we can see that Spark follows Master-Slave architecture where we have one central coordinator and multiple distributed worker nodes. The central coordinator is called Spark Driver and it communicates with all the Workers.
Each Worker node consists of one or more Executor(s) who are responsible for running the Task. Executors register themselves with Driver. The Driver has all the information about the Executors at all the time.
This working combination of Driver and Workers is known as Spark Application.
The Spark Application is launched with the help of the Cluster Manager. Cluster manager can be any one of the following –
1. **Spark Standalone Mode**
2. **YARN**
3. **Mesos**
4. **Kubernetes**

## DRIVER
Driver is a Java process. This is the process where the main() method of our Scala, Java, Python program runs. It executes the user code and creates a SparkSession or SparkContext and the SparkSession is responsible to create DataFrame, DataSet, RDD, execute SQL, perform Transformation & Action, etc.

## Responsibility of DRIVER
1. The main() method of our program runs in the Driver process. It creates SparkSession or SparkContext.
2. Conversion of the user code into Task (transformation and action). It looks at the user code and determines are the possible Tasks, i.e. the number of tasks to be performed is    decided by the Driver.
   But how does it determines the Tasks?
   – With the help of Lineage. (Discussed later in the blog)
3. Helps to create the Lineage, Logical Plan and Physical Plan.
4. Once the Physical Plan is generated, the Driver schedules the execution of the tasks by coordinating with the Cluster Manager.
5. Coordinates with all the Executors for the execution of Tasks. It looks at the current set of Executors and schedules our tasks.
6. Keeps track of the data (in the form of metadata) which was cached (persisted) in Executor’s (worker’s) memory.

## EXECUTOR:

Executor resides in the Worker node. Executors are launched at the start of a Spark Application in coordination with the Cluster Manager.
They are dynamically launched and removed by the Driver as per required.

## Responsibility of EXECUTOR

1. To run an individual Task and return the result to the Driver.
2. It can cache (persist) the data in the Worker node.

*Thinking how these Driver and Executor Processes are launched after submitting a job (spark-submit)?
Well, then let’s talk about the Cluster Manager.*

## CLUSTER MANAGER

Spark is dependent on the Cluster Manager to launch the Executors and also the Driver (in Cluster mode).
We can use any of the Cluster Manager (as mentioned above) with Spark i.e. Spark can be run with any of the Cluster Manager.

Spark provides a script named “spark-submit” which helps us to connect with a different kind of Cluster Manager and it controls the number of resources the application is going to get i.e. it decides the number of Executors to be launched, how much CPU and memory should be allocated for each Executor, etc.

##  Working Process

spark-submit –master <Spark master URL> –executor-memory 2g –executor-cores 4 WordCount-assembly-1.0.jar

1. Let’s say a user submits a job using “spark-submit”.
2. “spark-submit” will in-turn launch the Driver which will execute the main() method of our code.
3. Driver contacts the cluster manager and requests for resources to launch the Executors.
4. The cluster manager launches the Executors on behalf of the Driver.
5. Once the Executors are launched, they establish a direct connection with the Driver.
6. The driver determines the total number of Tasks by checking the Lineage.
7. The driver creates the Logical and Physical Plan.
8. Once the Physical Plan is generated, Spark allocates the Tasks to the Executors.
9. Task runs on Executor and each Task upon completion returns the result to the Driver.
10. Finally, when all Task is completed, the main() method running in the Driver exits, i.e. main() method invokes sparkContext.stop().
11. Finally, Spark releases all the resources from the Cluster Manager.

## LINEAGE:

When a new RDD is derived from an existing RDD using transformation, that new RDD contains a pointer to the parent RDD and Spark keeps track of all the dependencies between these RDDs using a component called the Lineage. In case of data loss, this lineage is used to rebuild the data. DataFrame, DataSet, SQL are internally converted to RDDs for computation as RDDs are the lowest level of abstraction in Spark. So, all the transformations that are involved internally in a DataFrame, DataSet, SQL can be seen by converting them to RDD.
