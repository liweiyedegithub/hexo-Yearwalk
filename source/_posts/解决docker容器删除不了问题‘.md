---
title: 解决docker容器删除不了问题
date: 2025-09-18 10:18:14
tags: 
  - Docker
top: false
categories: 
  - 疑难杂症
img: /medias/featureimages/8.jpg

---

# 解决docker容器删除不了问题
**问题现象是在删除容器时报错“Error response from daemon: container 2a6ace827d869c2ce77cf205ad8a2dd63386ce715ae1e0cabe0c121873648ef0: driver "overlay2" failed to remove root filesystem: unlinkat /data/docker_data/overlay2/e28e4407b147864099d96adc4f3026357e863d149662a52a8b27085b961e3af7/diff/data/aaa/bbb/ccc/classes/cn/db2/form: directory not empty”**
## 出现这个报错一定与异常启动、停止、重启容器导致的磁盘存储问题
``` bash
#按照网上资料通过以下方式：
#1.强制删除容器 （经实验不行）
docker rm -f 2a6ace827d86
#2.手动清理残留文件​ （经实验不行，执行删除后，实际上并没有删除，再删除容器还是报错）
docker inspect 2a6ace827d86 | grep Mounts
sudo rm -rf aaa/bbb/ccc/form/*
#3.直接删除 Overlay2 残留目录​ （经实验不行，删除的时候还是会提示“ directory not empty”）
#4.查看挂载存储对应的进程ID，然后杀掉进程（经实验不行，查不到哪个进程在使用挂载存储）
grep docker /proc/*/mountinfo 
grep docker /proc/*/mountinfo | grep 38735aa65b119c0c9cc620e07329279bcc20e9482feaaf81d85982c5ccc4543 | awk -F’:’‘{print $1}’ | awk -F’/’ ‘{print $3}’
#5.根据其他资料介绍的删除目录，重启docker，重启主机都不行，最后冒出来一个想法：找到
/data/docker_data/overlay2/e28e4407b147864099d96adc4f3026357e863d149662a52a8b27085b961e3af7这个目录，使用mv 命令 换成另外一个名字，然后重启docker，然后再rm 容器id 解决
