## 案例分析之qr-server

我们要创建一个二维码的服务

####工具依赖：

* qr-image       相对qr-image(依赖node-canvas)的这种大众包，显得小巧而且无依赖
* cluster 
* redis			 Nodejs的redis client
* memory-cache	 小巧的cache包，原理就是存储在{}
* sails			 web framework