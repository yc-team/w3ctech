## 案例分析之各种宝

在命令行展示各种宝的数据

如图：

![各种包](http://pan.baidu.com/s/1o6nylIi)

####工具依赖：

* request       请求必备
* iconv-lite    转码必备
* async         asynchronous
* cheerio       解析html
* cli-table     cli里面展示数据table利器

1. 发请求，爬有各种宝数据的网站

考虑几个问题，不一样的网站的编码不一样，比如有的不是utf-8，request返回的response如何设置

```shell
var options = {};

request(options, function(error, response, body){
	if (!error && response.statusCode == 200) {
		//...
	}
});
```

看一个options里面的参数：

```shell
`encoding` - Encoding to be used on `setEncoding` of response data. If `null`, the `body` is returned as a `Buffer`.
```

补充：

* uri || url - fully qualified uri or a parsed url object from url.parse()

很多人习惯不一样，其实两个都是一样的


2. 解码服务返回的html

```shell
//body是request请求返回的
iconv.decode(body, 'gbk');
```

我们来看一下iconv-lite的源码：

```shell

iconv.decode = function decode(buf, encoding, options){
	if (typeof buf === 'string') {
        if (!iconv.skipDecodeWarning) {
            console.error('Iconv-lite warning: decode()-ing strings is deprecated. Refer to https://github.com/ashtuchkin/iconv-lite/wiki/Use-Buffers-when-decoding');
            iconv.skipDecodeWarning = true;
        }

        buf = new Buffer("" + (buf || ""), "binary"); // Ensure buffer.
    }
}

```

所以：这个body必须是Buffer类型的


3. 获取body里面指定dom元素的值

```shell
var $ = cheerio.load(body);
//有没有jq的感觉
var fundName = $('div.bktit_new > a').text().trim();
```


4. 在cli里面展示table数据

注释：cli-table最新的0.0.3不支持中文head，还是用0.0.1的tag版本

```shell
var table = new Table({
    head: ['名称', '合作基金', '7日年化', '万份收益', '数据日期'],
    style : {compact : true, head: ['yellow']}
});

table.push(
	[],
	[]
);

console.log(table.toString());
```

补充：style可以设置更多：

我们看看源码：

```shell
style: {
    'padding-left': 1,
	'padding-right': 1,
	head: ['red'],
	border: ['grey'],
	compact : false
}
```

5. async只要是针对多个宝遍历

```shell
var funds = [
	{
        "code": "000198",
        "bao": "余额宝"
    },{
        "code": "000330",
        "bao": "现金宝"
    }
];
async.each(funds, function(fund, callback){
	
}, function(error){
	//..
});
```

我们先来看看async.each的源码：

```shell
var _each = function (arr, iterator) {
    if (arr.forEach) {
        return arr.forEach(iterator);
    }
    for (var i = 0; i < arr.length; i += 1) {
        iterator(arr[i], i, arr);
    }
};

var root = this;

function only_once(fn) {
    var called = false;
    return function() {
        if (called) throw new Error("Callback was already called.");
        called = true;
        fn.apply(root, arguments);
    }
}



async.each = function(arr, iterator, callback){
	callback = callback || function(){};

	if (!arr.length) {
		return callback();
	}

	var completed = 0;

	function done(err) {
		if (err) {
			callback(err);
			callback = function(){};
		} else {
			completed += 1;
			//completed >= arr.length
			if (completed >= arr.length) {
				callback();
			}
		}
	}

	_each(arr, function(x) {
		iterator(x, only_once(done));
	});


};
```

注释：

> async.forEach 和 async.each 是一样的



