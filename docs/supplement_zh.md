# 相关会议与预印本补充

[返回首页](../README.md) · [主目录](papers_zh.md)

以下 9 条记录用于补足与 MoonBright 高度相关的机制或平台背景，**不计入主目录 45 篇**。ICS、ICPP、IISWC、PACT、ISPASS 是相关专业会议，本次采用的主目录会议集合未包括它们；这不表示论文价值低。未确认正式顶会归属的预印本保持独立标记。

DREAM 与 GPUVM 的作者和机制有联系，暂按一条文献线索呈现，并保留二者链接；没有足够证据时不宣称二者全文或版本完全相同。

<a id="dream"></a>

## DREAM · ICS 2025

**DREAM: Device-Driven Efficient Access to Virtual Memory**

[原文](https://hpcrl.github.io/ICS2025-webpage/program/Proceedings_ICS25/ics25-9.pdf) · [代码](https://github.com/nnurlan008/dream)

- **核心机制：** 由 GPU 线程管理软件分页，并借助 RDMA 数据路径访问 CPU 内存，减少 CPU 服务开销。
- **与 MoonBright 的关系：** 与设备端自主内存管理高度相关；不能理解为 GPU 直接修改 CPU 的硬件页表。
- **实现层次 / 复现条件：** GPU 运行时与 RDMA NIC 等系统支持。
- **会议核验：** [会议程序或论文列表](https://hpcrl.github.io/ICS2025-webpage/program/program.html)。
- **相关预印本：** [GPUVM: GPU-driven Unified Virtual Memory](https://arxiv.org/abs/2411.05309)。作者和研究机制相关，可能存在版本演进；未确认可逐项等同，不将 GPUVM 另计为一篇顶会论文。
- **核验边界：** 相关会议 / 预印本，未计入主目录的顶会论文数；未运行实验。

<a id="dynamic-memory-ics22"></a>

## GPU Dynamic Memory (ICS 2022) · ICS 2022

**Dynamic Memory Management in Massively Parallel Systems: A Case on GPUs**

[原文](https://pmc.ncbi.nlm.nih.gov/articles/PMC9357265/)

- **核心机制：** 研究面向大量 GPU 并行线程的动态内存分配，减少集中式元数据争用。
- **与 MoonBright 的关系：** 设备端分配器补充；与 Gallatin 可对读。
- **实现层次 / 复现条件：** GPU 分配器软件。
- **核验边界：** 相关会议 / 预印本，未计入主目录的顶会论文数；未运行实验。

<a id="svm-ics24"></a>

## Shared Virtual Memory · ICS 2024

**Shared Virtual Memory: Its Design and Performance Implications for Diverse Applications**

[原文](https://arxiv.org/abs/2405.06811) · [DOI](https://doi.org/10.1145/3650200.3656608)

- **核心机制：** 研究 AMD Shared Virtual Memory 的设计、迁移与不同访问模式下的性能。
- **与 MoonBright 的关系：** 补充 NVIDIA UVM 之外的平台差异。
- **实现层次 / 复现条件：** 实际 AMD 平台性能研究。
- **核验边界：** 相关会议 / 预印本，未计入主目录的顶会论文数；未运行实验。

<a id="grace-hopper-icpp24"></a>

## Grace Hopper Memory · ICPP 2024

**Harnessing Integrated CPU-GPU System Memory for HPC: a first look into Grace Hopper**

[原文](https://arxiv.org/abs/2407.07850)

- **核心机制：** 考察 Grace Hopper 的 CPU-GPU 系统内存访问、分配方式和放置行为。
- **与 MoonBright 的关系：** 理解 ATS / 一致性互连背景；不把集成 CPU-GPU 内存等同于离散 GPU UVM。
- **实现层次 / 复现条件：** Grace Hopper 实机研究，平台相关。
- **核验边界：** 正式会议为 ICPP 2024，不是 SC；列为相关平台补充。

<a id="mi300a-iiswc25"></a>

## MI300A Unified Physical Memory · IISWC 2025

**Dissecting CPU-GPU Unified Physical Memory on AMD MI300A APUs**

[原文](https://arxiv.org/abs/2508.12743)

- **核心机制：** 分析 MI300A 上 CPU-GPU 统一物理内存行为。
- **与 MoonBright 的关系：** 区分统一地址空间、统一物理内存与迁移式 UVM。
- **实现层次 / 复现条件：** AMD MI300A 平台研究。
- **会议核验：** [会议程序或论文列表](https://iiswc.org/iiswc2025/program.html)。
- **核验边界：** 相关会议 / 预印本，未计入主目录的顶会论文数；未运行实验。

<a id="fs-hpt"></a>

## Fixed-Size Hashed Page Table · PACT 2024

**Rethinking Page Table Structure for Fast Address Translation in GPUs: A Fixed-Size Hashed Page Table**

[原文](https://scholarx.skku.edu/item/c26aa456-59d4-4c80-89b0-7b93fe6cc2fc) · [DOI](https://doi.org/10.1145/3656019.3676900)

- **核心机制：** 以固定大小哈希页表替代传统页表组织，优化 GPU 地址翻译。
- **与 MoonBright 的关系：** 页表结构补充；与构建和查找页表的成本直接相关。
- **实现层次 / 复现条件：** GPU 页表 / 地址翻译体系结构方案。
- **核验边界：** 相关会议 / 预印本，未计入主目录的顶会论文数；未运行实验。

<a id="early-adaptor"></a>

## Early-Adaptor · ISPASS 2023

**Early-Adaptor: An Adaptive Framework for Proactive UVM Memory Management**

[原文](https://ispass.org/ispass2023/accepted-papers.php) · [DOI](https://doi.org/10.1109/ISPASS57527.2023.00032)

- **核心机制：** 根据缺页等运行时信息自适应地提前管理 UVM 数据移动。
- **与 MoonBright 的关系：** UVM 预取与迁移策略补充。
- **实现层次 / 复现条件：** UVM 软件管理方案。
- **核验边界：** 相关会议 / 预印本，未计入主目录的顶会论文数；未运行实验。

<a id="vtensor"></a>

## vTensor · arXiv 2024

**vTensor: Flexible Virtual Tensor Management for Efficient LLM Serving**

[原文](https://arxiv.org/abs/2407.15309)

- **核心机制：** 围绕虚拟张量抽象组织 LLM 服务中的动态内存管理。
- **与 MoonBright 的关系：** 虚拟内存与张量接口相关；正式顶会归属未确认。
- **实现层次 / 复现条件：** 运行时软件方案；环境需查原文。
- **核验边界：** 仅核验为预印本；不与同名 VTensor 布局框架混为一篇，不计入主目录。

<a id="gmem"></a>

## GMEM · arXiv 2023

**GMEM: Generalized Memory Management for Peripheral Devices**

[原文](https://arxiv.org/abs/2310.12554)

- **核心机制：** 研究面向外设的通用内存管理抽象。
- **与 MoonBright 的关系：** 扩展到操作系统与外设内存管理接口的背景。
- **实现层次 / 复现条件：** OS / 设备内存管理设计。
- **核验边界：** 检索到预印本；提交过某会议不等于被该会议录用，因此未计入顶会主目录。

