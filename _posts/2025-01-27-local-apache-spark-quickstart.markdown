---
layout: post
title: "Local Apache Spark Quickstart"
date: 2025-01-27 00:00:00 -0700
tags: python spark
---

I wanted to get started with Apache Spark and SparkSQL on my local machine so
that I could learn how to operate on large scale geospatial data. This guide
walks through Docker configuration, a sample data set in CSV format, and a dead
simple script that we'll submit to Spark via Docker.

## Docker Configuration

This is the minimal configuration for setting up Spark on your local machine.

{% highlight yaml %}
services:
  spark-master:
    image: bitnami/spark:3.5.0
    ports:
      - "8080:8080" # Spark UI
      - "7077:7077" # Spark Master port
    environment:
      - SPARK_MODE=master
    volumes:
      - ./app:/opt/bitnami/spark/app
  spark-worker-1:
    image: bitnami/spark:3.5.0
    depends_on:
      - spark-master
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
    volumes:
      - ./app:/opt/bitnami/spark/app
volumes:
  app:
{% endhighlight %}

## Data Schema

I have a hilariously small data set for Apache Spark, 1671 rows, but this is
the general shape. This data should live in an `./app/` directory, in your
current working directory. This is linked to the Docker compose environment.

{% highlight csv %}
API,FTP_LATITUDE,FTP_LONGITUDE,LTP_LATITUDE,LTP_LONGITUDE,SPUD_DATE,CUM_OIL_06_MONTH
4238932145,31.3672544,-103.2766876,31.3602855,-103.2783914,1998-03-12 00:00:00,382.0
4238932290,31.1431862,-103.3475596,31.1557462,-103.3462224,2004-01-06 00:00:00,1.0
4238932446,31.2874187,-103.4865769,31.2867779,-103.493138,2006-12-29 00:00:00,27486.0
4238932457,31.2749213,-103.3917072,31.2746103,-103.4038642,2007-03-04 00:00:00,2773.0
4238932486,31.3818528,-103.3206018,31.3751552,-103.3222759,2007-08-30 00:00:00,12009.0
4238932498,31.3796621,-103.3130481,31.3875774,-103.3110596,2007-12-20 00:00:00,14062.0
4238932504,31.4099656,-103.2628631,31.4126112,-103.2619259,2008-02-03 00:00:00,36509.0
4238932522,31.2887026,-103.7434503,31.2789202,-103.7478164,2008-03-26 00:00:00,11.0
4238932532,31.417923,-103.4118079,31.4267432,-103.4098413,2008-04-18 00:00:00,4676.0
{% endhighlight %}

## Python Code

This script just prints out the number of rows in `./app/data.csv`, and the
column names in the CSV. This should sit next to the data file in `./app/`. We
do this so that the Docker image can "see" the script and the data.

{% highlight python %}
# filename: ./app/analysis_01.py

from pyspark.sql import SparkSession

if __name__ == "__main__":

    spark = SparkSession.builder.appName("Analysis 01").getOrCreate()
    df = spark.read.csv("/opt/bitnami/spark/app/data.csv", header=True, inferSchema=True)

    print("Number of rows:", df.count())
    print("Schema:")
    df.printSchema()

    spark.stop()
{% endhighlight %}

## Run the Code

To run this, we'll send a command to the Docker image using the
`docker exec -it` functionality. I created a `run.bash` script to do this; it
takes a single CLI argument, the container ID of the `spark-master` container.
You can figure this out by running `docker ps` then copy/pasting that ID when
running `bash run.bash <docker-container-id>`. Here's the contents of
`run.bash`.

{% highlight bash %}
#!/usr/bin/bash
docker exec -it $1 spark-submit --master local[*] /opt/bitnami/spark/app/analysis_01.py
{% endhighlight %}

At the end of it all, you should see some log statements where the Executor is
talking to the TaskScheduler, and the DAGScheduler, etc., and then our output,
and then log chatter about shutting everything down.

{% highlight console %}
25/01/27 21:53:30 INFO Executor: Finished task 0.0 in stage 5.0 (TID 4). 3995 bytes result sent to driver
25/01/27 21:53:30 INFO TaskSetManager: Finished task 0.0 in stage 5.0 (TID 4) in 41 ms on fc849b91cd54 (executor driver) (1/1)
25/01/27 21:53:30 INFO TaskSchedulerImpl: Removed TaskSet 5.0, whose tasks have all completed, from pool
25/01/27 21:53:30 INFO DAGScheduler: ResultStage 5 (count at NativeMethodAccessorImpl.java:0) finished in 0.051 s
25/01/27 21:53:30 INFO DAGScheduler: Job 4 is finished. Cancelling potential speculative or zombie tasks for this job
25/01/27 21:53:30 INFO TaskSchedulerImpl: Killing all running tasks in stage 5: Stage finished
25/01/27 21:53:30 INFO DAGScheduler: Job 4 finished: count at NativeMethodAccessorImpl.java:0, took 0.057968 s
Number of rows: 1671
Schema:
root
 |-- API: long (nullable = true)
 |-- FTP_LATITUDE: double (nullable = true)
 |-- FTP_LONGITUDE: double (nullable = true)
 |-- LTP_LATITUDE: double (nullable = true)
 |-- LTP_LONGITUDE: double (nullable = true)
 |-- SPUD_DATE: timestamp (nullable = true)
 |-- CUM_OIL_06_MONTH: double (nullable = true)

25/01/27 21:53:30 INFO SparkContext: SparkContext is stopping with exitCode 0.
25/01/27 21:53:30 INFO SparkUI: Stopped Spark web UI at http://fc849b91cd54:4040
25/01/27 21:53:30 INFO MapOutputTrackerMasterEndpoint: MapOutputTrackerMasterEndpoint stopped!
25/01/27 21:53:30 INFO MemoryStore: MemoryStore cleared
25/01/27 21:53:30 INFO BlockManager: BlockManager stopped
25/01/27 21:53:30 INFO BlockManagerMaster: BlockManagerMaster stopped
25/01/27 21:53:30 INFO OutputCommitCoordinator$OutputCommitCoordinatorEndpoint: OutputCommitCoordinator stopped!
25/01/27 21:53:30 INFO SparkContext: Successfully stopped SparkContext
25/01/27 21:53:30 INFO ShutdownHookManager: Shutdown hook called
25/01/27 21:53:30 INFO ShutdownHookManager: Deleting directory /tmp/spark-294fb3d4-6f43-4ffb-962a-5226b2e88219
25/01/27 21:53:30 INFO ShutdownHookManager: Deleting directory /tmp/spark-b18b2d8a-91c2-48df-ae39-aca16aec9da5
25/01/27 21:53:30 INFO ShutdownHookManager: Deleting directory /tmp/spark-b18b2d8a-91c2-48df-ae39-aca16aec9da5/pyspark-a218847b-d988-4df2-8514-f635f2855f04
{% endhighlight %}

So, that's not a whole lot of computation, but that's the *Hello World* for
using SparkSQL, with real world data, using Docker compose, locally.
