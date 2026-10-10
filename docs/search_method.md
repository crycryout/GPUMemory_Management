# 检索方法、范围与核验记录

[返回首页](../README.md)

## 时间范围

- 检索 / 核验日期：**2026-10-10（Asia/Shanghai）**。
- “近五年”采用滚动窗口：**2021-10-10 至 2026-10-10**。
- 已正式发表的论文按会议时间和正式年份归档，不按最早 arXiv 日期重复计数。
- SC 2021 于 2021 年 11 月举行，因此 ZeRO-Infinity 与 SC 2021 UVM 分析在范围内。
- 2021 年较早举行的会议不在窗口内；例如 HPCA 2021、ATC 2021 的相关工作可以作为历史背景，但不并入本次统计。
- 不将尚未到来的 2026 年会议或未确认录用的预印本自动视为已发表顶会论文。

## “顶会”的工作口径

本任务没有指定 CCF、CORE 或其他统一名单，因此采用**领域会议集合**，并公开这一选择，方便后续按用户标准收窄。这里没有声称这些会议拥有同一机构、同一年度的同一评级。

| 范围 | 本次主目录会议 |
|---|---|
| 系统、体系结构、HPC 与存储核心范围 | OSDI、SOSP、EuroSys、USENIX ATC、ASPLOS、ISCA、MICRO、HPCA、SC、PPoPP、FAST |
| GPU 内存直接相关的相邻领域 | MLSys、ICML、ICDE、PVLDB / VLDB |
| 单列补充、未计入主目录 | ICS、ICPP、IISWC、PACT、ISPASS、尚未确认正式顶会归属的 arXiv 工作 |

主目录 45 篇中，核心会议集合为 **41 篇**，相邻领域为 **4 篇**。若只关心与 MoonBright 同层次的机制，可按 `category` 选择 `allocator`、`uvm`、`multi_gpu`、`translation`，共 **24 篇（含 MoonBright）**。

## 检索路径

采用关键词检索、MoonBright 及相邻论文的引用 / 项目线索追踪，再以论文或出版记录核验。没有逐篇审阅上述会议五年内的全部 proceedings，因此本清单不宣称穷尽。

实际使用的检索词族包括：

| 线索 | 检索词 |
|---|---|
| 分配器与虚拟地址 | GPU memory allocator、device-side allocation、GPU virtual memory、virtual memory stitching、memory fragmentation |
| 页表和 TLB | GPU page table、page table materialization、software page walk、TLB coherence、TLB shootdown、address translation |
| UVM | unified virtual memory、GPU oversubscription、page migration、prefetching、zero-copy、access-aware UVM |
| 多 GPU | hierarchical unified memory、memory harvesting、multi-GPU page placement、object-aware page management |
| 模型运行时 | KV cache memory management、dynamic memory、GPU memory ballooning、memory handover、tensor migration |
| 已定位论文 | 精确标题 + 会议 / 年份 / 作者；用于核验和版本消歧 |

典型查询组合为“GPU memory allocator ASPLOS PPoPP”“GPU UVM oversubscription MICRO ISCA HPCA”以及“论文精确标题 + conference”。检索范围不只覆盖 NVIDIA；AMD SVM、MI300A 和 Grace Hopper 的平台工作放在相关补充中。

## 来源优先级与证据

1. 正式会议页面、proceedings、出版社 DOI 记录。
2. 作者托管的论文 PDF、作者 / 实验室出版列表与作者维护的 artifact。
3. arXiv 用于获取作者版本及区分预印本状态；DBLP 等索引用于发现和辅助核对。
4. 技术机制以论文、作者摘要或官方介绍为依据；未使用博客总结代替原论文作为机制证据。

本次核验使用的主要入口包括：

- [USENIX OSDI 2026：MoonBright](https://www.usenix.org/conference/osdi26/presentation/zhang-yangyu)
- [USENIX OSDI 2026：Prism](https://www.usenix.org/conference/osdi26/presentation/yu-shan)
- [MICRO 2025 官方程序](https://microarch.org/micro58/program/index.php)
- [SOSP 2025 录用论文列表](https://sigops.org/s/conferences/sosp/2025/accepted.html)
- [HPCA 2026：ARIADNE](https://2026.hpca-conf.org/details/hpca-2026-main-conference/66/ARIADNE-Adaptive-UVM-Management-for-Efficient-GPU-Memory-Oversubscription)
- [SC 2021：UVM 分析](https://sc21.supercomputing.org/proceedings/tech_paper/tech_paper_pages/pap249.html)
- [SC 2025：HELM](https://sc25.conference-program.com/presentation/?id=pap340&sess=sess370)
- [MLSys 2024：S-LoRA 论文](https://proceedings.mlsys.org/paper_files/paper/2024/file/906419cd502575b617cc489a1a696a67-Paper-Conference.pdf)
- [ICML 2023：FlexGen](https://proceedings.mlr.press/v202/sheng23a.html)
- [IISc 实验室论文列表](https://csl.csa.iisc.ac.in/publications.html)
- [ICS 2025 官方程序](https://hpcrl.github.io/ICS2025-webpage/program/program.html)

每篇的直接来源保存在 `data/papers.json` 和逐篇笔记中。核验精度不一致时，`verification_note_zh` 给出限制。例如 HELM 已核验会议和主题，但本次没有充分阅读其算法细节；CDFD 机制概括以出版摘要为主。没有填入未知性能数字或推测的代码链接。

## 去重与容易出错的归属

| 条目 | 本仓库采用的处理 |
|---|---|
| vAttention | ASPLOS 2025；2024 年为预印本时间 |
| STAlloc | EuroSys 2026；2025 年为预印本时间 |
| Pensieve | EuroSys 2025；不误记为 ASPLOS，也不按 2023 预印本重复统计 |
| Harmony | PVLDB / VLDB 2022；不误记为 OSDI |
| ObservUVM / SuperUVM | 用正式标题 Observability-aided GPU Memory Oversubscription，作者列表别名保留但不重复计数 |
| DREAM / GPUVM | 保存 ICS 2025 与相关预印本两种入口；未确认完整版本关系，不额外计为一篇顶会论文 |
| GMEM | “submitted to ASPLOS”不等于录用；仅放预印本补充 |
| vTensor | 未确认本条预印本的正式顶会归属，不与同名的其他 tensor 工作混同 |
| Grace Hopper memory study | ICPP 2024，列补充，不写成 SC |
| PagedAttention / vLLM | 一篇论文及其系统名，不重复计数 |
| SIRIUS | 使用论文正式标题，系统简称作为别名 |

## 排除规则与限制

不以“使用 GPU”或“使用内存”作为唯一纳入理由。主要是算子加速、普通缓存优化、纯模型结构、与 GPU 无关的 CPU 内存分层工作，不纳入本清单。一般 GPU 调度工作只有在显存管理是主要机制时才纳入。

这里提供中文机制摘要和原文入口，未镜像 PDF。论文和代码的版权与许可证归原作者。公开代码链接只表示发现作者维护的入口，不表示代码可在当前驱动上运行、项目仍活跃或结果已被独立复现。

链接依据检索结果与作者页面记录；部分出版社或作者网页的直接打开受限，未完成所有端点的下载验证。缺失的 `pdf_url`、`doi`、`code_url` 用 `null` 表示“本次未确认”，不表示不存在。

## 数据维护

稳定的 `id` 用于 Markdown 锚点；`scope` 决定是否计入主目录；`category` 与 `relation_level` 用于按研究层次筛选。所有条目带有核验日期。

新增论文时，应先核对正式标题、会议和年份，再补充原文链接、机制、与 MoonBright 的关系及实现层次。预印本正式发表后应更新同一记录，避免重复增加数量。若后续发现完整版本演进证据，再明确合并 DREAM / GPUVM 等关联记录。
