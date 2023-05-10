---
title: 混合模型
author: air.yan
typora-root-url: ..\..\public
date: 2021-09-05 20:34:56
updated:
index_img: /img/index_img/mixed_model.jpg
category:
  - 初出茅庐
tags:
published: true
---

混合模型是微软在2020年12月份上线的里程碑式功能，该功能被网友戏称为BI界的"圣杯"，因该功能BI的圣杯之战至此画上了句号，在可预见的未来里没有其他厂······································································································································································································																																DDDDDDDDD	商能与微软抗衡。

<!--more-->

# 混合模型（Composite Model）

<img src="/img/index_img/Composite Model.png" alt="Composite Model" style="zoom:80%;" />



## 一、简介

数据链接方式分为：

* Import
* Direct Query
* Live Connection
* Direct Query for Dataset or Analysis Service

模型的种类分为：

* Import Model
* Direct Query Model
* Live Connection Model
* Composite Model v1.0
* Composite Model v2.0

## 二、历史模型回顾

![Connection-Type-Outline](/img/mixed_model/Connection-Type-Outline.png)

*图片来自：https://powerbi.tips/2017/11/power-bi-connections-import/

### 1. Import Mode

**通用性：**✳✳✳

如果你从来没有了解过这些模式，那么你很可能只用过Import模式，它是PBI默认的数据模式，同时也是功能最强大的一种模式。

你所听过的Power BI各种强大的核心引擎，全部指的都是Import。像`Vertipaq引擎`，`内存计算 In-Memory Computing`，[`列储存模式`](https://zhuanlan.zhihu.com/p/127823207)，`表格模型 Tabular Model`，`OLAP(Online Analytical Processing)`。

所以呢，在可以的情况下，请尽量使用该模式。

注意，Import模式需要将数据copy到PBI中，因此如果要求数据不能上云…… 就只能不用这种模式，或者用Report Server做本地化部署了。

### 2. Direct  Query Mode

**通用性：**✡✡

Direct Query是直连数据库的意思，几乎所有的运算都推给数据库本身进行计算。所有的Dax公式会在推给SQL端时转换成SQL语言，因此优化此模式的性能时也需要查看转换后的SQL语言是否最优。

毋庸置疑，Direct Query模式的计算速度比Import模式要慢，所以只有在以下情况下，才考虑用Direct Query：

* 数据量过大，Import模式变得不太可能
* 维度爆炸，Import模式优化不能
* 近乎实时的数据更新需求
* 其他需要将计算推到数据库的理由（如Embedded成本考虑等等）

### 3. Live Connection Mode

**通用性：**✴

Live Connection模式与上述两种模式略有不同，Import和DQ都是连接数据源后，决定是用Vertipaq引擎承载计算还是用数据库直接计算，而Live Connection与数据源可以说是毫无联系。

此模式直接连接的是Power BI Online Dataset，也就是说，我们需要在Power BI Service中先发布一个完美的数据集，再去连接它。

经常在以下情况下，我们考虑用Live Connection：

* 做某已上线模型的数据验证
* 在已有数据集之上，直接建立新的报告
* 分享完美的数据集给别人，让别人建立新的报告

### 4. Composite Model V1.0（Import + Direct Query）

**通用性：**✳✳✳

在Composite Model出现之前，一个模型中支持的Direct Query数据源仅为一个，且不能有Import类的数据在其中。Composite Model的出现，打破了这一局限。

根据微软官方文档介绍，Composite Model的定义如下：

> A model that combines data from more than one DirectQuery source or that combines DirectQuery with import data is called a composite model.

但像微软官方文档说的一样，不同模式不同源的数据源之间，模型关系是不同的。在Composite Model下，我们需要注意跨数据源或跨模式的关系，因为这些关系很有可能影响性能。

> Any relationships that are cross-source are created with a cardinality of many-to-many, regardless of their actual cardinality.

Composite Model的局限基本上等同于Direct Query的局限。除此之外要注意的是，某些查询可能会跨数据源执行，因此会带来一些性能问题。具体可参考[官方文档](https://docs.microsoft.com/en-us/power-bi/transform-model/desktop-composite-models#performance-implications)。

Composite Model给我们带来了很大的便利，当我们需要做经典的各类分析时，我们用Import建模，同时再用Direct Query连接一个大宽明细表，支持相应维度下的明细查询。这么做的好处是，维度、颗粒度爆炸的那张表没有导入PBI模型，减轻刷新压力，同时又保留Import的分析能力。

例如，导入优化、汇总过的序时账做分析，再直连序时账查明细。

## 三、Composite Model V2.0（DQ for Dataset）

**通用性：**✳✳✳✳✳✳✳✳✳✳（直接爆表）

Composite Model V2.0 是微软在2020年12月份发布的一项功能，该功能被BI业界各种大佬称为BI界的王冠，甚至直接为BI竞争画上了Game Over的句号。各种彩虹屁大家可以自行搜索~

这里是官方文档：https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-directquery-datasets-azure-analysis-services

虽然该功能已出了快小一年，但依然在Preview阶段，可见该功能的复杂程度之高。

功能基本上是没有问题的，这个我已经在项目上实施过了。唯一的一个小坑是关于RLS权限的分配问题，该问题也会在后文提供解决方案。

上面提到，Composite Model V1.0混合了Import模式和Direct Query模式，大大提高了模型的可塑性、易用性；V2.0可不得了，它又加入了Online Dataset的直连模式（Direct Query for Dataset），这时我想到了俄罗斯套娃（但最多可套三层哈哈）。

换句话说，以前我们只能用Live Connection的方式连接一个数据源，现在我们可以直连一个或多个线上数据集，那些线上的数据集本身也可以直连其他数据集（套娃）。同时你在直连这些数据集之后，还可以在本地加入新的数据源，Excel也好数据库也好，Import也好Direct Query也好，都可以……

怎么说呢，Composite Model V2.0像是把所有能实现的功能都一口气发给你了一样，想怎么玩怎么玩。

关于使用方式呢，非常简单所以不讲了，请参考上述官方文档。

### 模型的拓展性直接爆表

比如我们想玩业财融合，先做了一套业务模型，又做了一套财务模型，然后将两个模型合并成Composite 2.0，维度一连，直接数据打通…… 再比如说官方的一种解释，一套通用模型，某人直连后加入了他们部门的预算，形成了预实对比模型 -- 完美~

### 突破本地内存限制，运用online资源制作复杂计算表

用Dax做计算表是某些情况下，需要提升性能时的无奈之举。但计算表如果写的逻辑比较复杂也会爆掉内存，所以该情况也可以通过将度量值写到Dataset里，再做Dax计算表来缓解电脑压力。

但在官方文档中明确指出，该方式暂只能在本地进行，一旦发布到云端则不支持。

> Calculated tables are not supported in the Service using this feature.

### 并行开发可能性

需要同时开发的两个模块，运用Compo 2.0，可以更方便的进行配合。以前需要多人做一个项目时，有可能需要考虑到模型合并的情况，其实还挺烦的。现在可以让两个人分开开发两个不同的模块，最后再用混合模型的方式合并到一起。（当然，有些时候看情况还是要合并到一个模型，不能用混合模型）

### RLS启用方式

对于混合模型来说，如果查看者想要看到报告，那么：

1. 需要分配所有被引用的所有数据集中的`生成 Build`权限，该权限在数据集权限设置中可见
2. 需要将数据集中的`RLS Roles 角色`赋予该用户
3. 需要在工作区中将用户设置成`查看器 viewer`，否则该用户将能看到所有角色的数据

RLS的启用参考了下方链接：

https://www.pbiusergroup.com/communities/community-home/digestviewer/viewthread?GroupId=547&MessageKey=b5c3365b-6a79-46c7-8e12-7676e8c30100&CommunityKey=b35c8468-2fd8-4e1a-8429-322c39fe7110&tab=digestviewer

https://radacad.com/directquery-for-power-bi-dataset-how-does-it-work

https://radacad.com/power-bi-user-access-levels-build-and-edit-are-different

### 计算组限制

在官方文档中，目前列示了Composite Model V2.0关于计算组的一条Limitation：

> Calculation groups on remote sources are not supported, with undefined query results.

根据文档，我们了解到远程模型中的计算组不可用。但本地模型中的计算组对远程模型中的度量值的效果如何呢？

根据我们测试得出：

* 一旦本地模型中的计算组包含任何Format Expression，远程模型的度量值就会**全部爆掉**。是的你没有听错，全部爆掉，不管该计算组有没有被筛选，都爆。对，我说的是所有远程度量值。
* 反之，不包含Format Expression时，不会爆。
* 有意思的是，如果任何计算组被筛选了，那么这个数还是能正确的算出来的，并且不会爆。

解决方案是，建立一个Priority最高的计算组，没有Format Expression，只写一个`SelectedMeasure()`，放到全部页面的筛选中，就可以解决。

我想该限制应该算是一个BUG，会随着PBI的迭代会慢慢被修好。但目前呢，我们只能暂时这么用，就这么卡BUG……

