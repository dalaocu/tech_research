## 环境与记号

- **群与配对**：设

  $$
  G_1, G_2, G_T
  $$

  为阶为 $q$ 的乘法循环群，

  $$
  e: G_1 \times G_2 \to G_T
  $$

  为双线性配对函数，满足：

  $$
  e(g_1^a, g_2^b) = e(g_1, g_2)^{ab}
  $$
- 生成元：

  $$
  g_1 \in G_1,
  g_2 \in G_2
  $$
- **哈希函数**：

  $$
  H_2: \{0,1\}^* \to \mathbb{Z}_q
  $$

  为随机预言模型下的密码学哈希函数。
- **消息**：

  $$
  m \in \{0,1\}^*
  $$


# 具体算法

## 步骤1： 用户生成私钥对


- **用户私钥**：$msk = (s, t)$，其中 $s \in G_1$, $t \in G_2$。

+ **用户公钥**：随机选取 $ h \gets G^T $，公开公钥 $ \mathit{pk} = (h, h^{s-t}) $。
+ **Nickname 生成**：用户随机选择 $ r $，计算：

$ \text{nickname} = h^r, \quad \text{proof} = h^{r(s-t)}. $

+ **验证**：验证者检查是否成立：

$ \text{proof} \stackrel{?}{=} \text{nickname}^{s-t}. $

+ **隐私性**：其他人无法从 `nickname` 或 `proof` 推导出 $ s-t $，也无法将其关联到特定用户。

这样，用户的私钥差值 $ s-t $ 被有效隐藏，而 nickname 的归属权仅由用户和生成者知晓。

## 签名者说明

签名者定义：

$$
Z := e(X, g_2) \cdot e(g_1, Y)^{-1} \in G_T.
$$

对于合法签名者 $signer = (X, Y)$，若其由私钥 $(s, t)$ 生成（即 $X = g_1^s$, $Y = g_2^t$），则存在秘密值

$$
w = s - t \in \mathbb{Z}_q,
$$

使得

$$
Z = e(g_1, g_2)^w.
$$

本签名机制本质上是**证明签名者知道 $w$ 使得 $Z = e(g_1, g_2)^w$** 的 Schnorr 类型零知识签名，并通过挑战 $c$ 绑定消息 $m$ 和昵称 $signer$，防止重放攻击。

## 签名算法 $\operatorname{Sig}(signer, m)$

**输入**：昵称 $signer = (X, Y)$，消息 $m$。消息m包括了转账金额，nonce, 附加数据等字段。
**输出**：签名 $\sigma = (T, r_w)$。

**步骤**：

1. 计算：

   $$
   Z = e(X, g_2) \cdot e(g_1, Y)^{-1}.
   $$
2. 随机选取：

   $$
   k \xleftarrow{\$} \mathbb{Z}_q.
   $$
3. 生成承诺：

   $$
   T = e(g_1, g_2)^k \in G_T.
   $$
4. 计算挑战：

   $$
   c = H_2\big( \mathrm{encode}( nickname(sender) \,\|\, nickname(receiver) ) \,\|\, m \,\|\, \mathrm{encode}(T) \big) \in \mathbb{Z}_q.
   $$
5. 利用私钥 $msk = (s, t)$ 计算：

   $$
   w = s - t, \quad r_w = k + c \cdot w \in \mathbb{Z}_q.
   $$
6. 输出签名：

   $$
   \sigma = (T, r_w, Z).
   $$

---

## 验签算法 $\operatorname{UVf}(signer, m, \sigma)$

**输入**： $signer = (X, Y)$，消息 $m$，签名 $\sigma = (T, r_w)$。
**输出**：$\text{True}$（接受）或 $\text{False}$（拒绝）。

**步骤**：

1. 获得签名：

   $$
   \sigma = (T, r_w, Z).
   $$
2. 重新计算挑战（使用接收到的 $T$）：

   $$
   c = H_2\big( \mathrm{encode}( nickname(sender) \,\|\, nickname(receiver) ) \,\|\, m \,\|\, \mathrm{encode}(T) \big).
   $$
3. 验证以下等式是否成立：

   $$
   e(g_1, g_2)^{r_w} \stackrel{?}{=} T \cdot Z^c \quad \text{in } G_T.
   $$
4. 若等式成立，则输出 $\text{True}$；否则输出 $\text{False}$。

> ✅ **关键验签公式**：
>
> $$
> \boxed{e(g_1, g_2)^{r_w} = T \cdot Z^c}
> $$

---

## 正确性说明

若签名合法，则：

- $Z = e(g_1, g_2)^w$，
- $T = e(g_1, g_2)^k$，
- $r_w = k + c w$，

代入左边得：

$$
e(g_1, g_2)^{r_w} = e(g_1, g_2)^{k + c w} = e(g_1, g_2)^k \cdot \left(e(g_1, g_2)^w\right)^c = T \cdot Z^c.
$$

因此验签等式成立。

---

## 安全性与绑定性

- **存在性安全**：只有知道 $w = s - t$ 的用户才能生成有效签名。
- **不可伪造性**：攻击者无法在不知 $w$ 的情况下通过挑战验证。
- **抗重放**：挑战 $c$ 依赖于 $receiver$、 $sender$、$m$ 和 $T$，确保签名与特定消息和昵称绑定。
- **零知识特性**：签名不泄露 $w$ 本身，仅证明知识
