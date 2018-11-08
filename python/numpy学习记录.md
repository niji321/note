title: numpy学习记录
author: xg wang
tags:
- numpy
categories:
- python
date: 2018-07-20 22:01:00
---

一些学习文档:
[NumPy User Guide](https://docs.scipy.org/doc/numpy-1.14.1/user/) 简单的介绍。
[NumPy Reference](https://docs.scipy.org/doc/numpy-1.14.1/reference/index.html) 全部API列表。
[100 numpy exercises](https://github.com/rougier/numpy-100),作完了，只能达到初级阶段的使用。更多技巧还需要自己实践中才能提高起来。

### NumPy User Guide：

1. ndarray : numpy最重要的数据结构
    1. ndarray.ndim
    2. ndarray.shape
    3. ndarray.size
    4. ndarray.dtype
    5. ndarray.itemsize


2. narray 生成函数：
    1. array
    2. zeros
    3. zeros_like
    4. one    
    5. ones_like
    6. empty
    7. empty_like
    8. arange
    9. linspace
    10. numpy.random.rand ：random samples from a uniform distribution over [0, 1].
    11. numpy.random.randn ： Return a sample (or samples) from the “standard normal” distribution.
    12. fromfile : tofile
    13. fromfunction : 使用 lambda表达式 np.fromfunction(lambda i, j: i == j, (3, 3), dtype=int)


3. 基础操作： 按照类别放到一起，很简单的就忽略掉吧。
    1. A*B : elementwise product
    2. A.dot(B)
    3. np.dot(A,B)
    4. A+=B : in place
    5. A*=B : in place

    6. **apply_along_axis**(func1d, axis, arr, *args, **kwargs)

    7.  ceil : 每个元素向上取整
    8.  floor : 每个元素向下取整
    9.  clip ： 根据上限下限修建

    10. **cov** : Estimate a covariance matrix, given data and weights
    11. **corrcoef** : Return Pearson product-moment correlation coefficients. X,y
    12. std ： Compute the standard deviation along the specified axis
    13. trace : Return the sum along diagonals of the array
    14. var : varianced
    15. vdot : Return the dot product of two vectors.
    16. **vectorize** : The vectorize function is provided primarily for convenience, not for performance. The implementation is essentially a for loop.

    17. sort ： Return a sorted copy of an array.
    18. argsort : Indirect sort ,return index.
    19. ndarray.sort : in-place sort.
    20. lexsort : Perform an indirect sort using a sequence of keys.根据一个序列的主键来排序。

    21. where : Return elements, either from x or y, depending on condition.
    22. nonzero : Return elements, either from x or y, depending on condition.
    23. **choose** : Construct an array from an index array and a set of arrays to choose from. 用法有点复杂

4. Indexing, Slicing and Iterating
    1. index 返回原来数组的内存。slicing返回一份新的view. view就是原来的内存。但是 y=x[0],对y操作，会改变x？为什么呢？
    1. x[1,2,...] is equivalent to x[1,2,:,:,:],
    2. x[4,...,5,:] to x[4,:,:,5,:].
    3. x[0,2] = x[0][2] 可以用单个数字去单行
    4. 对多维数组使用数组索引，仍然返回原数组内存，可以修改。
    5. numpy.ndenumerate : for index, x in np.ndenumerate(a):print(index, x)
    6. numpy.indices : Return an array representing the indices of a grid. row and col
    7. numpy.mgrid : 和上面的类似，但是返回的是一个nparray数组。
    8. numpy.meshgrid ： 生成一个网格结构，而不只是用来做索引结构。

5. Shape Manipulation:
    1. ravel
    2. faltten
    3. reshape
    4. resize
6. Stacking together different arrays:
    1. hstack
    2. vstack
    3. column_stack : 
    4. row_stack : 
    5. hspilt :
    6. dstack : Stack arrays in sequence depth wise (along third axis).
    7. block : Blocks can be of any dimension, but will not be broadcasted using the normal rules.
    8. atleast_1d
    9. atleast_2d
    10. numpy.concatenate
    11. **numpy.r_** : Translates slice objects to concatenation along the first axis. row
    12. numpy.c_ : Translates slice objects to concatenation along the second axis. columns
    13. numpy.ix_ : Construct an open mesh from multiple sequences.a[np.ix_([1,3],[2,5])] 可以取出来4个。

7. Copies and Views:
    1. not copy at all
    2. view : Different array objects can share the same data
    3. copy

8. Data types: 
    1. numpy 的int_  默认是 int64 .默认的dtype='i' 是int32。
    2. numpy.rec 和 pandas有点像。 通过 . 来访问某一列


9.  Subclassing ndarray ： 如何实现 ndarray的子类。略过。

**np.r_ :**

~~~python
np.r_['0,2,0', [1,2,3],[4,5,6]]
这个代码片段的控制参数0表示将在第一个维度对后面的序列进行合并，控制参数第二数2表示，合并后的结果最少要2维
所以在合并前对维度较少的序列进行维度提升。而这个提升的方式则是有第3个参数决定的，后面两个序列的维度是(3,)
由于三个参数是0，所以提升的维度在序列的维度元组中位置是0(即在维度数组的0号位置添加1)，即提升后的维度为
(3,1)，所以提升后的第一个序列应该为[[1],[2],[3]]，所以最后的结果是[[1],[2],[3],[4],[5],[6]]

np.r_['0,2,1', [1,2,3],[4,5,6]]则提升后应该为[[1,2,3]],所以结果为[[1,2,3],[4,5,6]]
~~~


### NumPy Reference:

1. Array objects
2. Universal functions
3. Routines