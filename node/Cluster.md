node-cluster-info
================

[核心模块](https://github.com/joyent/node/blob/master/lib/cluster.js)，主要用于Nodejs的多核处理，试用于一些负载均衡的集群环境

目前的状态:

> Stability: 1 - Experimental

[官方api地址](http://nodejs.cn/api/cluster#cluster_cluster_settings)

* cluster.isWorker

返回Boolean值，判断当前进程是否是从主进程的fork出来的，如果process.env.NODE_UNIQUE_ID有值的化，返回true

源码：

```shell
cluster.isWorker = ('NODE_UNIQUE_ID' in process.env);
```

* cluster.isMaster

返回Boolean值，判断是否是主进程

源码：

```shell
cluster.isMaster = (cluster.isWorker === false);
```



* cluster.fork
* cluster.worker
* cluster.worker.id
* cluster.worker.process




