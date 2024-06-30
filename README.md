# ArkGPT神经网络模型简介
    基于Linear-Attention的无限上下文神经网络模型，此模型尝试解决传统大语言模型内存开销过高的问题，采用以时间换空间的策略。
# ArkGPT模型架构
![模型架构](./Model%20Structure/ModelStructure.png)

# 注意力机制
注意力机制的定义式如下：
$attention = \frac{\sum_{i}^{n} sim(Q_{j}K_{i}^{T})V_{i}}{\sum_{i}^{n}sim(Q_{j}K_{i}^{T})+eps}$

其中，映射
    $sim(x) \in [0,1] $
一般选取sim(x) = exp(x)，也就是Softmax注意力机制，但是这样会增大内存开销，原因式每个
    $K_{i}^{T}$
都必须同时与同一个矢量
    $Q_{j}$
内积之后才能和
    $V_{i}$
作用，这个步骤的空间复杂度是
    $O(n^2)$
级别的，n为上下文长度，但是如果先处理
    $K_{i}^{T}V_{i}$
后再去作用
    $Q_{j}$
，可以降低空间复杂度为
    $O(n)$

# 线性注意力机制
虽然Softmax注意力机制在训练时可以采用并行计算，训练速度快，但是具有内存资源开销大、推理速度慢等缺点。采用线性注意力可以成功规避掉这些缺点。

* 基本假定:
    $$sim(QK^{T}) = \psi(Q)\psi(K)^{T}$$

* 核函数选取: 
    $$\psi(x) = [elu(x)+1]^{n}$$
    $n\in \{1,2,3……\}$
，当
    $n$
越大，注意力越集中，而当
    $n$
越小，注意力越涣散。但同时
    $n$
越大，越容易出现数值爆炸的问题，项目中$n = 5$

* 线性注意力的公式如下：
    $$a_{j} = \frac{\sum_{i}^{n} [\psi (Q_{j})\psi (K_{i})^{T}V_{i}]}{\sum_{i}^{n}\psi (Q_{j})\psi (K_{i})^{T}+eps }$$

* Memory项为：
    $$Memory(n) = Memory(n-1) + \psi (K_{n})^{T}V_{n}$$

* 归一化Zeta项为：
    $$Zeta(n) = Zeta(n-1) + \psi (K_{n})^{T}$$

* 线性注意力表现为:
    $$a_{j} = \frac{\psi (Q_{j})Memory(n)}{\psi(Q_{j})Zeta(n) + eps}$$

其中，eps为无穷小量，目的是为了训练的稳定性，防止出现分母为0的情况。


# 缺点
* 采用线性注意力模型的表达能力较低。原因是线性注意力模型的注意力过于弥散，不像Softmax那样集中，这一点再增加核函数的幂次$n$时得到改进，却也带来了数值爆炸的风险。
* 线性注意力虽然可以解决资源开销、推理速度慢的问题，但是训练时间被拉长。
* 模型的遗忘门机制则是采用手动遗忘，训练时需要每训练完一次就必须执行一次遗忘操作。理想状态下，在足够的浮点运算精度，模型可参考的上下文长度为无限长，不增加计算机内存的开销，但模型无法自主选择遗忘哪些东西。
# 项目文件
    Model.py:保存着神经网络层。
    ArkGPT.py:模型封装。
    run.py:模型运行器。

