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
| `document_minhash_deduplicator` | Deduplicator | MinHash + LSH 模糊去重，能识别近似重复 |
| `document_deduplicator` | Deduplicator | 基于 MD5 哈希的精确去重 |

---

## 基于大模型（效果最强，速度最慢，适合 SFT/DPO）

### Filter

| 算子 | 说明 |
|---|---|
| `llm_quality_score_filter` | LLM 给数据打质量分，过滤低质量样本 |
| `llm_difficulty_score_filter` | LLM 评估指令难度，过滤过于简单的样本 |
| `llm_task_relevance_filter` | LLM 判断数据是否与目标任务相关 |
| `llm_analysis_filter` | LLM 综合分析过滤 |
| `llm_perplexity_filter` | 用 LLM 算困惑度，比 KenLM 更准但更慢 |

### Mapper

| 算子 | 说明 |
|---|---|
| `calibrate_qa_mapper` | 用 LLM 校正/改写 QA 对 |
| `calibrate_query_mapper` | 用 LLM 改写 query |
| `calibrate_response_mapper` | 用 LLM 改写 response |
| `generate_qa_from_text_mapper` | 从原始文本自动生成 QA 对 |
| `generate_qa_from_examples_mapper` | 基于示例生成 QA 对 |

---

## DPO / GRPO 专项

| 算子 | 类型 | 说明 |
|---|---|---|
| `pair_preference_mapper` | Mapper | 给定 chosen response，调用 LLM 自动生成对立的 rejected response，直接产出 DPO 训练对 |
| `human_preference_annotation_mapper` | Mapper | 对接 Label Studio，将候选答案对推给人工标注，结果自动写回 chosen/rejected 字段 |
| `instruction_following_difficulty_filter` | Filter | 基于 IFD 分数过滤数据（见下方原理说明） |

---

## IFD 算法原理（instruction_following_difficulty_filter）

**论文**：[arxiv:2308.12032](https://arxiv.org/abs/2308.12032)

**核心公式**：

```
IFD = loss(response | query) / loss(response | 无 query)
```

用一个 HuggingFace CausalLM 模型（默认 Qwen2.5-0.5B）做推理，**不训练，只做 forward pass**，计算模型预测 response token 的 cross-entropy loss。

- **分母** `loss(无 query)`：模型凭空预测这段 response 的难度，作为基准线
- **分子** `loss(有 query)`：模型看到 query 之后预测 response 的难度

**IFD 越低越好**：

```
IFD 低（分子小）：有了 query，response 变得好预测
               → query 和 response 强相关 → 高质量数据，保留

IFD 高（接近1）：有没有 query 对预测 response 难度差不多
               → query 形同虚设 → 低质量数据，过滤
```

**使用方式**：只需设 `max_score`，过滤 IFD 过高的数据：

```yaml
- instruction_following_difficulty_filter:
    min_score: 0.0
    max_score: 0.7
```
