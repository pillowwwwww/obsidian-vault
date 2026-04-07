# FedDGC-B Final Method

  

## 1. 方法定位

  

本文最终采用的方法为：

  

**FedDGC-B (Federated Directional Gap Correction for LoRA-B，面向 LoRA-B 的联邦方向缺口校正)**

  

该方法只作用于 **LoRA-B**，不把 bundle（方向束）作为方法对象，不使用 subspace-based design（基于子空间的设计），不使用 residual accumulation（残差累积），不要求服务器访问客户端验证数据，也不采用"整个 `B` 永久本地化"的方案。

  

它的核心目标不是保留完整本地 `B`，而是在标准联邦聚合之后，仅恢复那些：

  

- 对客户端本地任务稳定存在

- 在聚合后没有被全局模型保住

- 同时具有本地功能意义

  

的少量 **direction-level atoms（方向级原子）**。

  

因此，FedDGC-B 的最终解释是：

  

> 服务器仍然学习共享的全局 `LoRA-B`；客户端只额外缓存一小组"共享聚合没有保住的方向缺口"，并在下一轮把这些方向缺口作为稀疏初始化校正注入回去。

  

## 2. 设计原则

  

FedDGC-B 必须同时满足以下约束：

  

1. **共享主导，不做整块本地化**

  全局 `LoRA-B` 仍然由标准联邦聚合得到。客户端只保留少量方向缺口缓存，而不是整个本地 `B`。

2. **方向级而非 bundle 级**

  方法对象是单个局部奇异方向原子 `phi_i`，不是滑窗 bundle。bundle 只保留在已有 motivation（动机）和 causal evidence（因果证据）解释层，不进入最终方法定义。

3. **只使用客户端本地可获得信息**

  所有方向筛选信号都来自：

  - 本地训练后的 `B`

  - 服务器回传的聚合后 `G`

  - 客户端自己的本地验证集

   服务器不接触客户端验证数据。

4. **缓存稀疏、可控、可覆盖**

  缓存不能退化成"另存一份本地 `B`"。每轮缓存按预算重新选择并覆盖，不做跨轮残差累积。

5. **优先补回"聚合没保住"的方向**

  方法的对象是 directional gap（方向缺口），不是任意本地方向，也不是全局残差。

  

## 3. 问题设定与记号

  

考虑联邦训练第 `t` 轮，参与客户端集合为 `S_t`。

  

对客户端 `c`、层 `l`，本地训练完成后的 LoRA-B 记为：

  

$$

B_{c,l}^{t,\mathrm{loc}} \in \mathbb{R}^{d_l \times r_l}

$$

  

服务器按标准方式聚合得到共享全局 LoRA-B：

  

$$

G_l^t = \sum_{c \in S_t} p_c B_{c,l}^{t,\mathrm{loc}},

\qquad

\sum_{c \in S_t} p_c = 1

$$

  

其中 `p_c` 为标准 FedAvg（联邦平均）权重，通常由本地样本数归一化得到。

  

对每个客户端、每层、每轮，在本地 LoRA-B 上做 SVD（奇异值分解）：

  

$$

B_{c,l}^{t,\mathrm{loc}} = \sum_{i=1}^{r_l} \sigma_{c,l,i}^t \, u_{c,l,i}^t \left(v_{c,l,i}^t\right)^\top

$$

  

定义方向级原子：

  

$$

\phi_{c,l,i}^t = u_{c,l,i}^t \left(v_{c,l,i}^t\right)^\top

$$

  

由于 `u` 和 `v` 都是单位向量，故有：

  

$$

\|\phi_{c,l,i}^t\|_F = 1

$$

  

因此，`sigma_i` 就是本地方向原子 `phi_i` 在 `B_{c,l}^{t,\mathrm{loc}}` 中的系数。

  

## 4. FedDGC-B 的核心对象

  

### 4.1 全局在本地方向上的带符号系数

  

定义全局矩阵 `G_l^t` 在客户端 `c` 的本地方向原子 `phi_{c,l,i}^t` 上的带符号投影系数为：

  

$$

\alpha_{c,l,i}^t = \left\langle G_l^t, \, \phi_{c,l,i}^t \right\rangle_F

$$

  

这里的 `alpha` 允许为负。

  

- 若 `alpha > 0`，说明全局模型在该本地方向上保留了同向成分。

- 若 `alpha = 0`，说明全局模型没有保住该方向。

- 若 `alpha < 0`，说明全局模型在该方向上发生了反向投影，存在方向翻转或冲突。

  

### 4.2 方向缺口

  

定义 directional gap（方向缺口）为：

  

$$

\delta_{c,l,i}^t = \max\left(0, \, \sigma_{c,l,i}^t - \alpha_{c,l,i}^t \right)

$$

  

它表示：

  

> 在客户端自己的局部方向坐标系里，当前全局模型相对于本地训练结果还缺多少该方向系数。

  

这一定义有三个关键优点：

  

1. 当 `0 < alpha < sigma` 时，`delta` 正好是未被保住的同向剩余量。

2. 当 `alpha < 0` 时，`delta` 自动变大，能把"方向翻转"作为更严重的缺口处理。

3. 当 `alpha >= sigma` 时，`delta = 0`，表示该方向无需额外保护。

  

### 4.3 方向脆弱性

  

定义 normalized fragility（归一化脆弱性）：

  

$$

f_{c,l,i}^t = \frac{\delta_{c,l,i}^t}{\sigma_{c,l,i}^t + \epsilon}

$$

  

`f_i` 越大，表示该本地方向越没有被聚合保住。

  

## 5. 六个漏洞的最终修补

  

下面给出对六个关键漏洞的最终修补版本。

  

### 5.1 漏洞一：方向缺口是否闭合

  

修补结论：

  

> FedDGC-B 将 `delta_i` 明确定义为 **client-local projection gap（客户端局部投影缺口）**，而不是矩阵意义上的完整残差。

  

这意味着方法只试图恢复：

  

$$

\left\{ \phi_{c,l,i}^t \right\}

$$

  

张成的本地方向原子集合上的缺口，而不试图解释 `G_l^t` 的全部成分。

  

因此方法的目标是闭合的：

  

- 本地方向由 `B_{c,l}^{t,\mathrm{loc}}` 的 SVD 唯一给出

- 全局在这些方向上的系数由 Frobenius inner product（Frobenius 内积）给出

- 方向缺口就是这些方向上的系数不足量

  

这已经足够支撑"下一轮初始化时补回本地方向缺口"的方法目标，不需要额外定义全空间残差。

  

### 5.2 漏洞二：`G + C` 是否会退化成"另存一份本地 B"

  

修补结论：

  

> 不直接存 dense matrix（稠密矩阵）`C`，而改为 **direction-sparse factorized cache（方向稀疏分解缓存）**。

  

客户端不存：

  

$$

C_{c,l}^t \in \mathbb{R}^{d_l \times r_l}

$$

  

的完整矩阵形式，而只存一个稀疏方向表：

  

$$

\mathcal{C}_{c,l}^t = \left\{ \left(i, \, \delta_{c,l,i}^t, \, u_{c,l,i}^t, \, v_{c,l,i}^t \right) \mid i \in I_{c,l}^{*,t} \right\}

$$

  

只有在下一轮初始化前，才临时物化：

  

$$

C_{c,l}^t = \sum_{i \in I_{c,l}^{*,t}} \delta_{c,l,i}^t \, \phi_{c,l,i}^t

$$

  

并注入：

  

$$

B_{c,l}^{t+1,\mathrm{init}} = G_l^t + \lambda_{c,t} \, C_{c,l}^t

$$

  

其中 `lambda_{c,t}` 是注入强度系数。

  

这样一来，缓存存储复杂度从：

  

$$

O(d_l r_l)

$$

  

降为：

  

$$

O\left(K_l^{*,t}(d_l + r_l)\right)

$$

  

其中 `K_l^{*,t} = |I_{c,l}^{*,t}|`，通常远小于 `r_l`。

  

因此，FedDGC-B 在存储语义上不再等价于"保存整个本地 `B`"。

  

### 5.3 漏洞三：round 0 到 round 1 的冷启动

  

修补结论：

  

> round 0 没有历史时，不让方法依赖 persistence（持续性）项，而使用更保守的选择门控和更弱的注入强度。

  

定义客户端最近一次缓存更新轮次：

  

$$

\tau_c(t) = \max\{s < t \mid c \in S_s\}

$$

  

若不存在，则视为无历史缓存。

  

在 round 0 或无历史缓存时：

  

$$

h_{c,l,i}^t = 0

$$

  

同时采用三项冷启动保护：

  

1. 更严格的候选阈值：

  - `f_i \ge \tau_f^{cold}`

  - `e_i \ge \tau_e^{cold}`

  - `u_i \ge \tau_u^{cold}` 或 `u_i^{proxy} \ge \tau_u^{cold}`

2. 更小的预算：

  - `K_max^{cold} < K_max`

  - `rho_max^{cold} < rho_max`

3. 更弱的注入强度：

  

$$

\lambda_{c,t} = \begin{cases} \lambda_{\mathrm{cold}}, & \text{if no valid history cache exists} \\ \lambda_{\mathrm{warm}}, & \text{otherwise} \end{cases} \qquad 0 < \lambda_{\mathrm{cold}} \le \lambda_{\mathrm{warm}} \le 1

$$

  

实践上可取：

  

- `lambda_cold = 0.3 ~ 0.5`

- `lambda_warm = 0.7 ~ 1.0`

  

若客户端连续若干轮未被选中，也可以加入 staleness decay（陈旧性衰减）：

  

$$

\lambda_{c,t} = \lambda_{\mathrm{warm}} \cdot \eta^{t - \tau_c(t) - 1}, \qquad \eta \in (0,1]

$$

  

这样可以抑制过旧缓存对新一轮初始化的干扰。

  

### 5.4 漏洞四：`u_i` 的本地计算成本过高

  

修补结论：

  

> 正式方法采用两阶段 utility（效用）估计：先做 proxy（代理）筛选，再对 shortlist（短名单）做 exact leave-one-out（精确逐方向剔除）。

  

#### 5.4.1 精确定义

  

精确本地效用定义为：

  

$$

B_{c,l}^{t,(-i)} = B_{c,l}^{t,\mathrm{loc}} - \sigma_{c,l,i}^t \, \phi_{c,l,i}^t

$$

  

$$

u_{c,l,i}^{t,\mathrm{exact}} = \mathcal{L}_{c}^{\mathrm{eval}}\!\left(B_{c,l}^{t,(-i)}\right) - \mathcal{L}_{c}^{\mathrm{eval}}\!\left(B_{c,l}^{t,\mathrm{loc}}\right)

$$

  

若 `u_i^{exact} > 0`，说明去掉该方向会恶化本地验证损失，因此该方向具有本地功能意义。

  

但如果对所有层、所有方向都做 exact leave-one-out，在线代价太高。

  

#### 5.4.2 一阶代理效用

  

对本地验证损失在当前 `B_{c,l}^{t,\mathrm{loc}}` 处求梯度：

  

$$

G_{c,l}^{t,\mathrm{eval}} = \nabla_{B_{c,l}} \mathcal{L}_{c}^{\mathrm{eval}}

$$

  

当从当前点移除方向 `i` 时，扰动为：

  

$$

\Delta B_{c,l,i}^t = -\sigma_{c,l,i}^t \, \phi_{c,l,i}^t

$$

  

其一阶损失变化近似为：

  

$$

u_{c,l,i}^{t,\mathrm{proxy}} \approx \left\langle G_{c,l}^{t,\mathrm{eval}}, \, \Delta B_{c,l,i}^t \right\rangle_F = -\sigma_{c,l,i}^t \left\langle G_{c,l}^{t,\mathrm{eval}}, \, \phi_{c,l,i}^t \right\rangle_F

$$

  

最终取非负化版本：

  

$$

u_{c,l,i}^{t,\mathrm{proxy},+} = \max\!\left( 0, \; -\sigma_{c,l,i}^t \left\langle G_{c,l}^{t,\mathrm{eval}}, \, \phi_{c,l,i}^t \right\rangle_F \right)

$$

  

这表示：

  

> 若沿 `-sigma_i phi_i` 方向的一阶变化会增大本地验证损失，则该方向对本地任务有正向作用。

  

#### 5.4.3 两阶段计算流程

  

在线版本采用：

  

1. 用 `f_i`、`e_i`、`h_i` 先做粗筛，得到候选短名单 `J_{c,l}^t`

2. 仅对短名单方向计算 `u_i^{proxy}`

3. 只对最终 Top-M 候选计算 `u_i^{exact}`

  

因此正式方法中的效用信号定义为：

  

$$

u_{c,l,i}^t = \begin{cases} u_{c,l,i}^{t,\mathrm{exact}}, & i \in \text{Top-M shortlist} \\ u_{c,l,i}^{t,\mathrm{proxy},+}, & i \in J_{c,l}^t \setminus \text{Top-M} \\ 0, & \text{otherwise} \end{cases}

$$

  

这样既保留了功能意义过滤，又把在线计算量控制在可接受范围内。

  

### 5.5 漏洞五：历史项如何定义，且不能演化成 residual accumulation

  

修补结论：

  

> 历史项只衡量当前方向与"最近一次缓存方向集合"的重合程度，不累计旧残差，不跨轮相加。

  

设客户端最近有效缓存为：

  

$$

\mathcal{C}_{c,l}^{\tau_c(t)} = \left\{ \left(j, \, \delta_{c,l,j}^{\tau_c(t)}, \, u_{c,l,j}^{\tau_c(t)}, \, v_{c,l,j}^{\tau_c(t)} \right) \right\}

$$

  

对应临时物化矩阵为：

  

$$

\widetilde{C}_{c,l}^{\tau_c(t)} = \sum_{j \in I_{c,l}^{*,\tau_c(t)}} \delta_{c,l,j}^{\tau_c(t)} \, \phi_{c,l,j}^{\tau_c(t)}

$$

  

定义当前方向 `i` 的持续性：

  

$$

h_{c,l,i}^t = \max\!\left( 0, \; \frac{ \left\langle \sigma_{c,l,i}^t \, \phi_{c,l,i}^t, \; \widetilde{C}_{c,l}^{\tau_c(t)} \right\rangle_F }{ \sigma_{c,l,i}^t \left\| \widetilde{C}_{c,l}^{\tau_c(t)} \right\|_F + \epsilon } \right)

$$

  

这一定义的含义是：

  

- 若当前方向与最近缓存中保护过的方向仍高度对齐，则 `h_i` 大

- 若当前方向与历史缓存无关，则 `h_i` 小

- 若内积为负，则经过 `max(0, ·)` 截断，不鼓励反向延续

  

关键点在于：

  

- `h_i` 只使用最近缓存

- 每轮缓存 **overwrite（覆盖）** 而不是加和

- 不保存服务器残差

- 不累积多轮纠错项

  

因此 FedDGC-B 仍然不是 residual accumulation 方法。

  

### 5.6 漏洞六：筛选和预算如何避免"每层选满小噪声"

  

修补结论：

  

> 最终选择机制改为"层内粗筛 + 客户端全局贪心预算分配"，并把预算绑定到 correction energy（校正能量）而不是原始方向标签。

  

#### 5.6.1 原始能量和校正能量

  

本地方向原始能量定义为：

  

$$

e_{c,l,i}^t = \frac{ \left(\sigma_{c,l,i}^t\right)^2 }{ \sum_j \left(\sigma_{c,l,j}^t\right)^2 + \epsilon }

$$

  

但真正被缓存并在下一轮注入的，是缺口校正量 `delta_i`，因此需要单独定义 correction energy（校正能量）：

  

$$

g_{c,l,i}^t = \frac{ \left(\delta_{c,l,i}^t\right)^2 }{ \sum_j \left(\sigma_{c,l,j}^t\right)^2 + \epsilon }

$$

  

最终预算约束应该作用在 `g_i` 上，而不是仅作用在 `e_i` 上。

  

#### 5.6.2 分层筛选得分

  

先定义 vulnerability-energy base score（脆弱性-能量基础分）：

  

$$

s_{c,l,i}^t = \operatorname{Norm}_l\!\left(f_{c,l,i}^t\right) \cdot \operatorname{Norm}_l\!\left(e_{c,l,i}^t\right)

$$

  

其中 `Norm_l` 表示在同一层内做归一化。

  

最终方向得分为：

  

$$

q_{c,l,i}^t = s_{c,l,i}^t + \beta \, \operatorname{Norm}_{J_{c,l}^t}\!\left(u_{c,l,i}^t\right) + \gamma \, h_{c,l,i}^t

$$

  

其中：

  

- `beta >= 0` 控制本地功能意义的重要性

- `gamma >= 0` 控制历史持续性的权重

  

#### 5.6.3 选择规则

  

正式方法分两级选择：

  

1. **层内粗筛**

  对每层保留满足下列条件的候选方向：

  - `f_i >= tau_f`

  - `e_i >= tau_e`

  - `u_i >= tau_u` 或 `u_i^{proxy} >= tau_u`

  - 每层最多保留 `K_pre` 个高分候选

2. **客户端全局贪心选择**

  把所有层的候选合并成客户端级候选池，按 `q_i / (g_i + eps)` 或按 `q_i` 进行排序，做全局贪心选择，并满足：

  - 每层最终最多保护 `K_max` 个方向

  - 客户端总校正能量预算不超过 `rho_max`

  

形式化地：

  

$$

I_{c}^{*,t} = \operatorname{GreedySelect} \left( \bigcup_l J_{c,l}^t, \; K_{\max}, \; \rho_{\max} \right)

$$

  

其中最终被选方向集合在层 `l` 上的投影记为：

  

$$

I_{c,l}^{*,t} = I_c^{*,t} \cap \{(l,i)\}

$$

  

预算约束为：

  

$$

\sum_{l} \sum_{i \in I_{c,l}^{*,t}} g_{c,l,i}^t \le \rho_{\max}

$$

  

这样能够防止：

  

- 小层中低绝对能量方向被机械选满

- 某一层过度占据全部缓存预算

  

## 6. 正式方法定义

  

综合以上修补，FedDGC-B 的正式定义如下。

  

### 6.1 客户端本地分解

  

客户端 `c` 在第 `t` 轮本地训练结束后，对每个 LoRA-B 层 `l` 做：

  

$$

B_{c,l}^{t,\mathrm{loc}} \xrightarrow{\mathrm{SVD}} \left\{ \sigma_{c,l,i}^t, \, u_{c,l,i}^t, \, v_{c,l,i}^t \right\}_{i=1}^{r_l}

$$

  

形成方向原子：

  

$$

\phi_{c,l,i}^t = u_{c,l,i}^t (v_{c,l,i}^t)^\top

$$

  

### 6.2 服务器标准聚合

  

服务器按原有 FedAvg（联邦平均）流程聚合：

  

$$

G_l^t = \sum_{c \in S_t} p_c B_{c,l}^{t,\mathrm{loc}}

$$

  

FedDGC-B 不改服务器主聚合规则。

  

### 6.3 客户端方向缺口估计

  

服务器广播 `G^t` 后，客户端对每层、每个本地方向计算：

  

$$

\alpha_{c,l,i}^t = \left\langle G_l^t, \, \phi_{c,l,i}^t \right\rangle_F

$$

  

$$

\delta_{c,l,i}^t = \max(0, \, \sigma_{c,l,i}^t - \alpha_{c,l,i}^t)

$$

  

$$

f_{c,l,i}^t = \frac{\delta_{c,l,i}^t}{\sigma_{c,l,i}^t + \epsilon}

$$

  

$$

e_{c,l,i}^t = \frac{(\sigma_{c,l,i}^t)^2}{\sum_j (\sigma_{c,l,j}^t)^2 + \epsilon}

$$

  

$$

g_{c,l,i}^t = \frac{(\delta_{c,l,i}^t)^2}{\sum_j (\sigma_{c,l,j}^t)^2 + \epsilon}

$$

  

### 6.4 本地效用估计

  

客户端在自己的本地验证集上估计：

  

$$

u_{c,l,i}^t \in \left\{ u_{c,l,i}^{t,\mathrm{proxy},+}, \; u_{c,l,i}^{t,\mathrm{exact}} \right\}

$$

  

在线版优先使用 proxy，再对短名单做 exact 精修。

  

### 6.5 历史持续性估计

  

若存在最近缓存：

  

$$

h_{c,l,i}^t = \max\!\left( 0, \; \frac{ \left\langle \sigma_{c,l,i}^t \, \phi_{c,l,i}^t, \; \widetilde{C}_{c,l}^{\tau_c(t)} \right\rangle_F }{ \sigma_{c,l,i}^t \left\| \widetilde{C}_{c,l}^{\tau_c(t)} \right\|_F + \epsilon } \right)

$$

  

否则 `h_i = 0`。

  

### 6.6 方向打分与筛选

  

$$

s_{c,l,i}^t = \operatorname{Norm}_l(f_{c,l,i}^t) \cdot \operatorname{Norm}_l(e_{c,l,i}^t)

$$

  

$$

q_{c,l,i}^t = s_{c,l,i}^t + \beta \, \operatorname{Norm}_{J_{c,l}^t}(u_{c,l,i}^t) + \gamma \, h_{c,l,i}^t

$$

  

之后先层内粗筛，再客户端全局贪心分配预算。

  

### 6.7 方向稀疏缓存

  

对每层最终得到：

  

$$

\mathcal{C}_{c,l}^t = \left\{ \left( i, \, \delta_{c,l,i}^t, \, u_{c,l,i}^t, \, v_{c,l,i}^t \right) \mid i \in I_{c,l}^{*,t} \right\}

$$

  

对应的临时矩阵形式为：

  

$$

C_{c,l}^t = \sum_{i \in I_{c,l}^{*,t}} \delta_{c,l,i}^t \, \phi_{c,l,i}^t

$$

  

但这只是推导和注入时的临时表达，不是持久化存储形式。

  

### 6.8 下一轮初始化

  

客户端下一轮开始前，用：

  

$$

B_{c,l}^{t+1,\mathrm{init}} = G_l^t + \lambda_{c,t} \, C_{c,l}^t

$$

  

作为 LoRA-B 初始化。

  

其中 `lambda_{c,t}` 由冷启动/暖启动规则决定。

  

## 7. 方法的关键性质

  

### 7.1 被保护方向上的系数恢复性质

  

对任何被选中的方向 `i`，注入后该方向上的系数为：

  

$$

\left\langle G_l^t + C_{c,l}^t, \, \phi_{c,l,i}^t \right\rangle_F = \alpha_{c,l,i}^t + \delta_{c,l,i}^t

$$

  

由 `delta_i = max(0, sigma_i - alpha_i)` 可知：

  

$$

\alpha_i + \delta_i = \max(\alpha_i, \, \sigma_i)

$$

  

因此：

  

- 若 `alpha_i < sigma_i`，该方向系数被恢复到 `sigma_i`

- 若 `alpha_i >= sigma_i`，不会额外放大

  

这说明 FedDGC-B 的作用不是任意增强本地方向，而是 **补足聚合没有保住的缺口**。

  

### 7.2 对符号翻转天然敏感

  

若 `alpha_i < 0`，则：

  

$$

\delta_i = \sigma_i - \alpha_i > \sigma_i

$$

  

因此符号翻转会被识别为更严重的方向缺口，这与"聚合后该方向被明显冲掉甚至反向"的直觉一致。

  

### 7.3 存储不等价于整块本地化

  

FedDGC-B 只保存少量方向原子和对应系数，不保存整个本地 `B`。因此它与"整个 `B` 永久本地"的方案在存储语义和通信语义上都不同。

  

### 7.4 不依赖服务器访问客户端验证数据

  

`u_i` 只在客户端本地计算。服务器只看到标准聚合所需的 LoRA 参数，不接触本地验证集。

  

## 8. 与 FedSA-LoRA 和 FRLoRA 的边界差异

  

### 8.1 与 FedSA-LoRA 的差异

  

FedSA-LoRA 的主线是 selective aggregation（选择性聚合），核心做法是让 LoRA 某一部分长期留在本地，典型形式是"只共享 `A`，`B` 保留本地"。这相当于把完整 `B` 视为本地特有组件。

  

FedDGC-B 不做这一点。

  

FedDGC-B 的立场是：

  

- 整个 `B` 仍然参与共享聚合

- 只在客户端侧缓存少量"聚合没有保住的方向缺口"

- 缓存是稀疏、方向级、按预算覆盖更新的

  

所以 FedDGC-B 不是 "keep all local B（保留整个本地 B）" 方法，而是 "keep only missing directions after aggregation（只补回聚合后丢失的少量方向）" 方法。

  

参考：

  

- Selective Aggregation for Low-Rank Adaptation in Federated Learning: [https://openreview.net/forum?id=iX3uESGdsO](https://openreview.net/forum?id=iX3uESGdsO)

  

### 8.2 与 FRLoRA 的差异

  

FRLoRA 的主线是 federated residual low-rank adaptation（联邦残差低秩适配），强调用 residual low-rank global update（残差低秩全局更新）和 SVD reinitialization（SVD 重初始化）提升全局更新表达能力。

  

FedDGC-B 不采用：

  

- residual accumulation（残差累积）

- 全局残差低秩重构

- 整轮的 SVD 重初始化逻辑

  

FedDGC-B 只做一件事：

  

> 在客户端自己的局部 LoRA-B 坐标系里，识别被聚合漏掉的方向缺口，并把这些缺口作为稀疏初始化校正注入下一轮。

  

参考：

  

- Federated Residual Low-Rank Adaptation of Large Language Models: [https://openreview.net/forum?id=e0rQRMUhs7](https://openreview.net/forum?id=e0rQRMUhs7)

  

## 9. 在线版、Oracle 版与离线回放版

  

### 9.1 在线版

  

正式在线方法如下：

  

1. 本地训练得到 `B_{c,l}^{t,loc}`

2. 服务器标准聚合得到 `G_l^t`

3. 客户端计算 `alpha_i, delta_i, f_i, e_i, g_i`

4. 客户端用本地验证集估计 `u_i^{proxy}`

5. 选出短名单后，对 Top-M 做 `u_i^{exact}`

6. 计算 `h_i`

7. 在 `K_max` 和 `rho_max` 预算下选方向

8. 保存方向稀疏缓存 `mathcal{C}_{c,l}^t`

9. 下一轮用 `G_l^t + lambda_{c,t} C_{c,l}^t` 初始化

  

### 9.2 Oracle 版

  

Oracle（上界）版本只用于实验上界，不作为正式部署版本。

  

它与在线版的区别只有一点：

  

- 对所有候选方向都计算 `u_i^{exact}`

  

因此 Oracle 版回答的问题是：

  

> 如果本地效用估计没有近似误差，FedDGC-B 的理论上界能到哪里。

  

### 9.3 离线回放版

  

离线回放版不重启训练，只在已保存的：

  

- 本地 pre-aggregation checkpoint（聚合前检查点）

- 同轮 global post-aggregation checkpoint（同轮聚合后全局检查点）

  

之间离线构造 `C`，再评测：

  

$$

G^t \quad \text{vs.} \quad G^t + \lambda C^t

$$

  

它只验证"方向缺口补回是否改善初始化"，适合作为正式在线实现前的低成本检验。

  

## 10. 推荐超参数与默认实现

  

下面给出一版适合当前主线的默认实现建议。

  

### 10.1 分数权重

  

- `beta = 1.0`

- `gamma = 0.5`

  

解释：

  

- 当前主线需要明确保留本地功能意义，所以 `beta` 不应过小

- 历史项有帮助，但不能压过当前轮的脆弱性和功能意义，因此 `gamma` 应弱于 `beta`

  

### 10.2 候选筛选

  

- `tau_f = 0.1`

- `tau_e = 0.005`

- `tau_u = 0`

- `K_pre = 8` per layer（每层）

  

### 10.3 最终预算

  

- `K_max = 2 ~ 4` per layer（每层）

- `rho_max = 0.05 ~ 0.10` per client-round（每客户端每轮）

  

### 10.4 冷启动

  

- `lambda_cold = 0.4`

- `lambda_warm = 0.8`

- `K_max_cold = floor(0.5 * K_max)`

- `rho_max_cold = 0.5 * rho_max`

  

### 10.5 陈旧性衰减

  

- `eta = 0.9`，当客户端跨多轮未参与时启用

  

## 11. 训练流程中的落地位置

  

如果后续进入实现，FedDGC-B 的主要改动位点应当是：

  

1. **客户端下一轮初始化前**

  在加载 `global_params` 后，把方向稀疏缓存物化成 `C` 并做：

  - `B_init = G + lambda C`

2. **每轮本地训练完成后**

  在客户端侧：

  - 对 LoRA-B 做 SVD

  - 读取同轮全局 `G`

  - 计算 `alpha, delta, f, e, g`

  - 用本地验证集估计 `u`

  - 选择方向并覆盖更新缓存

3. **服务器侧**

  保持标准 FedAvg 主聚合不变，只需要继续保存同轮聚合后 `global_postagg.pt`，供离线分析和可能的客户端后处理使用

  

## 12. 最终方法表述

  

最终，FedDGC-B 可以用下面这段话作为正式定义：

  

> FedDGC-B is a direction-level client-side correction method for federated LoRA-B training. After each round of standard global aggregation, each client decomposes its locally trained LoRA-B into singular direction atoms, measures how much of each local direction is preserved by the aggregated global model, and caches only a small budgeted set of missing directional coefficients in a sparse factorized form. In the next round, the client initializes LoRA-B with the shared global model plus this sparse directional gap correction. The method therefore preserves standard global sharing, avoids storing the entire local `B`, uses only client-local validation for utility estimation, and targets precisely the low-retention yet functionally meaningful local directions that global aggregation fails to maintain.

  

对应中文表述为：

  

> FedDGC-B 是一个面向联邦 LoRA-B 训练的方向级客户端侧校正方法。每轮标准全局聚合之后，客户端先把本地训练得到的 LoRA-B 分解为一组局部奇异方向原子，再测量全局聚合结果在这些本地方向上的保留程度，并仅缓存其中少量、受预算约束的方向缺口系数。下一轮开始时，客户端以"共享全局 LoRA-B + 稀疏方向缺口校正"作为初始化。该方法因此同时满足：保持标准全局共享、不保存整个本地 `B`、只使用客户端本地验证信息、并且只针对那些被聚合漏掉但对本地任务仍有功能意义的方向进行补回。

  

## 13. 当前结论边界

  

这版最终方法与当前证据链边界保持一致：

  

- 它的目标是保护 **client-specific + stable + functionally relevant + aggregation-vulnerable** 的 LoRA-B 方向

- 但当前整体论证仍应保持谨慎，不直接写成 "fully proven client-specific knowledge carrier（被完全证明的客户端特异知识载体）"

- 在方法目标上，更稳妥的表述是：

  

> FedDGC-B is designed to restore source-preferential, aggregation-vulnerable, and functionally meaningful LoRA-B directions without abandoning global sharing.