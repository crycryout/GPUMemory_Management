# 近五年 GPU 内存管理：逐篇笔记

[返回首页](../README.md) · [机制综述](survey_zh.md) · [补充文献](supplement_zh.md)

共 45 篇，包含 MoonBright。按研究层次排列；摘要为中文归纳，不是论文原文翻译，也不是复现实验结论。实现层次用于初筛，不代表任意驱动、GPU 或框架版本均可直接运行。

# 分配器、虚拟地址与碎片

<a id="moonbright"></a>

## MoonBright · OSDI 2026

**MoonBright: A GPU Memory Allocator with Device-Side Page Table Materialization and Deferred TLB Coherence**

[原文](https://www.usenix.org/conference/osdi26/presentation/zhang-yangyu) · [PDF](https://www.usenix.org/system/files/osdi26-zhang-yangyu.pdf) · [代码](https://github.com/MoonBright-project/OSDI-26-AE)

- **核心机制：** CPU 保留校验及分配器元数据管理，GPU 并行构造页表项；结合新虚拟地址和延迟 TLB 一致性处理降低映射开销。
- **与 MoonBright 的关系：** 基准论文：关注分配与映射控制路径，而非仅减少应用张量占用。
- **实现层次 / 复现条件：** 驱动与运行时修改；论文在现有 NVIDIA/AMD GPU 上实现，安全模型需结合可信负载前提阅读。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="gmlake"></a>

## GMLake · ASPLOS 2024

**GMLake: Efficient and Transparent GPU Memory Defragmentation for Large-scale DNN Training with Virtual Memory Stitching**

[原文](https://www.cs.sjtu.edu.cn/~leng-jw/pubs/guo2024asplos-gmlake.html) · [DOI](https://doi.org/10.1145/3620665.3640423) · [代码](https://github.com/antgroup/glake)

- **核心机制：** 通过虚拟地址拼接，把分散物理显存组织成连续虚拟区间，缓解训练分配器的外部碎片。
- **与 MoonBright 的关系：** 直接相关：同样使用 GPU 虚拟内存映射；GMLake 解决碎片，MoonBright 进一步降低映射本身的成本。
- **实现层次 / 复现条件：** CUDA VMM / 框架分配器；无需设计新的 GPU 硬件。
- **预印本：** [作者版本](https://arxiv.org/abs/2401.08156)，以正式会议年份归档。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="vattention"></a>

## vAttention · ASPLOS 2025

**vAttention: Dynamic Memory Management for Serving LLMs without PagedAttention**

[原文](https://apanwariisc.github.io/publications/asplos-2025-vattention/vattention-asplos25.pdf) · [DOI](https://doi.org/10.1145/3669940.3707256) · [代码](https://github.com/microsoft/vattention)

- **核心机制：** 为 KV cache 保留连续虚拟地址，按需提供物理页，使普通 attention kernel 可以使用动态分配的 KV 内存。
- **与 MoonBright 的关系：** 直接相关：把应用块表管理交给硬件地址翻译，映射成本与页粒度是关键问题。
- **实现层次 / 复现条件：** 运行时与 GPU VMM；论文原型的页粒度及驱动支持应按 artifact 环境确认。
- **预印本：** [作者版本](https://arxiv.org/abs/2405.04437)，以正式会议年份归档。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="gallatin"></a>

## Gallatin · PPoPP 2024

**Gallatin: A General-Purpose GPU Memory Manager**

[原文](https://prashantpandey.github.io/assets/pdf/uploads/ppopp24-final274.pdf) · [DOI](https://doi.org/10.1145/3627535.3638499) · [代码](https://github.com/saltsystemslab/gallatin)

- **核心机制：** 面向 GPU 线程并发 malloc/free，采用适合并行查找的分配结构，支持不同大小的动态分配。
- **与 MoonBright 的关系：** 直接相关于分配器；管理设备堆空间与安装 GPU 硬件 PTE 是不同层次。
- **实现层次 / 复现条件：** GPU 设备端分配器；现有 GPU 上的软件方案。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="stalloc"></a>

## STAlloc · EuroSys 2026

**STAlloc: Enhancing Memory Efficiency in Large-Scale Model Training with Spatio-Temporal Planning**

[原文](https://arxiv.org/abs/2507.16274) · [DOI](https://doi.org/10.1145/3767295.3769335) · [代码](https://github.com/infinigence/STAlloc)

- **核心机制：** 结合张量生命周期与地址空间规划，并处理动态分配，降低大模型训练的分配浪费。
- **与 MoonBright 的关系：** 分配策略层相关；可用于理解上层生命周期信息如何降低底层分配压力。
- **实现层次 / 复现条件：** 框架分配器 / 软件规划；论文提供 PyTorch 集成。
- **核验边界：** 2025 年为预印本；正式会议为 EuroSys 2026。


# UVM、缺页、预取与超额订阅

<a id="suv"></a>

## SUV · MICRO 2024

**SUV: Static analysis guided Unified Virtual Memory**

[原文](https://guilhermecox.github.io/dw/pratheek-micro24.pdf)

- **核心机制：** 静态分析访问模式与密度，指导 CPU/GPU 放置、固定驻留和预取，减少低效 UVM 迁移。
- **与 MoonBright 的关系：** 相邻底层机制：决定页放在哪里及何时移动，MoonBright 侧重如何低成本建立映射。
- **实现层次 / 复现条件：** 编译分析与运行时软件；不要求新增 GPU 硬件。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="forest"></a>

## Forest · ISCA 2025

**Forest: Access-aware GPU UVM Management**

[原文](https://guilhermecox.github.io/dw/lin-isca25.pdf) · [DOI](https://doi.org/10.1145/3695053.3731047)

- **核心机制：** 按对象访问行为构建差异化预取树，配合访问跟踪及编译器模式信息，改进 UVM 预取与迁移。
- **与 MoonBright 的关系：** 相邻底层机制：把统一迁移策略改为访问感知策略。
- **实现层次 / 复现条件：** 软硬件协同设计；不能仅凭其运行时部分视为现有 GPU 上的纯软件替换。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="ariadne"></a>

## ARIADNE · HPCA 2026

**ARIADNE: Adaptive UVM Management for Efficient GPU Memory Oversubscription**

[原文](https://2026.hpca-conf.org/details/hpca-2026-main-conference/66/ARIADNE-Adaptive-UVM-Management-for-Efficient-GPU-Memory-Oversubscription)

- **核心机制：** 流水化缺页处理，用 Sharing Degree 衡量访问共享，动态选择 GPU 驻留与 zero-copy。
- **与 MoonBright 的关系：** 相邻底层机制：优化超额订阅时缺页、迁移和远程访问的取舍。
- **实现层次 / 复现条件：** 修改 NVIDIA 开源 UVM 驱动；论文说明无需修改编译器或 GPU 硬件。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="observuvm"></a>

## ObservUVM · ISCA 2026

**Observability-aided GPU Memory Oversubscription**

[原文](https://www.csa.iisc.ac.in/~arkapravab/papers/ISCA26_ObservUVM.pdf) · [DOI](https://doi.org/10.1109/ISCA66397.2026.00108) · [代码](https://github.com/csl-iisc/ObservUVM)

- **核心机制：** 利用已有远程访问计数器及采样页获取 HBM 区域热度，为驱逐和预取提供运行时可观测性。
- **与 MoonBright 的关系：** 相邻底层机制：解决迁移策略缺少访问反馈的问题，与映射执行成本可以分别研究。
- **实现层次 / 复现条件：** 驱动 / 用户态策略软件；复用已有硬件功能。
- **名称辨析：** ObservUVM；SuperUVM: Empowering UVM with Observability。
- **核验边界：** 按论文 PDF 与出版记录使用正式标题；作者实验室列表中的 SuperUVM 作为别名，不重复计数。

<a id="deepum"></a>

## DeepUM · ASPLOS 2023

**DeepUM: Tensor Migration and Prefetching in Unified Memory**

[原文](https://gsds.snu.ac.kr/research-post/deepum-tensor-migration-and-prefetching-in-unified-memory/) · [DOI](https://doi.org/10.1145/3575693.3575736)

- **核心机制：** 围绕 DNN 张量访问设计 UVM 页迁移与相关性预取，支持超过显存容量的训练负载。
- **与 MoonBright 的关系：** 相邻底层机制：利用训练规律改进页迁移，而不是加速一般分配器的 PTE 构造。
- **实现层次 / 复现条件：** UVM 软件与张量访问协同；具体驱动版本需查 artifact。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="helm"></a>

## HELM · SC 2025

**HELM: Characterizing Unified Memory Accesses to Improve GPU Performance under Memory Oversubscription**

[原文](https://sc25.conference-program.com/presentation/?id=pap340&sess=sess370) · [DOI](https://doi.org/10.1145/3712285.3759812)

- **核心机制：** 研究 Unified Memory 访问行为及其对 GPU 显存超额订阅性能的影响。
- **与 MoonBright 的关系：** 为 UVM 管理提供访问分析视角；本索引不根据标题推断具体驱逐或预取算法。
- **实现层次 / 复现条件：** 实现细节待阅读全文核验。
- **核验边界：** 已核验正式会议、标题、作者及摘要开头；机制细节与硬件要求尚未充分核验。


# 多 GPU 页管理与显存共享

<a id="huvm"></a>

## HUVM / memHarvester · USENIX ATC 2022

**Memory Harvesting in Multi-GPU Systems with Hierarchical Unified Virtual Memory**

[原文](https://www.usenix.org/conference/atc22/presentation/choi-sangjin) · [PDF](https://www.usenix.org/system/files/atc22-choi-sangjin_1.pdf)

- **核心机制：** 把其他 GPU 的空闲显存作为分层 UVM 的一层，优先利用较快的 GPU 间互连扩展可用容量。
- **与 MoonBright 的关系：** 扩展物理内存来源；关注多 GPU 内存层级，而非单卡映射开销。
- **实现层次 / 复现条件：** 多 GPU UVM 软件；需要适合的 GPU 互连与访问能力。
- **名称辨析：** HUVM；memHarvester。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="grit"></a>

## GRIT · HPCA 2024

**GRIT: Enhancing Multi-GPU Performance with Fine-Grained Dynamic Page Placement**

[原文](https://yueqiwang42.github.io/assets/pdf/papers/GRIT_HPCA24.pdf)

- **核心机制：** 以细粒度动态页放置改善多 GPU 的本地性和远程访问开销。
- **与 MoonBright 的关系：** 相邻问题：映射管理还需要决定页应属于哪个 GPU。
- **实现层次 / 复现条件：** 体系结构与页放置机制；不能据此认定为通用驱动补丁。
- **项目页：** [作者项目](https://xzt102.github.io/projects/grit-oasis.html)。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="oasis"></a>

## OASIS · HPCA 2025

**OASIS: Object-Aware Page Management for Multi-GPU Systems**

[原文](https://mtarekibnziad.onrender.com/files/preprint_HPCA2025_OASIS.pdf) · [DOI](https://doi.org/10.1109/HPCA61900.2025.00124)

- **核心机制：** 引入对象级信息指导多 GPU 页管理，减少仅按独立页面决策的局限。
- **与 MoonBright 的关系：** 相邻问题：应用对象语义与底层多 GPU 页放置之间的接口。
- **实现层次 / 复现条件：** 体系结构 / 内存管理协同；复现环境需按原论文确认。
- **项目页：** [作者项目](https://xzt102.github.io/projects/grit-oasis.html)。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="cdfd"></a>

## CDFD · ISCA 2026

**Coarse-Grained Duplication First, Fine-Grained Deduplication Later: Duplication-Centric Multi-GPU Memory Management**

[原文](https://ieeexplore.ieee.org/abstract/document/11617834) · [DOI](https://doi.org/10.1109/ISCA66397.2026.00109)

- **核心机制：** 先粗粒度复制以利用 GPU 互连带宽，再细粒度去重，权衡重复驻留容量与远程访问成本。
- **与 MoonBright 的关系：** 相邻问题：复制策略会改变物理页分配、映射及一致性压力。
- **实现层次 / 复现条件：** 多 GPU 复制与去重管理；具体平台和实现边界需读原文。
- **核验边界：** 基于出版摘要核验机制；未对实现与 artifact 作深入核验。


# 页表遍历、TLB 与地址翻译

<a id="mcm-vm"></a>

## MCM GPU Virtual Memory · MICRO 2022

**Designing Virtual Memory System of MCM GPUs**

[原文](https://www.csa.iisc.ac.in/~arkapravab/papers/MICRO22_MGvm.pdf)

- **核心机制：** 研究多芯粒 GPU 的虚拟内存组织及地址翻译开销。
- **与 MoonBright 的关系：** 研究映射建立之后的翻译路径；为 MoonBright 的 TLB 设计背景提供上下文。
- **实现层次 / 复现条件：** GPU 体系结构方案。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="snakebyte"></a>

## SnakeByte · HPCA 2023

**SnakeByte: A TLB Design with Adaptive and Recursive Page Merging in GPUs**

[原文](https://yonsei.elsevierpure.com/en/publications/snakebyte-a-tlb-design-with-adaptive-and-recursive-page-merging-i-2/)

- **核心机制：** 利用页面连续性进行自适应递归合并，扩大 TLB 的有效覆盖。
- **与 MoonBright 的关系：** 同属页表 / TLB 研究；优化地址翻译覆盖率，与映射更新和失效成本不同。
- **实现层次 / 复现条件：** TLB 硬件设计，结合连续性友好的分配。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="barre-chord"></a>

## Barre Chord · ISCA 2024

**Barre Chord: Efficient Virtual Memory Translation for Multi-Chip-Module GPUs**

[原文](https://seonjinna.github.io/assets/pdf/BarreChord_isca24.pdf)

- **核心机制：** 面向 MCM GPU 优化翻译组织与合并，缓解共享翻译服务的瓶颈。
- **与 MoonBright 的关系：** 补充多芯粒拓扑下的页表和翻译路径问题。
- **实现层次 / 复现条件：** GPU 地址翻译体系结构设计。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="avatar"></a>

## Avatar · MICRO 2024

**A Case for Speculative Address Translation with Rapid Validation for GPUs**

[原文](https://zoon17.github.io/pdfs/avatar_micro24.pdf) · [DOI](https://doi.org/10.1109/MICRO61859.2024.00029)

- **核心机制：** 先推测地址翻译结果，再快速验证，减少等待页表遍历造成的停顿。
- **与 MoonBright 的关系：** 与 MoonBright 都讨论翻译相关开销，但这里优化内存访问阶段的翻译延迟。
- **实现层次 / 复现条件：** 需要推测翻译与验证的硬件支持。
- **名称辨析：** Avatar；CAST；CAVA。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="star"></a>

## STAR · MICRO 2024

**STAR: Sub-Entry Sharing-Aware TLB for Multi-Instance GPU**

[原文](https://yueqiwang42.github.io/assets/pdf/papers/STAR_MICRO24.pdf) · [DOI](https://doi.org/10.1109/MICRO61859.2024.00031)

- **核心机制：** 允许不同地址区域共享 TLB 子项，提高多实例 GPU 场景下的翻译缓存利用率。
- **与 MoonBright 的关系：** TLB 容量与隔离环境相关；不等同于延迟 TLB 一致性维护。
- **实现层次 / 复现条件：** TLB 硬件设计。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="marching-page-walks"></a>

## Marching Page Walks · HPCA 2025

**Marching Page Walks: Batching and Concurrent Page Table Walks for Enhancing GPU Throughput**

[原文](https://pure.korea.ac.kr/en/publications/marching-page-walks-batching-and-concurrent-page-table-walks-for-/)

- **核心机制：** 批量化和并发执行页表遍历，缓解大量地址翻译请求的排队。
- **与 MoonBright 的关系：** 同为降低页表相关瓶颈；处理 PTE 查询，而不是 PTE 安装。
- **实现层次 / 复现条件：** 页表遍历器及相关硬件机制。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="softwalker"></a>

## SoftWalker · MICRO 2025

**SoftWalker: Supporting Software Page Table Walk for Irregular GPU Applications**

[原文](https://skku-compasslab.github.io/compasslab/publications/250715-micro-softwalker/) · [DOI](https://doi.org/10.1145/3725843.3756056)

- **核心机制：** 使用 GPU 执行资源承担软件页表遍历，让翻译处理能力随负载扩展。
- **与 MoonBright 的关系：** 与 MoonBright 的设备端执行思路最值得对照：SoftWalker 查页表，MoonBright 构造页表项。
- **实现层次 / 复现条件：** 体系结构协同方案；不能把 software page walk 理解为无需硬件支持的普通 CUDA 库。
- **会议核验：** [会议程序或论文列表](https://microarch.org/micro58/program/index.php)。
- **核验边界：** 会议归属由 MICRO 官方程序核验，机制由作者实验室页面核验；作者页直接打开可能返回 404。

<a id="latpc"></a>

## LATPC · MICRO 2025

**LATPC: Accelerating GPU Address Translation Using Locality-Aware TLB Prefetching and MSHR Compression**

[原文](https://pure.skku.edu/en/publications/latpc-accelerating-gpu-address-translation-using-locality-aware-t/)

- **核心机制：** 结合局部性感知 TLB 预取和 MSHR 压缩，降低翻译请求带来的停顿与资源压力。
- **与 MoonBright 的关系：** 相邻翻译路径优化；关注访存并发而非分配调用。
- **实现层次 / 复现条件：** GPU 地址翻译硬件机制。
- **会议核验：** [会议程序或论文列表](https://microarch.org/micro58/program/index.php)。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="chiplet-locality"></a>

## Chiplet-local Memory Mapping · MICRO 2025

**Leveraging Chiplet-Locality for Efficient Memory Mapping in Multi-Chip Module GPUs**

[原文](https://pure.skku.edu/en/publications/leveraging-chiplet-locality-for-efficient-memory-mapping-in-multi/) · [DOI](https://doi.org/10.1145/3725843.3756090)

- **核心机制：** 利用芯粒局部性设计内存映射，权衡跨芯粒访问与地址翻译成本。
- **与 MoonBright 的关系：** 相邻映射问题：映射策略需要同时考虑拓扑和页面粒度。
- **实现层次 / 复现条件：** MCM GPU 体系结构 / 映射策略。
- **会议核验：** [会议程序或论文列表](https://microarch.org/micro58/program/index.php)。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。


# LLM KV cache、服务内存与交接

<a id="pagedattention"></a>

## PagedAttention / vLLM · SOSP 2023

**Efficient Memory Management for Large Language Model Serving with PagedAttention**

[原文](https://arxiv.org/abs/2309.06180) · [DOI](https://doi.org/10.1145/3600006.3613165) · [代码](https://github.com/vllm-project/vllm)

- **核心机制：** 把 KV cache 划分为逻辑块并通过块表访问，支持非连续分配及共享，减少浪费。
- **与 MoonBright 的关系：** 关键对照：应用块表由 attention kernel 解释；vAttention/MoonBright 利用或管理硬件虚拟内存。
- **实现层次 / 复现条件：** LLM 运行时与定制 attention kernel。
- **名称辨析：** vLLM。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="jenga"></a>

## Jenga · SOSP 2025

**Jenga: Effective Memory Management for Serving LLM with Heterogeneity**

[原文](https://pacman.cs.tsinghua.edu.cn/~zjd/publication/generated/dblp-confsosp-zhang-dlkmwlyllz-25/) · [代码](https://github.com/heheda12345/Jenga-SOSP25-AE)

- **核心机制：** 针对异构模型和 KV 形态设计分层内存管理，减少统一块大小带来的浪费。
- **与 MoonBright 的关系：** 应用分配层相关；可研究异构 KV 分配对底层映射吞吐的需求。
- **实现层次 / 复现条件：** LLM 运行时软件。
- **预印本：** [作者版本](https://arxiv.org/abs/2503.18292)，以正式会议年份归档。
- **会议核验：** [会议程序或论文列表](https://sigops.org/s/conferences/sosp/2025/accepted.html)。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="diffkv"></a>

## DiffKV · SOSP 2025

**DiffKV: Differentiated Memory Management for Large Language Models with Parallel KV Compaction**

[原文](https://ipads.se.sjtu.edu.cn/zh/publications/sosp25-zhang.pdf)

- **核心机制：** 差异化管理 KV cache，并行执行 KV 紧凑整理，降低异构精度或尺寸带来的内存浪费。
- **与 MoonBright 的关系：** 应用层碎片与整理相关；不直接替代通用 GPU 页表管理。
- **实现层次 / 复现条件：** LLM KV 管理与 GPU kernel。
- **会议核验：** [会议程序或论文列表](https://sigops.org/s/conferences/sosp/2025/accepted.html)。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="prism"></a>

## Prism · OSDI 2026

**Prism: Cost-Efficient Multi-LLM Serving via GPU Memory Ballooning**

[原文](https://www.usenix.org/conference/osdi26/presentation/yu-shan) · [代码](https://github.com/ovg-project/kvcached)

- **核心机制：** 通过 GPU memory ballooning 在多个模型服务之间弹性调整显存占用。
- **与 MoonBright 的关系：** 直接相关的应用场景：快速回收与再分配使底层映射开销更重要。
- **实现层次 / 复现条件：** 多模型服务运行时；实现依赖对应 VMM 支持。
- **预印本：** [作者版本](https://arxiv.org/abs/2505.04021)，以正式会议年份归档。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="sirius"></a>

## SIRIUS · USENIX ATC 2025

**Colocating ML Inference and Training with Fast GPU Memory Handover**

[原文](https://www.usenix.org/conference/atc25/presentation/wang-jiali) · [PDF](https://ipads.se.sjtu.edu.cn/_media/publications/sirius-atc25.pdf)

- **核心机制：** 利用训练语义快速腾出显存，配合内存交接策略支持训练与推理混部。
- **与 MoonBright 的关系：** 直接相关的应用场景：显存所有权快速转移与回收。
- **实现层次 / 复现条件：** 训练 / 推理运行时协同软件。
- **名称辨析：** SIRIUS。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="s-lora"></a>

## S-LoRA · MLSys 2024

**S-LoRA: Serving Thousands of Concurrent LoRA Adapters**

[原文](https://proceedings.mlsys.org/paper_files/paper/2024/file/906419cd502575b617cc489a1a696a67-Paper-Conference.pdf) · [代码](https://github.com/S-LoRA/S-LoRA)

- **核心机制：** 统一内存池管理不同 rank 的 LoRA 权重和不同序列长度的 KV cache，配合异构批处理 kernel。
- **与 MoonBright 的关系：** 应用层统一分页；此处 Unified Paging 不是 CUDA Unified Memory。
- **实现层次 / 复现条件：** 服务运行时与自定义 CUDA kernel。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="pensieve"></a>

## Pensieve · EuroSys 2025

**Stateful Large Language Model Serving with Pensieve**

[原文](https://arxiv.org/abs/2312.05516) · [DOI](https://doi.org/10.1145/3689031.3696086)

- **核心机制：** 保存和复用多轮对话状态，在 GPU 与 CPU 之间分层管理 KV cache。
- **与 MoonBright 的关系：** 容量与复用策略相关；管理对象是会话状态。
- **实现层次 / 复现条件：** LLM 运行时软件。
- **核验边界：** 正式发表为 EuroSys 2025；2023 是预印本年份，不应记为 ASPLOS。

<a id="llumnix"></a>

## Llumnix · OSDI 2024

**Llumnix: Dynamic Scheduling for Large Language Model Serving**

[原文](https://www.usenix.org/conference/osdi24/presentation/sun-biao) · [代码](https://github.com/AlibabaPAI/llumnix)

- **核心机制：** 在服务实例之间迁移请求及其 KV 状态，实现动态负载均衡。
- **与 MoonBright 的关系：** 延伸阅读：分布式 KV 状态迁移，不是底层分配器。
- **实现层次 / 复现条件：** 多实例 LLM 服务运行时。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="mooncake"></a>

## Mooncake · FAST 2025

**Mooncake: Trading More Storage for Less Computation — A KVCache-centric Architecture for Serving LLM Chatbot**

[原文](https://www.usenix.org/conference/fast25/presentation/qin)

- **核心机制：** 把分布式 KV cache 作为服务架构核心，利用跨设备存储资源复用计算结果。
- **与 MoonBright 的关系：** 延伸阅读：KV 存储与传输层级；名称相似不表示与 MoonBright 是同类分配器。
- **实现层次 / 复现条件：** 分布式服务与存储软件。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="infinigen"></a>

## InfiniGen · OSDI 2024

**InfiniGen: Efficient Generative Inference of Large Language Models with Dynamic KV Cache Management**

[原文](https://www.usenix.org/conference/osdi24/presentation/lee)

- **核心机制：** 预测后续 attention 所需的重要 KV 子集，动态取回缓存，减少 GPU 常驻容量。
- **与 MoonBright 的关系：** 应用层选择性 KV 管理；应单独评估模型精度，不属于完全透明的分配器。
- **实现层次 / 复现条件：** LLM 运行时和模型访问策略。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="flexgen"></a>

## FlexGen · ICML 2023

**FlexGen: High-Throughput Generative Inference of Large Language Models with a Single GPU**

[原文](https://proceedings.mlr.press/v202/sheng23a.html) · [代码](https://github.com/FMInference/FlexGen)

- **核心机制：** 联合规划 GPU、CPU 与存储之间的权重、激活和 KV 数据流，提高有限显存下的批量推理吞吐。
- **与 MoonBright 的关系：** 容量扩展与卸载调度相关；目标偏吞吐，不宜直接当作低延迟 allocator 对照。
- **实现层次 / 复现条件：** 推理运行时软件。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="aqua"></a>

## AQUA · ASPLOS 2025

**AQUA: Network-Accelerated Memory Offloading for LLMs in Scale-Up GPU Domains**

[原文](https://arxiv.org/abs/2407.21255) · [DOI](https://doi.org/10.1145/3676641.3715983)

- **核心机制：** 降低抢占式 LLM 服务中推理状态换出的成本，使突发请求下的时间片调度更可行。
- **与 MoonBright 的关系：** 显存分页与服务调度相关；着重 scale-up GPU 域内的数据搬运。
- **实现层次 / 复现条件：** 多 GPU 服务软件；依赖 scale-up 互连条件。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。


# 训练卸载与 GPU–CPU–存储层级

<a id="g10"></a>

## G10 · MICRO 2023

**G10: Enabling An Efficient Unified GPU Memory and Storage Architecture with Smart Tensor Migrations**

[原文](https://jianh.web.engr.illinois.edu/papers/g10-micro23.pdf) · [DOI](https://doi.org/10.1145/3613424.3614309)

- **核心机制：** 利用张量使用规律，在 GPU、CPU 与 SSD 层级之间规划数据迁移。
- **与 MoonBright 的关系：** 相邻容量扩展方案；把页管理与张量语义、存储层级结合。
- **实现层次 / 复现条件：** 统一内存与存储的体系结构协同设计。
- **预印本：** [作者版本](https://arxiv.org/abs/2310.09443)，以正式会议年份归档。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="bam"></a>

## BaM · ASPLOS 2023

**GPU-Initiated On-Demand High-Throughput Storage Access in the BaM System Architecture**

[原文](https://research.ibm.com/publications/gpu-initiated-on-demand-high-throughput-storage-access-in-the-bam-system-architecture)

- **核心机制：** GPU 线程按需发起存储访问，并用软件缓存管理数据，减少 CPU 在 I/O 关键路径上的参与。
- **与 MoonBright 的关系：** 设备端管理思路相关；对象是存储 I/O 和缓存，而非 GPU PTE 安装。
- **实现层次 / 复现条件：** GPU 软件缓存 / NVMe 访问；依赖硬件连接与系统配置。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="zero-infinity"></a>

## ZeRO-Infinity · SC 2021

**ZeRO-Infinity: Breaking the GPU Memory Wall for Extreme Scale Deep Learning**

[原文](https://arxiv.org/abs/2104.07857) · [DOI](https://doi.org/10.1145/3458817.3476205) · [代码](https://github.com/deepspeedai/DeepSpeed)

- **核心机制：** 结合训练状态分片与 CPU/NVMe 卸载，扩展可训练模型规模。
- **与 MoonBright 的关系：** 应用层容量扩展基线；与分配器映射成本属于不同层次。
- **实现层次 / 复现条件：** DeepSpeed 训练运行时软件。
- **核验边界：** SC 2021 在 2021 年 11 月举行，落在本次滚动五年窗口内；不按预印本发布日期排除。

<a id="tsplit"></a>

## TSPLIT · ICDE 2022

**TSPLIT: Fine-grained GPU Memory Management for Efficient DNN Training via Tensor Splitting**

[原文](https://ieeexplore.ieee.org/document/9835178)

- **核心机制：** 以张量拆分提供更细粒度的内存管理，改善训练中的容量与换入换出选择。
- **与 MoonBright 的关系：** 应用张量粒度与底层页面粒度的联系。
- **实现层次 / 复现条件：** DNN 训练运行时软件。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="harmony"></a>

## Harmony · PVLDB / VLDB 2022

**Harmony: Overcoming the Hurdles of GPU Memory Capacity to Train Massive DNN Models on Commodity Servers**

[原文](https://arxiv.org/abs/2202.01306) · [DOI](https://doi.org/10.14778/3551793.3551828) · [代码](https://github.com/msr-fiddle/harmony)

- **核心机制：** 重排训练计算和数据移动，减少容量不足的多 GPU 服务器上的反复换入换出。
- **与 MoonBright 的关系：** 应用调度减少迁移需求，与加速单次映射操作互补。
- **实现层次 / 复现条件：** 训练框架软件。
- **核验边界：** 按 PVLDB / VLDB 2022 归类，不是 OSDI。


# 内存行为测量与诊断

<a id="drgpum"></a>

## DrGPUM · ASPLOS 2023

**DrGPUM: Guiding Memory Optimization for GPU-Accelerated Applications**

[原文](https://www.jokeren.tech/publication/lin-2023-drgpum/) · [DOI](https://doi.org/10.1145/3582016.3582044)

- **核心机制：** 将 GPU 内存低效行为与应用对象和调用关联，为优化提供定位依据。
- **与 MoonBright 的关系：** 可用于识别 allocator、对象布局及数据移动问题；属于分析工具。
- **实现层次 / 复现条件：** 性能分析工具。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="uvm-analysis-sc21"></a>

## UVM In-depth Analysis · SC 2021

**In-Depth Analyses of Unified Virtual Memory System for GPU Accelerated Computing**

[原文](https://sc21.supercomputing.org/proceedings/tech_paper/tech_paper_pages/pap249.html) · [PDF](https://tallendev.github.io/assets/papers/sc21.pdf) · [DOI](https://doi.org/10.1145/3458817.3480855) · [代码](https://github.com/tallendev/uvm-eval)

- **核心机制：** 分解实际 UVM 中缺页、迁移及软硬件交互的性能开销。
- **与 MoonBright 的关系：** 建立 UVM 问题背景和测量方法，避免把所有开销都归到 GPU 算力。
- **实现层次 / 复现条件：** 真实 GPU / 驱动性能研究。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。


# 压缩及其他应用容量管理

<a id="ibp"></a>

## IBP · EuroSys 2026

**Reducing the GPU Memory Bottleneck with Lossless Compression for ML**

[原文](https://akkamath.github.io/publication/EuroSys26-IBP) · [PDF](https://syfi.cs.washington.edu/assets/papers/ibp.pdf) · [DOI](https://doi.org/10.1145/3767295.3803595) · [代码](https://github.com/AKKamath/InvariantBitPacking)

- **核心机制：** 用 Invariant Bit Packing 无损压缩减少张量传输量，配合 GPU 解压和异步 PCIe 传输。
- **与 MoonBright 的关系：** 容量与传输延伸阅读；不改变 GPU 页表管理语义。
- **实现层次 / 复现条件：** 压缩 / 解压库与 ML 运行时集成。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

<a id="clm"></a>

## CLM · ASPLOS 2026

**CLM: Removing the GPU Memory Barrier for 3D Gaussian Splatting**

[原文](https://tarzanzhao.github.io/CLM-GS/) · [PDF](https://cs.nyu.edu/~apanda/assets/papers/asplos26-cvm.pdf) · [代码](https://github.com/nyu-systems/CLM-GS)

- **核心机制：** 针对 3D Gaussian Splatting 将模型及优化器数据分层放置，选择性加载并流水化数据移动。
- **与 MoonBright 的关系：** 特定应用的显存容量管理；用于理解 GPU 内存问题在非 LLM 负载中的表现。
- **实现层次 / 复现条件：** 应用运行时软件，任务特定。
- **预印本：** [作者版本](https://arxiv.org/abs/2511.04951)，以正式会议年份归档。
- **核验边界：** 依据论文、会议页面或作者页面核验；未运行实验。

