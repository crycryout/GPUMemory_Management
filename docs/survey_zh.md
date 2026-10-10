# 从 MoonBright 出发：GPU 内存管理研究地图

[返回首页](../README.md) · [逐篇笔记](papers_zh.md) · [检索方法](search_method.md)

## 1. 先把“与 MoonBright 类似”分清楚

MoonBright 的中心问题是：**细粒度动态分配时，CPU 主导的页表构造和翻译状态维护成为开销来源，能否利用 GPU 并行性降低这些成本？**

论文把批量页表项构造移到设备端，但保留主机侧校验与分配元数据；新映射使用新虚拟地址，结合延迟一致性处理减少常见路径上的 TLB 维护。它涉及通用分配与映射控制路径，不能简单概括为“GPU 自己完成所有内存管理”。原文与实现见 [MoonBright 官方页面](https://www.usenix.org/conference/osdi26/presentation/zhang-yangyu)及[论文 PDF](https://www.usenix.org/system/files/osdi26-zhang-yangyu.pdf)。

据此，本综述将相关工作分成三圈：

| 关联层次 | 核心问题 | 代表工作 |
|---|---|---|
| 直接分配器 / VMM | 地址空间如何分配？碎片如何消除？设备线程如何 malloc/free？ | GMLake、vAttention、Gallatin、STAlloc |
| 底层相邻机制 | 页放哪里？何时迁移？如何查页表？TLB 如何组织？ | SUV、Forest、ARIADNE、ObservUVM、HUVM、SoftWalker、SnakeByte |
| 应用与容量管理 | KV / 张量怎么存、复用、回收和卸载？ | PagedAttention、Jenga、Prism、SIRIUS、G10、ZeRO-Infinity |

这些工作并非全部互为可替换基线。应用层方案可能使用底层分配器；页迁移策略也可能受映射成本影响，但二者组合是否有效需要实验。

## 2. 最值得优先比较的机制

下表中的“可组合”仅是研究判断，不表示已有兼容性证明。

| 工作 | 主要控制对象 | 主要手段 | 与 MoonBright 的关键差异 |
|---|---|---|---|
| [GMLake · ASPLOS 2024](papers_zh.md#gmlake) | 物理分配块和虚拟区间 | 虚拟地址拼接，缓解外部碎片 | 优化布局和复用；映射操作本身仍有成本 |
| [vAttention · ASPLOS 2025](papers_zh.md#vattention) | KV cache 的虚拟 / 物理页 | 连续虚拟地址、动态物理供给 | 服务于 LLM KV；是分析底层映射成本的直接场景 |
| [Gallatin · PPoPP 2024](papers_zh.md#gallatin) | GPU 设备堆 | 高并发动态分配 | 管理设备堆空闲空间，不能等同于安装硬件 PTE |
| [STAlloc · EuroSys 2026](papers_zh.md#stalloc) | 训练张量及其生命周期 | 时空规划与动态分配结合 | 利用应用信息减少浪费，处于更上层 |
| [SoftWalker · MICRO 2025](papers_zh.md#softwalker) | 地址翻译请求 | GPU 执行资源承担软件 page walk | 查询已有 PTE；MoonBright 的核心操作是构造 PTE |
| [SUV · MICRO 2024](papers_zh.md#suv) | CPU / GPU 数据放置 | 静态访问分析指导 UVM | 优化数据位置和迁移需求 |
| [Forest · ISCA 2025](papers_zh.md#forest) | 预取树与对象访问模式 | 访问感知的 UVM 管理 | 包含硬件支持，不能当成现有驱动的直接替换 |
| [ARIADNE · HPCA 2026](papers_zh.md#ariadne) | 超额订阅缺页与驻留 | 流水化处理、迁移 / zero-copy 自适应 | 优化缺页服务与远程访问取舍 |
| [ObservUVM · ISCA 2026](papers_zh.md#observuvm) | 页访问热度与驱逐 | 复用访问计数器获取观测 | 解决策略输入不足，侧重决策质量 |
| [HUVM · ATC 2022](papers_zh.md#huvm) | 多 GPU 空闲显存 | 增加对端 GPU 内存层级 | 解决容量来源与跨设备放置 |
| [Prism · OSDI 2026](papers_zh.md#prism) | 多模型之间的显存额度 | 内存 ballooning | 应用场景强调快速回收和重新供给 |
| [SIRIUS · ATC 2025](papers_zh.md#sirius) | 训练 / 推理显存交接 | 利用训练语义快速腾出内存 | 应用语义决定哪些内存可回收 |

**补充区仍有值得先读的论文。** [DREAM · ICS 2025](supplement_zh.md#dream) 将内存服务更多地交给 GPU 执行，和 MoonBright 的设备端管理动机接近；但其软件分页、RDMA 路径和 CPU 内存访问方式需要单独分析。它未列入本次主目录会议集合，相关 GPUVM 预印本也没有被另算一篇顶会论文。

## 3. 三条阅读路线

### A. 想研究 allocator、VMM 或 driver

按 [MoonBright](papers_zh.md#moonbright) → [GMLake](papers_zh.md#gmlake) → [vAttention](papers_zh.md#vattention) → [Gallatin](papers_zh.md#gallatin) 阅读，再对照 [SoftWalker](papers_zh.md#softwalker) 与 [DREAM](supplement_zh.md#dream)。

阅读时把操作拆开记录：虚拟地址预留、物理内存取得、PTE 安装、权限校验、翻译状态发布、回收与地址复用。这样才能判断某篇论文减少的是哪一段成本。

### B. 想研究 UVM 或显存超额订阅

按 [SC 2021 UVM 分析](papers_zh.md#uvm-analysis-sc21) → [DeepUM](papers_zh.md#deepum) → [SUV](papers_zh.md#suv) → [Forest](papers_zh.md#forest) → [ARIADNE](papers_zh.md#ariadne) → [ObservUVM](papers_zh.md#observuvm) 阅读。

逐篇比较：决策信息来自哪里、迁移粒度、预取时机、驱逐策略、何时选择远程访问，以及策略在超额订阅程度增加时如何退化。再看 [HUVM](papers_zh.md#huvm)、[GRIT](papers_zh.md#grit)、[OASIS](papers_zh.md#oasis) 与 [CDFD](papers_zh.md#cdfd)，将问题扩展到多 GPU。

### C. 想把内存机制用于 LLM 服务

先对读 [PagedAttention](papers_zh.md#pagedattention) 和 [vAttention](papers_zh.md#vattention)，明确软件逻辑块与硬件虚拟页的区别；再读 [Jenga](papers_zh.md#jenga)、[DiffKV](papers_zh.md#diffkv) 和 [S-LoRA](papers_zh.md#s-lora) 的异构内存需求，最后看 [Prism](papers_zh.md#prism) 与 [SIRIUS](papers_zh.md#sirius) 的动态资源回收。

[Pensieve](papers_zh.md#pensieve)、[Llumnix](papers_zh.md#llumnix)、[Mooncake](papers_zh.md#mooncake) 用于进一步理解跨请求、跨实例和跨存储层级的 KV 管理。它们与底层分配器的联系需要通过实际分配 / 迁移 trace 建立。

## 4. 实验前先区分改动层次

| 层次 | 例子 | 复现时首先确认 |
|---|---|---|
| 框架 / 分配器软件 | GMLake、Gallatin、STAlloc | GPU 架构、框架版本、VMM 支持、kernel 集成 |
| 驱动 / 运行时修改 | MoonBright、ARIADNE、ObservUVM | 驱动分支、内核版本、权限、GPU 功能、同步与隔离假设 |
| 软硬件协同或体系结构 | Forest、SoftWalker、SnakeByte、STAR、LATPC | 模拟器 / 硬件模型、修改范围、能否在现有硬件上验证部分机制 |
| 特定互连 / 存储条件 | HUVM、AQUA、BaM、DREAM | GPU 拓扑、P2P / RDMA / NVMe 路径及带宽 |
| 应用内存语义 | PagedAttention、InfiniGen、SIRIUS、CLM | 是否改计算 kernel、模型行为、精度要求或训练调度 |

“无需新 GPU 硬件”不代表无需改驱动，也不代表跨厂商或跨版本即插即用。本次没有运行任何 artifact。HELM、CDFD 等条目的核验深度已在[逐篇笔记](papers_zh.md)单独说明。

## 5. 从已有工作推导的研究问题

以下是本综述提出的**待验证问题**，不是对领域空白或论文性能的断言。

| 研究问题 | 为什么值得测 | 建议记录的量 |
|---|---|---|
| 更快的映射能否支持更细的动态 KV 分配？ | GMLake / vAttention 类方案的空间收益可能被小粒度映射成本抵消 | 分配与映射的 p50 / p99、PTE 吞吐、外部碎片、TTFT / TPOT |
| 新虚拟地址策略长期运行如何回收？ | 常见路径优化仍需处理地址复用和旧访问结束 | VA 消耗速度、回收时暂停、同步成本、长时间压力下的正确性 |
| 更便宜的映射会不会改变 UVM 的最优迁移粒度？ | 缺页、数据拷贝和映射维护的成本比例可能变化 | 每次缺页分解、迁移字节、无效预取、驱逐 / 回迁次数 |
| 如何兼顾动态回收与多租户隔离？ | 多模型 ballooning 和训练 / 推理交接需要安全地转移所有权 | 回收延迟、未完成 GPU 访问、权限撤销、错误路径行为 |
| 多 GPU 放置策略是否受映射更新限制？ | HUVM / GRIT / OASIS 的放置收益受互连和控制路径共同影响 | 本地 / 远程比例、互连流量、复制占用、映射与迁移开销 |

性能比较建议保留三个层次：分配 / 映射微基准，包含碎片与生命周期变化的 trace 重放，最后是相同模型、负载和 SLO 条件下的端到端实验。不同论文的“最高加速比”往往对应不同硬件、基线和任务，不宜拼成排行榜。

## 6. 几个容易混淆的概念

- **分配 ≠ 数据迁移。** 建立或改变 VA→PA 映射与把字节拷到另一个内存层级是不同操作。
- **PTE 构造 ≠ page-table walk。** 前者写入映射，后者为了访存查询映射。
- **PagedAttention 的块表 ≠ GPU 硬件页表。** 前者由应用 kernel 使用；后者参与硬件地址翻译。
- **UVM ≠ 统一物理内存。** 离散 GPU 迁移式 UVM、Grace Hopper 平台及 MI300A 的行为需要分别讨论。
- **TLB 容量优化 ≠ TLB 一致性优化。** 扩大覆盖率与保证更新后不使用旧映射属于不同目标。
- **预印本年份 ≠ 正式会议年份。** 本仓库用正式会议记录统一归档；待确认论文只放补充区。
