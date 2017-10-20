title: gzip压缩相关
date: 2017-06-15 21:51:32
tags: JavaScript
---

##  gzip压缩angular

在看angular源码时看到有check-size.sh脚本:

```shell
#!/bin/bash

grunt minify
gzip -c < build/angular.min.js > build/angular.min.js.gzip
ls -l build/angular.min.*
```

1.执行grunt 压缩命令
2.再用gzip压缩下
3.列出angular.min.* 文件的大小

参考资料：
[gzip-js is a pure JavaScript implementation of the GZIP file format](https://github.com/beatgammit/gzip-js)
