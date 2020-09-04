基于接收端的 `delay-base` 带宽预测算法主要的介绍都在[draf-ietf-rmcat-gcc-02]中，与基于发送端的 `delay-base` 带宽预测算法原理类似，其实现主要被分成四个部分：一个预过滤器（`pre-filtering`），一个到达时间过滤器（卡尔曼过滤器），一个过载检测器（`over-use detector`）和一个速率控制器（`rate controller`）。基于接收端和基于发送端的预测算法的区别，除了算法的核心到达时间过滤器外，其他大部分的实现都是类似的，所以在阅读这篇文件前，请先确保你已经读过[基于发送端的delay-base带宽预测算法实现]，因为这篇文章不会再对那些重复的内容进行讲解，而是主要讲到达时间过滤器，也就是卡尔曼滤波器的原理及实现。这部分代码的主要实现存在于 `WebRTC` 的 `OveruseEstimator` 类中。

# 到达时间模型

在到达时间模型上，由于我们的算法存在于接收端，所以与实现于发送端的算法相比，我们不再需要把计算得到的组间延迟变化量 $d_i$ 通过 `RTCP Feedback` 包反馈回去让发送端进行带宽预测，而是可以直接用它在接收端进行带宽的预测，然后通过 `RTMP` 包将预测得到的带宽反馈给发送端。

首先给组间延迟变量建模，我们希望通过这个值来预测当前带宽变化：

$$d(i) = w(i)$$

这里，$w(i)$ 是一个随机过程 $W$ 的样本，它是链路容量、当前交叉流量和当前发送比特率的函数。 我们将 $W$ 建模为一个白高斯过程。 如果我们过度使用信道，我们期望 $w(i)$ 的均值会增加，如果网络路径上的队列正在被清空，则 $w(i)$ 的均值会减少；否则 $w(i)$ 的均值为零。

从 $w(i)$ 中分解出均值 $m(i)$，使过程的均值为零。我们得到

$$d(i) = m(i) + v(i) \tag{0}$$

噪声项 $v(i)$ 代表了网络抖动和其他没有被模型捕捉到的延迟效应。因此我们只要算出这两个值就能得出当前网络带宽，在这之前我们先看下卡尔曼滤波器。

# 卡尔曼滤波器

维基百科对他的的解释为：[卡尔曼滤波（Kalman filter）是一种高效率的递归滤波器（自回归滤波器），它能够从一系列的不完全及包含噪声的测量中，估计动态系统的状态](https://zh.wikipedia.org/wiki/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2)。

看到其他文章中对他比较好的介绍是：[它一种利用线性系统状态方程，通过系统输入输出观测数据，对系统状态进行最优估计的算法。由于观测数据中包括系统中的噪声和干扰的影响，所以最优估计也可看作是滤波过程](http://www.suanfajun.com/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2%EF%BC%88kalmanfilter%EF%BC%89%E5%88%86%E6%9E%90%E5%8F%8A%E5%85%B6%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0%EF%BC%88c%E8%AF%AD%E8%A8%80matlab%EF%BC%89.html)。

代入到我们现在的场景下，我们希望得到 $i$ 时刻的最佳估计值 $\hat{m}(i)$，可以先利用它在 $i-1$ 时刻的状态，估计得到 $\hat{m}(i|i-i)$，再者可以通利用报文中的时间戳字段，测量得到 $d(i)$。但这两者哪个更准确呢？是通过前一个状态估算出来的值吗，还是直接测量出来的值，事实是这两者都可能有偏差，所以我们应该利用这两个值计算出一个新的最佳估计，卡尔曼滤波器可以很好的完成这个工作。

对卡尔曼滤波器公式的由来及推导过程可以参考以下几篇文章：[图说卡尔曼滤波，一份通俗易懂的教程](https://zhuanlan.zhihu.com/p/39912633)，[卡尔曼滤波（Kalman Filtering）分析及其算法实现（C/C++语言&Matlab源代码）](http://www.suanfajun.com/%E5%8D%A1%E5%B0%94%E6%9B%BC%E6%BB%A4%E6%B3%A2%EF%BC%88kalmanfilter%EF%BC%89%E5%88%86%E6%9E%90%E5%8F%8A%E5%85%B6%E7%AE%97%E6%B3%95%E5%AE%9E%E7%8E%B0%EF%BC%88c%E8%AF%AD%E8%A8%80matlab%EF%BC%89.html)，[Kalman滤波器的历史渊源](https://www.cnblogs.com/zhoug2020/p/8376509.html)。

设一个离散控制过程的系统的线性随机微分方程和系统测量值方程分别为：

$$
X(k) = A \cdot X(k-1) + B \cdot U(k) + W(k) \tag{1}
$$

$$
Z(k) = H \cdot X(k) + V(k) \tag{2}
$$

- $X(k)$ 是 $k$ 时刻的系统状态

- $U(k)$ 是 $k$ 时刻对系统的控制量，如人为介入等

- $W(k)$ 和 $V(k)$ 分别表示过程和测量的噪声，它们被假设成高斯白噪声，它们的协方差分别为 $Q,R$，这里我们假设他们不随系统状态变化而变化。 

- $A,B$ 是系统参数，对于多模型系统，$A,B$为矩阵

- $H$ 是测量系统的参数，对于多测量系统，$H$为矩阵

滤波器的递归逻辑为：

1. 先基于系统的上一状态预测现在的状态：

$$
X(k|k-1) = A \cdot X(k-1|k-1) + B \cdot U(k) + W(k) \tag{3}
$$

- $X(k|k-1)$ 为已知 $k-1$ 状态下估计 $k$ 状态的值, 所以 $X(k|k-1)$ 是利用上一状态预测的结果

- $X(k-1|k-1)$ 为上一状态的最优估计

2. 系统结果更新后，对 $X(k|k-1)$ 的协方差进行更新，这里用 $P$ 表示协方差 :

$$
P(k|k-1) = A \cdot P(k-1|k-1) \cdot A^T + Q \tag{4}
$$

- $P(k|k-1)$ 是 $X(k|k-1)$ 对应的协方差

- $P(k-1|k-1)$ 是 $X(k-1|k-1)$ 对应的协方差

- $A^T$ 是 $A$ 的转置矩阵
  
3. 经过前两步后就完成了对系统状态的预测，接下来再收集现在状态的测量值，再结合预测和测量值，就可以得到当前状态 $k$ 的最优估算值 $X(k|k)$

$$
X(k|k) = X(k|k-1) + Kg(k) \cdot (Z(k) - H \cdot X(k|k-1)) \tag{5}
$$

$Kg(k)$ 为卡尔曼增益（Kalman Gain），其计算方程为：

$$
Kg(k) = \frac{P(k|k-1) \cdot H^T}{(H \cdot P(k|k-1) \cdot H^T + R)} \tag{6}
$$

增益的含义就是估计量的方差占总方差（包括估计方差和测量方差）的比重，比重越大，说明真值接近预测值的概率越小（接近测量值的概率越大）。

4. 经过第三步我们已经得到我们想要的在 $k$ 状态下的最优估计 $X(k|k)$，为了让卡尔曼过滤器能在下一次递归中正确运行，我们需要更新 $k$ 状态下的协方差 $P(k|k)$：

$$
P(k|k) = (I - Kg(k) \cdot H) \cdot P(k|k-1) \tag{7}
$$

这样，算法就可以不断的自回归从而更新状态。这里由于第4步 $k$ 状态下协方差的更新并不需要最优估计 $X(k|k)$ 的参与，所以对最优估算值的计算可以放到最后进行，只要提前计算出卡尔曼增益就行。

# 实际代码实现

`WebRTC` 中卡尔曼滤波器的核心实现于 `OveruseEstimator::Update` 函数中。

``` cpp
// t_delta: 包组的到达时间间隔
// ts_delta: 包组的发送时间间隔
// size_delta: 包组大小的差值
void OveruseEstimator::Update(int64_t t_delta,
                              double ts_delta,
                              int size_delta,
                              BandwidthUsage current_hypothesis,
                              int64_t now_ms)
  const double t_ts_delta = t_delta - ts_delta;
  double fs_delta = size_delta;

  ...

  // 变量E_代表协方差矩阵
  // 公式（3），更新 X(k|k-1) 的协方差 P(k|k-1)
  E_[0][0] += process_noise_[0]; 
  E_[1][1] += process_noise_[1];

  ...

  // 更新测量系统参数 H
  const double h[2] = {fs_delta, 1.0};
  const double Eh[2] = {E_[0][0] * h[0] + E_[0][1] * h[1],
                        E_[1][0] * h[0] + E_[1][1] * h[1]};
  // 预测偏差，用于方程（5）与卡尔曼增益一起表示误差大小
  const double residual = t_ts_delta - slope_ * h[0] - offset_;

  ...

  // 公式（6）分母
  const double denom = var_noise_ + h[0] * Eh[0] + h[1] * Eh[1];
  // 卡尔曼增益
  const double K[2] = {Eh[0] / denom, Eh[1] / denom};

  const double IKh[2][2] = {{1.0 - K[0] * h[0], -K[0] * h[1]},{-K[1] * h[0], 1.0 - K[1] * h[1]}};
  const double e00 = E_[0][0];
  const double e01 = E_[0][1];

  // 更新 X(k|k) 的协方差矩阵 P(k|k)
  E_[0][0] = e00 * IKh[0][0] + E_[1][0] * IKh[0][1];
  E_[0][1] = e01 * IKh[0][0] + E_[1][1] * IKh[0][1];
  E_[1][0] = e00 * IKh[1][0] + E_[1][0] * IKh[1][1];
  E_[1][1] = e01 * IKh[1][0] + E_[1][1] * IKh[1][1];

  ...

  // 更新最佳估计 X(k|k)
  slope_ = slope_ + K[0] * residual;
  prev_offset_ = offset_;
  offset_ = offset_ + K[1] * residual;
```

首先最佳估计被建模为与变量 `slope` 和 `offset` 有关的方程：

$$
X(k-1|k-1) = \left[
 \begin{matrix}
   slope\underline{\hspace{5px}} \\\\
   offset\underline{\hspace{5px}}
  \end{matrix}
  \right]
$$

控制过程协方差 $Q$ 初始化为：

$$
process\underline{\hspace{5px}}noise\underline{\hspace{5px}} =
\left[
  \begin{matrix}
    1e^{-13} \\\\
    1e^{-3}
  \end{matrix}
\right]
$$

测量系统参数 $H$ 初始化为：

$$
h = \left[
 \begin{matrix}
   fs\underline{\hspace{5px}}delta & 1.0 \\\\
  \end{matrix}
  \right]
$$

协方差矩阵 P(k-1|k-1) 初始化为：

$$
E\underline{\hspace{5px}} = 
\left[
 \begin{matrix}
   100.0 & 0.0 \\\\
   0.0   & 1e^{-1}
  \end{matrix}
\right]
$$

系统测量值 $Z(k)$ 赋值为：

$$
Z(k) = t\underline{\hspace{5px}}ts\underline{\hspace{5px}}delta;
$$

然后逐步代入第二节中提到的递归过程。

首先在公式（3）中，假设当前网络的前一状态和后一状态是相同的，即有系统参数 $A = 1$，且过程没有对系统对控制，即有 $U(k) = 0$，得：

$$
X(k|k-1) = X(k-1|k-1)
$$

先求得部分结果存入变量中：
$$
Eh = E\underline{\hspace{5px}} \cdot h^T
$$

$$
\begin{aligned}
denom = var\underline{\hspace{5px}}noise + h \cdot E\underline{\hspace{5px}} \cdot h^T = var\underline{\hspace{5px}}noise + h \cdot Eh
\end{aligned}
$$

$$
residual = d(i) - h \cdot {\hat{x}_{k-1}}
$$


于是卡尔曼增益 $Kg(k)$ 通过公式（6）可得：

$$
Kg(k) = \frac{Eh}{denom}
$$

由公式（7） 更新 $k$ 状态时最佳估计的协方差矩阵 P(x|x):

$$
IKh = I - Kg(k) \cdot h^T
$$

$$
E\underline{\hspace{5px}} = IKh \cdot E\underline{\hspace{5px}}
$$

$I$ 为2阶单位矩阵，原公式中用1表示一价单位矩阵。


最后，将最佳估计通过公式（5）更新为：

$$
X(k|k) = X(k|k-1) + Kg(k) \cdot residual
$$

估计测量噪音使用一个指数平均过滤器进行更新:

$$
var\underline{\hspace{5px}}noise(x) = \alpha \cdot var\underline{\hspace{5px}}noise(x-1) + (1-\alpha) \cdot residual^2
$$

# 过载检测器 

通过前面得到 $X(x|x)$ 最佳估计，由于 $X(x|x)$ 被建模为与 `slope` 和 `offset` 相关的方程，所以在过载检测器中使用 `offset` 的预估值公式（0）中的 $m(i)$，从而做为到达时间滤波器的输出出，过载检测器的输入。

与发送端的过载检测器类似，它会与门限值进行比较，且需要持续被检测到过载，才能确定网络过载，且当 $m(i) < m(i-1)$时，即使满足上述条件，也不会被认定为过载。

大的门限值会使算法可以容忍较大的排除延迟，而小的门限值会使过载检测器对 `offset` 更加敏感，从而减小估算出来的可用带宽。所以动态调节门限值可以使得这个算法在大多数情况下有一个良好的表现，例如在和基于丢包的拥塞控制的流竞争的时候（例如TCP的Reno算法）。门限值的更新方程与发送端的实现一样，这里不做介绍。

[draf-ietf-rmcat-gcc-02]: https://tools.ietf.org/html/draft-ietf-rmcat-gcc-02
[基于发送端的delay-base带宽预测算法实现]: delay_based_bandwith_estimator.md
