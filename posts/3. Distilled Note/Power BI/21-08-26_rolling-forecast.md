---
title: 滚动预测
author: air.yan
typora-root-url: ..\..\public
date: 2021-08-26 18:40:01
updated:
index_img: /img/index_img/rolling_forecast.jpg
category:
  - 案例分析
tags:
published: true
---

> 滚动预测是财务经典应用场景之一。在BI中，这个案例考察了BI人员的综合能力，其中较为侧重上下文、变量、参数等应用。相对于 [动态趋势分析] 而言，是在其之上的进阶应用。

<!--more-->

# 滚动预测

![](/img/index_img/rolling_forecast.jpg)

## 一、介绍

之所以滚动预测被称之为“滚动预测”，是因为我们无法完全了解并预测未来的情况，预测的数据需要每过一段时间进行一次更新，预测时间与未来时间的时间间隔越短，预测精准度越高。

我们常见的财务预测方式有：

* 每年进行一次或多次，针对整年中每月的预测。每次预测覆盖替换上次预测的数据。
* 每月进行一次预测，预测后N月的数据。因此每月会有N个预测版本。

本文中将案例简单化，只针对整年预测进行讲解，不考虑预测版本的影响。

**通用性：**✳✳✳

预测是每一个企业都会做的事情，因此滚动预测模型的通用性不言而喻。

## 二、 题目背景

### 1. 数据

首先，我们会有两张事实表，我们这里分别管它们叫 `Fact_act`与`Fact_fcst`，它们分别承载了事实发生的数据及预测发生的数据。

这里将两张事实表进行了简化处理，维度只有日期且每月只有1号有一个值，因此颗粒度为月。两张表的结构如下表所示：

| date     | value    |
| -------- | -------- |
| 2021-1-1 | 201.0498 |
| 2021-2-1 | 171.7505 |
| ...      | ...      |

### 2. 需求

对于用户来讲，我们可以将他们的需求分为两类，**基本需求**与**增值需求**。

#### 基本需求

**难易度：**✡✡

他们的基本需求是，将Actual数据与Forecast数据合理的显示到趋势图中，此处的合理解释为：

* 页面中有典型的财务模型中运用的年、月切片器
* 趋势图Actual部分数据，显示年初到当前月份切片器所选月份
* 趋势图Forecast部分数据，显示当前切片器所选月份到年底月份
* 支持Current（当期）口径与YTD（年度累计）口径的切换

#### 增值需求

**难易度：**✴✴✴

他们的增值需求是，将滚动预测中增加分析功能

* 同期对比
* 提供当前预测口径的what-if调整
* 提供其他的预测口径
  * 同期值 * what-if 参数
  * 按去年权重与今年已有数据预测数据
  * 按去年环比增长率预测数据
* what-if调整
* 等等……

## 三、解题方案

首先，我们将上述事实表导入到数据模型中，形成Act与Fcst两张事实表。

接下来，按惯例进行日期表的建立：

> D_date  =<br><span class="Keyword" style="color:#035aca">ADDCOLUMNS</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">CALENDAR</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="StringLiteral" style="color:#D93124">"2019/1/1"</span>,  <span class="StringLiteral" style="color:#D93124">"2022/12/31"</span>  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">        </span><span class="StringLiteral" style="color:#D93124">"Year"</span>,  <span class="Keyword" style="color:#035aca">YEAR</span><span class="Parenthesis" style="color:#808080">  (</span>  [Date]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">        </span><span class="StringLiteral" style="color:#D93124">"Month"</span>,  <span class="StringLiteral" style="color:#D93124">"M"</span>    &  <span class="Keyword" style="color:#035aca">MONTH</span><span class="Parenthesis" style="color:#808080">  (</span>  [Date]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">        </span><span class="StringLiteral" style="color:#D93124">"Month_no"</span>,  <span class="Keyword" style="color:#035aca">MONTH</span><span class="Parenthesis" style="color:#808080">  (</span>  [Date]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4">        </span><span class="StringLiteral" style="color:#D93124">"yyyy-mm"</span>,  <span class="Keyword" style="color:#035aca">FORMAT</span><span class="Parenthesis" style="color:#808080">  (</span>  [Date],  <span class="StringLiteral" style="color:#D93124">"yyyy-mm"</span>  <span class="Parenthesis" style="color:#808080">)</span><br><span class="Parenthesis" style="color:#808080">)</span><br>

第三步，用上述代码建立一个无模型关系的日期表，用来做趋势图的x轴用。具体原因参见动态趋势一文。

第四步，将日期表1的年月切片器放入报告，将日期表2的yyyy-mm列放入趋势图的x轴。

以上为准备阶段，准备完毕后开始解题。

### 1. 基础需求方案

#### **Act-时间口径切换**

时间口径切换是基础知识，用法是通过新建一张表当做切片器，用Switch来判断切片器所选内容并进行度量值切换。

| Time Patterns | order |
| ------------- | ----- |
| Current       | 1     |
| YTD           | 2     |

基础加总：

> 1.  act_val  =<br><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">  (</span>  'Fact_actual'[value]  <span class="Parenthesis" style="color:#808080">)</span><br>

时间口径切换：

> 2.  act_time_patterns  =<br><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_time</span>  =<br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">SELECTEDVALUE</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_time_patterns'[Time  Patterns]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">SWITCH</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent8">                </span><span class="Variable" style="color:#49b0af">_time</span>,<br><span class="indent8">                </span><span class="StringLiteral" style="color:#D93124">"Current"</span>,  [1.  act_val],<br><span class="indent8">                </span><span class="StringLiteral" style="color:#D93124">"YTD"</span>,  <span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">  (</span>  [1.  act_val],  <span class="Keyword" style="color:#035aca">DATESYTD</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Date]  <span class="Parenthesis" style="color:#808080">)</span>  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent4">        </span><span class="Parenthesis" style="color:#808080">)</span><br>

#### **Act-动态趋势图**

首先我们要做的是，将事实数据的信息，用`动态趋势图`的方式显示到趋势图上。这里由于已经做过该攻略的讲解，因此不再赘述。请参考动态趋势。

> 3.  act_trend_axis  =<br><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_slicer_month</span>  =<br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Month_no]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent8">                </span>[2.  act_time_patterns],<br><span class="indent8">                </span><span class="Comment" style="color:#39a03b">//  视axis为日期表，保留日期筛选</span><br><span class="indent8">                </span><span class="Keyword" style="color:#035aca">KEEPFILTERS</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="Keyword" style="color:#035aca">VALUES</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date_axis'[Date]  <span class="Parenthesis" style="color:#808080">)</span>,  'D_date'[Date]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent8">                </span><span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">                </span><span class="Comment" style="color:#39a03b">//  忽略月份筛选，留年份</span><br><span class="indent8">                </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Month]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">                </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Month_no]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">                </span><span class="Comment" style="color:#39a03b">//  增加月份筛选，小于等于筛选器月份</span><br><span class="indent8">                </span>'D_date'[Month_no]  <=  <span class="Variable" style="color:#49b0af">_slicer_month</span><br><span class="indent4">        </span><span class="Parenthesis" style="color:#808080">)</span><br>

行文至此，事实数据的趋势已经可以上图了。

#### **Fcst-动态趋势**

对于预测数据，二话不说，首先建立底层汇总度量值，肯定没错。

> 1.  fcst_val  =<br><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">  (</span>  'Fact_actual'[value]  <span class="Parenthesis" style="color:#808080">)</span><br>

这里考虑到预测数据的YTD显示方式颇为复杂，逻辑上来讲计算分为两部分：

* 第一部分：年初至切片器月份所发生的Actual事实数据
* 第二部分：切片器月份之后，至x轴目前点所发生的Fcst预测数据

因此该计算并不是简单的YTD函数就能解决的。

我们应该养成的习惯是，将**简单的、可以复用的**逻辑放在底层，将**复杂的特殊逻辑**放在度量值外层来实现。这样既可以保证简单逻辑的复用，又可以保证复杂逻辑的独立性，这样即使你在测试时写的BUG如山一样高（Sh\*t Mountain），也不会影响其他模块……

以下是预测动态趋势的实现方式，与上述大同小异，唯一区别在于 -- 最后一个条件筛选了切片器月份之后的月。

> 2.  fcst_trend_axis  =<br><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_slicer_month</span>  =<br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Month_no]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent8">                </span>[1.  fcst_val],<br><span class="indent8">                </span><span class="Keyword" style="color:#035aca">KEEPFILTERS</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="Keyword" style="color:#035aca">VALUES</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date_axis'[Date]  <span class="Parenthesis" style="color:#808080">)</span>,  'D_date'[Date]  <span class="Parenthesis" style="color:#808080">)</span>  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">                </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Month]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">                </span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Month_no]  <span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8">                </span>'D_date'[Month_no]  >  <span class="Variable" style="color:#49b0af">_slicer_month</span><br><span class="indent4">        </span><span class="Parenthesis" style="color:#808080">)</span><br>

#### Fcst-时间切换

**难易度：**✡✡

滚动预测基础需求中，最困难的部分就是这预测度量值的最终显示部分了，这个度量值考虑了诸多的情况：

* 考虑了时间切换
  * Current当期引用了上述动态度量值，Easy
  * YTD说道可多了
    * _condition 变量，限制了度量值显示的范围（切片器月至年末），这对actual和forecast来说是一举两得的
    * _act_ytd 变量，巧妙引用了actual度量值，无需再做计算
    * _fcst_ytd 变量，简单的根据x轴限制了fcst的汇总范围

以下为该度量值的代码：

> 3.  fcst_time_patterns  =<br><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_time</span>  =<br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">SELECTEDVALUE</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_time_patterns'[Time  Patterns]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4">        </span><span class="Keyword" style="color:#035aca">SWITCH</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent8">                </span><span class="Variable" style="color:#49b0af">_time</span>,<br><span class="indent8">                </span><span class="StringLiteral" style="color:#D93124">"Current"</span>,  [2.  fcst_trend_axis],<br><span class="indent8">                </span><span class="StringLiteral" style="color:#D93124">"YTD"</span>,<br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_slicer_date</span>  =<br><span class="indent8">                </span><span class="indent8">                </span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Date]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_axis_date</span>  =<br><span class="indent8">                </span><span class="indent8">                </span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date_axis'[Date]  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_fcst_ytd</span>  =<br><span class="indent8">                </span><span class="indent8">                </span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">  (</span><br><span class="indent8">                </span><span class="indent8">                </span><span class="indent4">        </span>[1.  fcst_val],<br><span class="indent8">                </span><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">DATESBETWEEN</span><span class="Parenthesis" style="color:#808080">  (</span>  'D_date'[Date],  <span class="Variable" style="color:#49b0af">_slicer_date</span>  +  <span class="Number" style="color:#EE7F18">1</span>,  <span class="Variable" style="color:#49b0af">_axis_date</span>  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent8">                </span><span class="indent8">                </span><span class="Parenthesis" style="color:#808080">)</span><br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_act_ytd</span>  =  [2.  act_time_patterns]<br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">VAR</span>  <span class="Variable" style="color:#49b0af">_condition</span>  =<br><span class="indent8">                </span><span class="indent8">                </span><span class="Keyword" style="color:#035aca">YEAR</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="Variable" style="color:#49b0af">_slicer_date</span>  <span class="Parenthesis" style="color:#808080">)</span>  =  <span class="Keyword" style="color:#035aca">YEAR</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="Variable" style="color:#49b0af">_axis_date</span>  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent8">                </span><span class="indent8">                </span><span class="indent4">        </span>  &  &  <span class="Variable" style="color:#49b0af">_slicer_date</span>  <=  <span class="Variable" style="color:#49b0af">_axis_date</span><br><span class="indent8">                </span><span class="indent4">        </span><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent8">                </span><span class="indent8">                </span><span class="Keyword" style="color:#035aca">IF</span><span class="Parenthesis" style="color:#808080">  (</span>  <span class="Variable" style="color:#49b0af">_condition</span>,  <span class="Variable" style="color:#49b0af">_fcst_ytd</span>  +  <span class="Variable" style="color:#49b0af">_act_ytd</span>  <span class="Parenthesis" style="color:#808080">)</span><br><span class="indent4">        </span><span class="Parenthesis" style="color:#808080">)</span><br>

上述度量值仅运用了`上下文转换`与`IF判断`完成了滚动预测的需求。

看似很难，其实简单。说简单吧，但又都不会做。这种题型就是典型的，会者不难，难者不会。

因此，本案例中的基础需求部分，也是用来考量`筛选上下文`是否理解到位的题型。

### 2. 增值需求方案

增值需求方案题型难，通用性却不是那么高（万一客户不要求呢……）

类似这种给客户分析增值的方案，通用性都不高，因为它们的定位就不是硬需求，而是增值。

该部分由于时间不足，尚未编写，待同学们哪天看不够了我再来更新此部分。抱歉~

<img src="/img/coming_soon.jpg" style="zoom:50%;" />



## 四、Demo

下方为滚动预测的基础版demo，可以选择月份、时间口径来验证上述逻辑是否成功实现。

<iframe width="100%" height="450" src="https://app.powerbi.com/view?r=eyJrIjoiZWFlMmQwOGMtODllMS00NGQ1LWI2MDItNzk0NGNiZmJiMTBhIiwidCI6IjE3OGJkY2RlLTI5N2UtNGMyZi04NTllLTZlOTJlMDdlMDdjYSIsImMiOjEwfQ%3D%3D" frameborder="0" allowFullScreen="true"></iframe>