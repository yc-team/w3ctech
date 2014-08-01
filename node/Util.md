node-util-info
================

* util.format

```shell
var ip = '93.123.23.7';
var query = 'wandoujia';
var URL = 'http://%s/search?q=%s';

//http://93.123.23.7/search?q=wandoujia
console.log(util.format(URL, ip, query));
```




