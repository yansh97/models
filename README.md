# 模型规格与 API 价格目录

本仓库按模型 ID 整理语言模型和图像生成模型的公开规格与官方 API 单价，可用于比较上下文窗口、输入模态和可调档位，也可按 Token 用量估算请求费用。数据见 [catalog.json](./catalog.json)，字段定义见 [catalog.schema.json](./catalog.schema.json)。

在仓库根目录运行以下 `jq` 命令，可查看 DeepSeek Flash 的规格和价格。

```sh
jq '.models["deepseek-flash"]' catalog.json
```

## 模型字段

`models` 是以模型 ID 为键的对象。每条记录包含以下六个必填字段。

| 字段 | 含义 |
| --- | --- |
| `id` | 模型 ID，与字典键相同，字母全小写。 |
| `label` | 官方展示名称，保留品牌大小写，将连字符 `-` 替换为空格。 |
| `window` | 上下文窗口的近似 Token 数，取正整数。K、M 按十进制换算，例如 1M 记为 `1000000`。纯图像生成模型或窗口未知时为 `null`。 |
| `modalities` | 官方支持的输入模态，如 `text`、`image`、`audio`、`video`、`pdf`。 |
| `efforts` | 文本模型的推理强度或图像生成模型的图像质量。非空数组列出官方支持的可调档位，排除 `auto` 等自动选项。档位固定或不适用时为 `null`。 |
| `price` | 官方计价币种 `currency`（`USD` 或 `CNY`）、标准档 `standard` 和可选的长上下文扩展档 `extended`。若 `extended` 缺省或为 `null`，所有请求均使用标准档。 |

## 价格与费用估算

### 单价

每个价格档位都包含普通输入 `input`、输出 `output`、缓存读取 `cache_read` 和缓存写入 `cache_write`。每个 Token 的单价以十亿分之一货币单位表示，取非负整数。将官方每百万 Token 标价乘以 1000 即可换算，例如每百万 Token 2.5 美元记为 `2500`。

| 计费项写法 | 含义 |
| --- | --- |
| `{"_": 2500}` | 该项按统一单价计费。 |
| `{"text": 5000, "image": 8000}` | 该项按模态定价，分别取对应单价。此例为 `gpt-image-2.6-flare` 的普通输入价。 |
| `{}` | 该计费项不适用。例如，模型仅支持显式缓存写入时，`cache_write` 为 `{}`。 |
| `{"_": 0}` | 官方明确标为免费。按模态定价时，只将免费的模态记为 `0`。 |

同一档位下，同一模态的同一计费项按请求时段定价时，取各时段的**最高单价**。缓存读写只收录隐式（自动）缓存价格。缓存写入按有效期定价时，取各有效期中的**最低单价**。

### 估算一次请求

1. **选择档位。** 默认使用 `price.standard`。若设有扩展档，将普通输入、缓存读取和缓存写入的 Token 数相加。总数达到或超过 `price.extended.threshold` 时，整笔请求（含输出）使用 `price.extended.tier`。
2. **计算费用。** 按计费项和模态分别计算 `Token 数 × 目录单价`，求和后除以 `10⁹`，结果以 `price.currency` 计价。若接口返回的输入总量包含缓存用量，应先拆分，再分别计价。

例如，`gpt-6.2-astra` 的一次请求读取 270000 个缓存 Token，另有 2000 个普通输入 Token，输入总量恰好达到扩展阈值 272000。若生成 1000 个输出 Token，估算费用为 `(270000 × 800 + 2000 × 16000 + 1000 × 60000) / 10⁹ = 0.308` 美元。

## 收录范围与来源

仅收录能从供应商官方定价页或开发文档核实的模型。除非下表另有说明，各系列均排除面向特定行业或用途的专用模型及其变体，如 Legal、Law、Cyber。

同一产品线按供应商命名的规格档分别保留最新版，即使旧版仍可调用。例如，GPT Sol 更新时只替换旧版 Sol，Astra、Terra 和 Luna 各自独立更新。

| 系列 | 起始版本（含） | 收录范围 | 官方定价页 |
| --- | --- | --- | --- |
| Claude（Anthropic） | 5.2 | Fable、Opus、Sonnet、Haiku | [Claude Pricing](https://platform.claude.com/docs/en/about-claude/pricing) |
| GPT（OpenAI） | 6 | Flagship models 的 Standard 档，含 All models 中的型号 | [Flagship models](https://developers.openai.com/api/docs/pricing#text-tokens) |
| GPT Image（OpenAI） | 2.6 | Image generation models 的 Standard 档，含 All models 中的型号 | [Image generation models](https://developers.openai.com/api/docs/pricing#image-generation) |
| Gemini（Google） | 4 | Paid Tier 的 Standard 档 | [Gemini Pricing](https://aistudio.google.com/docs/pricing) |
| DeepSeek | 不限 | 通用型号 | [DeepSeek 模型与价格](https://api-docs.deepseek.com/zh-cn/quick_start/pricing/) |
| Kimi（Moonshot） | K3.1 | 通用型号 | [Kimi 模型定价](https://platform.kimi.com/docs/pricing/chat#模型定价) |
| GLM（智谱） | 5.3 | 旗舰模型和文本模型 | [智谱开放平台定价](https://docs.bigmodel.cn/cn/guide/start/pricing) |

- **独立型号。** 加速或优先服务须由供应商单独命名为模型才收录，如 `kimi-k3.1-pro-highspeed`。同一型号的服务层级或配置不另设条目。
- **价格来源。** 采用核验时生效的官方公开单价，包括活动价和限时免费。排除优惠券、充值返利、特定订阅套餐、批处理（Batch）折扣及预留吞吐量价格。
