+++
title = "Import custom package or module in PySpark"
date = 2020-04-02T22:24:00+08:00
lastmod = 2025-07-18T19:07:21+08:00
tags = ["Python", "Spark"]
categories = ["Programming"]
draft = false
author = "KK"
nocomment = true
nodate = true
nopaging = true
noread = true
+++

First zip all of the dependencies into zip file like this. Then you can use one of the following methods to import it.

```nil
|-- kk.zip
|   |-- kk.py
```


## Using --py-files in spark-submit {#using-py-files-in-spark-submit}

When submit spark job, add `--py-files=kk.zip` parameter. `kk.zip` will be distributed with the main scrip file, and `kk.zip` will be inserted at the beginning of `PATH` environment variable.

Then you can use `import kk` in your main script file.

This utilize Python's zip import feature. For more information, check this link: [zipimport](https://docs.python.org/3.8/library/zipimport.html)


## Using addPyFile in main script {#using-addpyfile-in-main-script}

You can also upload zip file to hdfs, and using `sc.addPyFile('hdfs://kk.zip')` after `SparkContext` is initialized.

This has the same effect as `--py-files`, but your import statement must be after this line.
