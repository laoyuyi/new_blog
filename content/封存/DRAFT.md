<div class="RichText ztext Post-RichText css-ob6uua" options="[object Object]">
    <p data-first-child data-pid="sEgqYL-t">去噪扩散概率模型（Denoising Diffusion
        Probabilistic Model,
        DDPM）在2020年被提出，向世界展示了扩散模型的强大能力，带动了扩散模型的火热。笔者出于兴趣自学相关知识，结合网络上的参考资料和自己的理解介绍DDPM。<b>需要说明的是，笔者能力很有限，学习过程中遇到了很多知识盲区，只能硬着头皮现学现卖。如果发现文中有错误，欢迎指出，大家共同进步。</b>
    </p>
    <hr />
    <h2 id="h_636776166_0" data-into-catalog-status="">前置知识</h2>
    <h3 id="h_636776166_1" data-into-catalog-status="">① 贝叶斯法则</h3>
    <p data-pid="4ZoIClKl">先验概率 <span class="ztext-math" data-eeimg="1" data-tex="P\left( \theta \right)">P\left( \theta
            \right)</span>
        ：根据以往的经验和分析得到的，假设 <span class="ztext-math" data-eeimg="1" data-tex="\theta">\theta</span> 发生的概率；</p>
    <p data-pid="04wVUz_z">似然函数 <span class="ztext-math" data-eeimg="1" data-tex="P\left( x\mid \theta \right)">P\left(
            x\mid \theta
            \right)</span> ：在假设 <span class="ztext-math" data-eeimg="1" data-tex="\theta">\theta</span> 已发生的前提下，发生事件
        <span class="ztext-math" data-eeimg="1" data-tex="x">x</span> 的概率；
    </p>
    <p data-pid="iH42Rf6U">后验概率 <span class="ztext-math" data-eeimg="1" data-tex="P\left( \theta\mid x \right)">P\left(
            \theta\mid x
            \right)</span> ：在事件 <span class="ztext-math" data-eeimg="1" data-tex="x">x</span> 已发生的前提下，假设 <span
            class="ztext-math" data-eeimg="1" data-tex="\theta">\theta</span> 成立的概率；</p>
    <p data-pid="ArU2xDMN">标准化常量 <span class="ztext-math" data-eeimg="1" data-tex="P\left( x \right)">P\left( x
            \right)</span> ：在已知所有假设下，事件 <span class="ztext-math" data-eeimg="1" data-tex="x">x</span> 发生的概率；</p>
    <p data-pid="YVc0uy2g">它们之间的关系为： <span class="ztext-math" data-eeimg="1"
            data-tex="P\left( \theta \mid x \right) = \frac{P\left( x \mid \theta \right)P\left( \theta \right)}{P\left( x \right)}">P\left(
            \theta \mid x \right) = \frac{P\left( x \mid \theta \right)P\left(
            \theta \right)}{P\left( x \right)}</span> </p>
    <h3 id="h_636776166_2" data-into-catalog-status="">② 贝叶斯公式</h3>
    <p data-pid="h8H8MdAQ"><span class="ztext-math" data-eeimg="1"
            data-tex="P(A,B) = P(B\mid A)P(A)=P(A\mid B)P(B)">P(A,B) = P(B\mid
            A)P(A)=P(A\mid B)P(B)</span> </p>
    <p data-pid="fRY3kIVW"><span class="ztext-math" data-eeimg="1"
            data-tex="P(A,B,C) = P(C\mid B,A)P(B,A)=P(C\mid B,A)P(B\mid A)P(A)">P(A,B,C)
            = P(C\mid B,A)P(B,A)=P(C\mid B,A)P(B\mid A)P(A)</span> </p>
    <p data-pid="h3isaQ4J"><span class="ztext-math" data-eeimg="1"
            data-tex="P(B,C\mid A) = P(B\mid A)P(C\mid A,B)">P(B,C\mid A) = P(B\mid
            A)P(C\mid A,B)</span> </p>
    <p data-pid="HDh6hTzo">若满足马尔科夫链关系 <span class="ztext-math" data-eeimg="1"
            data-tex="A\rightarrow B\rightarrow C">A\rightarrow B\rightarrow
            C</span> ，即当前时刻的概率分布仅与上一时刻有关，则有：</p>
    <p data-pid="vTSMHn-R"><span class="ztext-math" data-eeimg="1"
            data-tex="P(A,B,C) = P(C\mid B,A)P(B,A)=\color{red}{P(C\mid B)}P(B\mid A)P(A)">P(A,B,C)
            = P(C\mid B,A)P(B,A)=\color{red}{P(C\mid B)}P(B\mid A)P(A)</span> </p>
    <p data-pid="wywKA3aj"><span class="ztext-math" data-eeimg="1"
            data-tex="P(B,C\mid A) = P(B\mid A)\color{red}{P(C\mid B)}">P(B,C\mid A)
            = P(B\mid A)\color{red}{P(C\mid B)}</span> </p>
    <h3 id="h_636776166_3" data-into-catalog-status="">③ 高斯分布的概率密度函数、高斯函数的叠加公式</h3>
    <p data-pid="KcAmoSzd">给定均值为 <span class="ztext-math" data-eeimg="1" data-tex="\mu">\mu</span> ，方差为 <span
            class="ztext-math" data-eeimg="1" data-tex="\sigma^{2}">\sigma^{2}</span> 的单一变量高斯分布 <span class="ztext-math"
            data-eeimg="1" data-tex="\mathcal{N}(\mu , \sigma ^{2})">\mathcal{N}(\mu , \sigma
            ^{2})</span> ，其概率密度函数为：</p>
    <p data-pid="64KEBwlJ"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x) = \frac{1}{\sqrt{2\pi }\sigma }\exp \left ( -\frac{1}{2}\left ( \frac{x-\mu }{\sigma } \right )^2 \right )">q(x)
            = \frac{1}{\sqrt{2\pi }\sigma }\exp \left ( -\frac{1}{2}\left (
            \frac{x-\mu }{\sigma } \right )^2 \right )</span> </p>
    <p data-pid="CqLPgzlM">很多时候，为了方便起见，可以将前面的常数系数去掉，写成：</p>
    <p data-pid="u07G_Zeu"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x) \propto exp\left ( -\frac{1}{2}\left ( \frac{x-\mu }{\sigma } \right )^2 \right ) \ \ \Leftrightarrow \ \ q(x) \propto \exp \left ( -\frac{1}{2}\left ( \frac{1}{\sigma ^{2}}x^2 - \frac{2\mu }{\sigma ^{2}}x + \frac{\mu ^{2}}{\sigma ^{2}} \right ) \right )">q(x)
            \propto exp\left ( -\frac{1}{2}\left ( \frac{x-\mu }{\sigma } \right )^2
            \right ) \ \ \Leftrightarrow \ \ q(x) \propto \exp \left (
            -\frac{1}{2}\left ( \frac{1}{\sigma ^{2}}x^2 - \frac{2\mu }{\sigma
            ^{2}}x + \frac{\mu ^{2}}{\sigma ^{2}} \right ) \right )</span> </p>
    <p data-pid="jipt36Aw">给定两个高斯分布 <span class="ztext-math" data-eeimg="1"
            data-tex="X\sim \mathcal{N}(\mu_{1} , \sigma_{1} ^{2})">X\sim
            \mathcal{N}(\mu_{1} , \sigma_{1} ^{2})</span> ， <span class="ztext-math" data-eeimg="1"
            data-tex="Y\sim \mathcal{N}(\mu_{2} , \sigma_{2} ^{2})">Y\sim
            \mathcal{N}(\mu_{2} , \sigma_{2} ^{2})</span> ，则它们叠加后的 <span class="ztext-math" data-eeimg="1"
            data-tex="aX+bY">aX+bY</span> 满足：</p>
    <p data-pid="vFT35vQy"><span class="ztext-math" data-eeimg="1"
            data-tex="aX+bY\sim \mathcal{N}(a\times \mu _{1} + b \times \mu_{2},a^{2} \times \sigma _{1}^{2} + b^{2} \times \sigma _{2}^{2})">aX+bY\sim
            \mathcal{N}(a\times \mu _{1} + b \times \mu_{2},a^{2} \times \sigma
            _{1}^{2} + b^{2} \times \sigma _{2}^{2})</span></p>
    <h3 id="h_636776166_4" data-into-catalog-status="">④ KL散度</h3>
    <p data-pid="E8jFPZX6">KL散度可以用来衡量两个分布的差异，假设随机变量的真实概率分布为 <span class="ztext-math" data-eeimg="1"
            data-tex="P">P</span> ，而我们通过建模得到的一个近似分布为 <span class="ztext-math" data-eeimg="1" data-tex="Q">Q</span> ，则
        <span class="ztext-math" data-eeimg="1" data-tex="P">P</span> 与 <span class="ztext-math" data-eeimg="1"
            data-tex="Q">Q</span> 的KL散度满足下式：
    </p>
    <p data-pid="G6RU6Ql8"><span class="ztext-math" data-eeimg="1"
            data-tex="D_{KL}(P,Q) = -\sum P\log Q - (-\sum P\log P) = \sum P \log \frac{P}{Q}">D_{KL}(P,Q)
            = -\sum P\log Q - (-\sum P\log P) = \sum P \log \frac{P}{Q}</span></p>
    <p data-pid="bvoY8IqT">对于两个单一变量的高斯分布 <span class="ztext-math" data-eeimg="1"
            data-tex="p\sim \mathcal{N}(\mu _{1}, \sigma _{1}^{2})">p\sim
            \mathcal{N}(\mu _{1}, \sigma _{1}^{2})</span> 和 <span class="ztext-math" data-eeimg="1"
            data-tex="q\sim \mathcal{N}(\mu _{2}, \sigma _{2}^{2})">q\sim
            \mathcal{N}(\mu _{2}, \sigma _{2}^{2})</span> 而言，它们的KL散度为：</p>
    <p data-pid="IZkVFWl1"><span class="ztext-math" data-eeimg="1"
            data-tex="D_{KL}(p,q) = \log \frac{\sigma _{2}}{\sigma _{1}} + \frac{\sigma _{1}^{2} + (\mu _{1} - \mu _{2})^{2}}{2 \sigma _{2}^{2}} - \frac{1}{2}">D_{KL}(p,q)
            = \log \frac{\sigma _{2}}{\sigma _{1}} + \frac{\sigma _{1}^{2} + (\mu
            _{1} - \mu _{2})^{2}}{2 \sigma _{2}^{2}} - \frac{1}{2}</span></p>
    <h3 id="h_636776166_5" data-into-catalog-status="">⑤ 参数重整化（重参数技巧）</h3>
    <p data-pid="-Pu05ryj">若要从高斯分布 <span class="ztext-math" data-eeimg="1"
            data-tex="\mathcal{N}(\mu ,\sigma^{2} )">\mathcal{N}(\mu ,\sigma^{2}
            )</span> 中采样，可先从标准分布 <span class="ztext-math" data-eeimg="1" data-tex="\mathcal{N}(0 ,1)">\mathcal{N}(0
            ,1)</span> 中采样出 <span class="ztext-math" data-eeimg="1" data-tex="z">z</span> ，再得到 <span class="ztext-math"
            data-eeimg="1" data-tex="\sigma \ast z + \mu">\sigma
            \ast z + \mu</span> ，即我们的采样值。这样做的目的是将随机性转移到 <span class="ztext-math" data-eeimg="1" data-tex="z">z</span>
        上，让采样值对 <span class="ztext-math" data-eeimg="1" data-tex="\mu">\mu</span> 和 <span class="ztext-math"
            data-eeimg="1" data-tex="\sigma">\sigma</span> 可导。</p>
    <hr />
    <h2 id="h_636776166_6" data-into-catalog-status="">基本介绍</h2>
    <p data-pid="2p3kb4RU">如下图所示，DDPM模型主要分为两个过程：<b>加噪过程（从右往左）</b>和<b>去噪过程（从左往右）</b>。
    </p>
    <p data-pid="4KEkXCma">★ 加噪过程：给定真实图像 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        ，逐步对它添加高斯噪声，得到 <span class="ztext-math" data-eeimg="1" data-tex="x_{1},\ x_{2},\ \cdots">x_{1},\ x_{2},\
            \cdots</span> ，显然这是一个马尔科夫链过程，在进行了足够多的 <span class="ztext-math" data-eeimg="1" data-tex="T">T</span>
        次加噪后，图像会被高斯噪声淹没，可以认为是各向独立的高斯噪声的图像。
    </p>
    <p data-pid="3eG59NKP">★ 去噪过程：针对噪声图像 <span class="ztext-math" data-eeimg="1" data-tex="x_{T}">x_{T}</span>
        ，让神经网络模型对其逐步去噪，得到 <span class="ztext-math" data-eeimg="1" data-tex="x_{T-1},\ x_{T-2},\ \cdots">x_{T-1},\
            x_{T-2},\
            \cdots</span> ，最终复原出没有噪声的逼真图像 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        ，所以<b>加噪过程其实可以看作是在为去噪过程构建标签</b>。</p>
    <figure data-size="normal"><img src="https://pica.zhimg.com/v2-694b74ff67753c0909ce299376c62e72_1440w.jpg"
            data-caption="" data-size="normal" data-rawwidth="1526" data-rawheight="346" data-qrcode-action="none"
            data-original-token="v2-2be09aeee412ad29208ec3e7e6c74ed0" class="origin_image zh-lightbox-thumb"
            width="1526" data-original="https://pica.zhimg.com/v2-694b74ff67753c0909ce299376c62e72_r.jpg" />
    </figure>
    <hr />
    <h2 id="h_636776166_7" data-into-catalog-status="">前向过程（扩散过程，加噪过程）</h2>
    <p data-pid="KIR8mE3R">给定初始图像 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        ，向其中逐步添加高斯噪声，加噪过程持续 <span class="ztext-math" data-eeimg="1" data-tex="T">T</span>
        次，产生一系列带噪图像，达到破坏图像的目的。由 <span class="ztext-math" data-eeimg="1" data-tex="x_{t-1}">x_{t-1}</span> 加噪至 <span
            class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 的过程中，所加噪声的方差为 <span class="ztext-math"
            data-eeimg="1" data-tex="\beta_{t}">\beta_{t}</span>
        ，又称<b>扩散率</b>，是一个<b>给定的</b>，大于 0 小于 1 的，随扩散步数增加而逐渐增大的值。定义扩散过程如下式：</p>
    <p data-pid="AOtnxj2Q"><span class="ztext-math" data-eeimg="1"
            data-tex="x_{t}=\sqrt{1-\beta _{t}}x_{t-1}+\sqrt{\beta _{t}}z _{t},\hspace{1.5em}z_{t}\sim \mathcal N(0,\boldsymbol{I})">x_{t}=\sqrt{1-\beta
            _{t}}x_{t-1}+\sqrt{\beta _{t}}z _{t},\hspace{1.5em}z_{t}\sim \mathcal
            N(0,\boldsymbol{I})</span> </p>
    <p data-pid="WUdQbodf">根据定义，加噪过程可以看作在上一步的基础上乘了一个系数，然后加上均值为 0 ，方差为 <span class="ztext-math" data-eeimg="1"
            data-tex="\beta_{t}">\beta_{t}</span>
        的高斯分布。所以加噪过程是确定的，并不是可学习的过程，将其写成概率分布的形式，则有：</p>
    <p data-pid="AvDzSOEv"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t}\mid x_{t-1}) = \mathcal{N} (x_{t}; \sqrt{1 - \beta _{t}}x_{t-1}, \beta_{t}\boldsymbol{I})">q(x_{t}\mid
            x_{t-1}) = \mathcal{N} (x_{t}; \sqrt{1 - \beta _{t}}x_{t-1},
            \beta_{t}\boldsymbol{I})</span> </p>
    <p data-pid="lHnk29yu">此外，加噪过程是一个马尔科夫链过程，所以联合概率分布可以写成下式：</p>
    <p data-pid="appaWVCJ"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{1},x_{2},\cdots ,x_{T} | x_{0}) = q(x_{1} | x_{0})q(x_{2} | x_{1})\cdots q(x_{T}| x_{T-1}) = \prod_{t=1}^{T}q(x_{t}| x_{t-1})">q(x_{1},x_{2},\cdots
            ,x_{T} | x_{0}) = q(x_{1} | x_{0})q(x_{2} | x_{1})\cdots q(x_{T}|
            x_{T-1}) = \prod_{t=1}^{T}q(x_{t}| x_{t-1})</span> </p>
    <p data-pid="sJ20l3LB">定义 <span class="ztext-math" data-eeimg="1" data-tex="\alpha _{t} = 1 - \beta _{t}">\alpha
            _{t} = 1 - \beta
            _{t}</span> ，即 <span class="ztext-math" data-eeimg="1" data-tex="\alpha _{t} + \beta _{t} = 1">\alpha _{t} +
            \beta _{t} =
            1</span> ，代入 <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 表达式并迭代推导，可得 <span
            class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 到 <span class="ztext-math" data-eeimg="1"
            data-tex="x_{t}">x_{t}</span> 的公式：</p>
    <p data-pid="geFW99AD"><span class="ztext-math" data-eeimg="1"
            data-tex="x_{t} = \sqrt{1-\beta _{t}}x_{t-1} + \sqrt{\beta _{t}}z _{t} = \sqrt{\alpha _{t}}x_{t-1}+\sqrt{\beta _{t}}z _{t}">x_{t}
            = \sqrt{1-\beta _{t}}x_{t-1} + \sqrt{\beta _{t}}z _{t} = \sqrt{\alpha
            _{t}}x_{t-1}+\sqrt{\beta _{t}}z _{t}</span> </p>
    <p data-pid="YxBVDkt_"><span class="ztext-math" data-eeimg="1"
            data-tex="= \sqrt{\alpha _{t}}\color{red}{(\sqrt{\alpha _{t-1}}x_{t-2} + \sqrt{\beta _{t-1}}z _{t-1})} + \sqrt{\beta _{t}}z _{t}">=
            \sqrt{\alpha _{t}}\color{red}{(\sqrt{\alpha _{t-1}}x_{t-2} + \sqrt{\beta
            _{t-1}}z _{t-1})} + \sqrt{\beta _{t}}z _{t}</span> </p>
    <p data-pid="nTmWN_xb"><span class="ztext-math" data-eeimg="1"
            data-tex="=\sqrt{\alpha _{t}\alpha _{t-1}}x_{t-2}+\sqrt{\alpha _{t}\beta _{t-1}}z _{t-1}+\sqrt{\beta _{t}}z _{t}">=\sqrt{\alpha
            _{t}\alpha _{t-1}}x_{t-2}+\sqrt{\alpha _{t}\beta _{t-1}}z
            _{t-1}+\sqrt{\beta _{t}}z _{t}</span> </p>
    <p data-pid="fV5hz614"><span class="ztext-math" data-eeimg="1"
            data-tex="=\sqrt{\alpha _{t}\alpha _{t-1}}\color{red}{(\sqrt{\alpha _{t-2}}x_{t-3} + \sqrt{\beta _{t-2}}z _{t-2})} + \sqrt{\alpha _{t}\beta _{t-1}}z _{t-1} + \sqrt{\beta _{t}}z _{t}">=\sqrt{\alpha
            _{t}\alpha _{t-1}}\color{red}{(\sqrt{\alpha _{t-2}}x_{t-3} + \sqrt{\beta
            _{t-2}}z _{t-2})} + \sqrt{\alpha _{t}\beta _{t-1}}z _{t-1} + \sqrt{\beta
            _{t}}z _{t}</span> </p>
    <p data-pid="bd1tTTZq"><span class="ztext-math" data-eeimg="1"
            data-tex="=\sqrt{\alpha _{t}\alpha _{t-1}\alpha _{t-2}}x_{t-3} + \sqrt{\alpha _{t}\alpha _{t-1}\beta _{t-2}}z _{t-2} + \sqrt{\alpha _{t}\beta _{t-1}}z _{t-1} + \sqrt{\beta _{t}}z _{t}">=\sqrt{\alpha
            _{t}\alpha _{t-1}\alpha _{t-2}}x_{t-3} + \sqrt{\alpha _{t}\alpha
            _{t-1}\beta _{t-2}}z _{t-2} + \sqrt{\alpha _{t}\beta _{t-1}}z _{t-1} +
            \sqrt{\beta _{t}}z _{t}</span> </p>
    <p data-pid="0Vja1-NG"><span class="ztext-math" data-eeimg="1"
            data-tex="= \sqrt{\alpha _{t}\alpha _{t-1}\cdots \alpha _{1}}x_{0} + \sqrt{\alpha _{t}\alpha _{t-1}\cdots \alpha _{2}\beta _{1}}z _{1} + \sqrt{\alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2}}z _{2} + \cdots + \sqrt{\alpha _{t}\alpha _{t-1}\beta _{t-2}}z _{t-2} + \sqrt{\alpha _{t}\beta _{t-1}}z _{t-1} + \sqrt{\beta _{t}}z _{t}">=
            \sqrt{\alpha _{t}\alpha _{t-1}\cdots \alpha _{1}}x_{0} + \sqrt{\alpha
            _{t}\alpha _{t-1}\cdots \alpha _{2}\beta _{1}}z _{1} + \sqrt{\alpha
            _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2}}z _{2} + \cdots +
            \sqrt{\alpha _{t}\alpha _{t-1}\beta _{t-2}}z _{t-2} + \sqrt{\alpha
            _{t}\beta _{t-1}}z _{t-1} + \sqrt{\beta _{t}}z _{t}</span></p>
    <p data-pid="Q65L-oiO">
        上式从第二项到最后一项都是<b>独立的高斯噪声</b>，它们的均值都为0，方差为各自系数的平方。根据<b>高斯分布的叠加公式</b>，它们的<b>和</b>满足均值为0，方差为各项方差之和的高斯分布。又有上式每一项系数的平方和（包括第一项）为1，证明如下，注意始终有
        <span class="ztext-math" data-eeimg="1" data-tex="\alpha _{t} + \beta _{t} = 1">\alpha _{t} + \beta _{t} =
            1</span> ：
    </p>
    <p data-pid="lKCbRdm1"><span class="ztext-math" data-eeimg="1"
            data-tex="\alpha _{t}\alpha _{t-1}\cdots \alpha _{1} + \alpha _{t}\alpha _{t-1}\cdots \alpha _{2}\beta _{1} + \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2} + \cdots + \alpha _{t}\beta _{t-1} + \beta _{t}">\alpha
            _{t}\alpha _{t-1}\cdots \alpha _{1} + \alpha _{t}\alpha _{t-1}\cdots
            \alpha _{2}\beta _{1} + \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\beta
            _{2} + \cdots + \alpha _{t}\beta _{t-1} + \beta _{t}</span> </p>
    <p data-pid="iKiqKXOR"><span class="ztext-math" data-eeimg="1"
            data-tex="= \alpha _{t}\alpha _{t-1}\cdots \alpha _{2}(\alpha _{1} + \beta _{1}) + \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2} + \cdots + \alpha _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta _{t}">=
            \alpha _{t}\alpha _{t-1}\cdots \alpha _{2}(\alpha _{1} + \beta _{1}) +
            \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2} + \cdots + \alpha
            _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta
            _{t}</span> </p>
    <p data-pid="a8cotlrF"><span class="ztext-math" data-eeimg="1"
            data-tex="= \alpha _{t}\alpha _{t-1}\cdots \alpha _{2}\times \color{red}{1} + \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2} + \cdots + \alpha _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta _{t}">=
            \alpha _{t}\alpha _{t-1}\cdots \alpha _{2}\times \color{red}{1} + \alpha
            _{t}\alpha _{t-1}\cdots \alpha _{3}\beta _{2} + \cdots + \alpha
            _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta
            _{t}</span> </p>
    <p data-pid="m_VfBEbN"><span class="ztext-math" data-eeimg="1"
            data-tex="= \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}(\alpha _{2}+\beta _{2}) + \cdots + \alpha _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta _{t}">=
            \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}(\alpha _{2}+\beta _{2}) +
            \cdots + \alpha _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1}
            + \beta _{t}</span> </p>
    <p data-pid="pO0F9hxb"><span class="ztext-math" data-eeimg="1"
            data-tex="= \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\times \color{red}{1} + \cdots + \alpha _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta _{t}">=
            \alpha _{t}\alpha _{t-1}\cdots \alpha _{3}\times \color{red}{1} + \cdots
            + \alpha _{t}\alpha _{t-1}\beta _{t-2} + \alpha _{t}\beta _{t-1} + \beta
            _{t}</span> </p>
    <p data-pid="HdupdeuE"><span class="ztext-math" data-eeimg="1"
            data-tex="= \cdots \cdots = \alpha _{t} + \beta _{t} = 1">= \cdots
            \cdots = \alpha _{t} + \beta _{t} = 1</span> </p>
    <p data-pid="m3PBvGt4">那么，将 <span class="ztext-math" data-eeimg="1"
            data-tex="\alpha _{t}\alpha _{t-1}\cdots \alpha _{1}">\alpha _{t}\alpha
            _{t-1}\cdots \alpha _{1}</span> 记作 <span class="ztext-math" data-eeimg="1"
            data-tex="\bar{\alpha }_{t}">\bar{\alpha }_{t}</span>
        ，则正态噪声的方差之和为 <span class="ztext-math" data-eeimg="1" data-tex="1-\bar{\alpha }_{t}">1-\bar{\alpha }_{t}</span> ，
        <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 可表示为：
    </p>
    <p data-pid="BX855LZX"><span class="ztext-math" data-eeimg="1"
            data-tex="x_{t} = \sqrt{\bar{\alpha }_{t}}x_{0} + \sqrt{1-\bar{\alpha }_{t}}\bar{z}_{t},\hspace{1.5em}\bar{z}_{t} \sim \mathcal N(0,\boldsymbol{I})">x_{t}
            = \sqrt{\bar{\alpha }_{t}}x_{0} + \sqrt{1-\bar{\alpha
            }_{t}}\bar{z}_{t},\hspace{1.5em}\bar{z}_{t} \sim \mathcal
            N(0,\boldsymbol{I})</span> </p>
    <p data-pid="BjcBSltp">由该式可以看出， <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 实际上是原始图像 <span
            class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 和随机噪声 <span class="ztext-math"
            data-eeimg="1" data-tex="\bar{z}_{t}">\bar{z}_{t}</span>
        的线性组合，即只要给定初始值，以及每一步的扩散率，就可以得到任意时刻的 <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span>
        ，写成概率分布的形式：</p>
    <p data-pid="oW42VfGl"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t}\mid x_{0}) = \mathcal{N}(x_{t}; \sqrt{\bar{\alpha }_{t}}x_{0}, (1-\bar{\alpha }_{t})\boldsymbol{I})">q(x_{t}\mid
            x_{0}) = \mathcal{N}(x_{t}; \sqrt{\bar{\alpha }_{t}}x_{0},
            (1-\bar{\alpha }_{t})\boldsymbol{I})</span> </p>
    <p data-pid="cOsjvsYw">当加噪步数 <span class="ztext-math" data-eeimg="1" data-tex="T">T</span> 足够大时， <span
            class="ztext-math" data-eeimg="1" data-tex="\bar{\alpha }_{t}">\bar{\alpha }_{t}</span> 趋向于 0， <span
            class="ztext-math" data-eeimg="1" data-tex="1-\bar{\alpha }_{t}">1-\bar{\alpha }_{t}</span> 趋向于 1，所以 <span
            class="ztext-math" data-eeimg="1" data-tex="x_{T}">x_{T}</span>
        趋向于<b>标准高斯分布</b>。</p>
    <hr />
    <h2 id="h_636776166_8" data-into-catalog-status="">反向过程（逆扩散过程，去噪过程）</h2>
    <p data-pid="mOXRQVga">前向过程对原始图像 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 逐步加噪声变成 <span
            class="ztext-math" data-eeimg="1" data-tex="x_{T}">x_{T}</span> ，反向过程则是从 <span class="ztext-math"
            data-eeimg="1" data-tex="x_{T}">x_{T}</span> 逐步恢复到
        <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 。前向过程用
        <span class="ztext-math" data-eeimg="1" data-tex="q(x_{t}\mid x_{t-1})">q(x_{t}\mid x_{t-1})</span>
        来表示，那么反向过程则是求 <span class="ztext-math" data-eeimg="1" data-tex="q(x_{t-1}\mid x_{t})">q(x_{t-1}\mid
            x_{t})</span>
        。如果能实现这种逆转，就可以从一个随机的高斯噪声 <span class="ztext-math" data-eeimg="1"
            data-tex="\mathcal{N}(0,\boldsymbol{I})">\mathcal{N}(0,\boldsymbol{I})</span>
        中重建出一个真实的原始样本，实现<b>图像生成</b>的目的。
    </p>
    <p data-pid="1QRoXyBa">有文献证明，如果 <span class="ztext-math" data-eeimg="1" data-tex="q(x_{t}\mid x_{t-1})">q(x_{t}\mid
            x_{t-1})</span> 满足高斯分布且扩散率
        <span class="ztext-math" data-eeimg="1" data-tex="\beta_{t}">\beta_{t}</span> 足够小，则 <span class="ztext-math"
            data-eeimg="1" data-tex="q(x_{t}\mid x_{t-1})">q(x_{t}\mid
            x_{t-1})</span> 也满足高斯分布。虽然前向过程中每一步所加的噪声都采样自特定的高斯分布，但是采样有无数种可能性，所以 <span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}\mid x_{t})">q(x_{t-1}\mid x_{t})</span>
        不太可能从理论上求得，我们可以通过学习一个深度网络（参数为 <span class="ztext-math" data-eeimg="1" data-tex="\theta">\theta</span> ）来拟合。
    </p>
    <p data-pid="P8yGrfPF">反向过程仍然是一个马尔科夫链过程，网络以当前时刻 <span class="ztext-math" data-eeimg="1" data-tex="t">t</span>
        和当前时刻的状态 <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span>
        作为输入，构建反向过程条件概率，其中，均值和方差都是含参的，且都以 <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 和 <span
            class="ztext-math" data-eeimg="1" data-tex="t">t</span> 作为输入，有下式：</p>
    <p data-pid="Uo0vErUY"><span class="ztext-math" data-eeimg="1"
            data-tex="p_{\theta}(x_{t-1}\mid x_{t}) = \mathcal{N}\left ( x_{t-1}; \mu _{\theta}(x_{t}, t), \Sigma _{\theta}(x_{t}, t) \right )">p_{\theta}(x_{t-1}\mid
            x_{t}) = \mathcal{N}\left ( x_{t-1}; \mu _{\theta}(x_{t}, t), \Sigma
            _{\theta}(x_{t}, t) \right )</span> </p>
    <p data-pid="TGpbifhb"><span class="ztext-math" data-eeimg="1"
            data-tex="p_{\theta}(x_{0:T}) = p_{\theta}(x_{T})p_{\theta}(x_{T-1} \mid x_{T})\cdots p_{\theta}(x_{0}\mid x_{1}) = p_{\theta}(x_{T}) \prod_{t=1}^{T}p_{\theta}(x_{t-1}\mid x_{t})">p_{\theta}(x_{0:T})
            = p_{\theta}(x_{T})p_{\theta}(x_{T-1} \mid x_{T})\cdots
            p_{\theta}(x_{0}\mid x_{1}) = p_{\theta}(x_{T})
            \prod_{t=1}^{T}p_{\theta}(x_{t-1}\mid x_{t})</span> </p>
    <p data-pid="-p8Q3lCq">而<b>真实的反向过程</b>，或者称作<b>扩散过程的后验条件概率</b>，根据贝叶斯公式可以写成：</p>
    <p data-pid="iszdA3qL"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}\mid x_{t}) = q(x_{t}\mid x_{t-1}) \frac{q(x_{t-1})}{q(x_{t})}">q(x_{t-1}\mid
            x_{t}) = q(x_{t}\mid x_{t-1}) \frac{q(x_{t-1})}{q(x_{t})}</span> </p>
    <p data-pid="UG4HM-1R">其中， <span class="ztext-math" data-eeimg="1" data-tex="q(x_{t-1})">q(x_{t-1})</span>
        是不可知的，但是如果知道 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        ，则<b>扩散过程的后验条件概率</b>可以写成：</p>
    <p data-pid="45q-eN6E"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}\mid x_{t}, x_{0}) = \frac{q(x_{t}\mid x_{t-1}, x_{0})\times q(x_{t-1}\mid x_{0})}{q(x_{t}\mid x_{0})} = \mathcal{N}\left ( x_{t-1}, \color{blue}{\tilde{\mu }(x_{t}, x_{0})}, \color{red}{\tilde{\beta }_{t}}\boldsymbol{I} \right )">q(x_{t-1}\mid
            x_{t}, x_{0}) = \frac{q(x_{t}\mid x_{t-1}, x_{0})\times q(x_{t-1}\mid
            x_{0})}{q(x_{t}\mid x_{0})} = \mathcal{N}\left ( x_{t-1},
            \color{blue}{\tilde{\mu }(x_{t}, x_{0})}, \color{red}{\tilde{\beta
            }_{t}}\boldsymbol{I} \right )</span> </p>
    <p data-pid="jWnIgdU0">又根据前向过程的推导，有下面三个式子满足：</p>
    <p data-pid="aV3MEuP-"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}| x_{0}) = \sqrt{\bar{\alpha }_{t-1}}x_{0} + \sqrt{1-\bar{\alpha }_{t-1}}\bar{z}_{t-1} \hspace{0.8em} \sim \hspace{0.8em} \mathcal{N}\left ( x_{t-1}; \sqrt{\bar{\alpha }_{t-1}}x_{0}, \left ( 1-\bar{\alpha }_{t-1} \right )\boldsymbol{I} \right )">q(x_{t-1}|
            x_{0}) = \sqrt{\bar{\alpha }_{t-1}}x_{0} + \sqrt{1-\bar{\alpha
            }_{t-1}}\bar{z}_{t-1} \hspace{0.8em} \sim \hspace{0.8em}
            \mathcal{N}\left ( x_{t-1}; \sqrt{\bar{\alpha }_{t-1}}x_{0}, \left (
            1-\bar{\alpha }_{t-1} \right )\boldsymbol{I} \right )</span> </p>
    <p data-pid="ywF8Cdll"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t}\mid x_{0}) = \sqrt{\bar{\alpha }_{t}}x_{0} + \sqrt{1-\bar{\alpha }_{t}}\bar{z}_{t} \hspace{0.8em} \sim \hspace{0.8em} \mathcal{N}\left ( x_{t}; \sqrt{\bar{\alpha }_{t}}x_{0}, (1-\bar{\alpha }_{t})\boldsymbol{I} \right )">q(x_{t}\mid
            x_{0}) = \sqrt{\bar{\alpha }_{t}}x_{0} + \sqrt{1-\bar{\alpha
            }_{t}}\bar{z}_{t} \hspace{0.8em} \sim \hspace{0.8em} \mathcal{N}\left (
            x_{t}; \sqrt{\bar{\alpha }_{t}}x_{0}, (1-\bar{\alpha
            }_{t})\boldsymbol{I} \right )</span> </p>
    <p data-pid="sB9xAsJM"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t}\mid x_{t-1}, x_{0}) = q(x_{t}\mid x_{t-1}) = \sqrt{\alpha _{t}}x_{t-1} + \sqrt{\beta _{t}}z _{t} \hspace{0.8em} \sim \hspace{0.8em} \mathcal{N}\left ( x_{t}; \sqrt{\alpha _{t}}x_{t-1}, \beta _{t} \boldsymbol{I} \right )">q(x_{t}\mid
            x_{t-1}, x_{0}) = q(x_{t}\mid x_{t-1}) = \sqrt{\alpha _{t}}x_{t-1} +
            \sqrt{\beta _{t}}z _{t} \hspace{0.8em} \sim \hspace{0.8em}
            \mathcal{N}\left ( x_{t}; \sqrt{\alpha _{t}}x_{t-1}, \beta _{t}
            \boldsymbol{I} \right )</span> </p>
    <p data-pid="PtMJKaVe">将三个式子代入，并结合前置知识中的高斯函数概率密度函数，展开后合并同类项，有下式：</p>
    <p data-pid="SZUv20ra"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}\mid x_{t}, x_{0}) = \frac{\mathcal{N}(x_{t}; \sqrt{\alpha _{t}}x_{t-1}, \beta _{t} \boldsymbol{I}) \times \mathcal{N}(x_{t-1}; \sqrt{\bar{\alpha }_{t-1}}x_{0}, (1-\bar{\alpha }_{t-1})\boldsymbol{I})}{\mathcal{N}(x_{t}; \sqrt{\bar{\alpha }_{t}}x_{0}, (1-\bar{\alpha }_{t})\boldsymbol{I})}">q(x_{t-1}\mid
            x_{t}, x_{0}) = \frac{\mathcal{N}(x_{t}; \sqrt{\alpha _{t}}x_{t-1},
            \beta _{t} \boldsymbol{I}) \times \mathcal{N}(x_{t-1}; \sqrt{\bar{\alpha
            }_{t-1}}x_{0}, (1-\bar{\alpha
            }_{t-1})\boldsymbol{I})}{\mathcal{N}(x_{t}; \sqrt{\bar{\alpha
            }_{t}}x_{0}, (1-\bar{\alpha }_{t})\boldsymbol{I})}</span> </p>
    <p data-pid="9ypImCwe"><span class="ztext-math" data-eeimg="1"
            data-tex="\propto \exp \left ( -\frac{1}{2}\left ( \frac{(x_{t} - \sqrt{\alpha _{t}}x_{t-1})^{2}}{\beta _{t}} + \frac{(x_{t-1} - \sqrt{\bar{\alpha }_{t-1}}x_{0})^{2}}{1-\bar{\alpha }_{t-1}} - \frac{(x_{t} - \sqrt{\bar{\alpha }_{t}}x_{0})^{2}}{1 - \bar{\alpha }_{t}} \right ) \right )">\propto
            \exp \left ( -\frac{1}{2}\left ( \frac{(x_{t} - \sqrt{\alpha
            _{t}}x_{t-1})^{2}}{\beta _{t}} + \frac{(x_{t-1} - \sqrt{\bar{\alpha
            }_{t-1}}x_{0})^{2}}{1-\bar{\alpha }_{t-1}} - \frac{(x_{t} -
            \sqrt{\bar{\alpha }_{t}}x_{0})^{2}}{1 - \bar{\alpha }_{t}} \right )
            \right )</span> </p>
    <p data-pid="s3Qq2DIi"><span class="ztext-math" data-eeimg="1"
            data-tex="= \exp \left ( -\frac{1}{2}\left ( \frac{x_{t}^{2} - 2 \sqrt{\alpha _{t}}x_{t}\color{blue} {x_{t-1}} + \alpha_{t}\color{red} {x_{t-1}^{2}}}{\beta _{t}} + \frac{\color{red}{ x_{t-1}^{2}} - 2 \sqrt{\bar{\alpha }_{t-1}}x_{0}\color{blue}{x_{t-1}} + \bar{\alpha }_{t-1}x_{0}^{2}}{1-\bar{\alpha }_{t-1}} - \frac{(x_{t} - \sqrt{\bar{\alpha }_{t}}x_{0})^{2}}{1 - \bar{\alpha }_{t}} \right ) \right )">=
            \exp \left ( -\frac{1}{2}\left ( \frac{x_{t}^{2} - 2 \sqrt{\alpha
            _{t}}x_{t}\color{blue} {x_{t-1}} + \alpha_{t}\color{red}
            {x_{t-1}^{2}}}{\beta _{t}} + \frac{\color{red}{ x_{t-1}^{2}} - 2
            \sqrt{\bar{\alpha }_{t-1}}x_{0}\color{blue}{x_{t-1}} + \bar{\alpha
            }_{t-1}x_{0}^{2}}{1-\bar{\alpha }_{t-1}} - \frac{(x_{t} -
            \sqrt{\bar{\alpha }_{t}}x_{0})^{2}}{1 - \bar{\alpha }_{t}} \right )
            \right )</span> </p>
    <p data-pid="pDoJB6kj"><span class="ztext-math" data-eeimg="1"
            data-tex="=\exp \left ( -\frac{1}{2}\left ( \color{red} {\left ( \frac{\alpha _{t}}{\beta _{t}} + \frac{1}{1 - \bar{\alpha }_{t-1}} \right )}x_{t-1}^{2} - \color{blue} {\left ( \frac{2\sqrt{\alpha _{t}}}{\beta _{t}} x_{t}+ \frac{2 \sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha }_{t-1}}x_{0} \right )}x_{t-1} + \mathcal{C}\left ( x_{t}, x_{0} \right ) \right ) \right )">=\exp
            \left ( -\frac{1}{2}\left ( \color{red} {\left ( \frac{\alpha
            _{t}}{\beta _{t}} + \frac{1}{1 - \bar{\alpha }_{t-1}} \right
            )}x_{t-1}^{2} - \color{blue} {\left ( \frac{2\sqrt{\alpha _{t}}}{\beta
            _{t}} x_{t}+ \frac{2 \sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha
            }_{t-1}}x_{0} \right )}x_{t-1} + \mathcal{C}\left ( x_{t}, x_{0} \right
            ) \right ) \right )</span> </p>
    <p data-pid="RklgbtIJ">此式符合前置知识中高斯函数概率密度函数的展开形式，有以下两个式子满足：</p>
    <p data-pid="OTu05WZ8"><span class="ztext-math" data-eeimg="1"
            data-tex="\frac{1}{\tilde{\beta _{t}} ^{2}} = \color{red}{ \frac{\alpha _{t}}{\beta _{t}} + \frac{1}{1 - \bar{\alpha} _{t-1}}} \hspace{1em} and \hspace{1em} \frac{2 \tilde{\mu }(x_{t}, x_{0}) }{\tilde{\beta _{t}} ^{2}}=\color{blue} {\frac{2\sqrt{\alpha _{t}}}{\beta _{t}}x_{t} + \frac{2\sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha }_{t-1}}x_{0}}">\frac{1}{\tilde{\beta
            _{t}} ^{2}} = \color{red}{ \frac{\alpha _{t}}{\beta _{t}} + \frac{1}{1 -
            \bar{\alpha} _{t-1}}} \hspace{1em} and \hspace{1em} \frac{2 \tilde{\mu
            }(x_{t}, x_{0}) }{\tilde{\beta _{t}} ^{2}}=\color{blue}
            {\frac{2\sqrt{\alpha _{t}}}{\beta _{t}}x_{t} + \frac{2\sqrt{\bar{\alpha
            }_{t-1}}}{1 - \bar{\alpha }_{t-1}}x_{0}}</span> </p>
    <p data-pid="HC6FDYl6"><b>对第一个式子，有：</b></p>
    <p data-pid="0osm3AEX"><span class="ztext-math" data-eeimg="1"
            data-tex="\frac{1}{\tilde{\beta _{t}} ^{2}} = \frac{\alpha _{t}(1-\bar{\alpha }_{t-1}) + \color{darkred}{\beta _{t}}}{\beta _{t}(1 - \bar{\alpha }_{t-1})} = \frac{\alpha _{t} -\color{darkorange}{ \alpha _{t}\bar{\alpha }_{t-1}} + \color{darkred}{ 1 - \alpha _{t}}}{\beta _{t}(1 - \bar{\alpha }_{t-1})} = \frac{1 - \color{darkorange}{\bar{\alpha }_{t}}}{\beta _{t}(1 - \bar{\alpha }_{t-1})}">\frac{1}{\tilde{\beta
            _{t}} ^{2}} = \frac{\alpha _{t}(1-\bar{\alpha }_{t-1}) +
            \color{darkred}{\beta _{t}}}{\beta _{t}(1 - \bar{\alpha }_{t-1})} =
            \frac{\alpha _{t} -\color{darkorange}{ \alpha _{t}\bar{\alpha }_{t-1}} +
            \color{darkred}{ 1 - \alpha _{t}}}{\beta _{t}(1 - \bar{\alpha }_{t-1})}
            = \frac{1 - \color{darkorange}{\bar{\alpha }_{t}}}{\beta _{t}(1 -
            \bar{\alpha }_{t-1})}</span> </p>
    <p data-pid="UYIGDtIU"><b>对第二个式子，有：</b></p>
    <p data-pid="yI3S7tA2"><span class="ztext-math" data-eeimg="1"
            data-tex="\tilde{\mu }(x_{t}, x_{0}) = \left ( \frac{\sqrt{\alpha _{t}}}{\beta _{t}}x_{t} + \frac{\sqrt{\bar{\alpha } _{t-1}}}{1-\bar{\alpha }_{t-1}}x_{0} \right )\times \color{darkred}{\tilde{\beta _{t}} ^{2}} = \left ( \frac{\sqrt{\alpha _{t}}}{\beta _{t}}x_{t} + \frac{\sqrt{\bar{\alpha } _{t-1}}}{1-\bar{\alpha }_{t-1}}x_{0} \right )\times \color{darkred}{\frac{1-\bar{\alpha }_{t-1}}{1-\bar{\alpha }_{t}}\beta _{t}}">\tilde{\mu
            }(x_{t}, x_{0}) = \left ( \frac{\sqrt{\alpha _{t}}}{\beta _{t}}x_{t} +
            \frac{\sqrt{\bar{\alpha } _{t-1}}}{1-\bar{\alpha }_{t-1}}x_{0} \right
            )\times \color{darkred}{\tilde{\beta _{t}} ^{2}} = \left (
            \frac{\sqrt{\alpha _{t}}}{\beta _{t}}x_{t} + \frac{\sqrt{\bar{\alpha }
            _{t-1}}}{1-\bar{\alpha }_{t-1}}x_{0} \right )\times
            \color{darkred}{\frac{1-\bar{\alpha }_{t-1}}{1-\bar{\alpha }_{t}}\beta
            _{t}}</span> </p>
    <p data-pid="BsrVKYWC"><span class="ztext-math" data-eeimg="1"
            data-tex="= \frac{\sqrt{\alpha _{t}}(1-\bar{\alpha }_{t-1})}{1 - \bar{\alpha }_{t}}x_{t} + \frac{\sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha }_{t}}\beta _{t}\color{darkorange}{x_{0}} = \frac{\sqrt{\alpha _{t}}(1-\bar{\alpha }_{t-1})}{1 - \bar{\alpha }_{t}}x_{t} + \frac{\sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha }_{t}}\beta _{t}\color{darkorange}{\frac{x_{t} - \sqrt{1 - \bar{\alpha }_{t}}\bar{z}_{t} }{\sqrt{\bar{\alpha }_{t}}}}">=
            \frac{\sqrt{\alpha _{t}}(1-\bar{\alpha }_{t-1})}{1 - \bar{\alpha
            }_{t}}x_{t} + \frac{\sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha
            }_{t}}\beta _{t}\color{darkorange}{x_{0}} = \frac{\sqrt{\alpha
            _{t}}(1-\bar{\alpha }_{t-1})}{1 - \bar{\alpha }_{t}}x_{t} +
            \frac{\sqrt{\bar{\alpha }_{t-1}}}{1 - \bar{\alpha }_{t}}\beta
            _{t}\color{darkorange}{\frac{x_{t} - \sqrt{1 - \bar{\alpha
            }_{t}}\bar{z}_{t} }{\sqrt{\bar{\alpha }_{t}}}}</span> </p>
    <p data-pid="CSV0iQM3"><span class="ztext-math" data-eeimg="1"
            data-tex="= \left ( \frac{\sqrt{\alpha _{t}}(1 - \bar{\alpha }_{t-1})}{1 - \bar{\alpha }_{t}} + \frac{\color{magenta} {\beta _{t}}\color{darkgreen}{\sqrt{\bar{\alpha }_{t-1}}}}{\color{darkgreen}{\sqrt{\bar{\alpha } _{t}}}(1 - \bar{\alpha }_{t})} \right )x_{t} - \frac{\color{purple}{\sqrt{\bar{\alpha }_{t-1}}}\sqrt{1 - \bar{\alpha }_{t}}\beta _{t}\bar{z}_{t} }{\color{purple}{\sqrt{\bar{\alpha } _{t}}}(1 - \bar{\alpha }_{t})}">=
            \left ( \frac{\sqrt{\alpha _{t}}(1 - \bar{\alpha }_{t-1})}{1 -
            \bar{\alpha }_{t}} + \frac{\color{magenta} {\beta
            _{t}}\color{darkgreen}{\sqrt{\bar{\alpha
            }_{t-1}}}}{\color{darkgreen}{\sqrt{\bar{\alpha } _{t}}}(1 - \bar{\alpha
            }_{t})} \right )x_{t} - \frac{\color{purple}{\sqrt{\bar{\alpha
            }_{t-1}}}\sqrt{1 - \bar{\alpha }_{t}}\beta _{t}\bar{z}_{t}
            }{\color{purple}{\sqrt{\bar{\alpha } _{t}}}(1 - \bar{\alpha
            }_{t})}</span> </p>
    <p data-pid="N906zLu6"><span class="ztext-math" data-eeimg="1"
            data-tex="= \left ( \frac{\sqrt{\alpha _{t}}(1-\bar{\alpha }_{t-1})}{1 - \bar{\alpha }_{t}} + \frac{\color{magenta} {1 - \alpha _{t}}}{\color{darkgreen}{\sqrt{\alpha _{t}}}(1 - \bar{\alpha }_{t})} \right )x_{t} - \frac{\beta _{t}\bar{z}_{t} }{\color{purple}{\sqrt{\alpha _{t}}}\sqrt{1 - \bar{\alpha }_{t}}}">=
            \left ( \frac{\sqrt{\alpha _{t}}(1-\bar{\alpha }_{t-1})}{1 - \bar{\alpha
            }_{t}} + \frac{\color{magenta} {1 - \alpha
            _{t}}}{\color{darkgreen}{\sqrt{\alpha _{t}}}(1 - \bar{\alpha }_{t})}
            \right )x_{t} - \frac{\beta _{t}\bar{z}_{t}
            }{\color{purple}{\sqrt{\alpha _{t}}}\sqrt{1 - \bar{\alpha }_{t}}}</span>
    </p>
    <p data-pid="r0j4ETyN"><span class="ztext-math" data-eeimg="1"
            data-tex="= \frac{\alpha _{t}(1 - \bar{\alpha }_{t-1}) + 1 - \alpha _{t}}{\sqrt{\alpha _{t}}(1 - \bar{\alpha }_{t})}x_{t} - \frac{\beta _{t}\bar{z}_{t} }{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}} = \frac{1 - \color{blue}{\alpha _{t}\bar{\alpha }_{t-1}}}{\sqrt{\alpha _{t}}(1 - \bar{\alpha }_{t})}x_{t} - \frac{\beta _{t}\bar{z}_{t} }{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}}">=
            \frac{\alpha _{t}(1 - \bar{\alpha }_{t-1}) + 1 - \alpha
            _{t}}{\sqrt{\alpha _{t}}(1 - \bar{\alpha }_{t})}x_{t} - \frac{\beta
            _{t}\bar{z}_{t} }{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}} =
            \frac{1 - \color{blue}{\alpha _{t}\bar{\alpha }_{t-1}}}{\sqrt{\alpha
            _{t}}(1 - \bar{\alpha }_{t})}x_{t} - \frac{\beta _{t}\bar{z}_{t}
            }{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}}</span> </p>
    <p data-pid="Ovfhd7AD"><span class="ztext-math" data-eeimg="1"
            data-tex="= \frac{1 - \color{blue}{\bar{\alpha }_{t}}}{\sqrt{\alpha _{t}}(1 - \bar{\alpha }_{t})}x_{t} - \frac{\beta _{t}\bar{z}_{t} }{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}} = \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t} - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} \right )">=
            \frac{1 - \color{blue}{\bar{\alpha }_{t}}}{\sqrt{\alpha _{t}}(1 -
            \bar{\alpha }_{t})}x_{t} - \frac{\beta _{t}\bar{z}_{t} }{\sqrt{\alpha
            _{t}}\sqrt{1-\bar{\alpha }_{t}}} = \frac{1}{\sqrt{\alpha _{t}}}\left (
            x_{t} - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} \right
            )</span> </p>
    <p data-pid="nLg5vWu1">所以在给定 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        的条件下，反向过程真实的概率分布的均值只与 <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 和 <span
            class="ztext-math" data-eeimg="1" data-tex="\bar{z}_{t}">\bar{z}_{t}</span> 有关，满足下式：</p>
    <p data-pid="wg5ix4k4"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}\mid x_{t}, x_{0}) = \mathcal{N}\left ( x_{t-1}, \color{blue} {\tilde{\mu }(x_{t},x_{0})}, \color{red} {\tilde{\beta }_{t}^{2}}\boldsymbol{I} \right )=\mathcal{N}\left ( x_{t-1}, \color{blue} {\frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t} - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} \right )}, \color{red} {\frac{1-\bar{\alpha }_{t-1}}{1-\bar{\alpha }_{t}}\beta _{t}} \boldsymbol{I} \right )">q(x_{t-1}\mid
            x_{t}, x_{0}) = \mathcal{N}\left ( x_{t-1}, \color{blue} {\tilde{\mu
            }(x_{t},x_{0})}, \color{red} {\tilde{\beta }_{t}^{2}}\boldsymbol{I}
            \right )=\mathcal{N}\left ( x_{t-1}, \color{blue} {\frac{1}{\sqrt{\alpha
            _{t}}}\left ( x_{t} - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha
            }_{t}}}\bar{z}_{t} \right )}, \color{red} {\frac{1-\bar{\alpha
            }_{t-1}}{1-\bar{\alpha }_{t}}\beta _{t}} \boldsymbol{I} \right )</span>
    </p>
    <hr />
    <h2 id="h_636776166_9" data-into-catalog-status="">优化目标</h2>
    <p data-pid="KCfgqFDc">扩散模型的目标是得到尽可能真实的 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        ，即求模型参数 <span class="ztext-math" data-eeimg="1" data-tex="\theta">\theta</span> ，使其最终得到 <span class="ztext-math"
            data-eeimg="1" data-tex="x_{0}">x_{0}</span>
        的概率最大，这显然是一个极大似然估计问题，写出似然函数：</p>
    <p data-pid="78vB9-zc"><span class="ztext-math" data-eeimg="1"
            data-tex="p\left ( x_{0}\mid \theta \right ) = p_{\theta }(x_{0}) = \int_{x_{1}}\int _{x_{2}}\cdots \int _{x_{T}}p_{\theta}(x_{0}, x_{1}, x_{2},\cdots ,x_{T}) d_{x_{1}}d_{x_{2}}\cdots d_{x_{T}}">p\left
            ( x_{0}\mid \theta \right ) = p_{\theta }(x_{0}) = \int_{x_{1}}\int
            _{x_{2}}\cdots \int _{x_{T}}p_{\theta}(x_{0}, x_{1}, x_{2},\cdots
            ,x_{T}) d_{x_{1}}d_{x_{2}}\cdots d_{x_{T}}</span> </p>
    <p data-pid="GxiY2OtA"><span class="ztext-math" data-eeimg="1"
            data-tex="= \int_{x_{1}}\int _{x_{2}}\cdots \int _{x_{T}}\color{darkgreen}{q(x_{1:T}\mid x_{0}) }\frac{p_{\theta}(x_{0}, x_{1}, x_{2},\cdots ,x_{T})}{\color{darkgreen}{q(x_{1:T}\mid x_{0})}} d_{x_{1}}d_{x_{2}}\cdots d_{x_{T}}= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\frac{\color{darkred}{p_{\theta}(x_{0:T})}}{ q(x_{1:T}\mid x_{0})} \right]">=
            \int_{x_{1}}\int _{x_{2}}\cdots \int
            _{x_{T}}\color{darkgreen}{q(x_{1:T}\mid x_{0}) }\frac{p_{\theta}(x_{0},
            x_{1}, x_{2},\cdots ,x_{T})}{\color{darkgreen}{q(x_{1:T}\mid x_{0})}}
            d_{x_{1}}d_{x_{2}}\cdots d_{x_{T}}= \mathbb{E}_{ q(x_{1:T}\mid
            x_{0})}\left [\frac{\color{darkred}{p_{\theta}(x_{0:T})}}{ q(x_{1:T}\mid
            x_{0})} \right]</span> </p>
    <p data-pid="HoWuhRjl">由 <span class="ztext-math" data-eeimg="1" data-tex="Jensen">Jensen</span> 不等式，对任一凸函数 <span
            class="ztext-math" data-eeimg="1" data-tex="f">f</span>
        ，始终满足<b>函数值的期望大于等于期望的函数值</b>，对上式两边取负对数，得到负对数似然函数，满足：</p>
    <p data-pid="Q2lTXSmg"><span class="ztext-math" data-eeimg="1"
            data-tex="-\log p_{\theta }(x_{0}) = - \log \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [ \frac{ p_{\theta}(x_{0:T})}{ q(x_{1:T}\mid x_{0})} \right] \leq \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{ q(x_{1:T}\mid x_{0})}{ p_{\theta}(x_{0:T})}\right]">-\log
            p_{\theta }(x_{0}) = - \log \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [
            \frac{ p_{\theta}(x_{0:T})}{ q(x_{1:T}\mid x_{0})} \right] \leq
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{ q(x_{1:T}\mid
            x_{0})}{ p_{\theta}(x_{0:T})}\right]</span> </p>
    <p data-pid="vrtIG_s_">
        ​​​​​式子右侧称为<b>变分下界</b>，最小化负对数似然函数可以转换为最小化变分下界，结合<b>马尔科夫链的贝叶斯公式</b>将变分下界展开：
    </p>
    <p data-pid="xAf2PRb5"><span class="ztext-math" data-eeimg="1"
            data-tex="L_{VLB} = \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{ \color{darkblue}{q(x_{1:T}\mid x_{0})}}{ \color{darkred}{p_{\theta}(x_{0:T})}}\right]">L_{VLB}
            = \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{
            \color{darkblue}{q(x_{1:T}\mid x_{0})}}{
            \color{darkred}{p_{\theta}(x_{0:T})}}\right]</span> </p>
    <p data-pid="qU_0Q_0o"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{ \color{darkblue}{q(x_{1}\mid x_{0})q(x_{2}\mid x_{1})\cdots q(x_{T}\mid x_{T-1})}}{ \color{darkred}{p_{\theta}(x_{T})p_{\theta}(x_{T-1}\mid x_{T})\cdots p_{\theta}(x_{0}\mid x_{1})}}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{
            \color{darkblue}{q(x_{1}\mid x_{0})q(x_{2}\mid x_{1})\cdots q(x_{T}\mid
            x_{T-1})}}{ \color{darkred}{p_{\theta}(x_{T})p_{\theta}(x_{T-1}\mid
            x_{T})\cdots p_{\theta}(x_{0}\mid x_{1})}}\right]</span> </p>
    <p data-pid="yGL1dfMm"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{\color{darkblue}{\prod_{t=1}^{T} q(x_{t}\mid x_{t-1})}}{\color{darkred}{p_{\theta}(x_{T}) \prod_{t=1}^{T} p_{\theta}(x_{t-1}\mid x_{t})}}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log
            \frac{\color{darkblue}{\prod_{t=1}^{T} q(x_{t}\mid
            x_{t-1})}}{\color{darkred}{p_{\theta}(x_{T}) \prod_{t=1}^{T}
            p_{\theta}(x_{t-1}\mid x_{t})}}\right]</span> </p>
    <p data-pid="MyKiBI18"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=1}^{T}\log \frac{q(x_{t}\mid x_{t-1})}{p_{\theta}(x_{t-1}\mid x_{t})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=1}^{T}\log \frac{q(x_{t}\mid x_{t-1})}{p_{\theta}(x_{t-1}\mid
            x_{t})}\right]</span> </p>
    <p data-pid="JXbv8M_Q"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t}\mid x_{t-1})}}{p_{\theta}(x_{t-1}\mid x_{t})} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t}\mid
            x_{t-1})}}{p_{\theta}(x_{t-1}\mid x_{t})} + \log \frac{q(x_{1}\mid
            x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]</span> </p>
    <p data-pid="4xWopcVG"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t}\mid x_{t-1}, x_{0})}}{p_{\theta}(x_{t-1}\mid x_{t})} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t}\mid x_{t-1},
            x_{0})}}{p_{\theta}(x_{t-1}\mid x_{t})} + \log \frac{q(x_{1}\mid
            x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]</span> </p>
    <p data-pid="jT5FeC5T"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t}, x_{t-1}, x_{0})}}{p_{\theta}(x_{t-1}\mid x_{t})\color{purple}{q(x_{t-1},x_{0})}} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t}, x_{t-1},
            x_{0})}}{p_{\theta}(x_{t-1}\mid x_{t})\color{purple}{q(x_{t-1},x_{0})}}
            + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid
            x_{1})}\right]</span> </p>
    <p data-pid="JYPACI6E"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t-1}\mid x_{t},x_{0})q(x_{t}\mid x_{0})q(x_{0})}}{p_{\theta}(x_{t-1}\mid x_{t})\color{purple}{q(x_{t-1}\mid x_{0})q(x_{0})}} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{\color{purple}{q(x_{t-1}\mid
            x_{t},x_{0})q(x_{t}\mid x_{0})q(x_{0})}}{p_{\theta}(x_{t-1}\mid
            x_{t})\color{purple}{q(x_{t-1}\mid x_{0})q(x_{0})}} + \log
            \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]</span>
    </p>
    <p data-pid="HBlM4EpX"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid x_{t},x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})}\cdot \frac{q(x_{t}\mid x_{0})}{q(x_{t-1}\mid x_{0})} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid
            x_{t},x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})}\cdot \frac{q(x_{t}\mid
            x_{0})}{q(x_{t-1}\mid x_{0})} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta
            }(x_{0} \mid x_{1})}\right]</span> </p>
    <p data-pid="KDQBZRbf"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} + \color{red} {\sum_{t=2}^{T}\log \frac{q(x_{t}\mid x_{0})}{q(x_{t-1}\mid x_{0})}} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid x_{t},
            x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} + \color{red} {\sum_{t=2}^{T}\log
            \frac{q(x_{t}\mid x_{0})}{q(x_{t-1}\mid x_{0})}} + \log
            \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]</span>
    </p>
    <p data-pid="88bHH839"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) + \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} + \color{red} {\log \frac{q(x_{T}\mid x_{0})}{q(x_{1}\mid x_{0})}} + \log \frac{q(x_{1}\mid x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [- \log p_{\theta}(x_{T}) +
            \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid x_{t},
            x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} + \color{red} {\log
            \frac{q(x_{T}\mid x_{0})}{q(x_{1}\mid x_{0})}} + \log \frac{q(x_{1}\mid
            x_{0})}{p_{\theta }(x_{0} \mid x_{1})}\right]</span> </p>
    <p data-pid="6vycYdns"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid x_{0})}{p_{\theta}(x_{T})} + \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} - \log p_{\theta}(x_{0}\mid x_{1})\right]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid
            x_{0})}{p_{\theta}(x_{T})} + \sum_{t=2}^{T}\log \frac{q(x_{t-1}\mid
            x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} - \log
            p_{\theta}(x_{0}\mid x_{1})\right]</span> </p>
    <p data-pid="jp8pCHUn">因为<b>和的期望等于期望的和</b>，可得：</p>
    <p data-pid="tCHP2Y7x"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid x_{0})}{p_{\theta}(x_{T})} \right] + \sum_{t=2}^{T}\mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} \right] - \mathbb{E}_{ q(x_{1:T}\mid x_{0})} \left [ \log p_{\theta}(x_{0}\mid x_{1}) \right ]">=
            \mathbb{E}_{ q(x_{1:T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid
            x_{0})}{p_{\theta}(x_{T})} \right] + \sum_{t=2}^{T}\mathbb{E}_{
            q(x_{1:T}\mid x_{0})}\left [\log \frac{q(x_{t-1}\mid x_{t},
            x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} \right] - \mathbb{E}_{
            q(x_{1:T}\mid x_{0})} \left [ \log p_{\theta}(x_{0}\mid x_{1}) \right
            ]</span> </p>
    <p data-pid="A9Hk-zXO">因为<b>期望目标与部分时间步的概率无关可以直接省去</b>，可得：</p>
    <p data-pid="LHA97_rc"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid x_{0})}{p_{\theta}(x_{T})} \right] + \sum_{t=2}^{T}\mathbb{E}_{ q(x_{t},x_{t-1}\mid x_{0})}\left [\log \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} \right] - \mathbb{E}_{ q(x_{1}\mid x_{0})} \left [ \log p_{\theta}(x_{0}\mid x_{1}) \right ]">=
            \mathbb{E}_{ q(x_{T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid
            x_{0})}{p_{\theta}(x_{T})} \right] + \sum_{t=2}^{T}\mathbb{E}_{
            q(x_{t},x_{t-1}\mid x_{0})}\left [\log \frac{q(x_{t-1}\mid x_{t},
            x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} \right] - \mathbb{E}_{
            q(x_{1}\mid x_{0})} \left [ \log p_{\theta}(x_{0}\mid x_{1}) \right
            ]</span> </p>
    <p data-pid="vxy6j4q_">中间的求和项根据前置知识中的<b>贝叶斯公式</b>改写，可得：</p>
    <p data-pid="UyIWh5wG"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ q(x_{T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid x_{0})}{p_{\theta}(x_{T})} \right] + \sum_{t=2}^{T}\mathbb{E}_{ q(x_{t}\mid x_{0}) q(x_{t-1}\mid x_{t},x_{0})}\left [\log \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})} \right] - \mathbb{E}_{ q(x_{1}\mid x_{0})} \left [ \log p_{\theta}(x_{0}\mid x_{1}) \right ]">=
            \mathbb{E}_{ q(x_{T}\mid x_{0})}\left [\log \frac{q(x_{T}\mid
            x_{0})}{p_{\theta}(x_{T})} \right] + \sum_{t=2}^{T}\mathbb{E}_{
            q(x_{t}\mid x_{0}) q(x_{t-1}\mid x_{t},x_{0})}\left [\log
            \frac{q(x_{t-1}\mid x_{t}, x_{0})}{p_{\theta}(x_{t-1}\mid x_{t})}
            \right] - \mathbb{E}_{ q(x_{1}\mid x_{0})} \left [ \log
            p_{\theta}(x_{0}\mid x_{1}) \right ]</span></p>
    <p data-pid="7wNBueBR"><span class="ztext-math" data-eeimg="1"
            data-tex="= \color{purple} {D_{KL}\left ( q(x_{T}| x_{0})\parallel p_{\theta}(x_{T}) \right )} \color{darkgreen}{ + \sum_{t=2}^{T} \mathbb{E}_{ q(x_{t}\mid x_{0})} \left [ D_{KL}(q(x_{t-1}| x_{t},x_{0})\parallel p_{\theta}(x_{t-1}| x_{t})) \right ]} \color{darkred}{ - \mathbb{E}_{ q(x_{1}\mid x_{0})} \left [ \log p_{\theta}(x_{0}| x_{1}) \right ]}">=
            \color{purple} {D_{KL}\left ( q(x_{T}| x_{0})\parallel p_{\theta}(x_{T})
            \right )} \color{darkgreen}{ + \sum_{t=2}^{T} \mathbb{E}_{ q(x_{t}\mid
            x_{0})} \left [ D_{KL}(q(x_{t-1}| x_{t},x_{0})\parallel
            p_{\theta}(x_{t-1}| x_{t})) \right ]} \color{darkred}{ - \mathbb{E}_{
            q(x_{1}\mid x_{0})} \left [ \log p_{\theta}(x_{0}| x_{1}) \right
            ]}</span> </p>
    <p data-pid="iVHJfOk6"><span class="ztext-math" data-eeimg="1"
            data-tex="= \color{purple} {L_{T}} \color{darkgreen}{ + L_{T-1} + \cdots} \color{darkred}{ + L_{0}}">=
            \color{purple} {L_{T}} \color{darkgreen}{ + L_{T-1} + \cdots}
            \color{darkred}{ + L_{0}}</span> </p>
    <p data-pid="7qUXtLG9">对 <span class="ztext-math" data-eeimg="1" data-tex="L_{T}">L_{T}</span> 而言，先验分布 <span
            class="ztext-math" data-eeimg="1" data-tex="q(x_{T}\mid x_{0})">q(x_{T}\mid x_{0})</span>
        是确定的值，而 <span class="ztext-math" data-eeimg="1" data-tex="p_{\theta} (x_{T})">p_{\theta} (x_{T})</span>
        是一个各向同性的高斯分布，二者都不含参数，我们可将这一项视为常数，最小化变分上界时不用考虑。</p>
    <p data-pid="uu523N5t">对剩余的求和项，当 <span class="ztext-math" data-eeimg="1" data-tex="t">t</span> 的取值为 1 时，分子为 <span
            class="ztext-math" data-eeimg="1" data-tex="q(x_{0} \mid x_{1}, x_{0})">q(x_{0} \mid x_{1},
            x_{0})</span> ，即在已知 <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 的条件下求 <span
            class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 的概率分布，也是确定值，所以其实 <span class="ztext-math"
            data-eeimg="1" data-tex="L_{0}">L_{0}</span> 可以并入
        <span class="ztext-math" data-eeimg="1" data-tex="L_{t-1}">L_{t-1}</span>
        ，由此可将变分上界简化一些。
    </p>
    <p data-pid="QJRtZQyR">推导到这里，优化目标就被转化为了最小化<b>后验分布</b>与<b>网络参数化的高斯分布</b>之间的KL散度。
    </p>
    <p data-pid="G4s7WmOL">为了简化计算，DDPM对 <span class="ztext-math" data-eeimg="1"
            data-tex="p_{\theta}(x_{t-1}\mid x_{t})">p_{\theta}(x_{t-1}\mid
            x_{t})</span> 做了进一步简化，采用固定方差 <span class="ztext-math" data-eeimg="1"
            data-tex="\Sigma _{\theta}(x_{t}, t) = \sigma_{t} ^{2}\boldsymbol{I}">\Sigma
            _{\theta}(x_{t}, t) = \sigma_{t} ^{2}\boldsymbol{I}</span> ，这里的 <span class="ztext-math" data-eeimg="1"
            data-tex="\sigma_{t} ^{2}">\sigma_{t}
            ^{2}</span> 是一个无需训练的常量，文中提到，设置为 <span class="ztext-math" data-eeimg="1" data-tex="\beta _{t}">\beta
            _{t}</span> 或 <span class="ztext-math" data-eeimg="1" data-tex="\tilde{\beta }_{t}">\tilde{\beta
            }_{t}</span>
        有相似的效果，这里我们假定 <span class="ztext-math" data-eeimg="1" data-tex="\sigma_{t} ^{2} = \tilde{\beta }_{t}">\sigma_{t}
            ^{2} =
            \tilde{\beta }_{t}</span> ，则KL散度中的两项分别可写作：</p>
    <p data-pid="pWUBuUgK"><span class="ztext-math" data-eeimg="1"
            data-tex="q(x_{t-1}\mid x_{t},x_{0}) = \mathcal{N}\left ( x_{t-1}, \tilde{\mu }(x_{t},x_{0}), \color{red} {\tilde{\beta }_{t}}\boldsymbol{I} \right )= \mathcal{N}\left ( x_{t-1}, \tilde{\mu }(x_{t},x_{0}), \color{red} {\sigma_{t}^{2}}\boldsymbol{I} \right )">q(x_{t-1}\mid
            x_{t},x_{0}) = \mathcal{N}\left ( x_{t-1}, \tilde{\mu }(x_{t},x_{0}),
            \color{red} {\tilde{\beta }_{t}}\boldsymbol{I} \right )=
            \mathcal{N}\left ( x_{t-1}, \tilde{\mu }(x_{t},x_{0}), \color{red}
            {\sigma_{t}^{2}}\boldsymbol{I} \right )</span> </p>
    <p data-pid="HCF6cLLo"><span class="ztext-math" data-eeimg="1"
            data-tex="p_{\theta}(x_{t-1}\mid x_{t}) = \mathcal{N}\left ( x_{t-1}; \mu _{\theta}(x_{t}, t), \color{red}{\Sigma _{\theta}(x_{t}, t)} \right ) = \mathcal{N}(x_{t-1}; \mu _{\theta}(x_{t}, t), \color{red} {\sigma_{t}^{2}}\boldsymbol{I})">p_{\theta}(x_{t-1}\mid
            x_{t}) = \mathcal{N}\left ( x_{t-1}; \mu _{\theta}(x_{t}, t),
            \color{red}{\Sigma _{\theta}(x_{t}, t)} \right ) = \mathcal{N}(x_{t-1};
            \mu _{\theta}(x_{t}, t), \color{red}
            {\sigma_{t}^{2}}\boldsymbol{I})</span> </p>
    <p data-pid="WC9C7w2Q">结合前置知识中<b>高斯分布的KL散度公式</b>，有：</p>
    <p data-pid="ZceyvJhc"><span class="ztext-math" data-eeimg="1"
            data-tex="D_{KL}(q(x_{t-1}| x_{t},x_{0})\parallel p_{\theta}(x_{t-1}| x_{t}))">D_{KL}(q(x_{t-1}|
            x_{t},x_{0})\parallel p_{\theta}(x_{t-1}| x_{t}))</span> </p>
    <p data-pid="qvZ-qIYC"><span class="ztext-math" data-eeimg="1"
            data-tex="= D_{KL}\left ( \mathcal{N}(x_{t-1}, \tilde{\mu }(x_{t},x_{0}), \sigma_{t}^{2}\boldsymbol{I}) \parallel \mathcal{N}(x_{t-1}; \mu _{\theta}(x_{t}, t), \sigma_{t}^{2}\boldsymbol{I}) \right )">=
            D_{KL}\left ( \mathcal{N}(x_{t-1}, \tilde{\mu }(x_{t},x_{0}),
            \sigma_{t}^{2}\boldsymbol{I}) \parallel \mathcal{N}(x_{t-1}; \mu
            _{\theta}(x_{t}, t), \sigma_{t}^{2}\boldsymbol{I}) \right )</span> </p>
    <p data-pid="sQiQsRCY"><span class="ztext-math" data-eeimg="1"
            data-tex="= \log 1 + \frac{\sigma _{t}^{2} + \left \| \tilde{\mu }_{t}(x_{t},x_{0}) - \mu _{\theta}(x_{t}, t) \right \| ^{2}}{2 \sigma_{t} ^{2}} - \frac{1}{2} ">=
            \log 1 + \frac{\sigma _{t}^{2} + \left \| \tilde{\mu }_{t}(x_{t},x_{0})
            - \mu _{\theta}(x_{t}, t) \right \| ^{2}}{2 \sigma_{t} ^{2}} -
            \frac{1}{2} </span> </p>
    <p data-pid="SbWynzbx"><span class="ztext-math" data-eeimg="1"
            data-tex="= \frac{1}{2\sigma_{t} ^{2}}\left \| \tilde{\mu }_{t}(x_{t},x_{0}) - \mu _{\theta}(x_{t}, t) \right \| ^{2}">=
            \frac{1}{2\sigma_{t} ^{2}}\left \| \tilde{\mu }_{t}(x_{t},x_{0}) - \mu
            _{\theta}(x_{t}, t) \right \| ^{2}</span> </p>
    <p data-pid="sV_vQf-Y">将此式代回之前变分上界的式子，则优化目标 <span class="ztext-math" data-eeimg="1"
            data-tex="L_{t-1}">L_{t-1}</span> 可以写作：</p>
    <p data-pid="Gjd88-C-"><span class="ztext-math" data-eeimg="1"
            data-tex="L_{t-1} = \mathbb{E}_{ q(x_{t}\mid x_{0})}\left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \tilde{\mu }_{t}(x_{t},x_{0}) - \mu _{\theta}(x_{t}, t) \right \| ^{2} \right ]">L_{t-1}
            = \mathbb{E}_{ q(x_{t}\mid x_{0})}\left [ \frac{1}{2\sigma_{t}
            ^{2}}\left \| \tilde{\mu }_{t}(x_{t},x_{0}) - \mu _{\theta}(x_{t}, t)
            \right \| ^{2} \right ]</span> </p>
    <p data-pid="uHoV9RaV">也就是说，我们希望网络参数化高斯分布的<b>均值</b> <span class="ztext-math" data-eeimg="1"
            data-tex="\mu _{\theta}(x_{t},t)">\mu
            _{\theta}(x_{t},t)</span> 与后验分布的<b>均值</b> <span class="ztext-math" data-eeimg="1"
            data-tex="\tilde{\mu }_{t}(x_{t},x_{0})">\tilde{\mu
            }_{t}(x_{t},x_{0})</span> 一致。</p>
    <p data-pid="kHISlTF5">此式还可以继续扩展，在反向过程中， <span class="ztext-math" data-eeimg="1"
            data-tex="\tilde{\mu }_{t}(x_{t},x_{0})">\tilde{\mu
            }_{t}(x_{t},x_{0})</span> 可以写成用 <span class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 和
        <span class="ztext-math" data-eeimg="1" data-tex="\bar{z}_{t}">\bar{z}_{t}</span> 表示的形式，而在前向过程中， <span
            class="ztext-math" data-eeimg="1" data-tex="x_{t}">x_{t}</span> 又可以写成用
        <span class="ztext-math" data-eeimg="1" data-tex="x_{0}">x_{0}</span> 和
        <span class="ztext-math" data-eeimg="1" data-tex="\bar{z}_{t}">\bar{z}_{t}</span> 表示的形式，将它们代入上式，则有：
    </p>
    <p data-pid="md4tJdmS"><span class="ztext-math" data-eeimg="1"
            data-tex="L_{t-1} = \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})}\left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t}\left ( x_{0}, \bar{z}_{t}\right ) - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} \right ) - \mu _{\theta}\left ( x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t \right ) \right \| ^{2} \right ]">L_{t-1}
            = \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim
            \mathcal{N}(0,\boldsymbol{I})}\left [ \frac{1}{2\sigma_{t} ^{2}}\left \|
            \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t}\left ( x_{0},
            \bar{z}_{t}\right ) - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha
            }_{t}}}\bar{z}_{t} \right ) - \mu _{\theta}\left ( x_{t}\left ( x_{0},
            \bar{z}_{t}\right ), t \right ) \right \| ^{2} \right ]</span> </p>
    <p data-pid="XkRgtXqg">其中，参数化高斯分布的均值 <span class="ztext-math" data-eeimg="1" data-tex="\mu _{\theta}(x_{t},t)">\mu
            _{\theta}(x_{t},t)</span>
        可以相应改写成与真实均值 <span class="ztext-math" data-eeimg="1" data-tex="\tilde{\mu }_{t}(x_{t},x_{0})">\tilde{\mu
            }_{t}(x_{t},x_{0})</span> 一样的形式：</p>
    <p data-pid="sBS_oj6D"><span class="ztext-math" data-eeimg="1"
            data-tex="\mu _{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t) = \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t}\left ( x_{0}, \bar{z}_{t}\right ) - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}z_{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t) \right )">\mu
            _{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t) =
            \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t}\left ( x_{0},
            \bar{z}_{t}\right ) - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha
            }_{t}}}z_{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t) \right
            )</span> </p>
    <p data-pid="dHITwLdX">这里的 <span class="ztext-math" data-eeimg="1"
            data-tex="z_{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t))">z_{\theta}(x_{t}\left
            ( x_{0}, \bar{z}_{t}\right ), t))</span>
        是神经网络的拟合项，即优化目标由原来的<b>拟合均值</b>转换成了<b>拟合噪声</b>。将其代入 <span class="ztext-math" data-eeimg="1"
            data-tex="L_{t-1}">L_{t-1}</span> 的表达式中，则 <span class="ztext-math" data-eeimg="1"
            data-tex="L_{t-1}">L_{t-1}</span>
        可表达为：</p>
    <p data-pid="AQpSjlki"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})} \left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t}\left ( x_{0}, \bar{z}_{t}\right ) - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} \right ) - \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t} - \frac{\beta _{t}}{\sqrt{1-\bar{\alpha }_{t}}}z_{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t) \right ) \right \| ^{2} \right ]">=
            \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})}
            \left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{1}{\sqrt{\alpha
            _{t}}}\left ( x_{t}\left ( x_{0}, \bar{z}_{t}\right ) - \frac{\beta
            _{t}}{\sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} \right ) -
            \frac{1}{\sqrt{\alpha _{t}}}\left ( x_{t} - \frac{\beta
            _{t}}{\sqrt{1-\bar{\alpha }_{t}}}z_{\theta}(x_{t}\left ( x_{0},
            \bar{z}_{t}\right ), t) \right ) \right \| ^{2} \right ]</span></p>
    <p data-pid="QRIGwA1G"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})} \left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{x_{t}\left ( x_{0}, \bar{z}_{t}\right )}{\sqrt{\alpha _{t}}}-\frac{\beta _{t}}{\sqrt{\alpha _{t}} \sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} - \frac{x_{t}\left ( x_{0}, \bar{z}_{t}\right )}{\sqrt{\alpha _{t}}} + \frac{\beta _{t}}{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}}z_{\theta}(x_{t}\left ( x_{0}, \bar{z}_{t}\right ), t) \right \| ^{2} \right ]">=
            \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})}
            \left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{x_{t}\left ( x_{0},
            \bar{z}_{t}\right )}{\sqrt{\alpha _{t}}}-\frac{\beta _{t}}{\sqrt{\alpha
            _{t}} \sqrt{1-\bar{\alpha }_{t}}}\bar{z}_{t} - \frac{x_{t}\left ( x_{0},
            \bar{z}_{t}\right )}{\sqrt{\alpha _{t}}} + \frac{\beta
            _{t}}{\sqrt{\alpha _{t}}\sqrt{1-\bar{\alpha }_{t}}}z_{\theta}(x_{t}\left
            ( x_{0}, \bar{z}_{t}\right ), t) \right \| ^{2} \right ]</span> </p>
    <p data-pid="oaH92-F3"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})} \left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{\beta _{t}}{\sqrt{\alpha _{t}} \sqrt{1-\bar{\alpha }_{t}}}\left (\bar{z}_{t} - z_{\theta}\left ( \color{magenta}{x_{t}(x_{0},\bar{z}_{t})}, t \right ) \right ) \right \| ^{2} \right ]">=
            \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})}
            \left [ \frac{1}{2\sigma_{t} ^{2}}\left \| \frac{\beta
            _{t}}{\sqrt{\alpha _{t}} \sqrt{1-\bar{\alpha }_{t}}}\left (\bar{z}_{t} -
            z_{\theta}\left ( \color{magenta}{x_{t}(x_{0},\bar{z}_{t})}, t \right )
            \right ) \right \| ^{2} \right ]</span> </p>
    <p data-pid="q4qWpGFI"><span class="ztext-math" data-eeimg="1"
            data-tex="= \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})} \left [ \frac{\beta_{t}^{2}}{2\sigma ^{2} \alpha _{t}(1 - \bar{\alpha }_{t})}\left \| \bar{z}_{t} - z_{\theta}(\color{magenta}{\sqrt{\bar{\alpha }_{t}}x_{0} + \sqrt{1-\bar{\alpha }_{t}}\bar{z}_{t}}, t) \right \| ^{2} \right ]">=
            \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})}
            \left [ \frac{\beta_{t}^{2}}{2\sigma ^{2} \alpha _{t}(1 - \bar{\alpha
            }_{t})}\left \| \bar{z}_{t} -
            z_{\theta}(\color{magenta}{\sqrt{\bar{\alpha }_{t}}x_{0} +
            \sqrt{1-\bar{\alpha }_{t}}\bar{z}_{t}}, t) \right \| ^{2} \right
            ]</span> </p>
    <p data-pid="plGO628M">可以将系数项去掉，进一步简化：</p>
    <p data-pid="jv6dCRg7"><span class="ztext-math" data-eeimg="1"
            data-tex="L_{t-1}^{simple}= \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})} \left [ \left \| \bar{z}_{t} - z_{\theta}(\sqrt{\bar{\alpha }_{t}}x_{0} + \sqrt{1-\bar{\alpha }_{t}}\bar{z}_{t}, t) \right \| ^{2} \right ]">L_{t-1}^{simple}=
            \mathbb{E}_{ x_{0}, \bar{z}_{t} \sim \mathcal{N}(0,\boldsymbol{I})}
            \left [ \left \| \bar{z}_{t} - z_{\theta}(\sqrt{\bar{\alpha }_{t}}x_{0}
            + \sqrt{1-\bar{\alpha }_{t}}\bar{z}_{t}, t) \right \| ^{2} \right
            ]</span> </p>
    <p data-pid="tp01Donh">虽然推导比较复杂，但最终得到的优化目标非常简单，就是让网络预测的噪声与真实的噪声一致。</p>
</div>