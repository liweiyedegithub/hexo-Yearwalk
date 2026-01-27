---
title: Kubernetes 容器运行节点指定与调度控制
date: 2025-11-13 10:25:01
tags: 
  - Docker
  - Kubernetes
  - Yaml
top: false
categories: 
  - 性能优化
  - K8s节点调度
img: /medias/featureimages/13.jpg

---
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题  <!--最多6级标题-->

任务列表
- [ ] 任务一 未做任务 `- + 空格 + [ ]`
- [x] 任务二 已做任务 `- + 空格 + [x]`

对齐方式

<center>行中心对齐</center>
<p align="left">行左对齐</p>
<p align="right">行右对齐</p>


斜体、粗体、删除线、下划线、背景高亮


*斜体* 或 _斜体_
**粗体** 或 __粗体__
***加粗斜体***
~~删除线~~
++下划线++（又是CSDN不支持···）
==背景高亮==


超链接、页内链接、自动链接、注脚

欢迎阅读 [Lapland Stark](https://blog.csdn.net/weixin_45494811)


我经常去的几个网站[Google][1]、[Leanote][2]。

[1]:http://www.google.com
[2]:http://www.leanote.com

无序列表
* 无序列表项 一
+ 无序列表项 二
- 无序列表项 三
有序列表
1. 有序列表项 一
2. 有序列表项 二
3. 有序列表项 三
   

定义型列表
Markdown
:   轻量级文本标记语言（左侧有一个可见的冒号和四个不可见的空格）

插入图像

<center>  <!--开始居中对齐-->

![Lapland](https://i-blog.csdnimg.cn/blog_migrate/0ef591420251b009b3ce5151928bb742.png "Lapland")
</center> <!--结束居中对齐-->

转义字符、字体、字号、颜色

<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=12 face="黑体">黑体</font>
<font color=gray size=5>gray</font>
<font color=#00ffff size=3>null</font>

代码块

C语言里的函数 `scanf()` 怎么使用？

表示整行公式:
$$
E = m c^2
$$


分隔线

* * *
***
*****
- - -
-----------

HTML 原始码
第一个例子：
<div class="footer">
© 2004 Foo Corporation
</div>
第二个例子：
<center>

<table>
<tr>
<th rowspan="2">值班人员</th>
<th>星期一</th>
<th>星期二</th>
<th>星期三</th>
</tr>
<tr>
<td>李强</td>
<td>张明</td>
<td>王平</td>
</tr>
</table>

</center>

表格创建

| 姓名 | 年龄 |
| ------ | ------ |
| 小明 | 17 |

左对齐 :-----
| 姓名 | 年龄 |
| :------ | :------ |
| 小明 | 17 |

右对齐 -----:
| 姓名 | 年龄 |
| ------: | ------: |
| 小明 | 17 |

居中 :-----:
| 姓名 | 年龄 |
| :------: | :------: |
| 小明 | 17 |

可以使用Html注释 <!—这是注释---->
<strong>加粗标签 <em>斜体标签 <br>换行标签 <hr>分割线标签 <s>删除自标签 <del>删除字标签 <span>标签 <u>下划线标签 <ol>有序序列标签 <无序序列标签>