@(论文2)[Knowledge Distilling, Relation]
#Relational Knowledge Distillation
##Introduction
* 也是考虑蒸馏样本级别的相互关系的。除了样本之间的距离之外，还考虑了样本之间的角度关系。
* 感觉跟另一篇跟relation相关的文章（Correlation Congruence for Knowledge Distillation）相比，多了角度的相似性。
* 想到了A Gift from Knowledge Distillation这篇，是蒸馏各层对feature处理的变化的。他衡量feature进入某个block先后的方法可能本质上跟这篇文章差不多。因为他用的是gram矩阵的变形，里面的值都是点积，而点积既包含了方向，也包含了模长。
* 他这里说的就是，应该学习的是整个表达空间的结构而不是单纯的某个样本被映射为什么东西。
* 感觉这里是值得探讨的。表达空间的结构到底应该被怎么表达，他的做法是不是真的能完全代表表达空间的结构，都是可以商榷的
##Related Work
* 提到了两篇将知识蒸馏应用到半监督、无监督领域的工作【18，26】
* 提到了一个metric learning的应用，计算样本相似度【7】
##Our Approach
* 就是给定两个样本，要保证他们之间的某个距离度量被映射到teacher的空间和student的空间保持一致。
* 第一个度量就是欧氏距离度量。对这个讨论感觉并没有Correlation Congruence for Knowledge Distillation这篇讨论的全面。
* 这里本来我以为是考虑到不同距离的两个样本应该模仿的程度也不一样（对于较近的样本，欧氏距离是可以近似为真实距离的，但对于距离较远的两个样本则不能），所以才提出了一个分段的loss。但是他这里不是，是teacher跟student差别比较大就用l1 loss，比较小就用l2 loss，叫做huber loss
* 这里详细的讲一下这个loss：l1 loss的梯度是稳定的，无论在哪个点（差别多大）梯度都是1。这让他对于异常点（student学teacher学的特别不像，导致l1损失很大，可以看做是异常点）不敏感。缺点在于差别很小的时候梯度也是1，容易走过头，即优化的最优点附近梯度还是挺大，会振动。l2则相反，对异常点很敏感（差别大的时候梯度很大），但最优点附近梯度很小，容易收敛。
* 用这个我感觉在某种程度上可以减缓一点上面提到的这种缺陷。但他也没有说他的理解
* 角度的度量是三个样本一起计算的，三个样本构成一个夹角，要保证这个夹角被student映射后跟teacher映射的一样。他用余弦距离表示角度，然后也是用huber loss来作为损失。我感觉角度的loss好像不应该跟距离的loss这么对称
* 他说实验中发现，用了角度的关系计算，收敛会更快，性能也更好
##Experiments
* 提到要把样本的feature都正则化成单位向量，这倒是让距离跟角度匹配起来了，都是完全相反的时候差距为2
* 实验还是存在那个问题，就是结果不太整齐，对于有的网络只用距离关系好，有的只用角度关系好，有的就两个都用最好。不过效果提升挺明显。
* 此外，发现正则化成单位向量反而会降低精度。这个观察倒是很整齐。他的解释是从更大的embedding空间优化效果会更好，我不太能接受。考虑到他的huber loss没有正则化的时候可能大部分都处于l1距离的那一段，是否说明这样子的损失更有利于优化，进一步说明很多都样本的距离是不够稳定的（因此用l2损失算出来的巨大梯度对精度造成了负面影响）？从这个角度考虑，他就没有Correlation Congruence for Knowledge Distillation这篇考虑的全面
* 做了自蒸馏的实验。很有意思的是一次蒸馏提升很多很多，但是继续蒸馏效果反而会降低。他说在同主干网的前提下，他的方法很多都是soa
* 提到了这样的实验。他在一个数据集上做某种训练（简单的分类预训练，triplet loss，还有他的kd），然后再另一个数据集上测试这个模型提取的特征的可分程度，结果发现他的kd效果比较差，而且在原数据集上效果越好的训练方法表现就越差。
* 在分类上，别人方法的基础上用了他的方法都会有一定的提升。
* 有一个发现就是，其他人花里胡哨的方法好像还没有原始kd的效果好？
