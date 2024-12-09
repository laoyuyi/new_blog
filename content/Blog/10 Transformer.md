#### Transformer

[Transformer从零详细解读(可能是你见过最通俗易懂的讲解)](https://www.bilibili.com/video/BV1Di4y1c7Zm/?share_source=copy_web&vd_source=59e03da0555efae2883b4a6f21e8f47b)
[手推transformer_哔哩哔哩](https://www.bilibili.com/video/BV1UL411g7aX/?spm_id_from=333.337.search-card.all.click&vd_source=3749948871311220e961685a5ac2a7a7)

---

- Transformer结构框架图：
![[assets/Transformer.png|250]]

- **输入部分** 

  (1) 词嵌入($embedding$)： 
  > [!note] 注释
  > 两边的$Inputs$和$Outputs$分别代表样本和标签数据，$Embedding$是一个单层的全连接层，$Inputs$和$Outputs$经过$Embedding$的转换后可以表示单词或者语句的语义

  (2) 位置编码($positional$ $encoding$)：
 
   词向量偶数位置使用$\sin$，奇数位置使用$\cos$，将位置信息嵌入词向量中，原始版本的Transformer采用三角函数编码($Sinusoidal$ $Encoding$)：
$$PE_{(pos,2i)}=\sin(\frac{pos}{10000^{2i/d_{model}}})$$
$$PE_{(pos,2i+1)}=\cos(\frac{pos}{10000^{2i/d_{model}}})$$
$$
\begin{matrix}
	sin & cos & sin & cos & sin & cos\\
	↓ & ↓ & ↓ & ↓ & ↓ & ↓ &\\
	[0, & 1, & 2, & 3, & 4, & 5, & \cdots &]
\end{matrix}
$$
> [!note] 注释
> $pos$表示位置索引（从$0$开始），$𝑖$是词嵌入向量的维度索引，$𝑑_{𝑚𝑜𝑑𝑒𝑙}$是模型中词嵌入向量的维度。$带有位置信息的词embedding = 原始的词embedding+词embedding对应的PE计算结果$。RNN和LSTM是将文本一个词一个词输入网络，而transformer是并行计算的，意味着词与词之间不存在顺序关系，所以需要将位置信息嵌入词向量中，PE嵌入的是却对位置信息，通过频率、波长与位置的关系把信息嵌入词向量中。


- **Encoder部分**

  整个Transformer可以分为$Encoder$和$Decoder$两个部分。其中，$Encoder$可分成输入部分、注意力机制和前馈神经网络三个部分

  (1) 自注意力机制($Self$-$Attention$)：[zhihu](https://zhuanlan.zhihu.com/p/410776234)

   $Q(query)、K(key)、V(value)以及Attention$的计算：
$$Q = X \times W^Q$$
$$K = X \times W^K$$
$$V = X \times W^V$$
$$Attention(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}})V$$
> [!note] 注释
> $X$是经过词嵌入与位置编码后的输入，$W^Q$、$W^K$、$W^V$是三个可被训练的参数矩阵。$d_k$是键对应的向量维度，即$embedding$的维度。关于$Q、K、V$的作用：[※](https://www.zhihu.com/question/325839123)。$QK^T$表示查询权重，$Q$乘$K^T$相当于$Q$与$K$做数量积操作，数量级可以反映两个向量的相似度，结果越大表明两个向量的信息相似度越高，$softmax$的作用是确保权重之和为$1$，为了避免规模差异对相似度计算产生影响，通常会对向量进行缩放。为什么要除以$\sqrt{d_k}$？

  (2) 多头自注意力机制($Multi$-$Head$ $Self$-$Attention$)：[CSDN](https://blog.csdn.net/weixin_45662399/article/details/134384186)

  自注意力机制在对当前位置信息进行编码时，会过度的将注意力集中于自身的位置，因此引入多头自注意力机制。多头自注意力机制的步骤如下：

  首先，定义$n$组$W^Q_n、W^K_n和W^V_n$，生成多组$Q_n、K_n和V_n:$
$$Q_n = X \times W^Q_n$$
$$K_n = X \times W^K_n$$
$$V_n = X \times W^V_n$$
对多组$Q、K、V$进行$attention$计算，生成多个$Attention:$
$$Attention_n(Q_n,K_n,V_n)=softmax(\frac{Q_nK_n^T}{\sqrt{d_k}})V_n$$
将多组$Attention(Attention_0,Attention_1,...,Attention_n)$进行拼接(cancat)，再乘以矩阵$W^O$做一次线性变换降低维度，得到最终的$MultiHead:$
$$MultiHead = Concat(Attention_0,Attention_1,...,Attention_n)W^O$$
> [!note] 注释
> `nhead`需要能被`d_model`整除，因为所有头的维度之和等于`d_model`。即每个注意力头的输出维度是`d_model/nhead`.

  (3) 残差结构与前馈神经网络：

  前馈神经网络公式：
$$FFN(x) = max(0,xW_1+b_1)W_2+b_2$$
  > [!note] 注释
> 激活函数为ReLU。残差结构参考[CSDN](https://blog.csdn.net/qimo601/article/details/127410607)。

- **Dncoder部分**

  (1) $Masked$ $Multi$-$Head$ $Self$-$Attention$：
  
  $mask$-$attention$的计算公式如下：
$$Attention_{mask,n}(Q,K,V)=softmax(\frac{QK^T}{\sqrt{d_k}}+mask)V$$
$$
mask = 
\begin{pmatrix}
	0 & -∞ & -∞ & \cdots & -∞ \\
	0 & 0 & -∞ & \cdots & -∞ \\
	0 & 0 & 0 & \cdots & -∞\\
	\vdots &\vdots& \vdots&\ddots &\vdots\\
	0 & 0 & 0 & \cdots & 0 
\end{pmatrix}
$$
> [!note] 注释
> 由于transformer在训练时是将数据直接全部输入网络，所以需要使decoder看不到未来的信息。计算很简单，就是加一个mask矩阵

  (2) $Cross$-$Attention$：

  $Cross$-$attention$的计算公式如下：
$$Attention_n(Q_{Decoder},K_{Encoder},V_{Encoder})=softmax(\frac{QK^T}{\sqrt{d_k}}+mask)V$$
> [!note] 注释
> Encoder的最后一个模块的输出，输入进每个Decoder中，作为Decoder Multi-Head Self-Attention的$Key$和$Value$

- 详细过程：
![[assets/Transformer_Big.png]]

- **Transformer相关变体**
$LLaMA$

  $LLaMA$是由$Meta$公司开源的大语言模型，于$2023$年$2$月$25$日发布，包括$7B、13B和70B$三个版本。$LLaMA$是Transformer的变体，相比Transformer，其结构与特点如下：

  (1) $Encoder$-$Decoder$ $\Rightarrow$ $Decoder$-$Only$：
  > [!note] 注释
> Llama模型仅采用了Transfomer的解码器结构，相比Transfomer的样本和标签数据分别从编码器和解码器输入，$LLaMA$直接将样本和标签拼接在一起输入到神经网络

  (2) $LayerNorm$ $and$ $Post$-$norm$ $\Rightarrow$ $RMSNorm$ $and$ $Pre$-$norm$：
  > [!note] 注释
> 相比Transform的$LayerNorm$，$LLaMA$模型采用$RMSNorm$，$RMSNorm$比$Layernorm$计算更简便，节约了计算速度，此外，$LLaMA$还将$nomilization$层放在了注意力的前面，为啥不知道

  (3) 三角函数编码 → 旋转位置编码(RoPE)：[论文](http://arxiv.org/abs/2104.09864)[科学空间](https://kexue.fm/archives/10040)

   以二维词向量为例，将自注意力计算过程中的的$q$和$k$与二维旋转矩阵$R^{2×2}$相乘：
$$
R_n^{2×2} = 
\begin{pmatrix}
	cos(nθ) & -sin(nθ) \\
	sin(nθ) & cos(nθ)
\end{pmatrix}
$$
$$(R_m^{2×2}q)^⊤(R_n^{2×2}k)=q^{⊤}(R^{2×2}_m)^TR_n^{2×2}k=q^⊤R_{n−m}^{2×2}k$$
> [!note] 注释
> 其中，$θ$是一个常量，它的具体原理是让词向量在空间上发生旋转，从而把位置信息嵌入到词向量中，这也是为什么它叫旋转位置编码。可以看到相对位置信息$(n−m)$在$q$与$k$相乘后嵌入到了词向量中，形象过程可以看下图：

![[assets/RoPE.png|500]]

- 词向量一般不可能是二维，要想对高维词向量进行旋转，需要将旋转矩阵拓展到高维：

$$R_n^{d×d} =
\begin{pmatrix}
	cos(nθ_0) & -sin(nθ_0) & 0 & 0 & \cdots & 0 & 0\\
	sin(nθ_0) & cos(nθ_0) & 0 & 0 & \cdots & 0 & 0\\
	0 & 0 & sin(nθ_1) & cos(nθ_1) & \cdots & 0 & 0\\
	0 & 0 & sin(nθ_1) & cos(nθ_1) & \cdots & 0 & 0\\
	\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots\\
	0 & 0 & 0 & 0 & \cdots & cos(nθ_{d/2-1}) & -sin(nθ_{d/2-1})\\
	0 & 0 & 0 & 0 & \cdots & sin(nθ_{d/2-1}) & cos(nθ_{d/2-1})\\
\end{pmatrix}
$$

- 可以应用分块矩阵将旋转矩阵简化：

$$
\begin{align}
R_n^{d×d}
&=
\left( 
\begin{array}{cc|ccc}
	cos(nθ_0) & -sin(nθ_0) & 0 & 0 & \cdots & 0 & 0\\
	sin(nθ_0) & cos(nθ_0) & 0 & 0 & \cdots & 0 & 0\\
	\hline 
	0 & 0 & sin(nθ_1) & cos(nθ_1) & \cdots & 0 & 0\\
	0 & 0 & sin(nθ_1) & cos(nθ_1) & \cdots & 0 & 0\\
	\vdots & \vdots & \vdots & \vdots & \ddots & \vdots & \vdots\\
	0 & 0 & 0 & 0 & \cdots & cos(nθ_{d/2-1}) & -sin(nθ_{d/2-1})\\
	0 & 0 & 0 & 0 & \cdots & sin(nθ_{d/2-1}) & cos(nθ_{d/2-1})
\end{array}
\right)\\\\
&= 
\begin{pmatrix}
	R_{(0)} & 0 & 0 & \cdots & 0\\
	0 & R_{(1)} & 0 & \cdots & 0\\
	0 & 0 & R_{(2)} & \cdots & 0\\
	0 & 0 & 0 & \ddots & \vdots\\
	0 & 0 & 0 & \cdots & R_{(d/2-1)}
\end{pmatrix}
\end{align}
$$
- 计算公式改写为以下形式，不展开了：
$$(R_m^{d×d}q)^⊤(R_n^{d×d}k)=q^{⊤}(R^{d×d}_m)^TR_n^{d×d}k=q^⊤R_{n−m}^{d×d}k$$
> [!note] 注释
> prompt的理解是另外一个小型的训练样本适时的调整大模型的参数

- $BERT$

  (1) $Encoder$-$Decoder$ $\Rightarrow$ $Encoder$-$Only$：
  > [!note] 注释
> BERT模型就是Transformer左边的Encodeer结构，2018年10⽉由Google AI研究院提出，其架构图如下：

![[assets/Bert.png|600]]

- $GPT$

  (1) $Encoder$-$Decoder$ $\Rightarrow$ $Decoder$-$Only$：
  > [!note] 注释
> GPT模型，全称为‌(Generative Pre-trained Transformer)，是由‌OpenAI团队开发的一种基于深度学习的自然语言处理模型。GPT使用Transformer的Decoder作为其架构，但在Decoder的基础上做了一些改动，去掉了Cross-Attention。

- T5模型(Transfer Text-to-Text Transformer)
训练数据：高质量数据C4

---

![[assets/v2-7e06234980fe84aca696efceab3a5b06_1440w.png|800]]
全参微调、LoRA微调