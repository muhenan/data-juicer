# LLM 数据处理常用算子

面向文本数据的 SFT 和 DPO 场景，按类别整理的常用算子。

---

## 基于规则（无需模型，速度快）

| 算子 | 类型 | 说明 |
|---|---|---|
| `text_length_filter` | Filter | 过滤文本长度超出范围的样本 |
| `character_repetition_filter` | Filter | 过滤字符级 n-gram 重复率过高的样本 |
| `word_repetition_filter` | Filter | 同上，词级别，更准确 |
| `special_characters_filter` | Filter | 过滤特殊字符比例过高的样本 |
| `stopwords_filter` | Filter | 过滤停用词比例过低的样本（通常为乱码或纯代码堆砌） |
| `whitespace_normalization_mapper` | Mapper | 规范化空白字符和换行符 |
| `punctuation_normalization_mapper` | Mapper | 全角标点转半角 |
| `clean_html_mapper` | Mapper | 去除 HTML 标签，适用于网页爬取数据 |
| `clean_links_mapper` | Mapper | 去除文本中的 URL |

---

## 基于小模型（需下载轻量模型）

| 算子 | 类型 | 说明 |
|---|---|---|
| `language_id_score_filter` | Filter | 基于 fasttext 的语言识别，只保留目标语言样本 |
| `perplexity_filter` | Filter | 基于 KenLM 困惑度过滤语言质量差的文本 |
| `minhash_deduplicator` | Deduplicator | 模糊去重，能识别近似重复，比精确匹配更强 |
| `document_deduplicator` | Deduplicator | 基于哈希的精确去重 |

---

## 基于大模型（效果最强，速度最慢，适合 SFT/DPO）

| 算子 | 类型 | 说明 |
|---|---|---|
| `reward_model_scorer` | Mapper | 用 reward model 对每条样本打分 |
| `llm_judge_filter` | Filter | 调用 LLM（如 GPT/Qwen）判断样本质量 |
| `ifd_scorer` | Mapper | 计算指令跟随难度（IFD），过滤过于简单的样本 |
| `diversity_sampler` | Selector | 对 embedding 聚类后均衡采样，防止话题分布偏斜 |

---

## DPO 专项

| 算子 | 类型 | 说明 |
|---|---|---|
| `reward_gap_filter` | Filter | 过滤 chosen/rejected 分差过小的对，分差太小则没有学习信号 |
| `response_consistency_filter` | Filter | 检测标注错误，如 chosen 质量反而低于 rejected |
| `prompt_deduplicator` | Deduplicator | 对 prompt 去重，防止某类问题过多导致过拟合 |
