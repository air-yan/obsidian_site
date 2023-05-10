---
title: 动态趋势
author: air.yan
typora-root-url: ..\..\public
date: 2021-08-16 22:55:47
updated:
index_img: /img/index_img/dynamic_trend.bmp
category: 
  - 案例分析
tags:
---

> 趋势分析是报告必备的分析方式，此文将对最常用的动态趋势分析方法进行总结。

<!--more-->	

# 趋势分析

在Dax实现中，趋势分析考验的知识点有且只有一个：筛选上下文转换。因此，真正理解了上下文的同学们，收到类似这种需求可以举一反三，随时写出答案；尚未理解的同学们，可以参考此篇文章的思路，摸索上下文的转换逻辑。

在财务分析场景中，我们最常见的有月度分析、年度分析、自定义周分析、日分析等等。

月度分析最具有典型性，我们就来总结一下月度分析相关的动态趋势图的使用方法。

## 一、简介

为了测试，我在模型中建了一份简单的日期表，代码如下：

> D_date =<br><span class="Keyword" style="color:#035aca">ADDCOLUMNS</span><span class="Parenthesis" style="color:#808080"> (</span><br><span class="indent4">    </span><span class="Keyword" style="color:#035aca">CALENDAR</span><span class="Parenthesis" style="color:#808080"> (</span> <span class="StringLiteral" style="color:#D93124">"2020/1/1"</span>, <span class="StringLiteral" style="color:#D93124">"2024/12/31"</span> <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">    </span><span class="StringLiteral" style="color:#D93124">"Year"</span>, <span class="Keyword" style="color:#035aca">YEAR</span><span class="Parenthesis" style="color:#808080"> (</span> [Date] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">    </span><span class="StringLiteral" style="color:#D93124">"Month_number"</span>, <span class="Keyword" style="color:#035aca">MONTH</span><span class="Parenthesis" style="color:#808080"> (</span> [Date] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">    </span><span class="StringLiteral" style="color:#D93124">"Month"</span>, <span class="StringLiteral" style="color:#D93124">"M"</span> &amp; <span class="Keyword" style="color:#035aca">MONTH</span><span class="Parenthesis" style="color:#808080"> (</span> [Date] <span class="Parenthesis" style="color:#808080">)</span><br><span class="Parenthesis" style="color:#808080">)</span><br>

在模型中，我引入了一张Excel表，里面包含Date与Value两列，数据非常简单，可自行脑补一下。

报告中用上述日期表，制作了：

* 年份切片器，单选

* 月份切片器，单选

在月度分析报告中，报告中的筛选上下文为单一年、单一月份，整体报告呈现的都是该月份相关的数据截面，而**趋势图除外**，这也是我们需要对趋势图做特殊处理的原因。

在此篇文章中，介绍最常用的三种对趋势图的处理方式：

1. 取消月份交互
2. 动态显示年度至今的月份
3. 动态显示近N月月份


## 二、显示方式

### 1. 简易方法-取消联动

最最最最简单的方法，就是将趋势图与报告月份的切片器`取消联动`。这样的话无论月份切片器选择了什么，该趋势图都将呈现当年整体的趋势状态。

但该方法也有一些令人不舒服的地方。用户在切换月份时，看不到趋势图任何的变动，就算用户知道这是因为取消联动而导致的结果，也不容易直接定位到当前筛选月份在趋势图上的点在哪里。

非常不推荐此方法，除非时间紧张。

### 2. 动态显示年初至今的月份

对于财务来讲，完整年份的分析会更有参考价值，因此在月度分析时，我们经常选择从当年一月份开始进行观测。如果用户对上一年的信息感兴趣的话，我们会建立`同期度量值`，用来与今年的数字进行对比。

因此，年初至今的动态显示方式，在大多数情况下都非常合适。

对于YTD的显示方式，我们的目标是：当`月份切片器`单选了一个月份时，`趋势图`显示当年年初月份至选择月份的值。

#### 思考的陷阱

显而易见，我们最可能想到的处理方式是：

* 首先，我们先得到`外部筛选上下文`月份的数字。（你问我如何得到？用 `Allselected` 忽略可视化内部筛选器即可。用内、外来形容`Allselected`不是很严谨。更好的形容方式应该是 -- `Allselected`会停止当前行上下文中的上下文转换）

* 得到外部筛选月份之后，将其存入一个`变量`中。然后我们判断趋势图中，x轴的每一个点是否 <= 外部筛选中选取的月份，如果是则返回数值，否则不返回。

上述解决方案看似可行，但在第二点中我们做了一个**隐形的假设**：

**在年月切片器存在，且交互行为不被取消的情况下，我们假设趋势图的x轴依然能有1~12月份的存在。**

然而这个假设显而易见是错误的。由于外部筛选器与X轴都来自于一张表，因此一旦接受了年月份的筛选，可视化的行、列本身就已经被筛选了，更不用提度量值。

综上可得知：

1. 当需要得到外部日期筛选时，我们可以用`Allselected`忽略内部日期筛选上下文，找到外部日期筛选上下文。
2. 当外部筛选上下文直接控制了可视化的行或列时，在不动模型的前提下，除了取消交互以外没有任何办法。

哎，想到这里，少侠只能再一次另辟蹊径咯~

#### 解决方案

这里我们需要引入`第二张日期表`。第二章日期表的引入是非常常见的，不仅仅因为日期维度是大家都会分析的维度，还因为单一的日期表在模型上无法满足我们的需求。（不仅日期表，其他维度有时也会复制出第二个表作为辅助，比如多家公司对标）

我们的做法是：

1. 将日期表的代码原方不动的复制出来，制作出第二张日期表，并换一个命名。
2. 将原日期表作为常规页面筛选器
3. 将新日期表作为趋势图x轴
4. 建立以下度量值

> trend_showYTD =<br><span class="Keyword" style="color:#035aca">VAR</span> <span class="Variable" style="color:#49b0af">_slicer_month</span> =<br><span class="indent4">    </span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date'[Month_number] <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4">    </span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080"> (</span><br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080"> (</span> 'Fact'[value] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">KEEPFILTERS</span><span class="Parenthesis" style="color:#808080"> (</span> <span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080"> (</span> <span class="Keyword" style="color:#035aca">VALUES</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date_axis'[Date] <span class="Parenthesis" style="color:#808080">)</span>, 'D_date'[Date] <span class="Parenthesis" style="color:#808080">)</span> <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date'[Month] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date'[Month_number] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span>'D_date'[Month_number]  <= <span class="Variable" style="color:#49b0af">_slicer_month</span><br><span class="indent4">    </span><span class="Parenthesis" style="color:#808080">)</span><br>

上述度量值的作用是：

1. 计算Fact表中的value加总
2. 用`TreatAs`函数，将新日期表当做老日期表用（`Keepfilters`的作用是将`TreatAs`中的`All`函数去掉）
3. `All`掉老日期表里的月份、月份排序所带来的筛选
4. 并将月份的筛选替换为，小于等于当前选择月

此刻应有思考过程……



### 3. 动态显示近N月

动态显示近N月的处理方式与上述非常类似，因此思考过程不再赘述。

首先，我们要有`第二个日期表`，上面已经建好了。

之后，我们建立以下度量值：

> trend_showRecentN =<br><span class="Keyword" style="color:#035aca">VAR</span> <span class="Variable" style="color:#49b0af">_end_date</span> =<br><span class="indent4">    </span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date'[Date] <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">VAR</span> <span class="Variable" style="color:#49b0af">_start_date</span> =<br><span class="indent4">    </span><span class="Keyword" style="color:#035aca">EDATE</span><span class="Parenthesis" style="color:#808080"> (</span> <span class="Keyword" style="color:#035aca">MIN</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date'[Date] <span class="Parenthesis" style="color:#808080">)</span>, <span class="Number" style="color:#EE7F18">-11</span> <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4">    </span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080"> (</span><br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080"> (</span> 'Fact'[value] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080"> (</span> <span class="Keyword" style="color:#035aca">VALUES</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date_axis'[Date] <span class="Parenthesis" style="color:#808080">)</span>, 'D_date'[Date] <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date' <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">        </span><span class="Keyword" style="color:#035aca">DATESBETWEEN</span><span class="Parenthesis" style="color:#808080"> (</span> 'D_date'[Date], <span class="Variable" style="color:#49b0af">_start_date</span>, <span class="Variable" style="color:#49b0af">_end_date</span> <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent4">    </span><span class="Parenthesis" style="color:#808080">)</span><br>

该度量值的作用是：

1. 先在变量中计算出，近N月的开始日和结束日
2. 计算Fact中Value的加总
3. 用`TreatAs`函数，将新日期表视为老日期表（将x轴上的日期筛选当做原日期筛选）
4. `All`掉所有原日期表的筛选（忽略外部的年月筛选）
5. 用`DateBetween`函数，作为开始、结束日的筛选

此处需要继续思考片刻~

### 4. 最终效果

<iframe width="100%" height="650" src="https://app.powerbi.com/view?r=eyJrIjoiNDg3NzcxNDMtNTU0Ni00YWFiLTljMzUtN2I5ODkyMmVhMmE5IiwidCI6IjE3OGJkY2RlLTI5N2UtNGMyZi04NTllLTZlOTJlMDdlMDdjYSIsImMiOjEwfQ%3D%3D" frameborder="0" allowFullScreen="true"></iframe>