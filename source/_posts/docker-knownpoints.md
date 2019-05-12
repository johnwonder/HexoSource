title: docker容器学习资料整理
date: 2019-05-12 09:08:25
tags: docker
---

1. docker rm -f db01 db02 强制删除容器db01、db02
-f :通过SIGKILL信号强制删除一个运行中的容器

2. docker cp :用于容器与主机之间的数据拷贝。
docker cp  96f7f14e99ab:/www /tmp/
将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。
