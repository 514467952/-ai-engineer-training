# 实验结果分析写在这里!
## 一、各类切片的显著影响参数及原因
### 句子切片（SentenceSplitter）
- **核心参数**：chunk_size、separators、split_long_sentences
- 关键原因：chunk_size 决定单块句子数量，直接影响语义完整性与冗余度；separators 定义分割边界（如“\n\n”“.”），决定是否保留完整语义单元；split_long_sentences 避免超长句导致检索聚焦性下降。

### Token 切片（TokenTextSplitter）
- **核心参数**：chunk_size、tokenizer、separator
- 关键原因：chunk_size 是核心粒度，固定每块 Token 数，平衡检索精度与效率；tokenizer 决定分词逻辑，影响 Token 计数准确性；separator 若设置不当，会破坏词汇或短语完整性。

### 句子窗口切片（SentenceWindow 组件）
- **核心参数**：window_size、chunk_size、chunk_overlap
- 关键原因：window_size 控制命中句子前后补充的句子数量，直接决定上下文丰富度；chunk_size 限制窗口整体 Token 上限，避免冗余；chunk_overlap 保障相邻窗口语义衔接，防止信息断层。

### Markdown 切片（MarkdownNodeParser）
- **核心参数**：heading_levels、chunk_size、include_metadata
- 关键原因：heading_levels 定义标题层级拆分规则，决定是否保留文档结构；chunk_size 适配代码块、列表等结构长度，避免截断关键内容；include_metadata 影响检索时对结构信息的利用。

## 二、chunk_overlap 过大或过小的利弊
### 过大（超过单块长度 20%）
- 优势：最大程度避免语义断裂，适合逻辑紧密的长文本；降低分块位置敏感性，减少检索失效。
- 弊端：增加冗余内容，提升存储与计算开销，检索响应变慢；可能引入噪声，干扰核心信息判断，降低检索精准度。

### 过小（接近 0 或低于 5%）
- 优势：减少重复数据，降低资源压力，检索响应更快；切片独立性强，关键词精准匹配场景（如 API 文档）更易聚焦目标。
- 弊端：易造成上下文断层，跨块核心语义无法完整呈现；对分块位置极敏感，分割点若在关键节点会破坏信息完整性，导致召回率下降。

## 三、“精确检索”与“上下文丰富性”的权衡方法
1. **动态调整核心参数**：精确检索场景（技术文档、API 说明）用小 chunk_size（384-768Token）+ 低重叠率（10-15%）；需上下文支撑的场景（综述、小说）用大 chunk_size（768-1024Token）+ 中等重叠率（15-20%）。
2. **后处理补充上下文**：小粒度切片保障精确检索，通过 SentenceWindowNodePostprocessor 在命中后补充 2-3 个前后句子，兼顾精准性与连贯性。