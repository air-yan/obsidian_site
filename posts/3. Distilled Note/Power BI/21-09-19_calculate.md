---
title: Calculate深度理解
author: air.yan
published: true
date: 2021-09-19 17:41:57
updated:
index_img: /img/index_img/calculate_img.png
category: 
  - Dax基石
tags:
typora-root-url: ..\..\public
---

> 快速，正确的理解Calculate函数，应用到你的项目中。

<!--more-->

# Calculate

![calculate](/img/calculate/calculate.png)

今天我们来研究一下Calculate函数。有人说Calculate函数是万能的，只要我学会了Calculate，遇事不决就Calculate就完了。不管这个函数好用不好用，怎么用，它存在的本身揭示了Dax引擎的核心理念：

> 在Dax引擎中，或者说是在Power BI页面中，所有的计算都有且只有两步：先筛选，后计算。

计算我们暂且不说，因为我相信有机会读这篇文章的同学们，小学数学都没有问题，而BI中用到的计算90%都是小学数学。所以呢，这篇文章先谈理解，谈谈这个“先筛选”是什么意思，以及如何改变筛选，之后再谈一些Calculate相关的高阶知识。

为了通俗语言便于理解，本文中所有的“度量值计算（Measure Evaluation）”简称为“计算”，“筛选上下文（Filter Context）”简称为筛选。当然，在必要时我会引用全称。

## 一、先筛选后计算

Power BI中，所有的计算都是**先筛选后计算**。我们以下面两张图为例来讲解，图1标注了度量值，图2标注了筛选上下文。

![图1 - 标注度量值](/img/calculate/measures.png)

![图2 - 标注筛选上下文](/img/calculate/filter_context_example1.png)

在上图的例子中，我们可以看到最上方有一排切片器，左边有一堆卡片图，右边一个表格，构成非常简单。稍微值得注意的是，右方表格中，行里的项目名称是实体表中的一列数据，而表格中的TCV、Milestone进度等，是由不同的度量值组成的。

### 可视化与计算项

左侧指标图中，每一个可视化中的指标就是一次计算，一共10个可视化10次计算。右侧表格中，每一个"单元格"都是一次计算，一共有1个可视化 12 x 6 = 72 次计算。这11个可视化以及72个计算项就是我们要分析的目标。

接下来，让我们看看下图中**可视化**与**计算项**所对应的筛选分别是什么。请注意我上一句的说法，在上述BI页面中，我们要研究的对象有两类，一类是可视化，另一类是可视化中的计算项。

### 筛选上下文（Filter Context）

上图切片器中，其中有两个被选择了，它们分别是`① 产品类别 = “PS”`与`② 金额单位 = “百万”`。这两个筛选（图中标注的①和③）被应用到了BI页面中的所有可视化上，这是一目了然的事情。为了确认，你还可以用你的鼠标移动到任意一个可视化图形上，右上角会出现一个漏斗式图标，这个图标会清晰的告诉你，这个可视化目前所收到的筛选有且只有两个，就是产品类别与金额单位。

此处请划重点，对于可视化来说，它们的筛选上下文（Filter Context）就是漏斗图标中的筛选。

左侧的可视化非常简单，受到外部筛选上下文的影响后，它们只进行一次计算，返回一个结果，因此我可以拍着胸脯告诉你，左侧10个指标图所受到的筛选上下文，就是漏斗图标中显示的筛选上下文。

为了再次检验，同时为了让大家一目了然的看到后台代码，我用Dax Studio将左上角的指标图代码扒了出来。在下面代码中可见，在`Define`结构下面有俩个`VAR`变量，分别是我们刚刚确认的金额单位与项目。

指标图代码：

> <span class="Comment" style="color:#39a03b">//DAXQuery</span><br><span class="Keyword" style="color:#035aca">DEFINE</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">__DS0FilterTable</span>=<br><span class="indent8"></span><span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080">(</span>{<span class="StringLiteral" style="color:#D93124">"百万"</span>},'Para_金额单位'[金额单位]<span class="Parenthesis" style="color:#808080">)</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">__DS0FilterTable2</span>=<br><span class="indent8"></span><span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080">(</span>{<span class="StringLiteral" style="color:#D93124">"PS"</span>},'0Dim_项目基础-PM'[Productline]<span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">EVALUATE</span><br><span class="Keyword" style="color:#035aca">SUMMARIZECOLUMNS</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent4"></span><span class="Variable" style="color:#49b0af">__DS0FilterTable</span>,<br><span class="indent4"></span><span class="Variable" style="color:#49b0af">__DS0FilterTable2</span>,<br><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"TCV"</span>,<span class="Keyword" style="color:#035aca">IGNORE</span><span class="Parenthesis" style="color:#808080">(</span>'0收入成本毛利'[TCV]<span class="Parenthesis" style="color:#808080">)</span><br><span class="Parenthesis" style="color:#808080">)</span><br>

代码跑出的查询结果：

| TCV    |
| ------ |
| 429.84 |

读到这里你会发现：**所有可视化数据的背后全是查询，并且全是一维表！**

### 行上下文（Row Context）

而右侧的表格就比较有意思了，与左侧不同，它拥有一个全新的元素--项目名称。

通常来说，可视化内部除了度量值计算的数字以外，会有一些行、列、图例等其他元素来丰富这个可视化，这些行列等信息是来自模型实体表中的字段。

在后台跑出的查询表中，这些元素以行的形式存在（参考下方查询结果），它们所创造出来的这些行也被称为**行上下文**，作用有二：

1. 为每一个项目名称创建了一行位置
2. 在这些行中，每次计算都**可以**受到项目名称的筛选（关于如何控制筛选，涉及到上下文转换，见下文）

表格代码：

> <span class="Comment" style="color:#39a03b">//DAXQuery</span><br><span class="Keyword" style="color:#035aca">DEFINE</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">__DS0FilterTable</span>=<br><span class="indent8"></span><span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080">(</span>{<span class="StringLiteral" style="color:#D93124">"百万"</span>},'Para_金额单位'[金额单位]<span class="Parenthesis" style="color:#808080">)</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">__DS0FilterTable2</span>=<br><span class="indent8"></span><span class="Keyword" style="color:#035aca">TREATAS</span><span class="Parenthesis" style="color:#808080">(</span>{<span class="StringLiteral" style="color:#D93124">"PS"</span>},'0Dim_项目基础-PM'[Productline]<span class="Parenthesis" style="color:#808080">)</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">__DS0Core</span>=<br><span class="indent8"></span><span class="Keyword" style="color:#035aca">SUMMARIZECOLUMNS</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="indent4"></span><span class="Keyword" style="color:#035aca">ROLLUPADDISSUBTOTAL</span><span class="Parenthesis" style="color:#808080">(</span>'0Dim_项目基础-PM'[项目名称],<span class="StringLiteral" style="color:#D93124">"IsGrandTotalRowTotal"</span><span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="indent4"></span><span class="Variable" style="color:#49b0af">__DS0FilterTable</span>,<br><span class="indent8"></span><span class="indent4"></span><span class="Variable" style="color:#49b0af">__DS0FilterTable2</span>,<br><span class="indent8"></span><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"TCV"</span>,'0收入成本毛利'[TCV],<br><span class="indent8"></span><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"Milestone进度"</span>,'2项目进度'[Milestone进度],<br><span class="indent8"></span><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"收入确认进度"</span>,'2项目进度'[收入确认进度],<br><span class="indent8"></span><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"开票进度"</span>,'2项目进度'[开票进度],<br><span class="indent8"></span><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"回款进度"</span>,'2项目进度'[回款进度],<br><span class="indent8"></span><span class="indent4"></span><span class="StringLiteral" style="color:#D93124">"付款进度"</span>,'2项目进度'[付款进度]<br><span class="indent8"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">EVALUATE</span><br><span class="Variable" style="color:#49b0af">__DS0Core</span><br>

代码跑出的查询结果：

| 项目名称 | IsGrandTotalRowTotal | TCV    | Milestone进度 | 收入确认进度 | 开票进度 | 回款进度 | 付款进度 |
| -------- | -------------------- | ------ | ------------- | ------------ | -------- | -------- | -------- |
|          | True                 | 429.84 | 65%           | 27%          | 65%      | 38%      | 40%      |
| x        | False                | 117.17 | 83%           | 74%          | 84%      | 60%      | 49%      |
| y        | False                | 91.86  | 74%           |              | 74%      | 46%      |          |
| z        | False                | 51.82  | 33%           | 31%          | 31%      |          |          |
| a        | False                | 30.76  | 67%           |              | 67%      |          |          |
| b        | False                | 24.19  |               |              |          | 100%     |          |
| c        | False                | 22.66  | 17%           | 8%           | 17%      | 8%       | 2%       |
| d        | False                | 21.02  |               |              | 100%     | 32%      | 29%      |

通过上面的代码与结果，我们可以看到：

* 两个筛选上下文（单位百万与项目PS）依然存在
* 具体计算出的结果不再是单个一个值，而是一个一维表，因为：
  * SummarizeColumns函数中的第一个参数，提供了一个行上下文 -- 项目名称
  * SummarizeColumns函数中的第一个参数，提供了明细与总计的两种情况（IsGrandTotalRowTotal = true or false的两种情况）
  * 除了TCV以外，还有很多其他度量值，这些度量值各个独自作为一列存在



### 上下文转换（Context Transition）

上文我们说道，在行上下文中，我们可以选择计算是否受到行上下文中信息的筛选，那么如何选择呢？这就涉及到上下文转换的机制了：

* 当行上下文中的表达式为**度量值**时，转换生效 √
* 当行上下文中的表达式为**非度量值**时：
  * 所有**Calculate**及**CalculateTable**函数中的表达式，转换生效 √
  * 其他情况，不生效 x

另一种理解方式是，Calculate可以转换行上下文为筛选上下文，而度量值外有一层隐形的Calculate，因此这两种情况都能转换。

上述代码中，在行上下文中的代码很明显是度量值，度量值自带Calculate，转换行上下文为筛选上下文，因此受到筛选。

由于可视化中的计算必须填写度量值，因此我们可以断定，所有可视化中的行上下文都转换成了筛选上下文。只有在你自己写的代码中，才会出现行上下文未转换成筛选上下文的情况。

### 结论

让我们回到案例，有了上述的讨论与证明，我们现在可以澄清什么是“先筛选，后计算”。结论如下：

**先筛选：**

* 对于左侧的每一个指标图，以及右侧表格中的总计里的计算项，他们受到的筛选有二：
  * 外部筛选上下文：单位 = 百万
  * 外部筛选上下文：项目 = PS

* 对于右侧表格中除总计外的每一个计算项，他们受到的筛选有三：
  * 外部筛选上下文：单位 = 百万
  * 外部筛选上下文：项目 = PS
  * 行上下文转换的筛选上下文：项目名称 = xxx

**再计算：**

* 度量值是什么就计算什么

本文上述所有证明，都是以度量值为最小计算单位来证明，什么是先筛选后计算。当你再去往度量值里一层一层看时，你会看到不同的代码部分会有不同的筛选上下文，你会看到行上下文嵌套行上下文。但我们的评估方式不变 -- 先定位你的计算，再评估外部筛选上下文，再评估行上下文是否转换了筛选上下文。



## 二、所有的筛选都是表

行文至此，讲解的全部都是理解方面。此章节要让大家了解的是Calculate的第二个参数，因此，有必要回溯一下Calculate函数的官方文档。让一起看一下如何正确使用Calculate。

首先，Calculate的语法如下：

>  CALCULATE ( Expression, Filter, Filter, …)

| PARAMETER  | ATTRIBUTES          | DESCRIPTION                                                  | 理解                                                         |
| :--------- | :------------------ | :----------------------------------------------------------- | ------------------------------------------------------------ |
| Expression |                     | The expression to be evaluated.                              | 度量值，或汇总函数等                                         |
| Filter     | Optional Repeatable | A boolean (True/False) expression or a table expression that defines a filter. | 两种写法，分别是缩写及全写。参数2至N都是筛选，可写可不写，并列的筛选条件都是AND关系 |

接下来，让我们逐一了解一下这两种写法。

### Calculate缩写形式

> 缩写形式=<br><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">(</span>'Fact'[Column]<span class="Parenthesis" style="color:#808080">)</span>,'Dimension'[Column]=<span class="StringLiteral" style="color:#D93124">"SomeValue"</span><span class="Parenthesis" style="color:#808080">)</span><br>

缩写形式在技术上来讲，基本没什么好说的，但偏偏有很多新手对“筛选”这个逻辑有一些误解，所以在这里我也解释一下。

筛选几要素：

* 首先，缩写里的筛选条件，每一个参数位**只接受一个字段的筛选**，所以就别费劲想多个字段了……
* 然而，筛选毕竟是对字段的筛选，你要**确保在表达式中有该字段的非计算形式存在**，比如上述的 `'Dimension'[Column]`
* 表达式**返回的必须是一列布尔值**，也就是一列true或false值。为了达到这个效果，你需要做对比条件，比如 "="，">"，"<"，in，或任何能得到布尔值的表达式

如果说，上述几要素记起来比较困难，我们可以选择继续看看全写形式是如何书写的。

### Calculate全写形式

> 全写形式=<br><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">(</span>'Fact'[Column]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'Dimension'[Column]<span class="Parenthesis" style="color:#808080">)</span>,'Dimension'[Column]=<span class="StringLiteral" style="color:#D93124">"SomeValue"</span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Parenthesis" style="color:#808080">)</span><br>

上述结果为全写形式与简写形式的固定转换，转换过程有二：

* 将被筛选的字段放进Filter的第一个参数，套上All函数
* 将布尔表达式原方不动的放入Filter的第二个参数

此处我们发现，除了Calculate函数，我们又涉及到了Filter函数及All函数，让我们看一下简单介绍一下这两个函数。

#### All函数

[All函数](https://dax.guide/all/)的语法如下，返回的结果是一个表：

> ALL ( TableNameOrColumnName , ColumnName , ColumnName… )



| PARAMETER             | ATTRIBUTES          | DESCRIPTION                                                  |
| :-------------------- | :------------------ | :----------------------------------------------------------- |
| TableNameOrColumnName | Optional            | The name of an existing table or column.                     |
| ColumnName            | Optional Repeatable | A column in the same base table. The column can be specified in optional parameters only when a column is used in the first argument, too. |

ALL函数忽略了当前筛选上下文中被指定的那一列的筛选，返回一列数据或一张表。这里我们用到的是返回列数据的方式。

#### Filter函数

[Filter函数](https://dax.guide/filter/)的语法如下，返回的结果也是一个表：

> FILTER ( Table, FilterExpression )



| PARAMETER                          | DESCRIPTION                                                  |
| :--------------------------------- | :----------------------------------------------------------- |
| Table **[ITERATOR ]**              | The table to be filtered.                                    |
| FilterExpression **[ROW CONTEXT]** | A boolean (True/False) expression that is to be evaluated for each row of the table. |

Filter函数功能就是筛选，在第一个参数里我们提供了一列数据或一张表，然后在第二个参数里填写布尔表达式进行筛选。

同时，如果第一个参数里填写的是All函数类，那么被掉All的列、表将不受原本的筛选上下文影响。

### 结论

Calculate的简写方式先**忽略被筛选的字段的筛选上下文**，再对该字段**赋予新的筛选条件**。至此，我们知道了为什么Calculate被称为Context Modifier。

Calculate的**筛选条件永远都是表**，我们除了可以放标准的Filter、All组成的表以外，还可以放入一些奇奇怪怪的表比如用TopN函数筛选前几，用Values函数取到的数据来替换还原筛选上下文等等。

## 三、书写原则

新手书写原则：

* 尽量用一列数据作为筛选，因为性能好
* Calculate不缩写
* Calculate第二个参数用全写的方式，放入变量中

> 推荐全写形式=<br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_filter_table</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'Dimension'[Column]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span>'Dimension'[Column]=<span class="StringLiteral" style="color:#D93124">"SomeValue"</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">(</span>'Fact'[Column]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="Variable" style="color:#49b0af">_filter_table</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br>



## 四、死活题

这是本站第一道死活题，所以本次的题超级简单~ 死活题顾名思义，首先它是一道题，对这道题做出的解答只有两种结果，非死即活。

本次题目考的是特定情景下，计算某度量值的写法，写出并生效即是活，无效即是死。

在之后的出题中，会逐渐上升到一些高级情景，比如给定一部分需求完成模块，或整套项目的模拟死活题等。对于这类题目，会有一个综合评判，需求方满意则为活，需求方不满则为死。

### 主营业务收入

**难度：**✳

| 已知                               | 为                     |
| ---------------------------------- | ---------------------- |
| 报表项维度及字段（已与事实表相连） | 'D_报表项'[报表项一级] |
| 被筛选名称                         | "主营业务收入"         |
| 事实表加总字段                     | 'F_序时账'[借正贷负]   |

求主营业务收入度量值。

<details>   
    <summary>点击显示答案</summary>   
    <pre>
主营业务收入=<br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_filter_table</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'D_报表项'[报表项一级]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="Comment" style="color:#39a03b">--此处可All可Values，看情况而定</span><br><span class="indent8"></span>'D_报表项'[报表项一级]=<span class="StringLiteral" style="color:#D93124">"主营业务收入"</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">(</span>'F_序时账'[借正贷负]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="Variable" style="color:#49b0af">_filter_table</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br>
    </pre> 
</details>

### 去年同期（不用时间智能函数）

**难度：**✳

| 已知                               | 为                                 |
| ---------------------------------- | ---------------------------------- |
| 日期维度                           | 'D_date'[Date], 'D_date'[Year] ... |
| 被调整筛选的度量值                 | [主营业务收入]                     |
| 页面布局，筛选上下文中的日期筛选器 | 年份单选，月份单选                 |

求自定义去年同期度量值。

<details>   
    <summary>点击显示答案</summary>   
    <pre>
主营业务收入.同期=<br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_selected_year</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">(</span>'D_date'[Year]<span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_filter_table</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'D_date'[Year]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span>'D_date'[Year]=<span class="Variable" style="color:#49b0af">_selected_year</span>-<span class="Number" style="color:#EE7F18">1</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span>[主营业务收入],<br><span class="indent8"></span><span class="Variable" style="color:#49b0af">_filter_table</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br>
    </pre> 
</details>

### 客户最后购买金额、上次购买金额（不用时间智能函数）

**难度：**✡✡

假设有事实表'F_客户购买'，字段如下，且每个客户每天只有一条聚合过的购买记录。

| 购买日期 | 客户 | 金额 |
| -------- | ---- | ---- |
| 2021/9/1 | A    | 1000 |
| 2021/8/4 | A    | 600  |
| 2021/8/3 | A    | 400  |
| 2021/8/5 | B    | 2000 |
| 2021/3/5 | B    | 4000 |

假设可视化中，体现的结果如下

| 字段：事实表客户 | 度量值：最后购买金额 | 度量值：上次购买金额（除最后一次） |
| ---------------- | -------------------- | ---------------------------------- |
| A                | 1000                 | 600                                |
| B                | 2000                 | 4000                               |

求最后购买金额、上次购买金额度量值。

<details>   
    <summary>点击显示最后购买金额答案</summary>   
    <pre>
最后购买金额=<br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_last_date</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[购买日期]<span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_filter_table</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[购买日期]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="Comment" style="color:#39a03b">--此处可All可Values，看情况而定</span><br><span class="indent8"></span>'F_客户购买'[购买日期]=<span class="Variable" style="color:#49b0af">_last_date</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[金额]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="Variable" style="color:#49b0af">_filter_table</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br>
    </pre> 
</details>

<details>   
    <summary>点击显示上次购买金额答案</summary>   
    <pre>
上次购买金额=<br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_last_date</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">MAX</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[购买日期]<span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_temp_table</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[购买日期]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span>'F_客户购买'[购买日期]&lt;&gt;<span class="Variable" style="color:#49b0af">_last_date</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_last_date1</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">MAXX</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Variable" style="color:#49b0af">_temp_table</span>,<br><span class="indent8"></span>[购买日期]<br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">VAR</span><span class="Variable" style="color:#49b0af">_filter_table</span>=<br><span class="indent4"></span><span class="Keyword" style="color:#035aca">FILTER</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">ALL</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[购买日期]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span>'F_客户购买'[购买日期]=<span class="Variable" style="color:#49b0af">_last_date1</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br><span class="Keyword" style="color:#035aca">RETURN</span><br><span class="indent4"></span><span class="Keyword" style="color:#035aca">CALCULATE</span><span class="Parenthesis" style="color:#808080">(</span><br><span class="indent8"></span><span class="Keyword" style="color:#035aca">SUM</span><span class="Parenthesis" style="color:#808080">(</span>'F_客户购买'[金额]<span class="Parenthesis" style="color:#808080">)</span>,<br><span class="indent8"></span><span class="Variable" style="color:#49b0af">_filter_table</span><br><span class="indent4"></span><span class="Parenthesis" style="color:#808080">)</span><br>
    </pre> 
</details>
