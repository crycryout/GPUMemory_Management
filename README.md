# GPU Memory Management：从 MoonBright 出发的近五年论文索引

检索窗口：**2021-10-10 至 2026-10-10**。最近核验：2026-10-10。

主目录收录 **45 篇论文（含 MoonBright 本身，其他相关论文 44 篇）**，另列 **9 条相关会议 / 预印本记录**。范围以系统与体系结构顶会为核心，并纳入与 GPU 内存直接相关的 MLSys、ICML、ICDE 和 PVLDB / VLDB 工作。具体会议口径见[检索方法](docs/search_method.md)，不将所有“GPU memory”论文都视为与 MoonBright 同一层次。

其中 **24 篇**属于分配器、UVM、多 GPU 页管理和地址翻译，**21 篇**属于应用层、容量扩展或测量研究。

**先读这条主线：MoonBright → GMLake → vAttention → Gallatin → SoftWalker。** 前四篇分别帮助理解映射控制路径、碎片、连续虚拟地址和设备端分配；SoftWalker 用于对照 GPU 上构造页表与查询页表的区别。

| 文档 | 内容 |
|---|---|
| [中文综述与阅读路线](docs/survey_zh.md) | MoonBright 定位、机制对照、复现条件、可研究问题 |
| [45 篇逐篇笔记](docs/papers_zh.md) | 正式标题、会议年份、机制、与 MoonBright 的关系、原文及代码入口 |
| [补充文献](docs/supplement_zh.md) | DREAM / GPUVM、AMD SVM、Grace Hopper、vTensor 等 |
| [检索方法与边界](docs/search_method.md) | 日期口径、会议清单、去重、核验限制 |
| [机器可读索引](data/papers.json) | 54 条结构化记录，可按会议、类别、年份及关联层次筛选 |

这里保存的是**中文整理、书目信息与原文 / PDF / 代码链接**；未镜像论文 PDF 或第三方代码。未做 artifact 复现，也不把跨论文报告的加速比直接排序。

## 按研究层次浏览

| 方向 | 篇数 |
|---|---:|
| [分配器、虚拟地址与碎片](#allocator) | 5 |
| [UVM、缺页、预取与超额订阅](#uvm) | 6 |
| [多 GPU 页管理与显存共享](#multi_gpu) | 4 |
| [页表遍历、TLB 与地址翻译](#translation) | 9 |
| [LLM KV cache、服务内存与交接](#serving) | 12 |
| [训练卸载与 GPU–CPU–存储层级](#training_storage) | 5 |
| [内存行为测量与诊断](#measurement) | 2 |
| [压缩及其他应用容量管理](#capacity) | 2 |

<a id="allocator"></a>

## 分配器、虚拟地址与碎片

与 MoonBright 的 allocator / VMM 问题最直接相关。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [MoonBright](docs/papers_zh.md#moonbright) | OSDI 2026 | CPU 保留校验及分配器元数据管理，GPU 并行构造页表项；结合新虚拟地址和延迟 TLB 一致性处理降低映射开销。 | [原文](https://www.usenix.org/conference/osdi26/presentation/zhang-yangyu) · [PDF](https://www.usenix.org/system/files/osdi26-zhang-yangyu.pdf) · [代码](https://github.com/MoonBright-project/OSDI-26-AE) |
| [GMLake](docs/papers_zh.md#gmlake) | ASPLOS 2024 | 通过虚拟地址拼接，把分散物理显存组织成连续虚拟区间，缓解训练分配器的外部碎片。 | [原文](https://www.cs.sjtu.edu.cn/~leng-jw/pubs/guo2024asplos-gmlake.html) · [DOI](https://doi.org/10.1145/3620665.3640423) · [代码](https://github.com/antgroup/glake) |
| [vAttention](docs/papers_zh.md#vattention) | ASPLOS 2025 | 为 KV cache 保留连续虚拟地址，按需提供物理页，使普通 attention kernel 可以使用动态分配的 KV 内存。 | [原文](https://apanwariisc.github.io/publications/asplos-2025-vattention/vattention-asplos25.pdf) · [DOI](https://doi.org/10.1145/3669940.3707256) · [代码](https://github.com/microsoft/vattention) |
| [Gallatin](docs/papers_zh.md#gallatin) | PPoPP 2024 | 面向 GPU 线程并发 malloc/free，采用适合并行查找的分配结构，支持不同大小的动态分配。 | [原文](https://prashantpandey.github.io/assets/pdf/uploads/ppopp24-final274.pdf) · [DOI](https://doi.org/10.1145/3627535.3638499) · [代码](https://github.com/saltsystemslab/gallatin) |
| [STAlloc](docs/papers_zh.md#stalloc) | EuroSys 2026 | 结合张量生命周期与地址空间规划，并处理动态分配，降低大模型训练的分配浪费。 | [原文](https://arxiv.org/abs/2507.16274) · [DOI](https://doi.org/10.1145/3767295.3769335) · [代码](https://github.com/infinigence/STAlloc) |

<a id="uvm"></a>

## UVM、缺页、预取与超额订阅

研究页放置、迁移和容量不足时的处理。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [SUV](docs/papers_zh.md#suv) | MICRO 2024 | 静态分析访问模式与密度，指导 CPU/GPU 放置、固定驻留和预取，减少低效 UVM 迁移。 | [原文](https://guilhermecox.github.io/dw/pratheek-micro24.pdf) |
| [Forest](docs/papers_zh.md#forest) | ISCA 2025 | 按对象访问行为构建差异化预取树，配合访问跟踪及编译器模式信息，改进 UVM 预取与迁移。 | [原文](https://guilhermecox.github.io/dw/lin-isca25.pdf) · [DOI](https://doi.org/10.1145/3695053.3731047) |
| [ARIADNE](docs/papers_zh.md#ariadne) | HPCA 2026 | 流水化缺页处理，用 Sharing Degree 衡量访问共享，动态选择 GPU 驻留与 zero-copy。 | [原文](https://2026.hpca-conf.org/details/hpca-2026-main-conference/66/ARIADNE-Adaptive-UVM-Management-for-Efficient-GPU-Memory-Oversubscription) |
| [ObservUVM](docs/papers_zh.md#observuvm) | ISCA 2026 | 利用已有远程访问计数器及采样页获取 HBM 区域热度，为驱逐和预取提供运行时可观测性。 | [原文](https://www.csa.iisc.ac.in/~arkapravab/papers/ISCA26_ObservUVM.pdf) · [DOI](https://doi.org/10.1109/ISCA66397.2026.00108) · [代码](https://github.com/csl-iisc/ObservUVM) |
| [DeepUM](docs/papers_zh.md#deepum) | ASPLOS 2023 | 围绕 DNN 张量访问设计 UVM 页迁移与相关性预取，支持超过显存容量的训练负载。 | [原文](https://gsds.snu.ac.kr/research-post/deepum-tensor-migration-and-prefetching-in-unified-memory/) · [DOI](https://doi.org/10.1145/3575693.3575736) |
| [HELM](docs/papers_zh.md#helm) | SC 2025 | 研究 Unified Memory 访问行为及其对 GPU 显存超额订阅性能的影响。 | [原文](https://sc25.conference-program.com/presentation/?id=pap340&sess=sess370) · [DOI](https://doi.org/10.1145/3712285.3759812) |

<a id="multi_gpu"></a>

## 多 GPU 页管理与显存共享

研究跨 GPU 物理内存利用、本地性和复制。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [HUVM / memHarvester](docs/papers_zh.md#huvm) | USENIX ATC 2022 | 把其他 GPU 的空闲显存作为分层 UVM 的一层，优先利用较快的 GPU 间互连扩展可用容量。 | [原文](https://www.usenix.org/conference/atc22/presentation/choi-sangjin) · [PDF](https://www.usenix.org/system/files/atc22-choi-sangjin_1.pdf) |
| [GRIT](docs/papers_zh.md#grit) | HPCA 2024 | 以细粒度动态页放置改善多 GPU 的本地性和远程访问开销。 | [原文](https://yueqiwang42.github.io/assets/pdf/papers/GRIT_HPCA24.pdf) |
| [OASIS](docs/papers_zh.md#oasis) | HPCA 2025 | 引入对象级信息指导多 GPU 页管理，减少仅按独立页面决策的局限。 | [原文](https://mtarekibnziad.onrender.com/files/preprint_HPCA2025_OASIS.pdf) · [DOI](https://doi.org/10.1109/HPCA61900.2025.00124) |
| [CDFD](docs/papers_zh.md#cdfd) | ISCA 2026 | 先粗粒度复制以利用 GPU 互连带宽，再细粒度去重，权衡重复驻留容量与远程访问成本。 | [原文](https://ieeexplore.ieee.org/abstract/document/11617834) · [DOI](https://doi.org/10.1109/ISCA66397.2026.00109) |

<a id="translation"></a>

## 页表遍历、TLB 与地址翻译

大多属于体系结构方案；注意与现有 GPU 软件优化的区别。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [MCM GPU Virtual Memory](docs/papers_zh.md#mcm-vm) | MICRO 2022 | 研究多芯粒 GPU 的虚拟内存组织及地址翻译开销。 | [原文](https://www.csa.iisc.ac.in/~arkapravab/papers/MICRO22_MGvm.pdf) |
| [SnakeByte](docs/papers_zh.md#snakebyte) | HPCA 2023 | 利用页面连续性进行自适应递归合并，扩大 TLB 的有效覆盖。 | [原文](https://yonsei.elsevierpure.com/en/publications/snakebyte-a-tlb-design-with-adaptive-and-recursive-page-merging-i-2/) |
| [Barre Chord](docs/papers_zh.md#barre-chord) | ISCA 2024 | 面向 MCM GPU 优化翻译组织与合并，缓解共享翻译服务的瓶颈。 | [原文](https://seonjinna.github.io/assets/pdf/BarreChord_isca24.pdf) |
| [Avatar](docs/papers_zh.md#avatar) | MICRO 2024 | 先推测地址翻译结果，再快速验证，减少等待页表遍历造成的停顿。 | [原文](https://zoon17.github.io/pdfs/avatar_micro24.pdf) · [DOI](https://doi.org/10.1109/MICRO61859.2024.00029) |
| [STAR](docs/papers_zh.md#star) | MICRO 2024 | 允许不同地址区域共享 TLB 子项，提高多实例 GPU 场景下的翻译缓存利用率。 | [原文](https://yueqiwang42.github.io/assets/pdf/papers/STAR_MICRO24.pdf) · [DOI](https://doi.org/10.1109/MICRO61859.2024.00031) |
| [Marching Page Walks](docs/papers_zh.md#marching-page-walks) | HPCA 2025 | 批量化和并发执行页表遍历，缓解大量地址翻译请求的排队。 | [原文](https://pure.korea.ac.kr/en/publications/marching-page-walks-batching-and-concurrent-page-table-walks-for-/) |
| [SoftWalker](docs/papers_zh.md#softwalker) | MICRO 2025 | 使用 GPU 执行资源承担软件页表遍历，让翻译处理能力随负载扩展。 | [原文](https://skku-compasslab.github.io/compasslab/publications/250715-micro-softwalker/) · [DOI](https://doi.org/10.1145/3725843.3756056) |
| [LATPC](docs/papers_zh.md#latpc) | MICRO 2025 | 结合局部性感知 TLB 预取和 MSHR 压缩，降低翻译请求带来的停顿与资源压力。 | [原文](https://pure.skku.edu/en/publications/latpc-accelerating-gpu-address-translation-using-locality-aware-t/) |
| [Chiplet-local Memory Mapping](docs/papers_zh.md#chiplet-locality) | MICRO 2025 | 利用芯粒局部性设计内存映射，权衡跨芯粒访问与地址翻译成本。 | [原文](https://pure.skku.edu/en/publications/leveraging-chiplet-locality-for-efficient-memory-mapping-in-multi/) · [DOI](https://doi.org/10.1145/3725843.3756090) |

<a id="serving"></a>

## LLM KV cache、服务内存与交接

应用层相关工作，帮助定位底层机制的使用场景。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [PagedAttention / vLLM](docs/papers_zh.md#pagedattention) | SOSP 2023 | 把 KV cache 划分为逻辑块并通过块表访问，支持非连续分配及共享，减少浪费。 | [原文](https://arxiv.org/abs/2309.06180) · [DOI](https://doi.org/10.1145/3600006.3613165) · [代码](https://github.com/vllm-project/vllm) |
| [Jenga](docs/papers_zh.md#jenga) | SOSP 2025 | 针对异构模型和 KV 形态设计分层内存管理，减少统一块大小带来的浪费。 | [原文](https://pacman.cs.tsinghua.edu.cn/~zjd/publication/generated/dblp-confsosp-zhang-dlkmwlyllz-25/) · [代码](https://github.com/heheda12345/Jenga-SOSP25-AE) |
| [DiffKV](docs/papers_zh.md#diffkv) | SOSP 2025 | 差异化管理 KV cache，并行执行 KV 紧凑整理，降低异构精度或尺寸带来的内存浪费。 | [原文](https://ipads.se.sjtu.edu.cn/zh/publications/sosp25-zhang.pdf) |
| [Prism](docs/papers_zh.md#prism) | OSDI 2026 | 通过 GPU memory ballooning 在多个模型服务之间弹性调整显存占用。 | [原文](https://www.usenix.org/conference/osdi26/presentation/yu-shan) · [代码](https://github.com/ovg-project/kvcached) |
| [SIRIUS](docs/papers_zh.md#sirius) | USENIX ATC 2025 | 利用训练语义快速腾出显存，配合内存交接策略支持训练与推理混部。 | [原文](https://www.usenix.org/conference/atc25/presentation/wang-jiali) · [PDF](https://ipads.se.sjtu.edu.cn/_media/publications/sirius-atc25.pdf) |
| [S-LoRA](docs/papers_zh.md#s-lora) | MLSys 2024 | 统一内存池管理不同 rank 的 LoRA 权重和不同序列长度的 KV cache，配合异构批处理 kernel。 | [原文](https://proceedings.mlsys.org/paper_files/paper/2024/file/906419cd502575b617cc489a1a696a67-Paper-Conference.pdf) · [代码](https://github.com/S-LoRA/S-LoRA) |
| [Pensieve](docs/papers_zh.md#pensieve) | EuroSys 2025 | 保存和复用多轮对话状态，在 GPU 与 CPU 之间分层管理 KV cache。 | [原文](https://arxiv.org/abs/2312.05516) · [DOI](https://doi.org/10.1145/3689031.3696086) |
| [Llumnix](docs/papers_zh.md#llumnix) | OSDI 2024 | 在服务实例之间迁移请求及其 KV 状态，实现动态负载均衡。 | [原文](https://www.usenix.org/conference/osdi24/presentation/sun-biao) · [代码](https://github.com/AlibabaPAI/llumnix) |
| [Mooncake](docs/papers_zh.md#mooncake) | FAST 2025 | 把分布式 KV cache 作为服务架构核心，利用跨设备存储资源复用计算结果。 | [原文](https://www.usenix.org/conference/fast25/presentation/qin) |
| [InfiniGen](docs/papers_zh.md#infinigen) | OSDI 2024 | 预测后续 attention 所需的重要 KV 子集，动态取回缓存，减少 GPU 常驻容量。 | [原文](https://www.usenix.org/conference/osdi24/presentation/lee) |
| [FlexGen](docs/papers_zh.md#flexgen) | ICML 2023 | 联合规划 GPU、CPU 与存储之间的权重、激活和 KV 数据流，提高有限显存下的批量推理吞吐。 | [原文](https://proceedings.mlr.press/v202/sheng23a.html) · [代码](https://github.com/FMInference/FlexGen) |
| [AQUA](docs/papers_zh.md#aqua) | ASPLOS 2025 | 降低抢占式 LLM 服务中推理状态换出的成本，使突发请求下的时间片调度更可行。 | [原文](https://arxiv.org/abs/2407.21255) · [DOI](https://doi.org/10.1145/3676641.3715983) |

<a id="training_storage"></a>

## 训练卸载与 GPU–CPU–存储层级

通过调度和数据移动扩展模型容量。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [G10](docs/papers_zh.md#g10) | MICRO 2023 | 利用张量使用规律，在 GPU、CPU 与 SSD 层级之间规划数据迁移。 | [原文](https://jianh.web.engr.illinois.edu/papers/g10-micro23.pdf) · [DOI](https://doi.org/10.1145/3613424.3614309) |
| [BaM](docs/papers_zh.md#bam) | ASPLOS 2023 | GPU 线程按需发起存储访问，并用软件缓存管理数据，减少 CPU 在 I/O 关键路径上的参与。 | [原文](https://research.ibm.com/publications/gpu-initiated-on-demand-high-throughput-storage-access-in-the-bam-system-architecture) |
| [ZeRO-Infinity](docs/papers_zh.md#zero-infinity) | SC 2021 | 结合训练状态分片与 CPU/NVMe 卸载，扩展可训练模型规模。 | [原文](https://arxiv.org/abs/2104.07857) · [DOI](https://doi.org/10.1145/3458817.3476205) · [代码](https://github.com/deepspeedai/DeepSpeed) |
| [TSPLIT](docs/papers_zh.md#tsplit) | ICDE 2022 | 以张量拆分提供更细粒度的内存管理，改善训练中的容量与换入换出选择。 | [原文](https://ieeexplore.ieee.org/document/9835178) |
| [Harmony](docs/papers_zh.md#harmony) | PVLDB / VLDB 2022 | 重排训练计算和数据移动，减少容量不足的多 GPU 服务器上的反复换入换出。 | [原文](https://arxiv.org/abs/2202.01306) · [DOI](https://doi.org/10.14778/3551793.3551828) · [代码](https://github.com/msr-fiddle/harmony) |

<a id="measurement"></a>

## 内存行为测量与诊断

为机制设计和性能归因提供方法。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [DrGPUM](docs/papers_zh.md#drgpum) | ASPLOS 2023 | 将 GPU 内存低效行为与应用对象和调用关联，为优化提供定位依据。 | [原文](https://www.jokeren.tech/publication/lin-2023-drgpum/) · [DOI](https://doi.org/10.1145/3582016.3582044) |
| [UVM In-depth Analysis](docs/papers_zh.md#uvm-analysis-sc21) | SC 2021 | 分解实际 UVM 中缺页、迁移及软硬件交互的性能开销。 | [原文](https://sc21.supercomputing.org/proceedings/tech_paper/tech_paper_pages/pap249.html) · [PDF](https://tallendev.github.io/assets/papers/sc21.pdf) · [DOI](https://doi.org/10.1145/3458817.3480855) · [代码](https://github.com/tallendev/uvm-eval) |

<a id="capacity"></a>

## 压缩及其他应用容量管理

与显存容量有关的延伸阅读。

| 论文 | 会议 | 解决什么问题 | 入口 |
|---|---|---|---|
| [IBP](docs/papers_zh.md#ibp) | EuroSys 2026 | 用 Invariant Bit Packing 无损压缩减少张量传输量，配合 GPU 解压和异步 PCIe 传输。 | [原文](https://akkamath.github.io/publication/EuroSys26-IBP) · [PDF](https://syfi.cs.washington.edu/assets/papers/ibp.pdf) · [DOI](https://doi.org/10.1145/3767295.3803595) · [代码](https://github.com/AKKamath/InvariantBitPacking) |
| [CLM](docs/papers_zh.md#clm) | ASPLOS 2026 | 针对 3D Gaussian Splatting 将模型及优化器数据分层放置，选择性加载并流水化数据移动。 | [原文](https://tarzanzhao.github.io/CLM-GS/) · [PDF](https://cs.nyu.edu/~apanda/assets/papers/asplos26-cvm.pdf) · [代码](https://github.com/nyu-systems/CLM-GS) |

## 使用索引

`data/papers.json` 中，`scope=main` 对应主目录；`scope=supplement` 不计入 45 篇。`relation_level` 区分基准论文、直接分配器研究、底层相邻机制、应用层和测量工具。缺失的 DOI / PDF / 代码字段保留为 `null`，含义是本次未确认，不能据此断言作者没有公开材料。

例如，使用 Python 筛选 2025 年以来的底层研究：

```python
import json
from pathlib import Path

index = json.loads(Path("data/papers.json").read_text(encoding="utf-8"))
for p in index["papers"]:
    if (p["scope"] == "main" and p["year"] >= 2025
            and p["relation_level"] in {"anchor", "direct_allocator", "adjacent_low_level"}):
        print(p["short_name"], p["venue"], p["year"], p["primary_url"])
```

本索引采用关键词检索与引用追踪，属于可继续扩充的研究清单，并非逐篇扫描所有会议 proceedings 后的穷尽性统计。发现年份、链接或机制描述有误时，应优先依据正式出版记录与原论文修订。
