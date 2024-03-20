# attention
## 输入
X:**batch_size** * **seq_len** * **input_dim**
W_q:**batch_size** * **input_dim** * **dim_k**
W_k:**batch_size** * **input_dim** * **dim_k**
W_v:**batch_size** * **input_dim** * **dim_v**
Q:**batch_size** * **seq_len** * **dim_k**
K:**batch_size** * **seq_len** * **dim_k**
V:**batch_size** * **seq_len** * **dim_v**

## 具体操作
$Q,K$都是在X的基础上通过线性变换的来的，目的是提高提高模型的拟合能力。
$XX^T$的意义是X中的每一个行向量与自己与其他行向量做点乘，反应了每一个$X$中行向量之间的相似度，记$XX^T=Y$,那么Y中的第一个行向量的就是X中第一个行向量和其它所有行向量的相似度，记X的第一个行向量为$X_1$,那么$X_1$的第一个分量代表它与自身的相似度，第二个分量代表它与$X$中第二个行向量的相似度，以此类推。在$Softmax(XX^T)$之后可以把每一行的每一个分量转换为对应的0-1区间上的权重值,$Softmax(XX^T)X=Z$,$Z_{i,j}$就表示对$X$中所有行向量的第$j$个分量经过$Y$的第$i$行加权求和得来的第$i$个样本一次编码后的表示，它考虑了和所有向量的相关度。

$X$的形状是**batch_size** * **seq_len** * **input_dim**,
$X^T$的形状是**batch_size** * **input_dim** * **seq_len**,
则$Softmax(XX^T)=Y$的形状是**batch_size** * **seq_len** * **seq_len**,每个**seq_len** * **seq_len**矩阵中的第i行代表了第i个seq和其他seq之间的关联程度，反应出来就是数值高低。
$Softmax(XX^T)X$的形状是**batch_size** * **seq_len** * **input_dim**,注意到它的第i个行向量是由Y的第i个行向量乘X的每一个列向量得到的，所以其实就是把X中每一个seq的相同位置上分量加权求和了一遍，来重新对第i个seq进行表征。

Q,K,V同理。$Attention(Q,K,V)=Softmax(QK^T/\sqrt d_k)V$,$\sqrt d_k$是为了使$Softmax(QK^T)$的分布不会因为方差过大而陡峭。
在得到$Attention$后，经过残差连接和$layer\ normalization$就可以得到这一个encoder的输出，并将其作为下一个encoder的输入

## Swim Transformer
之前ViT的每一个patch在全局上的每一个patch做自注意力，复杂度与patch的数目成平方，也即O($N^2$)$N$是patch的数目，swim将每一个patch划分为固定大小，并只在本patch里做自注意力并与其他的patch进行拼接，因此复杂度是随patch线性变化的，也即$O(N)$