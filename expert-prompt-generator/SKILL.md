---
name: expert-prompt-generator
description: 根据用户的需求描述生成精简、专家级的 AI Prompt，并以追加方式写入当前工作目录的 describe_prompt.md。支持嵌入式（RV1126/MCU/MQTT/鸿蒙/驱动/BSP）与通用（代码/写作/分析/翻译/调试/产品）两种模式，会根据需求关键词自动选择领域专家身份。Use when the user asks 生成 prompt、写 prompt、设计角色、专家提示词、prompt engineering、提示词生成、给我一个 prompt，or mentions describe_prompt.md / discribe_prompt.md.
---

# Expert Prompt Generator

依据用户的一句话/一段话需求，产出**一段精简的专家级 prompt**，并**追加**写入当前工作目录的 `describe_prompt.md`。

---

## 工作流（必须按顺序执行）

### Step 1：解析需求

提取四要素，缺失则用合理默认值（不要反复追问用户，除非需求严重歧义）：

| 要素 | 说明 | 默认 |
|------|------|------|
| 领域 | 嵌入式 / 代码 / 写作 / 分析 / 翻译 / 调试 / 产品 / 其他 | 通用 |
| 任务 | 用户希望专家做什么 | 用户原话动词 |
| 关键约束 | 语言、长度、合规、性能、风格 | 无 |
| 输出形式 | Markdown / JSON / 代码 / 段落 / 表格 | Markdown |

### Step 2：自动选模式

扫描需求文本，命中以下任一关键词即用**嵌入式模式**，否则**通用模式**：

```
RV1126, MCU, MPU, STM32, MQTT, BSP, 驱动, 嵌入式, 鸿蒙, OpenHarmony,
RTOS, Linux 内核, 交叉编译, 工具链, 烧录, 固件, OTA, NPU, RKNN,
UART, I2C, SPI, GPIO, DTS, U-Boot, Buildroot
```

### Step 3：套用精简模板

**精简风格**：单段四行，不分小标题、不堆叠示例。

#### 通用模式模板

```
你是 [领域] 领域的资深专家，具备 [关键背景/经验]。
任务：[一句话任务]。
约束：[最关键的 1~3 条约束，用顿号分隔]。
输出：[格式 + 长度/结构要求]。
```

#### 嵌入式模式模板

```
你是嵌入式 [子领域：BSP/驱动/系统服务/AI 应用/测试] 资深工程师，熟悉 [芯片平台]、[关键技术栈]。
任务：[一句话任务]。
约束：遵循 P1-P11 工程规范、本地交叉链(P10)、[其他领域特定约束]。
输出：[格式 + 含可验证的量化指标]。
```

### Step 4：追加写入 describe_prompt.md

以**追加**方式写入当前工作目录（即用户当前 workspace 根，而非 skill 目录）的 `describe_prompt.md`：

- 文件不存在 → 创建并写入文件头
- 文件存在 → 在末尾追加新一节

每次写入一节，节标题用 ISO 时间戳，格式如下：

````markdown
## [YYYY-MM-DD HH:mm] — [一句话主题]

**原始需求**：
> [用户原话，引用块]

**模式**：通用 | 嵌入式

**生成 Prompt**：
```
[四行精简 prompt]
```

---
````

文件头（仅首次创建时写入）：

```markdown
# Prompt 库

> 由 expert-prompt-generator skill 自动追加，每节对应一次生成请求。
```

### Step 5：在对话中回显

向用户展示：
1. 识别的领域与模式（一行）
2. 生成的 prompt（代码块）
3. 已追加到 `describe_prompt.md`（含相对路径）

---

## 注意事项

- **文件名拼写**：固定使用 `describe_prompt.md`（不是 discribe_prompt.md）。若用户文档已存在 `discribe_prompt.md`，提示用户后再决定是否迁移。
- **工作目录**：写入路径始终是**用户当前 workspace 根**，不要写到 skill 目录或用户主目录。
- **追加而非覆盖**：用 Read 检查文件是否存在；存在则读取后在末尾拼接新节再 Write，或使用 StrReplace 在末尾追加。**禁止直接覆盖整个文件**。
- **精简优先**：prompt 必须在 4 行内完成，不要扩展成多段；如果用户后续要"详细版"再升级。
- **不要追问**：默认根据需求自行推断，仅当需求只有 1~2 个字（如"prompt"）才反问主题。
- **角色身份**：必须明确"你是 XX 领域 X 年经验/具备 X 背景的资深专家"，禁止用"你是一个 AI 助手"这类泛化身份。

---

## 示例

### 示例 1：通用模式

**用户输入**：帮我写个 prompt 用来审阅 Python 代码安全问题

**生成 prompt**：
```
你是网络安全与 Python 应用安全领域的资深专家，10 年代码审计经验，熟悉 OWASP Top 10 与 CWE。
任务：审阅给定 Python 代码，识别注入、反序列化、密钥泄露、依赖漏洞等安全风险。
约束：仅报告确证风险、按 严重/中/低 三档分级、给出最小可行修复 patch。
输出：Markdown 表格 | 列：风险点 / 位置 / 等级 / 修复建议。
```

### 示例 2：嵌入式模式

**用户输入**：需要个 prompt 帮我设计 RV1126 上 MQTT bridge 进程

**生成 prompt**：
```
你是嵌入式系统服务资深工程师，熟悉 RV1126、mosquitto、POSIX、systemd。
任务：设计 mcu_bridge 进程，将 STM32 UART 数据双向桥接到本地 MQTT Broker。
约束：遵循 P1-P11 工程规范、本地交叉链(P10)、单文件 ≤800 行、FIFO + 三级优先级、SIGTERM 优雅退出。
输出：Markdown 方案 | 含架构图、MQTT Topic 表、JSON Payload Schema、CLI 参数与退出码、可验证的延迟/丢包指标。
```
