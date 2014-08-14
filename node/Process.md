node-process-info
================

* [参考1](http://snoopyxdy.blog.163.com/blog/static/60117440201192841649337/)
* [官方api参考x](http://nodejs.org/docs/latest/api/process.html#process_process_argv)

#### process.argv

> 返回一个array，第一个是node，第二个是js文件名，后面是命令行添加的参数

#### process.version

> 返回NODE_VERSION[当前node的版本]，和node -v返回值一样

#### process.platform

> 返回当前使用的平台，比如 darwin

#### process.execPath

> 当前node进程的启动命令路径，node安装目录的绝对路径


#### process.env.SUDO_USER

> 如果是sudo操作的化，返回一个USER

#### process.env.NODE_ENV


#### process.env.NODE_UNIQUE_ID

#### process.exit(0)

> 以指定的code来结束当前的process

#### process.cwd()

> 返回当前执行进程的全路径

#### process.memoryUsage()

> 内存使用情况，看内存消耗很有用的方法


#### process.getuid()

> 只适合POSIX平台，不支持Windows，返回userid  - 比如sudo操作，返回0
