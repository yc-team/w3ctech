node-buffer-info
================

js没有处理二进制数据类型，而在Node里面提供了全局的Buffer来处理二进制相关操作。

#### 创建buffer实例

```shell
new Buffer(size)
new Buffer(array)

//encoding可选参数，默认是utf-8
new Buffer(str, [encoding])
```

比如：创建 1KB 的buffer

```shell
//输出随机的一些16进制内存单元
//<Buffer 05 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 90 f0 bf 5f ff 7f 00 00 00 00 00 00 02 00 00 00 02 00 00 00 06 00 00 00 01 00 00 ...>
var buf = new Buffer(1024);
```

如果参数类型是string：

```shell
if (type === 'string') {
	//直接将subject调用write方法，offset设置为0
	this.length = this.write(subject, 0, encoding);
}
```




这里面有一个 *8KB* 的说法：

```shell
Buffer.poolSize = 8 * 1024;
var pool;
function allocPool() {
  pool = new SlowBuffer(Buffer.poolSize);
  pool.used = 0;
}
```

分配buffer池，默认大小是8KB，其实8KB就是一个存储的空间


1. 如果创建的buffer大小 < 8KB && <= 当前剩余 8KB 里面能够分配的大小, 则此buffer实例存入，和其他buffer实例共享这个 8KB
2. 如果创建的buffer大小 > 8KB, 则此buffer存储在一个重新实例化的SlowBuffer

```shell
function Buffer(subject, encoding, offset) {
	//会和8KB比较
	if (this.length > Buffer.poolSize) {

		//如果大于8KB就再实例化SlowBuffer，赋值给this.parent就可以知道对应保存在哪个内存块
		this.parent = new SlowBuffer(this.length);
		//不需要公用内存，就至offset为0
		this.offset = 0;

	} else if (this.length > 0) {

		if (!pool || pool.length - pool.used < this.length) allocPool();

		this.parent = pool;
		this.offset = pool.used;

	} else {

		//把0传给SlowBuffer
		var zeroBuffer = new SlowBuffer(0);
		this.parent = zeroBuffer;
      	this.offset = 0;

	}
}
```

#### length的意义

```shell
var buf = new Buffer(1024);
console.log(buf.length);  //1024
buf.write('some string', 0, 'ascii');
console.log(buf.length);  //1024
```

所以length是buffer对象的分配内侧的总大小，buffer对象的内容发生变化,length不变


#### buffer和字符串如何转换

*. 字符串转buffer

```shell
var buf = new Buffer('zhangyaochun');
console.log(buf); // <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>
```

*. buffer转成string

```shell
var buf = new Buffer([0x7a, 0x68, 0x61, 0x6e, 0x67, 0x79, 0x61, 0x6f, 0x63, 0x68, 0x75, 0x6e]);
console.log(buf.toString()); // zhangyaochun
```

这里主要介绍一下buf.toString，它是用在：

> 对buffer进行解码，返回一个字符串

```shell
//encoding 可选，默认是uft8
//start 可选，默认是0
//end 可选，默认是buffer.length
buf.toString([encoding], [start], [end]);
```


#### 如何转成json？

```shell
var buf = new Buffer('zhangyaochun');
console.log(buf);  // <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>
console.log(buf.toJSON());  // [122,104,97,110,103,121,97,111,99,104,117,110]
```

可以看到，直接调用buf.toJSON() 返回JSON Array

还记得：JSON.stringify吗？

```shell
var buf = new Buffer('zhangyaochun');
console.log(buf);  // <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>
console.log(JSON.stringify(buf));  // [122,104,97,110,103,121,97,111,99,104,117,110]
```

会发现，这种返回的一样的.



那有没有人会疑问，那咋再转回buf呢？

```shell
var buf = new Buffer('zhangyaochun');
console.log(buf);  // <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>
var json = JSON.stringify(buf);  
console.log(json); // [122,104,97,110,103,121,97,111,99,104,117,110]

var copy = new Buffer(JSON.parse(json));
console.log(copy); // <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>
```

其实就是再实例化一个Buffer。



#### 如何设置和获取对应index下标的buffer呢？

```shell
buf[index]
```

返回0-255 或者 0x00-0xFF

看下面的代码示例：

```shell
var buf = new Buffer('zhangyaochun');
console.log(buf.length); //12
console.log(buf[0]);     //122 
console.log(buf[1]);     //104
console.log(buf[2]);     //97
```


#### slice

```shell
Buffer.prototype.slice = function(start, end) {
	var len = this.length;

	//... 对参数start和end进行处理

	//原理还是new Buffer
	return new Buffer(this.parent, end - start, start + this.offset());

};
```

我们来看看示例代码：

```shell
var buf = new Buffer([0x7a, 0x68, 0x61, 0x6e, 0x67, 0x79, 0x61, 0x6f, 0x63, 0x68, 0x75, 0x6e]);
console.log(buf);  // <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>

var sub = buf.slice(2);
console.log(sub);  // <Buffer 61 6e 67 79 61 6f 63 68 75 6e>

sub[0] = 0x65;
console.log(sub);  // <Buffer 65 6e 67 79 61 6f 63 68 75 6e>
console.log(buf);  // <Buffer 7a 68 65 6e 67 79 61 6f 63 68 75 6e>
```

> 我们看到居然buf改变了，slice会作用于原来的Buffer



#### buf.write

把指定的string用指定的encoding，写入buffer

```shell
//offset 可选 默认 0
//length 可选 buffer.length - offset
//encoding 可选 默认 utf8
buf.write(string, [offset], [length], [encoding])
```

实例：

```shell
var buf = new Buffer(1024);

// <Buffer 7a 68 61 6e 67 79 61 6f 63 68 75 6e>
console.log(new Buffer('zhangyaochun')); 

// <Buffer a9 7b 22 f4 e6 17 00 00 00 00 00 00 b6 01 00 00 11 7e 22 f4 e6 17 00 00 00 9e 00 01 01 00 00 00 00 00 00 00 00 00 00 00 e0 f9 03 01 01 00 00 00 04 00 00 ...>
console.log(buf); 

//write指定了offset,注意从0开始

var len = buf.write('zhangyaochun', 2);
console.log(len); //12

// <Buffer a9 7b 7a 68 61 6e 67 79 61 6f 63 68 75 6e 00 00 11 7e 22 f4 e6 17 00 00 00 9e 00 01 01 00 00 00 00 00 00 00 00 00 00 00 e0 f9 03 01 01 00 00 00 04 00 00 ...>
console.log(buf);
```




#### 什么时候用buffer

当保存非utf-8类型的字符串，或者2进制格式的时候，能用字符串的还是用字符串。


#### 判断参数是否是Buffer

```shell
Buffer.isBuffer(obj)
```

看下面的代码示例：

```shell
var buf = new Buffer('zhangyaochun');
var str = 'zhangyaochun';
Buffer.isBuffer(str); // false
Buffer.isBuffer(buf); // true
```


#### 参考链接：

* [node官网关于buffer](http://nodejs.org/api/buffer.html#buffer_buffer)





