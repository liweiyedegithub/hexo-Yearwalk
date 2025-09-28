---
title: Linux操作系统常用环境配置
date: 2020-05-04 10:40:14
tags: 
  - Linux
  - 环境配置
top: false
categories: 
  - Linux
img: /medias/featureimages/9.jpg
---

# 快捷建设置
``` shell
#编辑或创建~/.bashrc文件
vi ~/.bashrc
# rm -i → 每次删除文件前都会提示确认
alias rm='rm -i'
# cp -i → 复制文件前提示确认，如果目标文件已存在，会问你是否覆盖。
alias cp='cp -i'
# mv -i → 移动或重命名文件时，如果目标文件已存在，会提示确认。
alias mv='mv -i'
# 告诉系统你使用的终端类型支持 256 色 让 Vim、less、top 等程序能正确显示颜色。
export TERM=xterm-256color
#-l → 长格式显示（详细信息：权限、大小、时间等）
#-r → 逆序排列
#-t → 按修改时间排序（最新的文件排在最后，如果加 -r 则最早的文件排在前）
alias ll='ls -lrt'
#保存退出
#生效
source ~/.bashrc
```
# 常用工具安装
``` shell
sudo apt update

#网络连通性工具：curl wget telnet ping traceroute
sudo apt install -y curl wget telnet net-tools iproute2 nmap openssh-client

#系统监控工具：htop iotop iftop dstat
sudo apt install -y htop iotop iftop sysstat dstat

#文本处理工具：vim grep awk sed less tree
sudo apt install -y vim nano less grep tree zip unzip rsync locate

#Git 和自动化工具：git python3 jq tmux docker
sudo apt install -y git

```