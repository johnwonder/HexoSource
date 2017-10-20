title: nodejs的命令行是如何调用的
date: 2017-06-17 22:07:06
tags: nodejs
---

## nodejs命令是从哪里调用的

研究了下hexo的命令调用

### npm config
运行了下 npn config list命令，输出如下:

```
; cli configs
scope = ""
user-agent = "npm/4.2.0 node/v7.10.0 win32 x64"

; userconfig C:\Users\Administrator\.npmrc
cache = "D:\\npmcache"
prefix = "D:\\npmglobal"
registry = "https://registry.npm.taobao.org/"

; builtin config undefined

; node bin location = D:\nodejs\node.exe
; cwd = F:\code\blognew
; HOME = C:\Users\Administrator
; "npm config ls -l" to show all defaults.
```

prefix和cache是npm的全局路径，

我们到该目录下发现有个hexo.cmd和hexo 文件，打开hexo.cmd，代码如下：

```
@IF EXIST "%~dp0\node.exe" (
  "%~dp0\node.exe"  "%~dp0\node_modules\hexo-cli\bin\hexo" %*
) ELSE (
  @SETLOCAL
  @SET PATHEXT=%PATHEXT:;.JS;=;%
  node  "%~dp0\node_modules\hexo-cli\bin\hexo" %*
)
```

那我们就到当前目录的node_modules中看```hexo-cli\bin\hexo```了:

```js
#!/usr/bin/env node

'use strict';

require('../lib/hexo')();

console.log('hi');
```

在任意目录下执行hexo命令会输出

```js
hi
Usage: hexo <command>

Commands:
help     Get help on a command.
init     Create a new Hexo folder.
version  Display version information.

Global Options:
--config  Specify config file instead of using _config.yml
--cwd     Specify the CWD
--debug   Display all verbose messages in the terminal
--draft   Display draft posts
--safe    Disable all plugins and scripts
--silent  Hide output on console

For more help, you can use 'hexo help [command]' for the detailed information
or you can check the docs: http://hexo.io/docs/
```

### DOS批处理
DOS批处理命令：
%~dp0 代表当前目录

SETLOCAL
开始批处理文件中环境改动的本地化操作。在执行 SETLOCAL 之后所做的环境改动只限于批处理文件。要还原原先的设置，必须执行 ENDLOCAL。 达到批处理文件结尾时，对于该批处理文件的每个尚未执行的 SETLOCAL 命令，都会有一个隐含的 ENDLOCAL 被执行。

PathExt修改 让我们的Bat文件执行的优先级别在其他可执行文件之前就OK了
[DOS批处理中%~dp0表示什么意思](http://blog.csdn.net/hncsl/article/details/59015497)
[批处理，%~d0 cd %~dp0 代表什么意思](https://zhidao.baidu.com/question/117267593.html)
[cmd SETLOCAL使用介绍](http://www.jb51.net/article/36043.htm)
[修改Pathext，让你的东东优先执行](http://tieba.baidu.com/p/73557959)
