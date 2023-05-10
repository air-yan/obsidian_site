#python #攻略

## 1. Python
---
[Kaggle数据科学平台 - Python 免费攻略](https://www.kaggle.com/learn/python)：免费快速上手，适合小白，适合喜欢自己研究类型的，适合边学边做的。快速上手，世界最强的数据科学平台。Jupyter Notebook式代码输入。

[B站 - Python 基础知识系统学习 - By Mosh](https://www.bilibili.com/video/BV1ng4y1i7Uk/?spm_id_from=333.337.search-card.all.click&vd_source=bf2e9801960190a12c013603c3ae44e2)：B站免费搬运课程，专业，系统，适合长时间学习。



## 2. Anaconda / PyCharm / Jupyter-Notebook
---
[官网](https://www.anaconda.com/)：Anaconda 官方网站
[官方文档](https://docs.anaconda.com/)：有问题找文档
[B站一站式教程，Anaconda + PyCharm + Python 安装与使用](https://www.bilibili.com/video/BV1K7411c7EL/?spm_id_from=333.337.search-card.all.click&vd_source=bf2e9801960190a12c013603c3ae44e2)：一口气学完所有相关工具。

### 2.1 步骤
由于我们做的项目主要是数据分析类，所以我们用到的Python集成软件包是**Anaconda**（https://www.anaconda.com/）。下载后安装即可。

其中我们主要会用到的是一款命令行工具 -- `Conda` 或者说 `Anaconda Prompt`。这款工具帮助我们的主要方式，是用来建立独立的虚拟环境（Virtual Environment），并在这个独立的python虚拟环境中安装不同的依赖包（Package）。

打开conda后，我第一件事情就是，将pip与conda的网络镜像设定为国内清华站点，这样下载速度就会起飞。

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```



第二件事，创建自己的环境，可以参考下面网站。
[Documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)
[Cheatsheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)


第三件事，用Conda打开你的虚拟环境，安装依赖包：

ETL神器Pandas

```
pip install pandas
pip install pymssql
pip install jupyter
```



## 3. Pandas
---
[Kaggle Pandas 免费攻略](https://www.kaggle.com/learn/pandas)：免费快速上手，适合小白，适合喜欢自己研究类型的，适合边学边做的。快速上手，世界最强的数据科学平台。
[Udemy Pandas专业课](https://www.udemy.com/course/data-analysis-with-pandas/)：包含从A到Z的Pandas学习，其中囊括了Python的基础知识和Anaconda环境的配置与安装。
