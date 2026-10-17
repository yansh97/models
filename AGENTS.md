# 模型目录维护指南 (Agent Guide)

本仓库维护主流大语言模型的公开元数据与官方 API 标价，数据文件为 `catalog.json`，结构由 `catalog.schema.json` 定义。

---

## 1. 核心准则

- **官方依据**：以官方定价页与开发文档当前展示的公开单价为准；严禁使用第三方聚合站或主观猜测。
- **中立与精确**：不收录特定订阅套餐、异步批处理（Batch）或预留吞吐量折扣；单价通过公式精确换算为整数，不留模糊精度。
- **克制变更**：仅修改有据可查的条目，保留既有键位顺序；不顺带重命名、不擅自淘汰既有旧模型。
- **动态演进与主动确认**：厂商会随时更新价格、推出全新系列或调整计费模式。遇到**全新模型系列、定价规则重大调整、未曾覆盖的计费形式或收录边界存疑**时，绝不凭猜测决断，务必主动向用户说明情况并确认方案。
- **合规表达**：若官方计费形式超出当前 Schema 表达能力（如按次、按时长收费），严禁通过硬填 `0` 或擅自放宽 Schema 绕过问题。

---

## 2. 收录范围与权威信源

更新前核实以下来源；链接失效或版本无法确认时保留原数据并报告，不擅自扩大收录范围。

| 系列 / 厂商 | 官方来源 | 收录范围与版本下限 | 典型型号示例 |
| :--- | :--- | :--- | :--- |
| **Claude** (Anthropic) | [Claude Platform Pricing](https://platform.claude.com/docs/en/about-claude/pricing) | Claude Fable ≥ 5、Claude Opus ≥ 5、Claude Sonnet ≥ 5、Claude Haiku ≥ 5 | `Claude Fable` (5, 5.1)、`Claude Opus` (5, 5.5)、`Claude Sonnet` (5, 5.5)、`Claude Haiku` (5.5) |
| **GPT** (OpenAI) | [OpenAI · Text tokens](https://developers.openai.com/api/docs/pricing#text-tokens) | GPT ≥ 5.6；Flagship models 的 Standard 档，展开 All models，排除 Cyber models | `GPT 5.6` (Sol, Terra, Luna)、`GPT 6` (Astra, Sol, Luna)、`GPT 6.1` (Sol) |
| **GPT Image** (OpenAI) | [OpenAI · Image generation](https://developers.openai.com/api/docs/pricing#image-generation) | GPT Image ≥ 2；Image generation models 的 Standard 档，展开 All models | `GPT Image 2`、`GPT Image 2.5` (Flare, Sunburst) |
| **Gemini** (Google) | [Gemini Platform Pricing](https://aistudio.google.com/docs/pricing) | Gemini ≥ 3.8；付费层级（Paid Tier）的 Standard 档 | `Gemini 3.8 Flash` 等旗舰与主流调用档位 |
| **Grok** (xAI) | [xAI Platform Pricing](https://docs.x.ai/developers/pricing) | Grok ≥ 4.5；仅 Text API | `Grok 4.5`、`Grok 4.6`、`Grok 4.7` |
| **Muse** (Meta) | [Meta Platform Pricing](https://dev.meta.ai/docs/pricing-rate-limits) | 文本模型的 Standard 档 | `Muse Spark` (1.1, 1.2, 1.3) |
| **DeepSeek** | [DeepSeek Platform Pricing](https://api-docs.deepseek.com/zh-cn/quick_start/pricing/) | 全部模型 | `DeepSeek Flash`、`DeepSeek V4 Pro` |
| **Kimi** (Moonshot) | [Kimi Platform Pricing](https://platform.kimi.com/docs/pricing/chat#模型定价) | Kimi ≥ K3 | `Kimi K3` |
| **GLM** (智谱) | [BigModel Platform Pricing](https://docs.bigmodel.cn/cn/guide/start/pricing) | GLM ≥ 5.3；仅旗舰模型和文本模型 | `GLM 5.3`、`GLM 5.3 Flash`、`GLM 5.3 FlashX` |
| **Qwen** (千问) | [Qwen Platform Pricing](https://platform.qianwenai.com/pricing/api) | Qwen ≥ 3.8；Max、Plus、Flash 等非参数规模的文本生成版本 | `Qwen 3.8 Max`、`Qwen 3.8 Flash` |

> **关于活动与排除项**：
> - **已落入表格的活动价**：若官方定价页已直接更新价格表格（如标注“即日起至某日特惠价/免费”，且当前表格已生效），直接以当前表格展示的价格收录。
> - **严格排除**：领券、充值返利、特定订阅绑定的套餐价、Batch 异步调用折扣或第三方代理商报价。

---

## 3. 数据契约与字段定义

数据根节点为 `{"models": { "<model_id>": { ... } }}`，不能改成数组。字典键与对象内部的 `id` 必须完全一致（唯一、非空、全小写）。

### 3.1 模型基础属性

| 字段 | 类型 | 说明与规范 |
| :--- | :--- | :--- |
| `id` | `string` | 唯一规范标识，全小写，与字典键严格一致（如 `gpt-6-astra`、`claude-sonnet-5-5`）。 |
| `label` | `string` | 官方展示名称，保留官方品牌大小写（如 `GPT 6 Astra`、`Claude Sonnet 5.5`）。 |
| `window` | `integer \| null` | 上下文总窗口近似正整数（便于 K/M 展示，如 1M 记为 `1000000`，500K 记为 `500000`）。纯生图或未知时为 `null`；**计费阈值不取整**。若取整后窗口小于扩展阈值，先确认取值，不改写阈值。 |
| `modalities` | `string[]` | 支持的输入模态列表，常用 `text`、`image`、`audio`、`video`、`pdf`。按官方标明填写。 |
| `efforts` | `string[] \| null` | 官方支持调节的推理强度档位。不适用或固定时显式为 `null`，严禁写成 `[]`。常用：`none`、`minimal`、`low`、`medium`、`high`、`xhigh`、`max`。 |
| `price` | `object` | 计费对象，包含 `currency`、`standard`，可选 `extended`、`peak`。 |

### 3.2 计费单位换算

- `currency` 仅支持官方原始币种（`USD` 或 `CNY`），禁止自行换汇。
- 单价为非负整数，单位是**每 Token 的十亿分之一货币单位**：
  $$\text{整数单价} = \text{官方每百万 Token (1M Tokens) 标价} \times 1000$$
  - `$2.5/M` $\rightarrow$ `2500`，`$0.15/M` $\rightarrow$ `150`，`¥0.028/M` $\rightarrow$ `28`，明确免费项 $\rightarrow$ `0`。

### 3.3 价格档位 (`PriceTier`)

每个价格档位完整包含 `cache_read`、`cache_write`、`input`、`output` 四项：

- `cache_read`、`input`、`output` 格式为“用量标签 → 单价”。统一定价写为 `"_"`，按模态区分写具体标签（如 `"text"`、`"image"`）。
- `cache_write` 格式为“缓存时长 → 用量标签 → 单价”。无细分时长写为 `"_"`，分时长如 `"5m"`、`"1h"`。
- **`{}` 与 `0` 的严格区分**：`{}` 表示不支持该计费特性，`0` 表示已确认免费，二者不能混淆。

```json
// 价格档位结构示例
{
  "cache_read": { "_": 1000 },
  "cache_write": { "5m": { "_": 12500 }, "1h": { "_": 25000 } },
  "input": { "text": 2500, "image": 5000 },
  "output": { "_": 50000 }
}
```

### 3.4 扩展上下文与高峰倍率

- **扩展上下文 (`price.extended`)**：包含 `threshold`（正整数阈值）与 `tier`（完整价格档位）。
  - 当 `缓存读取 + 缓存写入 + 普通输入` 之和 $\ge$ `threshold` 时，整笔请求套用扩展档；输出不参与区间判定。
  - **阈值按官方原值精确填写，严禁取整**。
  - 典型案例：`gpt-6-astra`、`grok-4.7`、`claude-haiku-5-5`。
- **高峰倍率 (`price.peak`)**：包含 `condition` 与正浮点数 `multiplier`。倍率统一作用于所选档位各项费用。
  - `condition.timezone` 仅支持 `"Asia/Shanghai"`。
  - `condition.weekdays` 为 `true` 时仅工作日（周一至周五）生效，`false` 每日生效（不处理节假日与调休）。
  - `condition.windows` 为本地时段数组，格式 `HH:MM` 或 `HH:MM:SS`，左闭右开（`start` $\le$ 时间 $<$ `end`），同一天内 `start < end` 且各时段不重叠。
  - 典型案例：`deepseek-flash`、`deepseek-v4-pro` 在工作日高峰时段加收费用（`multiplier: 2`）。

---

## 4. 维护工作流 (SOP)

```
[1. 核验信源与范围] ---> [2. 转换映射与编辑] ---> [3. 机器与语义复核] ---> [4. 交付与汇报]
```

1. **核验信源与范围**：查验官方定价页与模型文档，核对版本下限、模态、窗口、推理档位及价格结构。
2. **转换映射与编辑**：按公式计算整数单价，注意区分 `{}` 与 `0`。按现有键位顺序更新 `catalog.json`，不添加额外的备注或时间戳字段。
3. **校验与复核**：
   - **Schema 结构校验**：
     ```bash
     uv run --no-project --with check-jsonschema check-jsonschema --schemafile catalog.schema.json catalog.json
     ```
   - **语义规则复核**：字典键与 `id` 一致小写；`window` 仅合理取整而 `threshold` 保持精确原值；时段顺序无重叠；复算样本费用（`Token 数 × 整数单价` 之和乘倍率后除以 $10^9$）。
4. **交付与汇报**：检查 `git diff`，简要汇报更新模型、核验日期、官方链接；遇规则重大变更或 Schema 无法表达时，主动向用户说明并确认方案。
