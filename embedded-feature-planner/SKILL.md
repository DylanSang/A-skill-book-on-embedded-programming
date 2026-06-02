---
name: embedded-feature-planner
description: 嵌入式新功能需求规划器。当用户提出新功能需求、开始实现一个新功能、或进入新工程/已有工程开发时自动使用。自动检测工程状态（新工程 vs 已有工程），从多角色视角分析需求，输出 方案.md、Todo.md、debug.json 三份规划文档。Use when the user mentions 新功能、需求实现、方案设计、功能规划、new feature, or starts working in an empty or existing project directory.
---

# 嵌入式新功能需求规划器

## 角色资产路径

本 Skill 依赖以下角色提示词（读取对应文件以获取角色视角）：
```
C:\Users\maitekai\Desktop\AI\embedded_ai_roles\embedded_ai_roles\embedded_ai_roles\
├── 01_product_manager.md      # 需求可行性、用户价值
├── 02_system_architect.md     # 架构设计、模块划分
├── 03_hardware_engineer.md    # 硬件接口约束
├── 04_bsp_engineer.md         # 底层驱动可行性
├── 05_driver_engineer.md      # Linux 驱动支持
├── 06_system_engineer.md      # 系统集成、性能
├── 07_application_engineer.md # AI 推理、业务逻辑
├── 08_test_engineer.md        # 可测试性、风险
├── 09_devops_engineer.md      # 构建、部署
└── 10_project_manager.md      # 工作量、优先级
```

---

## 工作流：自动检测工程状态

### 步骤 1：判断工程类型

检查当前工作目录：
- **新工程**：目录为空，或仅有少量非源码文件（无 `.c/.cpp/.h/CMakeLists.txt/Makefile` 等）
- **已有工程**：存在源码文件、构建脚本、或项目配置

**新工程 → 执行流程 A**
**已有工程 → 执行流程 B**

### 步骤 0：意图识别（默认彻底修复）

当用户输入包含以下任一意图时，**默认进入“全局彻底修复模式”**，不需要用户再额外说明“彻底”：

- `修复bug` / `修复 bug`
- `解决bug` / `解决 bug`
- `解决问题` / `排查问题` / `处理报错`
- `编译失败` / `链接失败` / `无法运行`

进入该模式后，必须强制执行：

1. `全局排查范围（6条）`
2. `高频故障闭环模板（A~D）`
3. `全局 Debug 最小验收（5条）`
4. `门禁变更后强制自检`

禁止行为：

- 只修当前报错行后结束
- 只加门禁不补实现
- 只给建议不落地改代码/脚本/构建

> 只有在用户明确要求“仅做临时绕过/仅定位不修复”时，才允许降级为非彻底模式，并必须在 `debug.json` 记录降级原因。

---

## 流程 A：新工程

### A1. 多角色需求分析

依次从以下角色视角分析用户需求（读取对应角色文件获取判断依据）：

| 角色 | 分析重点 |
|------|---------|
| 产品经理 | 需求是否清晰、用户价值、验收标准 |
| 系统架构师 | 模块划分、MCU/MPU 职责边界、接口设计 |
| 硬件工程师 | 硬件接口约束、信号要求 |
| BSP/驱动工程师 | 底层支持可行性、驱动现状 |
| 应用工程师 | AI 推理路径、MQTT 通信方案 |
| 测试工程师 | 可测试性、风险点 |
| 项目经理 | 工作量估算、优先级 |

统合各角色意见，对不同实现路径进行优劣对比。

### A2. 生成三份文档

执行完 A1 后，生成以下三个文件到当前工作目录：

**`方案.md`** — 见 [方案.md 模板](#方案md-模板)
**`Todo.md`** — 见 [Todo.md 模板](#todomd-模板)
**`debug.json`** — 见 [debug.json 模板](#debugjson-模板)

---

## 流程 B：已有工程

### B1. 检查 topStruct.md 是否存在

```
存在 topStruct.md → 跳到 B2
不存在           → 执行 B1a
```

### B1a. 生成 topStruct.md（切换模型）

**切换到 claude-sonnet 或更强模型** 执行以下分析：

1. 扫描工程目录结构，识别所有功能模块
2. 分析整机各模块的调用关系（系统级视图）
3. 深入各模块内部，梳理关键接口的调用链
4. 记录重点接口的输入/输出、MQTT 主题、数据格式
5. 梳理硬件架构（芯片型号、外设连接、引脚分配）
6. 记录 Flash/存储信息（分区表、各分区用途、大小）

输出 `topStruct.md`，要求：
- **功能模块化**：每个功能模块独立一节
- **两级视图**：整机调用图 → 各模块内部接口说明
- **重点接口**：参数、返回值、副作用、依赖项
- **硬件架构**：MCU/MPU 型号、外设、通信总线
- **Flash/存储**：分区表、各分区名称与用途

### B2. 多角色需求分析（参考 topStruct.md）

在读取 `topStruct.md` 了解现有架构后，执行与流程 A 相同的多角色分析，但需额外关注：
- 新功能与现有模块的集成点
- 对已有接口的影响与兼容性
- 复用现有代码的可能性

### B3. 生成三份文档

与流程 A 的 A2 相同，生成 `方案.md`、`Todo.md`、`debug.json`。

---

## 输出文档模板

### 方案.md 模板

```markdown
# 方案：[功能名称]

## 架构
[系统架构图或模块关系描述，明确 MCU/MPU 职责边界，进程划分，MQTT 主题]

## 接口定义
[关键接口：函数签名、MQTT 主题、数据结构、CLI 参数]

## 优先级
| 功能点 | 优先级 | 说明 |
|--------|--------|------|

## 重难点
[技术挑战、风险点、不确定项及应对思路]

## 工作量估算
| 模块 | 预估工时 | 负责角色 |
|------|---------|---------|

## 关键成果
[可量化的验收标准，如：延迟 <X ms、准确率 >Y%、通过 self-test]
```

### Todo.md 模板

```markdown
# Todo：[功能名称]

## 评分规则
- 每项 Todo 满分 10 分
- 完成度：代码实现完整得 4 分
- 测试通过：self-test/单测通过得 3 分
- 文档完整：接口注释/README 齐全得 2 分
- 代码质量：无 warning、符合规范得 1 分

## 任务列表

### 阶段一：[基础搭建]
- [ ] **[模块名]**：[具体任务描述]
  - 参考方案.md §[章节]
  - 预期产物：[文件/接口/功能]
  - 得分权重：[X/100]

### 阶段二：[集成联调]
- [ ] ...

### 阶段三：[测试验收]
- [ ] ...

## 总分追踪
| 任务 | 满分 | 当前得分 | 状态 |
|------|------|---------|------|
```

### debug.json 模板

```json
{
  "feature": "[功能名称]",
  "created_at": "[日期]",
  "requirement": {
    "input": "[用户原始需求描述]",
    "clarifications": []
  },
  "implementation": {
    "modules": [],
    "interfaces": [],
    "mqtt_topics": []
  },
  "debug_log": [],
  "status": "planning"
}
```

`debug_log` 条目格式：
```json
{
  "timestamp": "...",
  "module": "...",
  "issue": "...",
  "fix": "...",
  "result": "pass|fail|pending"
}
```

---

## 代码操作强制规则

**进行任何代码相关操作时（生成代码、编译、调试、交接），必须自动遵守以下规则文件中的所有约束：**

| 规则文件 | 触发场景 | 核心约束 |
|----------|---------|---------|
| `rv1126-safety-guard.mdc` | **始终生效** | 禁止刷机/分区/固件写入，保护板卡底层 |
| `rv1126-code-generation.mdc` | 生成/修改 `.c/.cpp/.h/CMakeLists.txt` | POSIX 接口、C++11、进程独立、MQTT 通信 |
| `rv1126-compilation.mdc` | 涉及构建脚本 `CMakeLists.txt/Makefile` | arm-linux-gnueabihf 工具链、故障自修复、编译报告 |
| `rv1126-debug-workflow.mdc` | **始终生效** | adb 推送→运行→日志→patch 闭环调试流程 |
| `rv1126-module-handover.mdc` | **始终生效** | 模块完成后生成标准七类交付清单 |

规则文件路径：
```
C:\Users\maitekai\Desktop\AI\embedded_ai_roles\embedded_ai_roles\rules\
├── rv1126-safety-guard.mdc
├── rv1126-code-generation.mdc
├── rv1126-compilation.mdc
├── rv1126-debug-workflow.mdc
└── rv1126-module-handover.mdc
```

**规则优先级（高→低）：**
```
safety-guard > code-generation > compilation > debug-workflow > module-handover
```

当规则与其他指令冲突时，规则具有更高优先级，尤其是 `safety-guard` 在任何情况下不可绕过。

### 生成代码相关角色职责（强化版，基于已发生 bug）

> 以下职责在“开始实现”后默认生效。任何角色未履责导致的编译失败，必须立即回滚到对应职责清单逐项补齐。

| 角色 | 代码生成阶段强制职责 | 对应已发生问题 |
|------|----------------------|---------------|
| **Application Engineer** | 1) 每个新增 `.c/.cpp/.h` 文件按“符号→头文件”映射显式 `#include`；2) 头文件若暴露平台类型（如 `speed_t`）必须自包含其定义头（如 `<termios.h>`）；3) 新增源码文件后同步校验是否已加入构建列表；4) C/C++ 代码提交前执行一次“最小可编译检查”；5) 若代码使用 `pthread_*`，在接口文档与实现注释中明确“需要 pthread 链接” | `speed_t has not been declared`、源文件漏进 CMake、pthread 链接失败 |
| **Driver Engineer** | 1) 审核 POSIX 相关类型/接口头文件完整性；2) 对串口/IO 模块重点检查 `termios/select/unistd/fcntl` 头文件是否直接包含；3) 避免依赖“间接包含”；4) 对涉及线程/同步的驱动交互代码标注线程库依赖 | 串口头文件缺失、线程依赖遗漏 |
| **System Engineer** | 1) 当项目存在 `.cpp` 或 `project(... CXX)`，在方案中显式声明“必须具备 C++ 交叉编译链”；2) 保证模块边界不引入隐式依赖（头文件可独立编译）；3) 审核 CMake 源清单与模块设计一致；4) 方案中显式声明链接依赖（如 `pthread/m/rt`） | CXX 配置遗漏、架构声明与工程不一致、链接库遗漏 |
| **DevOps Engineer** | 1) bootstrap 必须在 cmake 前完成 `gcc/g++` 成对探测；2) toolchain 写入编译器绝对路径；3) 自动适配 `-9` 变体；4) 编译脚本强制 UTF-8 无 BOM + LF；5) 脚本失败时打印可执行修复命令；6) 每次改构建逻辑后清理并重建 `build_arm` 验证；7) 对 `pthread_*`/`clock_gettime` 等符号执行链接依赖检查并自动补齐 `target_link_libraries` | `/bin/bash^M`、`CMAKE_CXX_COMPILER not found`、`undefined reference to pthread_*` |
| **Test Engineer** | 1) 增加“构建前门禁测试”：编码/换行/权限/JSON 合法性；2) 增加“编译链完整性测试”：`gcc/g++` 同时可用；3) 新增“头文件自包含检查”：公共头单独编译 smoke test；4) 新增“链接阶段 smoke test”：检查常见库符号（pthread/rt）可解析 | 环境伪故障反复出现、链接阶段故障漏检 |
| **Project Manager** | 1) 维护 `debug.json.debug_log` 为唯一故障事实源；2) 每次 `pass` 级修复后，推动将经验反写到本 Skill；3) 禁止把报错日志粘贴在 JSON 外 | debug 记录方式不规范导致 JSON 失效 |

#### 角色交付门禁（生成代码时必须满足）

在“代码已生成，准备编译”时，按角色勾选：

```
[Application]  新增头文件可独立编译；平台类型对应头文件已显式包含
[Driver]       串口/IO 模块 POSIX 头文件检查通过（termios/select/unistd/fcntl）
[System]       若含 .cpp，方案与构建均声明 CXX 编译链
[DevOps]       gcc/g++ 成对可用，toolchain 为绝对路径，脚本为 LF 无 BOM，链接库自动核对
[Test]         构建前门禁脚本通过（编码+权限+JSON+编译器+链接符号）
[PM]           debug.json 已追加本轮 issue/fix/result 且 JSON 可解析
```

任一项未通过：禁止进入 `cmake --build`。

---

## 操作脚本生成规范（强制）

生成方案/Todo 时，凡涉及"执行步骤"（编译、部署、测试、环境配置）的部分，**必须同时产出对应平台的可执行脚本文件**，不能仅在 Markdown 里写命令块。

| 目标环境 | 格式 | 存放位置 |
|---------|------|---------|
| Linux / 板卡 | Bash `.sh`，UTF-8 无 BOM，LF，`#!/bin/bash` + `set -euo pipefail` | `scripts/bootstrap_linux.sh` |
| Windows 开发机 | PowerShell `.ps1`，UTF-8 无 BOM，LF，`$ErrorActionPreference="Stop"` | `scripts/bootstrap_windows.ps1` |
| 跨平台流程 | 两份同时生成 | `bootstrap_linux.sh` + `bootstrap_windows.ps1` |

### 一键 bootstrap 优于零散补丁（重要）

**有且只有一个 `scripts/bootstrap_<platform>.sh / .ps1`** 作为编译入口，必须：

1. **自给自足**：脚本内嵌 `CMakeLists.txt` / `toolchain-*.cmake` 等关键配置的干净内容，启动时强制覆盖写入
2. **`set -euo pipefail`** / `$ErrorActionPreference="Stop"`：任一步失败立即退出
3. **字节级自校验**：写关键文件后用 `head -c 3 | od -tx1` 确认无 `EF BB BF`
4. **绝对路径**：禁止 `..` / 相对路径，全部 `$PROJECT_ROOT/...`
5. **每步标号**：`echo "=== [N/total] ==="` 让用户能立即定位失败步骤
6. **结尾给下一步命令**：明确告诉用户编译完了怎么 adb push / 运行

**禁止反模式**：出一个错就加一个 `fix_xxx.sh` / `patch_*.py` / `normalize_*.sh`，scripts/ 不可堆零散补丁。所有修复逻辑必须收敛进 `bootstrap_<platform>` 里。

**判断依据**：看"这段命令在哪台机器上运行"：
- 板卡 RV1126 上运行 → bash
- Windows 开发机上运行 → PowerShell
- 先在 Windows 上触发编译、再 adb push 到板子 → 两份都生成

### 环境与编码强制规则（新增，必须执行）

> **目的**：避免再次出现 `bash: ... /bin/bash^M: bad interpreter` 这类由 CRLF/BOM 引起的环境伪故障。

#### 1) 任何脚本执行前，先做环境预检（不可跳过）

每次要运行 `scripts/*.sh` 前，必须先检查：

- `shell`：目标系统是否为 Linux/bash（非 PowerShell）
- `换行`：`.sh` 文件必须是 **LF**，禁止 CRLF
- `编码`：必须 **UTF-8 无 BOM**
- `权限`：Linux 侧脚本必须具备可执行权限（`chmod +x`）

若任一检查失败：**立即修复后再执行脚本**，不得带错继续编译。

#### 2) bootstrap 脚本必须内建“自检 + 失败即退出”

`scripts/bootstrap_linux.sh` 顶部必须包含并保持以下行为：

- `set -euo pipefail`
- 对关键脚本/配置文件做 BOM + CRLF 检查
- 发现 `EF BB BF` 或 `\r` 直接 `exit 1`，并打印明确报错
- 输出“如何修复”的提示（例如：统一转为 UTF-8 无 BOM + LF）

#### 3) Windows 写入策略（防止再次污染）

在 Windows 生成 `.sh` 或 `CMakeLists.txt`/`.cmake` 时：

- 必须显式使用 UTF-8 无 BOM 写入
- 写入后强制将 `\r\n` 归一化为 `\n`
- 禁止依赖编辑器默认编码/默认行尾

#### 4) debug.json 记录规则（强制）

出现环境类错误（编码/换行/权限/解释器）时：

- 必须追加到 `debug.json.debug_log`
- 必须使用合法 JSON 条目，不得在 JSON 文件尾写 `//` 注释
- 条目包含：`timestamp/module/issue/fix/result`

#### 5) 执行门禁（Gate）

在 `/embedded-feature-planner` 执行“开始实现”后，**第一条执行命令前**必须满足：

```
□ scripts/*.sh 已确认为 UTF-8 无 BOM + LF
□ bootstrap_linux.sh 可执行
□ debug.json 为合法 JSON（可被解析）
```

不满足任一项时，先修复门禁项，再进入编译/联调流程。

---

## 代码兼容性硬约束（强制）

> 生成 C/C++/脚本时，**必须先按下表自检兼容性**，否则交叉编译会因头文件缺失、严格警告、Python API 差异等"伪故障"反复返工。

### 角色责任分工（兼容性专项）

| 检查项 | 主责角色 | 备查规则文件 |
|--------|---------|-------------|
| POSIX 头文件完备性（sys/time.h / sys/select.h / inttypes.h / unistd.h 等不可漏） | **Application Engineer + Driver Engineer** | `rv1126-code-generation.mdc §C/C++ 兼容性硬约束` |
| 严格警告零容忍（unused-result / format-truncation / stringop-truncation） | **System Engineer** | 同上 |
| FORTIFY_SOURCE 友好写法（snprintf 用 `%.Ns`、不用 strncpy 截断） | **System Engineer** | 同上 |
| 工具链版本下界（C11 / C++11 / CMake 3.12 / Python 3.6 / Bash 4.x / GNU sed） | **DevOps Engineer** | 同上 + `rv1126-compilation.mdc` |
| 交叉编译器完整性（`gcc/g++` 必须成对可用，支持 `-v` 正常输出） | **DevOps Engineer** | `rv1126-compilation.mdc` |
| C/C++ 编译器路径确定性（优先绝对路径，禁止仅写裸编译器名） | **DevOps Engineer** | `rv1126-compilation.mdc` |
| C++ 入口可用性（项目含 `.cpp` 时必须先验证 `CMAKE_CXX_COMPILER`） | **System Engineer + DevOps Engineer** | `rv1126-compilation.mdc` |
| Python 辅助脚本兼容（禁 `Path.write_text(newline=)` 等 3.10+ API） | **DevOps Engineer** | 同上 |
| 跨平台编码（无 BOM + LF） | **DevOps Engineer** | `rv1126-code-generation.mdc §跨平台文件编码` |

### 生成前自检清单（每个 C 文件都过一遍）

```
□ 用到 struct timeval / gettimeofday → 已 #include <sys/time.h>
□ 用到 fd_set / select / FD_*        → 已 #include <sys/select.h>
□ 用到 PRIu64 / PRIx64 / SCNu64       → 已 #include <inttypes.h>
□ 用到 sleep / read / write / close   → 已 #include <unistd.h>
□ pread/pwrite/ftruncate/fscanf 调用 → 用 (void) 或显式判返回值
□ snprintf 写不定长字符串            → 用 %.Ns 限长
□ 不要 strncpy(dst, src, sizeof(dst)-1) → 改 memcpy + 手动 \0
□ Python 辅助脚本                    → open(...,newline='\n')，不用 Path.write_text(newline=)
□ Bash 脚本                          → set -e；目标 bash 4.x；sed 不依赖 \xNN 转义
```

**Agent 在写完每个源文件后必须按此清单回扫一遍，再交付。**

### 编译器故障强制规则（新增）

> **目标**：避免再次出现 `CMAKE_CXX_COMPILER not found` / `CXX compiler identification is unknown` 这类编译器配置故障。

#### 角色分工（编译器专项）

| 场景 | 主责角色 | 强制动作 |
|------|---------|---------|
| `project(... CXX)` 或存在 `.cpp` 文件 | **System Engineer** | 在方案阶段显式声明“需要 C++ 交叉编译器”，不得仅写 C 编译链 |
| 生成 `toolchain-*.cmake` | **DevOps Engineer** | 写入 `CMAKE_C_COMPILER` 与 `CMAKE_CXX_COMPILER`，优先绝对路径 |
| 执行 bootstrap 前 | **DevOps Engineer** | 先探测 `arm-linux-gnueabihf-gcc` 与 `arm-linux-gnueabihf-g++`（含 `-9` 变体） |
| CMake 首次配置失败 | **DevOps Engineer** | 立即解析报错并回写 toolchain，不允许“让用户手动补命令”后中断流程 |

#### 编译前门禁（新增，必须全部通过）

在 `cmake -DCMAKE_TOOLCHAIN_FILE=...` 之前，必须先满足：

```
□ command -v arm-linux-gnueabihf-gcc 或 arm-linux-gnueabihf-gcc-9 成功
□ command -v arm-linux-gnueabihf-g++ 或 arm-linux-gnueabihf-g++-9 成功
□ toolchain-*.cmake 内 C/CXX 编译器为可执行绝对路径
□ 当存在 .cpp 文件时，禁止仅配置 C 编译器
```

#### 故障闭环（强制）

若出现以下关键字任一：

- `CMAKE_CXX_COMPILER ... not found in the PATH`
- `The CXX compiler identification is unknown`
- `No CMAKE_CXX_COMPILER could be found`

必须执行闭环：

1. 在 bootstrap 内自动探测可用 `g++`/`g++-9`
2. 重写 `toolchain-*.cmake` 为绝对路径
3. 清理并重建 `build_arm/` 后重跑 cmake
4. 将本次根因与修复追加到 `debug.json.debug_log`
5. 若仍失败，给出单行可执行安装命令（例如 `sudo apt-get install g++-arm-linux-gnueabihf`）

#### 链接阶段故障闭环（新增）

若出现以下关键字任一：

- `undefined reference to pthread_create/pthread_join`
- `undefined reference to clock_gettime`
- `ld returned 1 exit status`

必须执行闭环：

1. 从报错符号反查依赖库（`pthread_create -> pthread`，`clock_gettime -> rt`）
2. 自动修正 `target_link_libraries(...)`（保持 Linux/Windows bootstrap 内嵌 CMake 同步）
3. 清理并重建 `build_arm/` 后重编
4. 将根因与修复追加到 `debug.json.debug_log`
5. 在角色门禁中补充“链接符号检查”防止复发

#### 禁止反模式（编译器）

| 反模式 | 危害 | 替代方案 |
|--------|------|---------|
| 仅检查 `gcc`，不检查 `g++` | CMake 在 `project(... CXX)` 阶段必失败 | C/C++ 编译器成对预检 |
| toolchain 写裸名 `arm-linux-gnueabihf-g++` | PATH 环境差异导致不可复现 | 写 `command -v` 解析出的绝对路径 |
| 报 `CXX compiler unknown` 后继续 `cmake --build` | 流程无效、重复报错 | 先修 toolchain 再重配 |
| 代码用了 `pthread_*` 但 CMake 未链接 `pthread` | 链接阶段失败 | 生成代码时同步声明并校验链接库 |
| 把故障贴在 JSON 外部注释 | `debug.json` 失效 | 仅追加 `debug_log` 合法条目 |

---

## 配置与代码分离硬约束（强制）

> **背景**：用户（HR/产线/现场调试人员）拿到代码后最常做的事情是**改几个参数**——波特率、设备路径、超时、CSV 落盘位置、IP/端口。如果这些参数散落在 `port.c / channel.c / main.c` 里，用户每次都要全局搜索 + 编辑多文件 + 担心改漏，体验极差。**所有"运行时可变项"必须收敛到一个 `config.h`（或 `config.json/.ini`）里**，让用户只改一个文件即可。

### 何时必须独立 config 文件

只要满足下面任一条，就要建立独立 `config.h`：

- 用户/产线/现场需要在编译期或部署期调整该参数
- 同一参数在多处出现（如波特率同时影响 termios 配置和日志显示）
- 参数值与硬件/环境强相关（设备前缀、串口路数、CSV 路径）
- 参数与"协议常量/类型定义"**性质不同**（协议常量稳定，配置项常变）

### 角色责任分工（配置分层专项）

| 检查项 | 主责角色 |
|--------|---------|
| 识别"用户可调参数"清单（PRD 中标记） | **Product Manager** |
| 设计配置分层架构（config.h / 运行时参数 / 协议常量 各归各位） | **System Architect** |
| 把 magic number 全部替换为 `CFG_*` 间接引用，写 `common.h` 别名层 | **Application Engineer** |
| 在 bootstrap 脚本中**保护 config.h 不被覆盖**，仅覆盖生成产物 | **DevOps Engineer** |
| 设计配置矩阵回归测试（不同 baud/路数组合各跑一次自测） | **Test Engineer** |
| 在交付物里加"用户可调项一览表"，README 顶端就指向 config.h | **Project Manager** |

### 配置文件写法规范

```c
// src/config.h —— 用户唯一需要编辑的文件
#ifndef CONFIG_H
#define CONFIG_H

/* ── 串口 ─────────────────────────────────────
 * 配对常量必须并列声明，并在注释里给出对照表
 * 修改时同步修改 CFG_BAUD_RATE（数字）和 CFG_BAUD_TERMIOS（B*）
 * --------------------------------------------- */
#define CFG_BAUD_RATE         115200
#define CFG_BAUD_TERMIOS      B115200

/* 常用对照：9600/19200/38400/57600/115200/230400/460800/921600 */

#endif
```

`common.h` 里写**别名层**保持原代码零改动：
```c
#include "config.h"
#define DEFAULT_BAUD   CFG_BAUD_RATE   /* 别名，源自 config.h */
```

### 配置项自检清单

每写一个新模块（或新增一个常量）时回扫：

```
□ 这个数字/路径用户会改吗？     → 是 → 移到 config.h 并加注释
□ 这个常量出现 ≥ 2 次了吗？      → 是 → 移到 config.h（DRY 原则）
□ 这个值与硬件/环境绑定吗？      → 是 → 移到 config.h
□ 这是"协议常量"还是"配置项"？   → 协议常量留 common.h，配置项进 config.h
□ 是配对常量（数值+平台编码）吗？→ 必须并列声明，注释里给对照表
□ bootstrap 脚本会覆盖 config.h 吗？→ 必须 NOT 覆盖，仅覆盖纯生成产物
```

### 禁止反模式

| 反模式 | 危害 | 替代方案 |
|--------|------|---------|
| 用户改个波特率要动 3 个 `.c` 文件 | 漏改 / 不一致 | 全收敛到 `config.h` 一处 |
| `B115200` 直接出现在 `port.c` | 改成 230400 时漏改 | `cfsetispeed(&t, CFG_BAUD_TERMIOS)` |
| config 写在 `common.h` 末尾 | 用户找不到 / 混着协议常量 | 独立 `config.h`，文件顶部即可见 |
| bootstrap 脚本把 config.h 一起覆盖 | 用户改完一编译就还原 | bootstrap 显式排除 config.h |
| `#define CFG_BAUD 115200` 但 termios 还用硬编码 `B115200` | 二者不一致 → 跑出错 | 配对常量必须并列且注释强调"二者必须同步改" |

---

## 注意事项

- 修复编译/链接问题时，**禁止只加门禁不补实现**；必须同时完成：
  1) 根因定位（文件/符号级）  
  2) 代码修复（函数/源文件/CMake 至少一项）  
  3) 脚本门禁（防复发）  
  4) 重新编译验证（通过后才标记 `pass`）
- debug 时**禁止只盯当前文件**；必须执行“全局链路检查”（声明/定义/调用 + 构建 + 脚本 + 配置 + 文档）
- 每轮“修复bug/解决问题”回复时，必须附带“全局修复执行清单（已做/待做）”，至少覆盖：符号链路、构建链路、脚本链路、配置链路、环境链路、文档链路、编译验证、debug.json 记录状态
- 生成所有文档前，先向用户确认功能需求已理解正确
- topStruct.md 分析耗时较长，生成前告知用户
- 所有文档生成到**当前工作目录根目录**，不要创建子目录
- debug.json 在后续调试中持续追加，不要覆盖已有条目
- 每次方案中包含构建/部署/测试步骤时，**DevOps Engineer 角色必须同步产出 `scripts/` 目录下的对应脚本文件**
- 每个 C/C++ 源文件生成后，**主责角色必须按"生成前自检清单"逐项确认**，再产出

---

## 全局 Debug 闭环（新增）

> 用户要求“彻底修复”时，默认执行全局模式；禁止局部补丁后直接结束。

### 全局排查范围（强制）

1. **符号链路**：声明（`.h`）→ 定义（`.c/.cpp`）→ 调用点（caller）一致性
2. **构建链路**：`CMakeLists.txt/Makefile/BUILD.gn` 源清单、链接库、编译选项
3. **脚本链路**：`bootstrap_*.sh/.ps1` 是否会覆盖、回滚或误判修复
4. **配置链路**：`config.h(.example)`、`common.h`、默认宏映射是否闭环
5. **环境链路**：编码（LF/BOM）、权限、PATH、工具可用性（避免工具缺失误报）
6. **文档链路**：`README/方案/Todo/debug.json` 是否同步最新行为

### 全局 Debug 最小验收（强制）

```
□ 不仅修报错文件，还检查至少 1 个上游和 1 个下游文件
□ 已验证构建系统确实纳入修复源文件
□ 已验证 bootstrap 门禁不会误拦截
□ 已执行完整构建验证（非仅静态检查）
□ debug.json 记录根因、修复、验证结果
```

---

## 高频故障闭环模板（新增）

> 以下模板为强制执行模板，优先级高于“经验性建议”。

### 模板 A：`debug.json` 被污染（JSON 外粘贴日志）

触发信号：
- `JSONDecodeError: Extra data`
- 文件尾出现 `}` 之后还有终端输出

必须动作：
1. 删除 JSON 外所有文本
2. 将终端现象转换为 `debug_log` 合法条目（`timestamp/module/issue/fix/result`）
3. 执行 JSON 解析校验，未通过不得继续下一步

### 模板 B：`undefined reference to ...`

触发信号：
- `ld returned 1 exit status`
- `undefined reference to xxx`

必须动作（缺一不可）：
1. **实现层**：确认符号在某个 `.c/.cpp` 中有定义；无定义则补实现（可含 weak fallback）
2. **构建层**：确认该源文件已在 `CMakeLists.txt/Makefile` 中
3. **门禁层**：在 `bootstrap` 中增加“符号存在 + 源文件入构建”检查
   - 若采用 fallback 方案，门禁必须使用 **OR 逻辑**：`主实现存在 OR fallback 存在`
   - 禁止门禁只检查主实现文件，否则会误报并阻断可用构建
4. **验证层**：清理构建目录并重编；通过后再写 `result: pass`

### 模板 C：同一错误重复出现 ≥2 次

必须动作：
1. 给出“一次性修复”方案（代码 + 脚本 + 文档三处同步）
2. 禁止仅追加临时脚本（`fix_xxx.sh`）  
3. 必须在 `debug_log` 记录“复发根因”与“防复发措施”

### 模板 D：门禁规则与实现策略冲突

触发信号：
- 门禁报错，但代码实际可链接（例如已存在 weak fallback）
- 新增门禁后出现“误拦截”而非真实编译错误

必须动作：
1. 复盘实现策略（主实现、fallback、弱符号覆盖关系）
2. 将门禁条件改为与实现一致的逻辑（AND/OR 明确化）
3. 对门禁规则执行“正反用例自检”：
   - 用例1：主实现存在，fallback 存在（应通过）
   - 用例2：主实现缺失，fallback 存在（应通过）
   - 用例3：主实现缺失，fallback 缺失（应失败）
4. 将本次“规则冲突根因 + 修复逻辑”写入 `debug_log`

---

## 门禁变更后强制自检（新增）

每次修改 `bootstrap_*.sh` 的门禁逻辑后，必须在交付前完成：

```
□ 门禁条件与当前实现策略一致（主实现 / fallback）
□ 不会误拦截已可用构建（至少 1 个正向样例）
□ 能拦截真实缺陷（至少 1 个反向样例）
□ 对应 CMakeLists/Makefile 与门禁检查项同步
□ debug.json 记录本次门禁变更与验证结果
```
