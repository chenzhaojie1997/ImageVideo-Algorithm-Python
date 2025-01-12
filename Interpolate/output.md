# NEDI

## NEDI
1. 论文名New edge-directed interpolation。基于维纳滤波思想，求解最优滤波器
2. 考虑这样一个问题：在你观察到一条有噪声的曲线x，已经知道他上面具有噪声n，问能不能求得一个最优的滤波器，使得 h * x的结果是最优的。答案是肯定的，那我们要知道些什么信息呢？ 维纳滤波的思路是：如果我们知道噪声的互相关程度，也知道信号的互相关程度，那么就可以知道 
3. 互相关程度本质上是在说这样一件事情，当前时刻是t，如果把当前时刻和附近时刻的相似程度刻画出来，就是互相关函数。在不考虑噪声的情况下，如果我们知道了互相关函数，那么我们只需要在 -T到T时刻加权，就可以知道t时刻的结果。此时h就是信号的自相关，但是当噪声越大的时候，噪声的互相关函数变得更强。此时就需要把信号的互相关函数除以互相关函数加噪声相关函数。如果设信号的互相关函数为Ryy，噪声的互相关函数为Rnn。那么h = Ryy / (Ryy + Rnn) = Rxy / (Ryy + Rnn) 也就是权重会根据噪声的强度进行调整 
4. 这样关键在于求解图像信号的自相关函数和噪声相关函数。但是求噪声的自相关是一个很强的先验信息。作者这里使用了 h = Rxy / Rxx的形式，需要求xy的互相关和xx的自相关。Rxx的自相关是好求的，图像patch对应位置进行点积就可以了。Rxy的互相关是比较难求的，因为y是真值，是不知道的 
5. 作者这里比较巧妙的一点是使用下采样的patch作为真值的patch 
6. 效果上，m=4时效果明显优于双线性插值、略优于双三次插值和Lanczos。m=2插值结果明显不对；m=4效果较优；m=6相比m=4边缘锯齿、噪声减轻明显，但清晰度还在同一水准；m=8相比m=6改善较小，改善方向仍然是锯齿噪声的减轻。m=6相比m=4，窗口变大了2.25倍，在保证清晰度的情况下，锯齿、噪声能明显改善。而m=8相比m=6改善并不明显。因为首先m=6已获得较好效果，再度大幅增加很难；其次窗口信息的增益并不明显 

#### 改进NEDI —— 性能改进1
1. 改进目标：在原始NEDI算法，直接使用矩阵点乘结果作为相似度度量，能否使用更合理的相似性度量指标
2. 改进方法：具体的相似性度量指标换成了协方差，而不是直接向量点乘
3. 使用协方差方法产生的边缘更好，但是代价是更多的锯齿和噪声
4. 这种方法，可能会产生过度处理的边缘
5. 随着m增大，展示出和NEDI一样的规律：随着m增大，使用的信息更多，在边缘的锯齿、图像的噪声会变小
6. 随着m增大，改进方法的优势在减少。在m=6情况下，改进前后在边缘上已没有明显差异
7. 一些感想
  （1）做超分本质上好像在做边缘
  （2）基于（1），更好的超分方法本质上就是产生更锐利边缘的同时伴随更窄的过度区域、带来更少的锯齿、更轻的噪声
  （3）合理的、可以接受的边缘锯齿会一定程度上带来图像清晰的感觉；但更明显的边缘锯齿会带来更多噪声；所以同一水平的算法应该要选择好在锯齿、噪声等方面做好平衡

## 其他应用
1. 原论文中提到，这种方法也可以应用在去马赛克任务上，这里先不深入研究，留下个种子。

# INTR

## 1. 原始INTR
1. 原论文是 An Edge-Guided Image Interpolation Algorithm via Directional Filtering and Data Fusion。第一作者张磊
2. 这里主要实现原论文中的简化版本。核心思想是根据方差自适应调整45°和135°插值结果的权重，以达到插值结果主要沿着灰度变化不大方向进行的目的
3. 总体定性来说，边缘相比双线性有明显优势，但是明显差于NEDI
4. m取从3变化到9，差别不大。这是可以预期的，因为窗口m大小的作用只在预计方差上。而使用方差的形势判断一个像素附近是否有边缘，使用更大的窗口收益并不大
5. 有一个明显的缺点就是容易受噪声影响。这是可以预期的，假如待插值附近的点有一个噪点，那么使用方差方法很有可能把它认为是边缘，于是赋给它更大权重。噪声于是就被保留了下来。
6. 给我的一个提示，或者经验就是，考虑到用边缘信息的，都应该考虑噪声的影响，或者考虑将它们区分开。因为它们在数据上有类似的统计量属性，但是视觉上却产生完全不同的效果。

# Lanczos
1. 基于原始Lanczos方法改进，改进之处在于可以调整插值核。开组会时，小组长曾经提过，以前H的PQ人员会专门对插值权重做调整，本质上就是调整插值核
2. 类似于Lanczos这种可以缩放任意倍的插值方法还是有其价值，当需要放大的场景不是2,4,8等倍数时，一种可行的方法是先用优秀的超分方法（传统超分、AISR等）放大到一个更高的倍数，再使用任意倍率插值方法下采样到想要的分辨率
3. Lanczos的精髓在于认为每个像素的影响会按照sinc函数方式。Lanczos核在距离超过1，小于2时候会变成负数，这也是合理的。回想使用拉普拉斯锐化边缘时候的核也是有正有负的，会得到更锐利的边缘。
4. 直接使用4*4的Lanczos方法会导致有点“栅格效应”。OPENCV中的Lanczos_4方法则使用了8×8的领域，很好地缓解了这种效应。二者清晰度并没有变化

# 总结
1. 插值性能上，NEDI最优，INTR略优于Lanczos；但是INTR最容易受噪声影响
2. 窗口大小会影响芯片算法落地的line buffer和行延时，从这个角度上看，Lanczos仍然是最好的选择，其次则是INTR，NEDI的窗口最大
