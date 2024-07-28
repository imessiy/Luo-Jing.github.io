# 1 Abstract
低温计算得益于漏电功耗和导线电阻的降低，使得计算性能显著提高，但是与之而来的挑战是如何设计并部署和低温计算相适配的架构

这篇论文提出了一种与低温计算适配的Cache结构，作者首先分析了各种片上存储器在77K下运行的成本效益和可行性，然后基于可行的技术构建了Cache，并且评估结果表明这个Cache架构在低温下运行相较于传统Cache架构在常温下运行有2倍访问速度和2倍容量的优势

# 2 Introduction
- 从性能，功耗等方面评估主流的Cache单元技术，最后选择**6T-SRAM**和**3T-eDRAM**
- 证明设计的Cache在访问延迟和容量方面的优势
- 通过降低供电电压来减少cooling cost
- 最后提出Cache设计方案并评估性能

# 3 Cell technologies for cryogenic caches
![](https://pic.imgdb.cn/item/66a5d575d9c307b7e970854d.png)
Cache单元结构技术对比

# 4 Cryogenic cache modeling and analysis
运用CryoRAM（一款低温存储器模型工具）对**6T_SRAM**和**3T-eDRAM**进行仿真。由于这款工具没有**3T-eDRAM**的模型，所以需要对其建模，建模之后再将模型仿真的结果（延迟，功耗等）与实际结果进行对比，从而验证建模是否合理正确

# 5 CryoCache: 77K-optimal cache design
## 5.1 Vdd and Vth scaling
77K运行cooling cost很高，为了减小cooling cost，需要减小动态功耗，所以需要调节Vdd和Vth，确定合适的Vdd和Vth使得功耗最低
## 5.2 Latency analysis
![](https://pic.imgdb.cn/item/66a5e4ced9c307b7e97ea99a.png)
## 5.3 Energy consumption analysis
![](https://pic.imgdb.cn/item/66a5e608d9c307b7e97faeb7.png)

# 6 Evaluation using 77K caches
## 6.1 Evaluation methodology
评估方案：使用**Gem5 timing simulator**,，参数设置，能耗计算公式
## 6.2&6.3 Performance&Energy evaluation
![](https://pic.imgdb.cn/item/66a5ea8fd9c307b7e9838cc2.png)

# 7 Discussion
- 不仅仅局限于低温Cache，可以使整个计算机系统都工作在低温状态
- cache-to-pipelinerr interface
- relevant to quantum computing

