# MLLM-Finetuning-Demo

一个基于 LLaMA-Factory 的多模态（LLaVA）预训练与指令微调最小示例，包含数据示例、训练配置与一键导出/上传流程。

---

## 目录结构

```text
config/                     # 训练与导出配置
data/
  ├─ dataset_info.json      # 数据集元信息（LLaMA-Factory 兼容）
  ├─ mllm_demo.json         # SFT 数据示例（指令微调）
  ├─ mllm_pt_demo.json      # PT 数据示例（特征对齐）
  └─ mllm_demo_data/        # 配图示例（1.jpg, 2.jpg, ...）
upload_dataset.py           # 将数据集上传到 Hugging Face 脚本
README.md                   # 本说明文档
```

## 环境要求

- Python 3.9+
- CUDA 可用的 PyTorch（建议遵循官方安装指引）
- Git、pip/conda

## 快速开始

### 1) 安装 LLaMA-Factory

```shell
git clone https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e .[torch,metrics]
cd ..
```

### 2) 准备数据集

- 已提供最小可跑示例：`data/mllm_pt_demo.json`、`data/mllm_demo.json` 与配图在 `data/mllm_demo_data/`。
- 如需自定义数据，请对照 `dataset_info.json` 与示例 JSON 的字段格式进行扩展。

### 3) 预训练（特征对齐）

基于已经预训练好的模型则可跳过（本次比赛可以跳过）

冻结 `language_model` 与 `vision_tower`，仅训练 `multi_modal_projector`（见 `config/llava_pt.yaml`）。

- Linux/macOS:
```shell
CUDA_VISIBLE_DEVICES=0 llamafactory-cli train config/llava_pt.yaml
```

- Windows (cmd.exe):
```shell
set CUDA_VISIBLE_DEVICES=0 && llamafactory-cli train config/llava_pt.yaml
```

### 4) 指令微调（LoRA）

使用 `config/llava_lora_sft.yaml` 进行 LoRA SFT。

- Linux/macOS:
```shell
CUDA_VISIBLE_DEVICES=0 llamafactory-cli train config/llava_lora_sft.yaml
```

- Windows (cmd.exe):
```shell
set CUDA_VISIBLE_DEVICES=0 && llamafactory-cli train config/llava_lora_sft.yaml
```

### 5) 网页聊天（Web UI）

下例使用已训练的适配器进行交互式验证。

- Linux/macOS:
```shell
CUDA_VISIBLE_DEVICES=0 llamafactory-cli webchat \
--model_name_or_path llava-hf/llava-1.5-7b-hf \
--adapter_name_or_path saves/llava1_5-7b/lora/sft \
--template llava
```

- Windows (cmd.exe):
```shell
set CUDA_VISIBLE_DEVICES=0 && llamafactory-cli webchat --model_name_or_path llava-hf/llava-1.5-7b-hf --adapter_name_or_path saves/llava1_5-7b/lora/sft --template llava
```

## 上传数据集到 Hugging Face

在 `upload_dataset.py` 中填写您的 `HF_TOKEN` 或相关密钥后执行：

```shell
python upload_dataset.py
```

## 导出与上传模型到 Hugging Face Hub

在 `config/llava_lora_sft_export.yaml` 中填写：

- `export_hub_model_id`: 目标仓库名（如 `your-username/your-model`）
- `hf_hub_token`: 您的 Hugging Face 访问令牌

- Linux/macOS:
```shell
CUDA_VISIBLE_DEVICES=0 llamafactory-cli export config/llava_lora_sft_export.yaml
```

- Windows (cmd.exe):
```shell
set CUDA_VISIBLE_DEVICES=0 && llamafactory-cli export config/llava_lora_sft_export.yaml
```

## 常见问题（FAQ）

- 显存不足：
  - 将 batch size、`gradient_accumulation_steps`、图像分辨率降低，或启用更高级别的量化/LoRA。
- 找不到数据文件：
  - 确认 `dataset_info.json` 中的数据集名称、路径与 JSON 字段一致。
- Windows 环境变量：
  - 在 `cmd.exe` 使用 `set VAR=VALUE && <command>`，在 PowerShell 使用 `$env:VAR="VALUE"; <command>`。

