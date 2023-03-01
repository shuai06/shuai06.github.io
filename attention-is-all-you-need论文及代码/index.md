# Attention Is All You Need论文及代码

<script type="text/javascript" src="/js/src/bai.js"></script>

## 简介
论文地址：https://arxiv.org/abs/1706.03762
该论文提出了Transformer模型，完全基于Attention mechanism，抛弃了传统的RNN和CNN，是第一个只依赖于自注意力来做encoder-decoder架构的模型，可谓是大道至简！


## Transformer架构
先整体看一下架构图，
![架构图](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009133916.png)

对于上图左边的`Nx`标注出来的部分，就是encoder的一层，一共有6层。同理右边的decoder也是。

输入序列经过word embedding和positional encoding相加后，输入到encoder。
输出序列经过word embedding和positional encoding相加后，输入到decoder。
最后，decoder输出的结果，经过一个线性层，然后计算softmax。


### Encoder
encoder由6层相同的层组成，每一层分别由两部分组成：
- 第一部分是一个`multi-head self-attention mechanism`
- 第二部分是一个`position-wise feed-forward network`，是一个全连接层
两个部分都有一个残差连接(`residual connection`)，然后接着一个Layer Normalization





### Decoder
和encoder类似，decoder由6个相同的层组成，每一个层包括以下3个部分：
- 第一个部分是`multi-head self-attention mechanism`
- 第二部分是`multi-head context-attention mechanism`
- 第三部分是一个`position-wise feed-forward network`
- 还是和encoder类似，上面三个部分的每一个部分，都有一个残差连接，后接一个Layer Normalization。




## Attention机制
attention是指，对于某个时刻的输出y，它在输入x上各个部分的注意力。这个注意力实际上可以理解为权重。

attension机制有很多种，下面连接有一张较全面的表格：https://lilianweng.github.io/posts/2018-06-24-attention/
![Attention机制种类](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009134755.png)

上图中，第一种加性注意力(additive attention)在seq2seq模型里面用的很多，但是这里transformer中使用的另外一种**乘性注意力(multiplicative attention)，即两个隐状态进行点积**



### Self-Attention
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009153650.png)
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009153626.png)


前面说attention机制的时候，都会说两个隐状态：
- $h_i$：输入序列第i个位置产生的隐状态
- $s_t$：输出序列在第t个位置产生的隐状态

所谓self-attention实际上就是，**输出序列就是输入序列！因此，计算自己的attention得分，就叫做self-attention！**


### Context-attention
它是encoder和decoder之间的attention。所以，你也可以称之为encoder-decoder attention。

不管是self-attention还是context-attention，它们计算attention分数的时候，可以选择很多方式，比如上面表中提到的：
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009134755.png)

Transformer模型采用的是：`scaled dot-product attention`。


### Scaled dot-product attention
通过确定Q和K之间的相似程度来选择V。
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009135734.png)


论文也提供了结构图：
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009135800.png)


其中，K和Q和V是一个东西，在之前复制了三份。
- 在encoder的self-attention中，Q、K、V都来自同一个地方（**相等**），他们是上一层encoder的输出。对于第一层encoder，它们就是word embedding和positional encoding相加得到的输入。
- 在decoder的self-attention中，Q、K、V都来自于同一个地方（**相等**），它们是上一层decoder的输出。对于第一层decoder，它们就是word embedding和positional encoding相加得到的输入。但是对于decoder，我们不希望它能获得下一个time step（即将来的信息），因此我们需要进行sequence masking。
- 在encoder-decoder attention中，Q来自于decoder的上一层的输出，K和V来自于encoder的输出，K和V是一样的。
- Q、K、V三者的维度一样，即d_q=d_k=d_v 


**Scaled dot-product attention的实现：**
```python
import torch
import torch.nn as nn

class ScaledDotProductAttention(nn.Module):
    """Scaled dot-product attention mechanism."""

    def __init__(self, attention_dropout=0.0):
        super(ScaledDotProductAttention, self).__init__()
        self.dropout = nn.Dropout(attention_dropout)
        self.softmax = nn.Softmax(dim=2)

    def forward(self, q, k, v, scale=None, attn_mask=None):
        """前向传播.

        Args:
            q: Queries张量，形状为[B, L_q, D_q]
            k: Keys张量，形状为[B, L_k, D_k]
            v: Values张量，形状为[B, L_v, D_v]，一般来说就是k
            scale: 缩放因子，一个浮点标量
            attn_mask: Masking张量，形状为[B, L_q, L_k]

        Returns:
            上下文张量和attetention张量
        """
        attention = torch.bmm(q, k.transpose(1, 2))  # bmm 是tensor乘法
        # 相乘的两个矩阵，要满足一定的维度要求：input（p,m,n) * mat2(p,n,a) ->output(p,m,a)。
        # 这个要求，可以类比于矩阵相乘。前一个矩阵的"列"等于后面矩阵的"行"才可以相乘。
        if scale:
            attention = attention * scale
        if attn_mask:
            # 给需要mask的地方设置一个负无穷
            attention = attention.masked_fill_(attn_mask, -np.inf)  # 用value填充tensor中与mask中值为1位置相对应的元素。np.inf 表示+∞
        # 计算softmax
        attention = self.softmax(attention)
        # 添加dropout
        attention = self.dropout(attention)
        # 和V做点积
        context = torch.bmm(attention, v)
        return context, attention

```


### Multi-head attention
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009153550.png)
将Q、K、V通过一个线性映射之后，分成`h`份，对每一份进行scaled dot-product attention效果更好。
然后，把各个部分的结果合并起来，再次经过线性映射，得到最终的输出。这就是所谓的multi-head attention。
上面的超参数h就是heads数量。论文默认是`8`。

multi-head attention结构图：
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009143410.png)



![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009143614.png)

**Multi-head attention的实现：**
```python

import torch
import torch.nn as nn

class MultiHeadAttention(nn.Module):

    def __init__(self, model_dim=512, num_heads=8, dropout=0.0):
        super(MultiHeadAttention, self).__init__()

        self.dim_per_head = model_dim // num_heads
        self.num_heads = num_heads
        self.linear_k = nn.Linear(model_dim, self.dim_per_head * num_heads)
        self.linear_v = nn.Linear(model_dim, self.dim_per_head * num_heads)
        self.linear_q = nn.Linear(model_dim, self.dim_per_head * num_heads)

        self.dot_product_attention = ScaledDotProductAttention(dropout)
        self.linear_final = nn.Linear(model_dim, model_dim)
        self.dropout = nn.Dropout(dropout)
        # multi-head attention之后需要做layer norm
        self.layer_norm = nn.LayerNorm(model_dim)

    def forward(self, key, value, query, attn_mask=None):
        # 残差连接
        residual = query

        dim_per_head = self.dim_per_head
        num_heads = self.num_heads
        batch_size = key.size(0)

        # linear projection
        key = self.linear_k(key)
        value = self.linear_v(value)
        query = self.linear_q(query)

        # split by heads
        key = key.view(batch_size * num_heads, -1, dim_per_head)
        value = value.view(batch_size * num_heads, -1, dim_per_head)
        query = query.view(batch_size * num_heads, -1, dim_per_head)

        if attn_mask:
            attn_mask = attn_mask.repeat(num_heads, 1, 1)
        # scaled dot product attention
        scale = (key.size(-1) // num_heads) ** -0.5
        context, attention = self.dot_product_attention(query, key, value, scale, attn_mask)

        # concat heads
        context = context.view(batch_size, -1, dim_per_head * num_heads)

        # final linear projection
        output = self.linear_final(context)

        # dropout
        output = self.dropout(output)

        # add residual and norm layer
        output = self.layer_norm(residual + output)

        return output, attention

```


其中
### Residual connection
![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009144335.png)

$F(x) + x$，其中的`+x`就是一个shortcut
那么残差结构有什么好处呢？显而易见：因为增加了一项x，那么该层网络对x求偏导的时候，多了一个常数项1！所以在反向传播过程中，梯度连乘，也不会造成梯度消失！
```python
# transformer架构图中的Add & Norm中的Add也就是指的这个shortcut。
def residual(sublayer_fn,x):
    return sublayer_fn(x)+x
```


### Layer Normalization
> Normalization有很多种，但是它们都有一个共同的目的，那就是把输入转化成均值为0方差为1的数据。


- BN的主要思想就是：在每一层的每一批数据上进行归一化。BN的具体做法就是对每一小批数据，在批这个方向上做归一化
- 那么什么是Layer normalization呢？:它也是归一化数据的一种方式，不过LN是在**每一个样本上计算均值和方差，而不是BN那种在批方向计算均值和方差！**

PyTorch已经实现了LN的代码，如果想要自己实现可以看下面的代码：
```python
import torch
import torch.nn as nn

class LayerNorm(nn.Module):
    """实现LayerNorm。其实PyTorch已经实现啦，见nn.LayerNorm。"""

    def __init__(self, features, epsilon=1e-6):
        """Init.

        Args:
            features: 就是模型的维度。论文默认512
            epsilon: 一个很小的数，防止数值计算的除0错误
        """
        super(LayerNorm, self).__init__()
        # alpha
        self.gamma = nn.Parameter(torch.ones(features))
        # beta
        self.beta = nn.Parameter(torch.zeros(features))
        self.epsilon = epsilon

    def forward(self, x):
        """前向传播.

        Args:
            x: 输入序列张量，形状为[B, L, D]
        """
        # 根据公式进行归一化
        # 在X的最后一个维度求均值，最后一个维度就是模型的维度
        mean = x.mean(-1, keepdim=True)
        # 在X的最后一个维度求方差，最后一个维度就是模型的维度
        std = x.std(-1, keepdim=True)
        return self.gamma * (x - mean) / (std + self.epsilon) + self.beta
```



## Mask
mask顾名思义就是掩码，在我们这里的意思大概就是对某些值进行掩盖，使其不产生效果。

在t实现不会看到t时间以及之后的输入，从而保证训练和预测的时候行为一致


Transformer模型里面涉及两种mask。分别是
- padding mask：所有的scaled dot-product attention里面都需要用到
- sequence mask：只有在decoder的self-attention里面用到


### padding mask
我们的每个批次输入序列长度是不一样的！也就是说，我们要对输入序列进行对齐！具体来说，就是给在较短的序列后面填充0。因为这些填充的位置，其实是没什么意义的，所以我们的attention机制不应该把注意力放在这些位置上，所以我们需要进行一些处理。

具体的做法是，**把这些位置的值加上一个非常大的负数(可以是负无穷)，这样的话，经过softmax，这些位置的概率就会接近0！**
```python

def padding_mask(seq_k, seq_q):
    # seq_k和seq_q的形状都是[B,L]
    len_q = seq_q.size(1)
    # `PAD` is 0
    pad_mask = seq_k.eq(0)
    pad_mask = pad_mask.unsqueeze(1).expand(-1, len_q, -1)  # shape [B, L_q, L_k]
    return pad_mask

```




### sequence mask
sequence mask是为了使得decoder不能看见未来的信息。也就是对于一个序列，在time_step为t的时刻，我们的解码输出应该只能依赖于t时刻之前的输出，而不能依赖t之后的输出。因此我们需要想一个办法，把t之后的信息给隐藏起来。

那么具体怎么做呢？**产生一个上三角矩阵，上三角的值全为1，下三角的值全为0，对角线也是0。把这个矩阵作用在每一个序列上**，就可以达到我们的目的啦
```python
def sequence_mask(seq):
    batch_size, seq_len = seq.size()
    mask = torch.triu(torch.ones((seq_len, seq_len), dtype=torch.uint8), diagonal=1)  # 返回矩阵（2-D张量）或矩阵 input 批次的上三角部分，结果张量 out 的其他元素设置为0
    mask = mask.unsqueeze(0).expand(batch_size, -1, -1)  # [B, L, L]
    return mask
```


attn_mask参数有几种情况？
- 对于decoder的self-attention，里面使用到的scaled dot-product attention，同时需要padding mask和sequence mask作为attn_mask，具体实现就是两个mask相加作为attn_mask。
- 其他情况，attn_mask一律等于padding mask。





## Position Encoding
> 对序列中的词语出现的位置进行编码。如果对位置进行编码，那么我们的模型就可以捕捉顺序信息


![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009150149.png)


PE的代码实现，按照公式即可：
```python

import torch
import torch.nn as nn

class PositionalEncoding(nn.Module):

    def __init__(self, d_model, max_seq_len):
        """初始化。

        Args:
            d_model: 一个标量。模型的维度，论文默认是512
            max_seq_len: 一个标量。文本序列的最大长度
        """
        super(PositionalEncoding, self).__init__()

        # 根据论文给的公式，构造出PE矩阵
        position_encoding = np.array([
          [pos / np.pow(10000, 2.0 * (j // 2) / d_model) for j in range(d_model)]
          for pos in range(max_seq_len)])
        # 偶数列使用sin，奇数列使用cos
        position_encoding[:, 0::2] = np.sin(position_encoding[:, 0::2])
        position_encoding[:, 1::2] = np.cos(position_encoding[:, 1::2])

        # 在PE矩阵的第一行，加上一行全是0的向量，代表这`PAD`的positional encoding
        # 在word embedding中也经常会加上`UNK`，代表位置单词的word embedding，两者十分类似
        # 那么为什么需要这个额外的PAD的编码呢？很简单，因为文本序列的长度不一，我们需要对齐，
        # 短的序列我们使用0在结尾补全，我们也需要这些补全位置的编码，也就是`PAD`对应的位置编码
        pad_row = torch.zeros([1, d_model])
        position_encoding = torch.cat((pad_row, position_encoding))

        # 嵌入操作，+1是因为增加了`PAD`这个补全位置的编码，
        # Word embedding中如果词典增加`UNK`，我们也需要+1。看吧，两者十分相似
        self.position_encoding = nn.Embedding(max_seq_len + 1, d_model)
        self.position_encoding.weight = nn.Parameter(position_encoding,
                                                     requires_grad=False)
    def forward(self, input_len):
        """神经网络的前向传播。

        Args:
          input_len: 一个张量，形状为[BATCH_SIZE, 1]。每一个张量的值代表这一批文本序列中对应的长度。

        Returns:
          返回这一批序列的位置编码，进行了对齐。
        """

        # 找出这一批序列的最大长度
        max_len = torch.max(input_len)
        tensor = torch.cuda.LongTensor if input_len.is_cuda else torch.LongTensor
        # 对每一个序列的位置进行对齐，在原序列位置的后面补上0
        # 这里range从1开始也是因为要避开PAD(0)的位置
        input_pos = tensor(
          [list(range(1, len + 1)) + [0] * (max_len - len) for len in input_len])
        return self.position_encoding(input_pos)
```


### Word embedding
它实际上就是一个二维浮点矩阵，里面的权重是可训练参数，我们只需要把这个矩阵构建出来就完成了word embedding的工作。
```python
import torch.nn as nn

embedding = nn.Embedding(vocab_size, embedding_size, padding_idx=0)
# 获得输入的词嵌入编码
seq_embedding = seq_embedding(inputs)*np.sqrt(d_model)

# 上面vocab_size就是词典的大小，embedding_size就是词嵌入的维度大小，论文里面就是等于d_model=512
# 所以word embedding矩阵就是一个vocab_size*embedding_size的二维张量
```



## Position-wise Feed-Forward network

![](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20221009150539.png)

```python
import torch
import torch.nn as nn

class PositionalWiseFeedForward(nn.Module):

    def __init__(self, model_dim=512, ffn_dim=2048, dropout=0.0):
        super(PositionalWiseFeedForward, self).__init__()
        self.w1 = nn.Conv1d(model_dim, ffn_dim, 1)
        self.w2 = nn.Conv1d(model_dim, ffn_dim, 1)
        self.dropout = nn.Dropout(dropout)
        self.layer_norm = nn.LayerNorm(model_dim)

    def forward(self, x):
        output = x.transpose(1, 2)
        output = self.w2(F.relu(self.w1(output)))
        output = self.dropout(output.transpose(1, 2))

        # add residual and norm layer
        output = self.layer_norm(x + output)
        return output
```



## Transformer
至此，所有的细节都已经解释完了。现在来完成我们Transformer模型的代码。

首先，我们需要实现6层的encoder和decoder

encoder:
```python
import torch
import torch.nn as nn

class EncoderLayer(nn.Module):
    """Encoder的一层。"""

    def __init__(self, model_dim=512, num_heads=8, ffn_dim=2018, dropout=0.0):
        super(EncoderLayer, self).__init__()

        self.attention = MultiHeadAttention(model_dim, num_heads, dropout)
        self.feed_forward = PositionalWiseFeedForward(model_dim, ffn_dim, dropout)

    def forward(self, inputs, attn_mask=None):

        # self attention
        context, attention = self.attention(inputs, inputs, inputs, padding_mask)

        # feed forward network
        output = self.feed_forward(context)

        return output, attention


class Encoder(nn.Module):
    """多层EncoderLayer组成Encoder。"""

    def __init__(self,
               vocab_size,
               max_seq_len,
               num_layers=6,
               model_dim=512,
               num_heads=8,
               ffn_dim=2048,
               dropout=0.0):
        super(Encoder, self).__init__()

        self.encoder_layers = nn.ModuleList(
          [EncoderLayer(model_dim, num_heads, ffn_dim, dropout) for _ in
           range(num_layers)])

        self.seq_embedding = nn.Embedding(vocab_size + 1, model_dim, padding_idx=0)
        self.pos_embedding = PositionalEncoding(model_dim, max_seq_len)

    def forward(self, inputs, inputs_len):
        output = self.seq_embedding(inputs)
        output += self.pos_embedding(inputs_len)

        self_attention_mask = padding_mask(inputs, inputs)

        attentions = []
        for encoder in self.encoder_layers:
            output, attention = encoder(output, self_attention_mask)
            attentions.append(attention)

        return output, attentions

```

decoder:
```python
import torch
import torch.nn as nn

class DecoderLayer(nn.Module):

    def __init__(self, model_dim, num_heads=8, ffn_dim=2048, dropout=0.0):
        super(DecoderLayer, self).__init__()

        self.attention = MultiHeadAttention(model_dim, num_heads, dropout)
        self.feed_forward = PositionalWiseFeedForward(model_dim, ffn_dim, dropout)

    def forward(self,
              dec_inputs,
              enc_outputs,
              self_attn_mask=None,
              context_attn_mask=None):
        # self attention, all inputs are decoder inputs
        dec_output, self_attention = self.attention(
          dec_inputs, dec_inputs, dec_inputs, self_attn_mask)

        # context attention
        # query is decoder's outputs, key and value are encoder's inputs
        dec_output, context_attention = self.attention(
          enc_outputs, enc_outputs, dec_output, context_attn_mask)

        # decoder's output, or context
        dec_output = self.feed_forward(dec_output)

        return dec_output, self_attention, context_attention


class Decoder(nn.Module):

    def __init__(self,
               vocab_size,
               max_seq_len,
               num_layers=6,
               model_dim=512,
               num_heads=8,
               ffn_dim=2048,
               dropout=0.0):
        super(Decoder, self).__init__()

        self.num_layers = num_layers

        self.decoder_layers = nn.ModuleList(
          [DecoderLayer(model_dim, num_heads, ffn_dim, dropout) for _ in
           range(num_layers)])

        self.seq_embedding = nn.Embedding(vocab_size + 1, model_dim, padding_idx=0)
        self.pos_embedding = PositionalEncoding(model_dim, max_seq_len)

    def forward(self, inputs, inputs_len, enc_output, context_attn_mask=None):
        output = self.seq_embedding(inputs)
        output += self.pos_embedding(inputs_len)

        self_attention_padding_mask = padding_mask(inputs, inputs)
        seq_mask = sequence_mask(inputs)
        self_attn_mask = torch.gt((self_attention_padding_mask + seq_mask), 0)

        self_attentions = []
        context_attentions = []
        for decoder in self.decoder_layers:
            output, self_attn, context_attn = decoder(
            output, enc_output, self_attn_mask, context_attn_mask)
            self_attentions.append(self_attn)
            context_attentions.append(context_attn)

        return output, self_attentions, context_attentions
```


最后，我们把encoder和decoder组成Transformer模型！
```python

import torch
import torch.nn as nn

class Transformer(nn.Module):

    def __init__(self,
               src_vocab_size,
               src_max_len,
               tgt_vocab_size,
               tgt_max_len,
               num_layers=6,
               model_dim=512,
               num_heads=8,
               ffn_dim=2048,
               dropout=0.2):
        super(Transformer, self).__init__()

        self.encoder = Encoder(src_vocab_size, src_max_len, num_layers, model_dim,
                               num_heads, ffn_dim, dropout)
        self.decoder = Decoder(tgt_vocab_size, tgt_max_len, num_layers, model_dim,
                               num_heads, ffn_dim, dropout)

        self.linear = nn.Linear(model_dim, tgt_vocab_size, bias=False)
        self.softmax = nn.Softmax(dim=2)

    def forward(self, src_seq, src_len, tgt_seq, tgt_len):
        context_attn_mask = padding_mask(tgt_seq, src_seq)

        output, enc_self_attn = self.encoder(src_seq, src_len)

        output, dec_self_attn, ctx_attn = self.decoder(
          tgt_seq, tgt_len, output, context_attn_mask)

        output = self.linear(output)
        output = self.softmax(output)

        return output, enc_self_attn, dec_self_attn, ctx_attn

```



## Note
self-attention有很多变形，以后的重点就是如何减少运算量




## 参考链接
感谢这篇文章的大佬：https://www.jianshu.com/p/3b550e903e78


---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/attention-is-all-you-need%E8%AE%BA%E6%96%87%E5%8F%8A%E4%BB%A3%E7%A0%81/  

