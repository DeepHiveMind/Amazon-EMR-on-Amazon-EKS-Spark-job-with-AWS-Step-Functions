# Understanding the working of Spark Driver and Executor

In this page we will understand how Spark’s Driver and Executors communicate with each other to process a given job 
First, let’s see what Apache Spark is. The official definition of Apache Spark says that **“Apache Spark™ is a unified analytics engine for large-scale data processing.”** It is an in-memory computation processing engine where the data is kept in random access memory (RAM) instead of some slow disk drives and is processed in parallel.

Let’s look into the architecture of Apache Spark.



## Spark Architecture
As we can see that Spark follows Master-Slave architecture where we have one central coordinator and multiple distributed worker nodes. The central coordinator is called Spark Driver and it communicates with all the Workers.
Each Worker node consists of one or more Executor(s) who are responsible for running the Task. Executors register themselves with Driver. The Driver has all the information about the Executors at all the time.
This working combination of Driver and Workers is known as Spark Application.
The Spark Application is launched with the help of the Cluster Manager. Cluster manager can be any one of the following –
1. **Spark Standalone Mode**
2. **YARN**
3. **Mesos**
4. **Kubernetes**
