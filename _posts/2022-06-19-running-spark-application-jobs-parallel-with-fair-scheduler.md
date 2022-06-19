---
title: Running spark application jobs(actions) parallel with FAIR scheduler
date: 2022-06-13T09:51:20+01:00
layout: single
---

This article is about, how to run spark application jobs parallel and available spark configurations.

**What is spark  job?**
- Job is nothing but spark action (write, count, collect etc.) in a spark submit.
  Every Spark application divided into multiple jobs, job into stages and stage into  tasks.

![job-stage-task](https://raw.githubusercontent.com/sumitppawar/draw/master/Untitled%20Diagram.jpg)

- To make action (job) concurrent we need to run action on separate thread, but executing them concurrently is depends on the scheduler mode and resource required by each jobs.
- **By default** spark job scheduler executes job in `FIFO`, irrespective of jobs are created concurrently or sequentially in a spark submit.
- In `FIFO` mode, all Resource is given to first job, if some left then it will be given to next in queue. If first job taking all resources then next queued job has to wait for resources.

**How to execute concurrent jobs in parallel ?**
- Spark provides another job scheduler mode called `FAIR`, it will distribute resources fairly to jobs.
- Set configuration  `spark.scheduler.monde`  to `FAIR`, note this can only be set while creating spark  session.
- Another note, even scheduler mode is `FAIR` but jobs are in sequence (not used separate thread) , it will be executed in sequence.

**Let's understand with example and code**

In the below image, at right side we have input table and spark application will read input, union data to get more rows, finally write it to five times on local file system concurrently.

![spark-app](https://raw.githubusercontent.com/sumitppawar/draw/master/a.png)

Below code is in `Scala 2.13`
- Jobs(1,2,3,4,5) are running concurrently  using Scala  `Future`, on separate thread.
- Data file is [customers.csv](https://github.com/sumitppawar/data/blob/main/customers.csv)
- Also doing `union` on customer data before concurrent write to get more rows.

Firstly, let's run with default scheduler mode **`FIFO`**

<script src="https://gist.github.com/sumitppawar/d43307acffacd43d181e68456d2de8ee.js"></script>

Spark UI screenshot for jobs, using `FIFO` mode.
- Jobs(1,2,3,45) are submitted concurrently, but only one is running

![enter image description here](https://raw.githubusercontent.com/sumitppawar/draw/master/fifo.png)

Let's set scheduler mode to **`FAIR`**  (`.config("spark.scheduler.mode", "FAIR")`)

Below is spark UI screenshot for jobs, using `FAIR` mode.
- UI is showing `Scheduling Mode: FAIR`
- Jobs(1,2,3,4,5) submitted concurrently, as the are active all together
- `Job(2)` and `Job(3)` are running parallel

![enter image description here](https://raw.githubusercontent.com/sumitppawar/draw/master/fair.png)


**Conclusion**
- By default, spark scheduler mode is `FIFO` , which executes jobs(actions) in sequence as per resource availability. If some resource left after starting head job then it will be given to next in queue.
- In `FIFO` even with jobs(Actions) are concurrent, it will be executing in sequence if the first one needs all resources.
- We can change schedule mode by using config `spark.scheduler.mode` , this can be set only while creating spark session.
- In`FAIR` scheduling mode, resources are distributed among jobs fairly.
- In `FAIR`, if jobs(actions) are in sequence, then it will be executed in sequence.
- To run jobs concurrently we need to use `FAIR` mode and job needs to run on separate threads. 
