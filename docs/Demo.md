# 如何跑一个 Demo

## 环境安装

建议在独立虚拟环境中安装，避免污染 base 环境：

```bash
conda create -n data-juicer python=3.10
conda activate data-juicer
pip install py-data-juicer
```

---

## 需要准备什么

跑一个 Demo 只需要两个文件：

### 1. 数据文件（JSONL 格式）

每行一条 JSON，字段名自己定义：

```jsonl
{"instruction": "请解释牛顿第一定律", "response": "牛顿第一定律指出..."}
{"instruction": "光合作用是什么？", "response": "光合作用是植物利用光能..."}
```

JSONL 是通用格式，每行一个独立 JSON 对象，适合流式处理大文件。

### 2. Pipeline 配置文件（YAML 格式）

```yaml
project_name: 'my-sft-demo'
dataset_path: './demos/my_sft_demo/raw_sft_data.jsonl'
np: 1
text_keys: 'response'   # 指定用哪个字段做质量判断，与 JSONL 中的字段名对应

export_path: './demos/my_sft_demo/output/processed_sft_data.jsonl'

process:
  # 过滤 response 太短的样本
  - text_length_filter:
      min_len: 20
      max_len: 2000

  # 过滤重复率过高的样本（如模型退化输出"好好好好好..."）
  - character_repetition_filter:
      rep_len: 5
      min_ratio: 0.0
      max_ratio: 0.2

  # 只保留中文数据
  - language_id_score_filter:
      lang: 'zh'
      min_score: 0.5
```

---

## 运行

```bash
dj-process --config demos/my_sft_demo/pipeline.yaml
```

`dj-process` 是 data-juicer 安装后注册的命令行工具，等价于：

```bash
python -m data_juicer.tools.process_data --config demos/my_sft_demo/pipeline.yaml
```

---

## 输出

处理后的数据会写入 `export_path` 指定的路径。运行日志会显示每个算子处理后剩余的样本数：

```
[1/3] OP [text_length_filter] Done. Left 7 samples.
[2/3] OP [character_repetition_filter] Done. Left 6 samples.
[3/3] OP [language_id_score_filter] Done. Left 5 samples.
```

---

## 参考 Demo

项目内置了若干可直接运行的 Demo，见 `demos/` 目录：

| 目录 | 说明 |
|---|---|
| `demos/process_simple/` | 最简单的 Demo，单算子，适合入门 |
| `demos/my_sft_demo/` | 针对 SFT 文本数据的三算子 Demo |
| `demos/process_on_ray/` | 分布式 Ray 执行示例 |
