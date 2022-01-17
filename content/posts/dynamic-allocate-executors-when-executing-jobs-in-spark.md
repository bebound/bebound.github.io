+++
title = "Dynamic Allocate Executors when Executing Jobs in Spark"
author = ["KK"]
date = 2021-07-18T16:52:00+08:00
lastmod = 2022-01-15T14:25:28+08:00
tags = ["Spark"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

I wrote a Spark program to process logs. The number of logs always changes as time goes by. To ensure logs can be processed instantly, the number of executors is calculated by the maximum of logs per minutes. As a consequence, the CPU usage is low in executors. In order to decrease resource waste, I tried to find a way to schedule executors during the execution of program.

As shown below, the maximum number of logs per minutes can be a dozen times greater than the minimum number in one day.

{{< figure src="/images/dynamic_log_number.png" >}}

If I can modify the executor number by size of data to proceed, the resource usage should increase.


## Dynamic Allocation {#dynamic-allocation}

Spark provide a similar configuration to control the number of executors. By enable `spark.dynamicAllocation.enabled`, spark will change number of running executors by task number automatically.


### How does Dynamic Allocation Work? {#how-does-dynamic-allocation-work}


#### Request Executors {#request-executors}

As is known to all, the action operators(such as `count`, `collect`) create Spark job. Each job is divided into stages by shuffle operation, and each data partition in the stage will become independent jobs. When dynamic allocation is enabled, if there have been pending tasks for `spark.dynamicAllocation.schedulerBacklogTimeout` seconds, driver will request for more executors. If the pending task still exists, the executor request will be triggered every `spark.dynamicAllocation.sustainedSchedulerBacklogTimeou` seconds. Furthermore, the number of executors requested in each round increases exponentially from the previous round. For instance, an application will add 1 executor in the first round, and then 2, 4, 8 and so on executors in the subsequent rounds. The number of total running executor should not exceed `spark.dynamicAllocation.maxExecutors`.

When receiving the first executor request, driver ask cluster manager to create executor. After the new executor is created, driver checks if there are more request waiting to created and handle all of the pending request.

The reason to use this strategy to create executor is to avoid creating too many executor when payload just peak for a short time and make sure there are enough executor to be created in a period of time if the payload keeps high.


#### Release Executors {#release-executors}

After the executor is idle for `spark.dynamicAllocation.executorIdleTimeout` seconds, it will be released. The one which contains cache data will not be removed. To prevent the executor which keeps the shuffle data from being removed, a additional spark service is needed before spark 3.0. From 3.0, the external shuffle service is not required if `spark.dynamicAllocation.shuffleTracking.enabled` is used.

Dynamic allocation is easy to used, but there are two disadvantage:

1.  Slow scheduling. Creating executors is serial. If two or more executor is requested, driver will ask cluster manager to create executors for at least two times. This is an issue if pods creation takes time. In general, that is fine as the K8s 1.6 SLO is that 99% of pods should be created in 5s in a 5000 node cluster.
2.  Hard to release executor if each task is short. The release is based on the idle time. If there are so many short task, the executor is not like to idle as tasks are assigned uniformly.

In our spark program, the task is short and data must be processed in 1 minutes. So dynamic allocation not suitable.


## Manual Allocation {#manual-allocation}

Luckily, spark also provide a way to control the number of executors manually. We can use `sc.requestExecutors` and `sc.killExecutors` to create and delete executors.

In order to use these two function, we have to know the number of running executors and their IDs.

**Number of Running Executors**

The Spark program's RAM usage can be obtained from `sc.getExecutorMemoryStatus`. It returns a dict list like this: `[Map(10.73.3.67:59136 -> (2101975449,2101612350))]`. The key is IP with port and value is a tuple contains the max RAM and available RAM. Please note that driver is also included in the return data.

**IDs of Running Executors**

IDs is required when calling `sc.killExecutors`. This can be found in [Spark REST API](https://spark.apache.org/docs/latest/monitoring.html#rest-api). The executors information such as ID, cores and tasks is record in `/applications/[app-id]/executors`.

With the help of `sc.requestExecutors`, we can create as many executors as we want in one request. But the pod create time is still too long. To eliminate the pod create request, I used these strategies:

1.  The running executors is expected to finish job in 50s, fot the purpose of reversing some time for delayed tasks.
2.  When the expected executor is close to current running executors, no executor is requests or released.
3.  If there is backlog data, request more executors.


## Result {#result}

After using manually allocation, the CPU usage grows a lot and reaches 40%. The cores used by Spark programs drop from 1700 to 800. Furthermore, the Spark program can scale automatically.

{{< figure src="/images/dymanic_cpu_before.png" >}}

{{< figure src="/images/dymanic_cpu_after.png" >}}