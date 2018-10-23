###PROCESSING GDELT DATASET IN AWS CLUSTERS USING SPARK

####1.GDELT DATASET

<p>The GDelt 2.0 Global Knowledge Graph (GKG) indexes
persons,organizations, companies, locations, themes, and even emotions from live news reports in print, broadcast and internet sources all over the world. 
</p>

####2.TASK
<p>
	The task in hand was to write an application in spark that analyzes the given dataset and constructs the 10 most talked about topics per day. To achieve this, we had to run the application for the whole dataset for which we have used AWS EMR service.It is a layer on top of EC2 which allow users to deploy Spark applications.The GDELT GKG was hosted on S3 in the US EAST Region.
	We have used spot instances to bid for unused machines to process the data.
</p>

####3.CODE
<p></p>

###4.ISSUES FACED WHILE RUNNING THE APPLICATION FOR FEW FILES
<p>
  For our initial run, We decided to use spark's default configuration for the cluster and did not modify any configurations. We faced few issues described below:

  1. **Executor nodes becomes idle** : We noticed during our first run that load on different executors node varied quite a lot. See the aggregated load graph taken from Ganglia. After digging into spark history server, we found a lot of executors were killed and spawned at different times.
  2. **GC overhead limit in driver node** : After 32 mins of execution, our application throws \texttt{java.lang.OutOfMemoryError: GC overhead limit exceeded}. This exception came in the driver node.  

</p>

####5.Configuration Changes Made
<p>
	After reading through blogs and Spark's configuration , we collected information about different configuration parameters and used 5 configuration for our future run.
	
1.**spark.executor.instances,spark.executor.memory,spark.executor.cores** : We fixed the number of executors and assigned fixed memory and cores to these executors to have a more balanced distribution of load. The numbers were calculated using the available number of cores, Ram and number of nodes.


2.**spark.driver.memory** : We figured out the OutOfMemoryError could be because of low driver memory assigned to driver process by default(1G). It might indicate that memory space is not used efficiently by Spark process or application. Thus, we tried increasing the value for the same.


3.**spark.storage.memoryFraction**: Since we were not using any cache or persist objects, we wanted to give smallest possible value so that JVM has more memory at it's disposal for execution and tranformation tasks.
</p>

##FINAL RESULTS
<p>
After deciding the two implementations(dataframe and sql) and required configuration parameters, We ran our spark application over the whole GDelt dataset. We found that cluster load and cpu time for each executor node was very similar for both implementations. Though, Dataframe implementation with only SQL Queries outperformed the other implementation by **4.3 mins.**
The Dataframe implementation using queries to replace Map/Filter functions took 16.9 mins while other implementation took **12.6mins**. More Detailed timing  can be found in Figure

**FIGURE**

The load graph for better implementation across all implementations can be viewed in the Figure

 **FIGURE**
</p>