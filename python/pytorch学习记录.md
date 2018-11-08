title: pytorch学习记录
author: xg wang
tags:
- pytorch
categories:
- python
date: 2018-010-25 21:01:00
---

### 深度学习框架PyTorch：入门与实践


####  实现细节

`nn.Module`对象在构造函数中的行为看起来有些怪异，如果想要真正掌握其原理，就需要看两个魔法方法`__getattr__`和`__setattr__`。在Python中有两个常用的buildin方法`getattr`和`setattr`，`getattr(obj, 'attr1')`等价于`obj.attr`，如果`getattr`函数无法找到所需属性，Python会转而调用`obj.__getattr__('attr1')`方法，即`getattr`函数无法找到的交给`__getattr__`函数处理，没有实现`__getattr__`或者`__getattr__`也无法处理的就会raise AttributeError。`setattr(obj, 'name', value)`等价于`obj.name=value`，如果obj对象实现了`__setattr__`方法，setattr会直接调用`obj.__setattr__('name', value)`，否则调用buildin方法。总结一下：
- result  = obj.name会调用buildin函数`getattr(obj, 'name')`，如果该属性找不到，会调用`obj.__getattr__('name')`
- obj.name = value会调用buildin函数`setattr(obj, 'name', value)`，如果obj对象实现了`__setattr__`方法，`setattr`会直接调用`obj.__setattr__('name', value')`

nn.Module实现了自定义的`__setattr__`函数，当执行`module.name=value`时，会在`__setattr__`中判断value是否为`Parameter`或`nn.Module`对象，如果是则将这些对象加到`_parameters`和`_modules`两个字典中，而如果是其它类型的对象，如`Variable`、`list`、`dict`等，则调用默认的操作，将这个值保存在`__dict__`中。



##### Sequential的三种写法

```python
net1 = nn.Sequential()
net1.add_module('conv', nn.Conv2d(3, 3, 3))
net1.add_module('batchnorm', nn.BatchNorm2d(3))
net1.add_module('activation_layer', nn.ReLU())

net2 = nn.Sequential(
        nn.Conv2d(3, 3, 3),
        nn.BatchNorm2d(3),
        nn.ReLU()
        )

from collections import OrderedDict
net3= nn.Sequential(OrderedDict([
          ('conv1', nn.Conv2d(3, 3, 3)),
          ('bn1', nn.BatchNorm2d(3)),
          ('relu1', nn.ReLU())
        ]))
print('net1:', net1)
print('net2:', net2)
print('net3:', net3)
```


##### 为不同子网络设置不同的学习率

```python
optimizer =optim.SGD([
                {'params': net.features.parameters()}, # 学习率为1e-5
                {'params': net.classifier.parameters(), 'lr': 1e-2}
            ], lr=1e-5)
```

对于如何调整学习率，主要有两种做法。一种是修改optimizer.param_groups中对应的学习率，另一种是更简单也是较为推荐的做法——新建优化器，由于optimizer十分轻量级，构建开销很小，故而可以构建新的optimizer。但是后者对于使用动量的优化器（如Adam），会丢失动量等状态信息，可能会造成损失函数的收敛出现震荡等情况。

#### nn.Module 区别联系 nn.functional

对于不具备可学习参数的层（激活层、池化层等），将它们用函数代替，这样则可以不用放置在构造函数`__init__`中。对于有可学习参数的模块，也可以用functional来代替，只不过实现起来较为繁琐，需要手动定义参数parameter，如前面实现自定义的全连接层，就可将weight和bias两个参数单独拿出来，在构造函数中初始化为parameter。

```python
from torch.nn import functional as F
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = F.pool(F.relu(self.conv1(x)), 2)
        x = F.pool(F.relu(self.conv2(x)), 2)
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x
```
```python
class MyLinear(nn.Module):
    def __init__(self):
        super(MyLinear, self).__init__()
        self.weight = nn.Parameter(t.randn(3, 4))
        self.bias = nn.Parameter(t.zeros(3))
    def forward(self):
        return F.linear(input, weight, bias)
```

### 保存模型的两种办法

```python
# 保存模型
t.save(net.state_dict(), 'net.pth')

# 加载已保存的模型
net2 = Net()
net2.load_state_dict(t.load('net.pth'))
```

第二种严重依赖模型定义方式及文件路径结构等，很容易出问题，因而不建议使用。
```python
t.save(net, 'net_all.pth')
net2 = t.load('net_all.pth')
net2
```

### 多个GPU运行程序

两种方式：
```python
nn.parallel.data_parallel(module, inputs, device_ids=None, output_device=None, dim=0, module_kwargs=None)

torch.nn.DataParallel(module, device_ids=None, output_device=None, dim=0)
```

DataParallel并行的方式，是将输入一个batch的数据均分成多份，分别送到对应的GPU进行计算，各个GPU得到的梯度累加。与Module相关的所有数据也都会以浅复制的方式复制多份，在此需要注意，在module中属性应该是只读的。


### nn和autograd的关系
nn.Module利用的也是autograd技术，其主要工作是实现前向传播。在forward函数中，nn.Module对输入的tensor进行的各种操作，本质上都是用到了autograd技术。这里需要对比autograd.Function和nn.Module之间的区别：
- autograd.Function利用了Tensor对autograd技术的扩展，为autograd实现了新的运算op，不仅要实现前向传播还要手动实现反向传播
- nn.Module利用了autograd技术，对nn的功能进行扩展，实现了深度学习中更多的层。只需实现前向传播功能，autograd即会自动实现反向传播
- nn.functional是一些autograd操作的集合，是经过封装的函数

作为两大类扩充PyTorch接口的方法，我们在实际使用中应该如何选择呢？如果某一个操作，在autograd中尚未支持，那么只能实现Function接口对应的前向传播和反向传播。如果某些时候利用autograd接口比较复杂，则可以利用Function将多个操作聚合，实现优化，正如第三章所实现的`Sigmoid`一样，比直接利用autograd低级别的操作要快。而如果只是想在深度学习中增加某一层，使用nn.Module进行封装则更为简单高效。

### tensor

torch.tensor always copies data. 避免复制数据的办法
- torch.Tensor.requires_grad_()
- torch.Tensor.detach()
- torch.from_numpy()

layout=torch.strided | torch.sparse_coo
torch.cat torch.split torch.chunk
torch.transpose(input, dim0, dim1)

rand [0-1]
randint [low-high]
randn [normal mean 0 std 1]
random [form to]
randperm [n]

torch.mul broadcastable 按照元素相称
torch.reciprocal 
torch.lerp : linear interpolation
torch.sigmoid

还有一些傅立叶变换函数， 
- Bartlett window function
- Blackman window function
- Hamming window function
- Hann window function

torch.meshgrid
torch.histc : Computes the histogram of a tensor 

BLAS and LAPACK Operations
- torch.addbmm(beta=1, mat, alpha=1, batch1, batch2, out=None) → Tensor
- torch.addmm(beta=1, mat, alpha=1, mat1, mat2, out=None) → Tensor
- torch.addmv(beta=1, tensor, alpha=1, mat, vec, out=None) → Tensor
- torch.addr(beta=1, mat, alpha=1, vec1, vec2, out=None) → Tensor  外集
- torch.baddbmm(beta=1, mat, alpha=1, batch1, batch2, out=None) → Tensor
- torch.bmm(batch1, batch2, out=None) → Tensor
- torch.btrifact(A, info=None, pivot=True) Batch LU factorization.
- torch.btrifact_with_info(A, pivot=True) Batch LU factorization with additional error information
- torch.btrisolve(b, LU_data, LU_pivots) Batch LU solve.
- torch.dot(tensor1, tensor2) → Computes the dot product (inner product) of two tensors
- torch.gels(B, A, out=None)
- torch.geqrf
- torch.ger
- torch.gesv
- torch.inverse
- torch.det
- torch.logdet
- torch.slogdet
- torch.matmul
- torch.mm
- torch.mv
- torch.orgqr orthogonal matrix Q of a QR factorization
- torch.ormqr
- torch.pinverse
- torch.potri
- torch.pstrf
- torch.potrs
- torch.qr
- torch.svd
- torch.trtrs
- torch.symeig


### Tensor  multi-dimensional matrix containing elements of a single data type

- Tensor.expand(*size)
- Tensor.expand_as(other)
- Tensor.long()
- Tensor.short()
- Tensor.double()
- Tensor.permute(*dim) 重排维度

- index_add_
- index_copy_
- index_fill_
- index_put_
- index_select
- narrow(dimension, start, length) → Tensor

- repeat
- expand
- put_ tensor is treated as if it were a 1-D tensor
- scatter_
- scatter_add_

### torch.cuda

很多cuda操作的函数，包括内存管理。设备管理，随机数生成器，*Streams and events*，多显卡数组操作

Streams and events 不太明白

### torch.nn

nn.Parameter
nn.Module
    1. add_module(name, module)
    1. apply(fn)
    1. register_backward_hook(hook)
    1. register_buffer(name, tensor)
    1. train(mode=True)
    1. zero_grad()
nn.Sequential
nn.ModuleList
    1. append
    1. extend
nn.ParameterList
    1. append
    1. extend


### torch.optim

1. optim.Optimizer(params, defaults)
    1. add_param_group(param_group)
    1. load_state_dict(state_dict)
    1. state_dict()
    1. step(closure)
    1. zero_grad()
1. Adadelta
1. Adagrad
1. Adam
1. SparseAdam
1. Adamax
1. ASGD
1. LBFGS
1. RMSprop
1. Rprop
1. SGD


### torch.utils.data

1. Dataset
2. TensorDataset
3. ConcatDataset
4. Subset
5. DataLoader
6. 各种sampler

### torch.utils.bottleneck

使用 python profiler 分析性能 。自己之后测试一下。
```python -m torch.utils.bottleneck /path/to/source/script.py [args]```

### torch.autograd

1. autograd.backward
1. autograd.grad
1. autograd.no_grad
1. autograd.enable_grad
1. autograd.set_grad_enabled
1. **torch.autograd.Function**
1. autograd.gradcheck
1. autograd.gradgradcheck


### torch.multiprocessing


### torch.distributed

