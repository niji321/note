title: pandas学习记录
author: xg wang
tags:
- pandas
categories:
- python
date: 2018-07-20 22:01:00
---

pandas 可以说是一个很重要的库了，各种对于数据的操作十分方便。而且还有很多配套的开源项目来处理数据，可以看到，数据处理是多么繁琐的一步。。

一些资料：
1. [10 Minutes to pandas](http://pandas.pydata.org/pandas-docs/stable/10min.html)
2. [Cookbook](http://pandas.pydata.org/pandas-docs/stable/cookbook.html) : This is a repository for short and sweet examples and links for useful pandas recipes
3. [Modern pandas](https://github.com/TomAugspurger/effective-pandas) : github
4. [Exercises for new users](https://github.com/guipsamora/pandas_exercises) : github
5. **pandas的文档界面虽然丑，但是已经很完善了。好好的读一下吧。**


pandas 生态系统中有用的库：
1. Featuretools
2. Dask：Dask is a flexible parallel computing library for analytics. Dask provides a familiar DataFrame interface for out-of-core, parallel and distributed computing.
3. Dask-ML ： Dask-ML enables parallel and distributed machine learning using Dask alongside existing machine learning libraries like Scikit-Learn, XGBoost, and TensorFlow.
4. Blaze ： Blaze provides a standard API for doing computations with various in-memory and on-disk backends: NumPy, Pandas, SQLAlchemy, MongoDB, PyTables, PySpark.
5. Odo ： Odo provides a uniform API for moving data between different formats. It uses pandas own read_csv for CSV IO and leverages many existing packages such as PyTables, h5py, and pymongo to move data between non pandas formats. Its graph based approach is also extensible by end users for custom formats that may be too specific for the core of odo.

这里一篇一篇的文章慢慢的看。很有意思。

1. Intro to Data Structures
2. Essential Basic Functionality
3. Working with Text Data
4. Options and Settings
5. Indexing and Selecting Data
6. MultiIndex / Advanced Indexing
7. Computational tools
8. Working with missing data
9. Group By: split-apply-combine
10. Merge, join, and concatenate
11. Reshaping and Pivot Tables
12. Time Series / Date functionality
13. Time Deltas
14. Categorical Data
15. Visualization
16. Styling
17. IO Tools (Text, CSV, HDF5, …)
18. Enhancing Performance
19. Sparse data structures
20. Frequently Asked Questions (FAQ)