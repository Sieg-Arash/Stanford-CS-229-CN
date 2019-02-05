# CS229 课程讲义中文翻译
CS229 Lecture notes

|原作者|翻译|
|---|---|
|[Andrew Ng  吴恩达](http://www.andrewng.org/),Kian Katanforoosh|[CycleUser](https://www.zhihu.com/people/cycleuser/columns)|

|相关链接|
|---|
|[Github 地址](https://github.com/Kivy-CN/Stanford-CS-229-CN)|
|[知乎专栏](https://zhuanlan.zhihu.com/MachineLearn)|
|[斯坦福大学 CS229 课程网站](http://cs229.stanford.edu/)|
|[网易公开课中文字幕视频](http://open.163.com/movie/2008/1/M/C/M6SGF6VB4_M6SGHFBMC.html)|

# 深度学习(Deep Learning)

现在开始学深度学习.在这部分讲义中,我们要简单介绍神经网络,讨论一下向量化以及利用反向传播(backpropagation)来训练神经网络.

## 1 神经网络(Neural Networks)

我们将从小处开始逐渐构建一个神经网络,一步一步来.回忆一下最开始本课程的时候就见到的那个房价预测模型:给定房屋的面积,我们要预测其价格.

在之前的章节中,我们学到的方法是在数据图像中拟合一条直线.现在咱们不再拟合直线了,而是通过设置绝对最低价格为零来避免出现有负值房价出现.这就在图中让直线拐了个弯,如图1所示.

![](https://raw.githubusercontent.com/Kivy-CN/Stanford-CS-229-CN/master/img/cs229notedlf1.png)

我们的目标是输入某些个输入特征$x$到一个函数$f(x)$中,然后输出房子$y$的价格.规范来表述就是:$f:x\rightarrow y$.可能最简单的神经网络就是定义一个单个神经元(neuron)的函数$f(x)$,使其满足$f(x)=\max(ax+b,0)$,其中的$a,b$是参数(coefficients).这个$f(x)$所做的就是返回一个单值:要么是$(ax+b)$,要么是0,就看哪个更大.在神经网络的领域,这个函数叫做一个ReLU(英文读作'ray-lu'),或者叫整流线性单元(rectified linear unit).更复杂的神经网络可能会使用上面描述的单个神经元然后堆栈(stack)起来,这样一个神经元的输出就是另一个神经元的输入,这就得到了一个更复杂的函数.

现在继续深入房价预测的例子.除了房子面积外,假如现在你还指导了卧房数目,邮政编码,以及邻居的财富状况.构建神经网络的国产和乐高积木(Lego bricks)差不多:把零散的砖块堆起来构建复杂结构而已.同样也适用于神经网络:选择独立的神经元并且对战起来创建更复杂的神经元网络.

有了上面提到的这些特征(面积,卧房数,邮编,社区财富状况),就可以决定这个房子的价格是否和其所能承担的最大家庭规模有关.加入家庭规模是房屋面积和卧室数目的一个函数(如图2所示).邮编(zip code)则可以提供关于邻居走动程度之类的附加信息(比如你能走着去杂货店或者去哪里都需要开车).结合邮编和邻居的财富状况就可以预测当地小学的教育质量.给了上面这三个推出来的特征(家庭规模,交通便利程度,学校教育质量),就可以依据这三个特征来最终推断房价了.

![](https://raw.githubusercontent.com/Kivy-CN/Stanford-CS-229-CN/master/img/cs229notedlf2.png)

我们就已经描述了上面这个神经网络了,就如同读者应该已经理解了确定这三个因素来最终影响房屋.神经网络的一个神奇之处就在于你只需要有输入特征向量x以及输出y,而其他的具体过程都交给神经网络自己来完成.用神经网络学习中介特征(intermediate features)的这个过程叫做端到端学习(end-to-end learning).

参考上面关于房价的例子,严格来说,输入到神经网络的是一个输入特征的集合$x_1,x_2,x_3,x_4$.我们将这四个特征连接到三个神经元(neurons).这三个"内部(internal)"神经元叫做隐藏单元(hidden units).这个神经网络的目标是要去自动判定三个相关变量来借助着三个变量来预测房屋价格.我们只需要给神经网络提供充足数量的训练样本$(x^{(i)},y^{(i)})$.很多时候,神经网络能够发现对预测输出很有用的一些复杂特征,但这些特征对于人类来说可能不好理解,因为可能不具备通常人类所理解的常规含义.因此有人就把神经网络看作是一个黑箱(black box,意思大概就是说内部过程不透明),因为神经网络内部发掘的特征可能是难以理解的.

接下来我们将神经网络的概念以严格术语进行表述.假设我们有三个输入特征$x_1,x_2,x_3$,这三个特征共同称为输入层(input layer),然后又四个隐藏单元(hidden units)共同称为隐藏层(hidden layer),一个输出神经元叫做输出层(output layer).隐藏层之所以称之为隐藏,是因为不具备足够的事实依据或者训练样本值来确定这些隐藏单元.这是受到输入和输出层限制的,对输入输出我们所了解的基本事实就是$(x^{(i)},y^{(i)})$.

第一个隐藏单元需要输入特征$x_1,x_2,x_3$,然后输出一个记作$a_1$的输出值.我们哲龙字幕a是因为这个可以表示神经元的"激活(activation)"的值.在这个具体的案例中,我们使用了一个单独的隐藏层,但实际上可能有多个隐藏层.假设我们用$a_1^{[1]}$来表示第一个隐藏层中的第一个隐藏单元.对隐藏层用从零开始的索引来指代层号.也就是输入的层是第0层,第一层隐藏层是第1层,输出层是第二层.再次强调一下,更复杂的神经网络就可能有更多的隐藏层.有了上述数学记号,第2层的输出就表达做$a_1^{[2]}$.统一记号就得到了:

$$
\begin{aligned}
x_1 &= a^{[0]}_1 &\quad\text{(1.1)}\\
x_2 &= a^{[0]}_2 &\quad\text{(1.2)}\\
x_3 &= a^{[0]}_3 &\quad\text{(1.3)}\\
\end{aligned}
$$

这里要说清的是,用方括号$[1]$商标的元素表示一切和第1层相关的,带圆括号的$x^{(i)}$表示的则是第i个训练样本,而$a^{[l]}_j$表示的是第j个单元在第l层的激活.可以将逻辑回归函数$g(x)$看做一个单个神经元(如图3所示):

$$
g(x)=\frac{1}{1+\exp(-w^Tx)}
$$

向上面逻辑回归函数$g(x)$中输入的就是三个特征$x_1,x_2,x_3$,而输出的是对y的估计值.可以将上面这个$g(x)$表示成神经网络中的一个单个神经元.可以将这个函数拆解成两个不同的计算:(1)$z=w^Tx+b$;(2)$a=\sigma(z),\sigma(z)=\frac{1}{1+e^{-z}}$.要注意这里的记号上的差别:之前我们使用的是$z=\theta^Tx$但现在使用的是$z=w^Tx+b$,这里面的$w$是一个向量.后面的讲义中会看到如果是表示矩阵就用大写字母$W$了.这里的记号差别是为了遵循标准的神经网络记号.更通用的写法,还要加上$a=g(z)$,这里面这个$g(z)$可以试试某种激活函数.举几个例子,激活函数可以包括下面几种:

$$
\begin{aligned}
g(z) &= \frac{1}{1+e^{-z}} &\quad\text{sigmoid}\quad\text{(1.4)}\\
g(z) &= \max(z,0) &\quad\text{ReLU}\quad\text{(1.5)}\\
g(z) &= \frac{e^z-e^{-z}}{e^z+e^{-z}} &\quad\text{tanh}\quad\text{(1.6)}\\
\end{aligned}
$$

一般来说,$g(z)$都是非线性函数(non-linear function).

![](https://raw.githubusercontent.com/Kivy-CN/Stanford-CS-229-CN/master/img/cs229notedlf3.png)


回到前面的那个神经网络,第一隐藏层的第一个隐藏单元会进行下面的计算:

$$
z^{[1]}_1={W^{[1]}_1}^T x+b^{[1]}_1\quad,\quad a^{[1]}_1=g(z^{[1]}_1)   \quad\text{(1.7)}
$$

上式中的$W$是一个参数矩阵,而$W_1$指的是这个矩阵的第1行(row).和第一个隐藏单元相关联的参数包括了向量$W_1^{[1]} \in R^3$和标量$b_1^{[1]} \in R^3$.对第1个隐藏层的第二和第三个隐藏单元,计算定义为:

$$
\begin{aligned}
z^{[1]}_2&={W^{[1]}_2}^T x+b^{[1]}_2\quad,\quad a^{[1]}_2&=g(z^{[1]}_2)  \\
z^{[1]}_3&={W^{[1]}_3}^T x+b^{[1]}_3\quad,\quad a^{[1]}_3&=g(z^{[1]}_3)  \\
\end{aligned}
$$

其中每个隐藏单元都有各自对应的参数$W$和$b$.接下来就是输出层的计算:

$$
z^{[1]}_1={W^{[1]}_1}^T x+b^{[1]}_1\quad,\quad a^{[1]}_1=g(z^{[1]}_1)   \quad\text{(1.8)}
$$

上面式子中的$a^{[1]}$定义是所有第一层激活函数的串联(concatenation):

$$
a^{[1]}=\begin{bmatrix} a^{[1]}_1 \\
a^{[1]}_2 \\
a^{[1]}_3 \\
a^{[1]}_4 \\
 \end{bmatrix} \quad\text{(1.9)}
$$

激活函数$a^{[2]}_1$来自第二层,是一个单独的标量,定义为$a^{[2]}_1=g(z^{[2]}_1)$,表示的是神经网络的最终输出预测结果.要注意对于回归任务来说,通常不用严格为正的非线性函数(比如ReLU或者Sigmoid),因为对于某些任务来说,基本事实$y$的值实际上可能是负值.

## 2 向量化(Vectorization)

为了一合理的速度来实现一个神经网络,我们必须谨慎使用循环(loops).要计算在第一层中的隐藏单元的激活函数,必须要计算出来$z_1,...,z_4$和$a_1,...,a_4$.

$$
\begin{aligned}
z^{[1]}_1&={W^{[1]}_1}^T x+b^{[1]}_1\quad,\quad a^{[1]}_1&=g(z^{[1]}_1) \quad\text{(2.1)} \\
&...& ...\quad\text{(2.2)} \\
z^{[1]}_4&={W^{[1]}_4}^T x+b^{[1]}_4\quad,\quad a^{[1]}_4&=g(z^{[1]}_4) \quad\text{(2.3)} \\
\end{aligned}
$$

最自然的实现上述目的的方法自然是使用for循环了.深度学习在机器学习领域中最大的特点就是深度学习的算法有更高的算力开销.如果你用了for循环,代码运行就会很慢.

这就需要使用向量化了.向量化和for循环不同,能够利用矩阵线性代数的优势,还能利用一些高度优化的数值计算的线性代数包(比如BLAS),因此能使神经网络的计算运行更快.在进行深度学习领域之前,小规模数据使用for循环可能就足够用了,可是对现代的深度学习网络和当前规模的数据集俩说,for循环就根本不可行了.

### 2.1 输出计算向量化(Vectorizing the Output Computation)

接下来将的方法是不使用for循环来计算$z_1,...,z_4$.使用矩阵线性代数方法,可以用如下方法计算状态:

$$
\begin{aligned}
\begin{bmatrix} z^{[1]}_1 \\ ...\\...\\ z^{[1]}_4 \end{bmatrix} &=\begin{bmatrix}-&{W{[1]}_1}^T - \\ -&{W{[1]}_2}^T -\\&...\\ -&{W{[1]}_4}^T -\end{bmatrix}\begin{bmatrix}x_1\\x_2\\x_3 \end{bmatrix} + \begin{bmatrix} b^{[1]}_1 \\ b^{[1]}_2 \\...\\ b^{[1]}_4\end{bmatrix}  \quad\text{(2.4)}\\
z^{[1]}&\in  R^{4\times 1}\\
W^{[1]}&\in  R^{4\times 3}\\
b^{[1]}&\in  R^{4\times 1}\\
\end{aligned}
$$

上面的矩阵下面所标注的$R^{n\times m}$表示的是对应矩阵的维度.直接用矩阵记号表示是这样的:$z^{[1]}= W^{[1]}x+b^{[1]}$.要计算$a^{[1]}$而不实用for循环,可以利用MATLAB/Octave或者Python里面先有的向量化库,这样通过运行分布式的针对元素的运算就可以非常快速的计算出$a^{[1]}=g(z^{[1]})$.数学上可以定义一个S型函数(sigmoid function)$g(z)$:

$$
g(z)=\frac{1}{1+e^{-1}} \quad\text{, } z\in R \quad\text{(2.5)}
$$

不够,这个S型函数不尽力针对标量(scalars)来进行定义,也可以对向量(vectors)定义.以MATLAB/Octave风格的伪代码,就可以如下方式定义这个函数:

$$
g(z)=1 ./(1+\exp(-z)) \quad\text{, } z\in R \quad\text{(2.6)}
$$

上式中的$./$表示的是元素对除.这样有了向量化的实现后,$a^{[1]}=g(z^{[1]})$就可以很快计算出来了.

总结一下目前位置对神经网络的了解,给定一个输入特征$x\in R^3$,就可以利用$z^{[1]}=W^{[1]}x+b^{[1]}$和$a^{[1]}=g(z^{[1]})$计算隐藏层的激活,要计算输出层的激活状态(也就是神经网络的输出),要用:

$$\begin{aligned}
z^{[2]}&=W^{[2]} a^{[1]}+b^{[2]}\quad,\quad a^{[2]}&=g(z^{[2]})\quad\text{(2.7)}\\
z^{[2]}&\in  R^{1\times 1}\\
W^{[2]}&\in  R^{1\times 4}\\
a^{[1]}&\in  R^{4\times 1}\\
b^{[2]}&\in  R^{1\times 1}\\
\end{aligned}
$$

为什么不对$g(z)使用同样的函数呢?为啥不用$g(z)=z$呢?假设$b^{[1]}$和$b^{[2]}$都是零值的.利用等式(2.7)就得到了:

$$\begin{aligned}
z^{[2]}&=W^{[2]}a^{[1]} &\text{} \quad\text{(2.8)}\\
&= W^{[2]} g(z^{[1]}) &\text{根据定义} \quad\text{(2.9)}\\
&= W^{[2]}z^{[1]}&\text{因为}g(z)=z \quad\text{(2.10)}\\
&= W^{[2]}W^{[1]}x&\text{参考等式(2.4)} \quad\text{(2.11)}\\
&= \tilde W x&\text{其中的}\tilde W  =W^{[2]}W^{[1]} \quad\text{(2.12)}\\
\end{aligned}
$$

这样之前的$W^{[1]}W^{[2]}$就合并成了$\tilde W $.这是因为对一个线性函数应用另一个线性函数会得到原结果的一个线性函数(也就是你可以构建一个$\tilde W $来使得$\tilde W  x=W^{[2]}W^{[1]}x).这也使得神经网络失去了很多的代表性,因为有时候我们要预测的输出和输入可能是存在非线性关系的.没有了非线性激活函数,神经网络就只是简单地进行线性回归了.

### 2.2 训练样本集向量化(Vectorization Over Training Examples)

假如你有了一个三个样本组成的训练集.每个样本的激活函数如下所示:

$$
\begin{aligned}
z^{[1](1)} &= W^{[1]}x^{(1)}+b^{[1]}\\
z^{[1](2)} &= W^{[1]}x^{(2)}+b^{[1]}\\
z^{[1](3)} &= W^{[1]}x^{(3)}+b^{[1]}\\
\end{aligned}
$$

要注意上面的括号是有区别的,方括号[]内的数字表示的是层数(layer number),而圆括号()内的数字表示的是训练样本编号(training example number).直观来看,似乎可以用一个for循环实现这个过程.但其实也可以通过向量化来实现.首先定义:

$$
X=\begin{bmatrix} |&|&|&\\ 
x^{(1)}&x^{(2)}&x^{(3)}\\ 
|&|&|&\\ 
\end{bmatrix}\quad\text{(2.13)}
$$

注意,我们是在列上排放训练样本而非在行上.然后可以将上面的式子结合起来用单个的统一公式来表达:

$$
Z^{[1]}=\begin{bmatrix} |&|&|&\\ 
z^{[1](1)}&z^{[1](2)}&z^{[3](3)}\\ 
|&|&|&\\ 
\end{bmatrix} =W^{[1]}X+b^{[1]}\quad\text{(2.14)}
$$

你或许已经注意到了我们试图在$W^{[1]}X \in R^{4\times 3}$的基础上添加一个$b^{[1]}\in R^{4\times 1}$.严格按照线性代数规则的话,这是不行的.不过在实际应用的时候,这个加法操作是使用广播(boradcasting)来实现的.创建一个中介$\tilde b^{[1]}\in R^{4\times 3}$:

$$
\tilde b^{[1]} =\begin{bmatrix} |&|&|&\\ 
b^{[1]}&b^{[1]}&b^{[1]}\\ 
|&|&|&\\ 
\end{bmatrix}\quad\text{(2.15)}
$$

然后就可以计算:$Z^{[1]}= =W^{[1]}X+\tilde b^{[1]}$.通常都没必要去特地构建一个$\tilde b^{[1]}$.检查一下等式(2.14),你就可以假设$b^{[1]}\in R^{4\times 1}$能够正确广播(broadcast)到$W^{[1]}X \in R^{4\times 3}$上.

综上所述:加入我们有一个训练集:$(x^{(1)},y^{(1)}),...,(x^{(m)},y^{(m)})$,其中的$x^{(i)}$是一个图形而$y^{(i)}$是一个二值化分类标签,标示的是一个图片是否包含一只猫(比如y-1就表示有一只猫).首先要将参数$W^{[1]},b^{[1]},W^{[2]},b^{[2]}$初始化到比较小的随机值.对每个样品,都计算其从S型函数(sigmoid function)$a^{[2](i)}$的输出概率.然后利用逻辑回归(logistic regression)对数似然函数(log likelihood):

$$
\sum^m_{i=1}(y^{(i)}\log a^{[2](i)} +(1-y^{(i)}\log(1-a^{[2](i)})\quad\text{(2.16)}
$$

最终,利用梯度上升法(gradient ascent)将这个函数最大化.这个最大化过程对应的就是对神经网络的训练.

## 3 反向传播(Backpropagation)

### 3.1 参数初始化(ParameterInitialization)

### 3.2 优化(Optimization)

### 3.3 参数分析(Analyzing the Parameters)

#### 3.3.1 L2规范化(Regularization)

#### 3.3.2 参数共享(Parameter Sharing)