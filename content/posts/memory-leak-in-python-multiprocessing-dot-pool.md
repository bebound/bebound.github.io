+++
title = "Memory Leak in Python multiprocessing.Pool"
author = ["KK"]
date = 2022-03-16T21:04:00+08:00
lastmod = 2022-03-16T21:14:37+08:00
tags = ["Python"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

There is a historical memory leak problem in our Django app and I fixed it recently. As time goes by, the memory usage of app keeps growing and so does the CPU usage.

{{< figure src="/images/pool_before.png" >}}

After some research, I figure out the cause. Some views does not close `multiprocessing.Pool` after using it. The problem disappears when I use `Pool` with `with` statement.

{{< figure src="/images/pool_after.png" >}}

But I'm still interested in it and wrote some testing code. The script is run in Python 3.6.8 and produce similar result when using `multiprocessing.ThreadPool`.

```python3
import time
from multiprocessing import Pool


def func(i):
    return i


def ori():
    # create many thread as time goes by, when i==300 cpu grow to 300%, run out of 16g ram and stuck I have to kill process
    p = Pool(4)
    r.append(p.map(func, range(4)))


def with_close():
    # 100% cpu, 0.1 ram, create 40 thread, takes 41s
    p = Pool(4)
    r.append(p.map(func, range(4)))
    p.close()


def with_terminate():
    # 5% cpu, 0.1 ram, create 4 thread, takes 425s
    p = Pool(4)
    r.append(p.map(func, range(4)))
    p.terminate()


def with_with():
    # same as terminate
    with Pool(4) as p:
        r.append(p.map(func, range(4)))


r = []
s = time.time()
for i in range(4000):
    ori()
    # with_close()
    # with_terminate()
    # with_with()

    if i % 100 == 0:
        print(i)

print(f'takes {time.time() - s} seconds')
```

As you can see, there are four functions. The `ori` function is `Pool` with no `close` and `terminate`, the RAM keeps growing and the script stuck. `with_close`, `with_terminate` and `with_with` will exit normally but time is different.


## Why `close()` is faster than `terminate()` {#why-close-is-faster-than-terminate}

`Pool.terminate()` will call `terminate()` in each worker. `Pool.close()` just change the pool states and each worker will terminate itself. You can find the source code on [GitHub](https://github.com/python/cpython/blob/v3.6.8/Lib/multiprocessing/pool.py).


## Verify Memory Leak {#verify-memory-leak}

```python3
import gc
import time
import weakref


from multiprocessing import Pool

def func(i):
    return i


p = Pool(4)
wr = weakref.ref(p)
p.map(func, range(4))
print(wr())
print(gc.get_referents(wr()))
# p.close()
# p.terminate()
time.sleep(1)
del p
gc.collect()
print(wr())
print(gc.get_referents(wr()))
```

If not calling `close` or `terminate`, after execution, `p` is still referred by some objects:

```nil
<multiprocessing.pool.Pool object at 0x7fc0e6db0828>
[{'_ctx': <multiprocessing.context.ForkContext object at 0x7fc0e6d455c0>, '_inqueue': <multiprocessing.queues.SimpleQueue object at 0x7fc0e6db0860>, '_outqueue': <multiprocessing.queues.SimpleQueue object at 0x7fc0e5babac8>, '_quick_put': <bound method _ConnectionBase.send of <multiprocessing.connection.Connection object at 0x7fc0e620e8d0>>, '_quick_get': <bound method _ConnectionBase.recv of <multiprocessing.connection.Connection object at 0x7fc0e4d241d0>>, '_taskqueue': <queue.Queue object at 0x7fc0e4d24320>, '_cache': {}, '_state': 0, '_maxtasksperchild': None, '_initializer': None, '_initargs': (), '_processes': 4, '_pool': [<ForkProcess(ForkPoolWorker-1, started daemon)>, <ForkProcess(ForkPoolWorker-2, started daemon)>, <ForkProcess(ForkPoolWorker-3, started daemon)>, <ForkProcess(ForkPoolWorker-4, started daemon)>], '_worker_handler': <Thread(Thread-1, started daemon 140466410379008)>, '_task_handler': <Thread(Thread-2, started daemon 140466401986304)>, '_result_handler': <Thread(Thread-3, started daemon 140466393593600)>, '_terminate': <Finalize object, callback=_terminate_pool, args=(<queue.Queue object at 0x7fc0e4d24320>, <multiprocessing.queues.SimpleQueue object at 0x7fc0e6db0860>, <multiprocessing.queues.SimpleQueue object at 0x7fc0e5babac8>, [<ForkProcess(ForkPoolWorker-1, started daemon)>, <ForkProcess(ForkPoolWorker-2, started daemon)>, <ForkProcess(ForkPoolWorker-3, started daemon)>, <ForkProcess(ForkPoolWorker-4, started daemon)>], <Thread(Thread-1, started daemon 140466410379008)>, <Thread(Thread-2, started daemon 140466401986304)>, <Thread(Thread-3, started daemon 140466393593600)>, {}), exitprority=15>}, <class 'multiprocessing.pool.Pool'>]
<multiprocessing.pool.Pool object at 0x7fc0e6db0828>
[{'_ctx': <multiprocessing.context.ForkContext object at 0x7fc0e6d455c0>, '_inqueue': <multiprocessing.queues.SimpleQueue object at 0x7fc0e6db0860>, '_outqueue': <multiprocessing.queues.SimpleQueue object at 0x7fc0e5babac8>, '_quick_put': <bound method _ConnectionBase.send of <multiprocessing.connection.Connection object at 0x7fc0e620e8d0>>, '_quick_get': <bound method _ConnectionBase.recv of <multiprocessing.connection.Connection object at 0x7fc0e4d241d0>>, '_taskqueue': <queue.Queue object at 0x7fc0e4d24320>, '_cache': {}, '_state': 0, '_maxtasksperchild': None, '_initializer': None, '_initargs': (), '_processes': 4, '_pool': [<ForkProcess(ForkPoolWorker-1, started daemon)>, <ForkProcess(ForkPoolWorker-2, started daemon)>, <ForkProcess(ForkPoolWorker-3, started daemon)>, <ForkProcess(ForkPoolWorker-4, started daemon)>], '_worker_handler': <Thread(Thread-1, started daemon 140466410379008)>, '_task_handler': <Thread(Thread-2, started daemon 140466401986304)>, '_result_handler': <Thread(Thread-3, started daemon 140466393593600)>, '_terminate': <Finalize object, callback=_terminate_pool, args=(<queue.Queue object at 0x7fc0e4d24320>, <multiprocessing.queues.SimpleQueue object at 0x7fc0e6db0860>, <multiprocessing.queues.SimpleQueue object at 0x7fc0e5babac8>, [<ForkProcess(ForkPoolWorker-1, started daemon)>, <ForkProcess(ForkPoolWorker-2, started daemon)>, <ForkProcess(ForkPoolWorker-3, started daemon)>, <ForkProcess(ForkPoolWorker-4, started daemon)>], <Thread(Thread-1, started daemon 140466410379008)>, <Thread(Thread-2, started daemon 140466401986304)>, <Thread(Thread-3, started daemon 140466393593600)>, {}), exitprority=15>}, <class 'multiprocessing.pool.Pool'>]
```

After calling `close()` or `terminate()`, the last two lines become:

```nil
None
[]
```


## Document Update {#document-update}

The Python3.7 document adds this [warning](https://docs.python.org/3.7/library/multiprocessing.html#multiprocessing.pool.Pool):

> `multiprocessing.pool` objects have internal resources that need to be properly managed (like any other resource) by using the pool as a context manager or by calling `close()` and `terminate()` manually. Failure to do this can lead to the process hanging on finalization.
> Note that is not correct to rely on the garbage colletor to destroy the pool as CPython does not assure that the finalizer of the pool will be called (see `object.__del__()` for more information).


## The Bug is Fixed in Python 3.8 {#the-bug-is-fixed-in-python-3-dot-8}

In python 3.8.6, the script exits normally and the total execution time also decreases without calling `close()`. I found this issue is fixed in Python bug tracker: [multiprocessing.Pool and ThreadPool leak resources after being deleted](https://bugs.python.org/issue34172).