# Claude Code 快捷初始化工作流插件
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
这是一个专为 Claude Code 打造的快捷初始化工作流插件。以下是插件的安装方式以及建议的工作区框架与相关配置文件。

## 下载与安装

在你的 Claude Code 终端中，依次输入并运行以下两条命令即可完成安装：

**1. 添加我的插件市场：**
`/plugin marketplace add SuperEldridge/Init-plugin`

**2. 安装本插件：**
`/plugin install Init-plugin@Init-plugin`

> 配置说明：
> 如果打不开，可能是终端没连上梯子，可以依次在终端运行以下命令：
```text
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global http.version HTTP/1.1
git config --global http.sslBackend schannel
```

---

## 建议配置的工作区框架

请参考以下目录结构搭建你的本地工作区：

```text
F:\workspace\
├── .claude/
│   ├── hooks/
│   │   ├── format_coding.py
│   │   └── ...
│   ├── skills/
│   │   └── find-skills/
│   │       ├── references/
│   │       └── SKILL.md
│   ├── rules/
│   │   ├── env-mcp.md          # 存放 Skill 和 MCP 的下载/配置规则
│   │   ├── embedded-c.md       # 存放 C/C++、HAL库、硬件防坑守则
│   │   ├── agent-flow.md       # 存放 SubAgent 唤醒和协作逻辑
│   │   └── project-init.md     # 存放新建项目时的初始化 SOP
│   └── settings.json           # 见下方详细配置
├── projects/
│   └── <你的具体项目>/
│       ├── progress.md         # (通过规则自动生成) 项目进度与待办
│       └── LESSONS.md          # (通过规则自动生成) 项目级专属错题本
├── Doc/
│   └── CLAUDE.md.bak           # (存放过时或暂时用不到的 CLAUDE.md)
├── .mcp.json
├── CLAUDE.md                   # 主控路由文件
└── Project_LESSONS.md          # 错题本，自进化用
```

> 配置说明：
> * 全局配置：以上为核心工作区搭建，你还可以在全局设置 Agent、CLAUDE.md 和 GLOBAL_LESSONS.md 等等，全局配置需结合个人情况具体分析。
> * 路径适配：如果遇到脚本跑不通的情况，请检查绝对路径。示例中使用的是 F 盘，请根据你的实际情况改成你自己的盘符和对应路径。
> * 子智能体：把模型换成你自己的，默认是moonshot的k2-0905-preview

---

## settings.json 配置文件

请将以下内容完整复制并保存到 `.claude/settings.json` 文件中：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "^(Edit|Write)$",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/format_coding.py"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "^Bash$",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/check_dangerous_cmd.py"
          }
        ]
      }
    ],
    "SubagentStart": [
      {
        "matcher": "^chaos[-_]tester$",
        "hooks": [
          {
            "type": "command",
            "command": "python .claude/hooks/chaos_alarm.py"
          }
        ]
      }
    ]
  }
}
```

---

## 工作区CLAUDE.md示例

```text
# Workspace Meta-Router (工作区路由守则)

> **【系统级强制隔离 - YOU MUST FOLLOW】**
> 以下规则适用于本工作区下的所有项目。在进行任何代码生成、修改、调试或终端操作前，必须严格遵守。


## 工作区目录与项目隔离铁律 (Strict Project Sandbox Isolation)
> **【沙盒隔离铁律 - YOU MUST FOLLOW】**
> 1. **强制根目录约束**：每次开始新任务，必须进入 `projects/` 文件夹去操作，如果没有相关项目则在 `projects/`里新建项目文件夹。绝对禁止在 `F:\workspace\` 根目录下生成任何业务代码或文件夹。
> 2. **绝对路径锁定**：确定项目后，所有文件读写必须严格锁定在 `F:\workspace\projects\<项目名>\` 内部，严禁越界。

> **【交互与版本控制规范】**
> 1. **强制中文交互**：所有对话、方案解释、错误排查输出必须使用专业中文。
> 2. **规范化 Git 提交**：格式强制为 `<type>(<scope>): <中文描述>` (如 `feat(adc): 增加 DMA 采样`)。
> 3. **原子化提交**：每次 commit 只专注于一件事。


## 代码与注释规范
* **详尽的中文注释**：所有生成的代码（尤其是 C/C++、汇编或相关构建脚本），必须包含详细的中文注释。
* **注释核心诉求**：对于关键逻辑分支、状态机轮转、硬件寄存器位操作、以及中断服务函数（ISR），必须在注释中清晰解释“为什么这么设计（Why）”以及“对应的硬件物理意义”，而不仅仅是描述代码语法（What）。


## 动态上下文触发器 (Read on Demand)
在执行特定任务前，你**必须优先静默读取**对应的领域规则文件：

1. **环境与工具配置:** 
   * **触发条件:** 当要求下载、安装、配置 plugin、Skills 或添加 MCP 服务器时。
   * **动作:** 立即读取 `F:\workspace\.claude\rules\mcp-skills.md`。

2. **新建项目初始化:**
   * **触发条件:** 当要求在 `projects/` 下创建一个新项目，或执行类似“初始化工作区”的任务时。
   * **动作:** 立即读取 `F:\workspace\.claude\rules\project-sop.md` 并严格按其步骤执行。

3. **C/C++ 与硬件编码:**
   * **触发条件:** 当任务涉及编写/修改 C/C++ 代码、STM32/ESP32 外设驱动、查阅 Datasheet 时。
   * **动作:** 立即读取 `F:\workspace\.claude\rules\embedded.md`。

4. **复杂测试与 Agent 协作:**
   * **触发条件:** 当代码生成完毕、触发代码测试、架构审查，或明确要求“压力测试”、“代码Review”时。
   * **动作:** 立即读取 `F:\workspace\.claude\rules\agent-flow.md` 唤醒对应 SubAgent。

## 经验总结与错题记录机制
当你与我共同排查并成功解决了一个项目中的 Bug、编译错误或逻辑难题后：
1. **项目级记录:** 你必须主动将【错误特征】和【解决方案】追加写入到该项目的 `projects/<当前项目名>/LESSONS.md` 中。
2. **工作区级记录：** 如果是犯了误操作工作区文件的错误（例如把项目放到了工作区根目录里），则写入`F:\workspace\Project_LESSONS.md`中。
3. **全局级记录判断:** 如果该 Bug 属于跨项目的通用硬件/C语言级大坑，你还需将其追加到全局错题本 `C:\Users\Yourname\.claude\GLOBAL_LESSONS.md` 中。
4. **错题记录读取机制** 当你在被分配任何新的编码、架构或 Debug 任务之前，你需要读取对应的错题记录，并确保不会再犯同样的错误。
```

> 配置说明：
> * 路径适配：把CLAUDE.md内容的文件路径改成你自己的，指向rules，亲测能大幅减少上下文占用，在需要的时候也调的出来。

