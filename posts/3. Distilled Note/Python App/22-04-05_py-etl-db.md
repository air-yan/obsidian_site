---
title: 搭建Python与MySQL环境
author: air.yan
published: true
date: 2021-04-05 08:32:00
updated:
index_img: /img/index_img/py_etl_db.png
category: 
  - Pythonic
tags:
typora-root-url: ..\..\public

---

> 用免费的方式，从底层搭建BI架构，Python ETL与MySQL数据库。这是第一步，环境安装。

<!--more-->

# 搭建Python与MySQL环境



## 一、Python环境搭建

网上有很多类似攻略，也可自行查询。



### 1. Anaconda -- Python的集成环境安装包

由于我们做的项目主要是数据分析类，所以我们用到的Python集成软件包是**Anaconda**（https://www.anaconda.com/）。下载后安装即可。



其中我们主要会用到的是一款命令行工具 -- Conda 或者说 Anaconda Prompt。这款工具帮助我们的主要方式，是用来建立独立的虚拟环境（Virtual Environment），并在这个独立的python虚拟环境中安装不同的依赖包（Package）。





打开conda后，我第一件事情就是，将pip与conda的网络镜像设定为国内清华站点，这样下载速度就会起飞。

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```



第二件事，创建自己的环境，可以参考下面网站。

Documentation：https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html

Cheatsheet：https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf

注意，在创建环境时，先不要选择Python3.9，用3.8目前比较稳定（是为了后续与SSAS内核连接做的考虑，其中的依赖包不支持3.9版本）。



第三件事，用Conda打开你的虚拟环境，安装依赖包：

ETL神器Pandas

```
pip install pandas
```

MySql Connector pymysql
```
pip install pymysql
```



### 2. Pycharm 代码编辑器

安装**Pycharm Community**环境，这款代码编辑器可以帮助你更好的编写python。

https://www.jetbrains.com/pycharm/download/#section=windows



注意，执行代码时，右下角的环境要选择为上面的虚拟环境。



## 二、MySQL环境搭建

这个攻略就更多了，也可以自行查询。我居然看了个印度口音的攻略，不习惯的请出门右转bilibili……

https://www.youtube.com/watch?v=6dC0xjdIPZ0



### 1. MySQL 开发环境安装

打开官网，点击[MySQL Installer for Windows](https://dev.mysql.com/downloads/windows/)，下载后一步步安装即可。其中涉及到各种依赖包可以依次安装。



### 2. Navicat 数据库操作工具安装

买不起还不能免费试用吗？点击下面的link直接官网下载，过期了再想办法重置试用期吧。

https://www.navicat.com/en/



## 三、性能优化

为什么我把性能优化放在这里？因为我懒得再开一个post

https://towardsdatascience.com/read-excel-files-with-python-1000x-faster-407d07ad0ed8

总结：

* excel巨慢，最好不要保存excel‘
* csv稍快，让用户导出csv文件可以让速度提升近10倍
* 并行读取 -- 用DASK
* Pickle file最快，但最好只应用于没有用户要看的情况，比如历史数据和中间过程文件



## 四、代码说明

