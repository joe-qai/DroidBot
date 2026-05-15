# DroidBot 技术栈速成 + Superpowers 入门学习方案设计文档

> **版本**：v1.0
> **日期**：2026-05-15
> **状态**：已确认

***

## 1. 概述

### 1.1 问题背景

DroidBot 全栈开发计划（P0-P6）预估 39 人天，采用 FastAPI + Vue3 + SQLAlchemy + uiautomator2 技术栈。当前开发者处于**零基础**状态（Python 和 JS 都没做过项目），业余投入 2h/天。

如果不先学习技术栈直接开发：
- AI 生成的代码看不懂 → 无法判断质量 → 多轮修正浪费时间
- AI 加速效果仅 1.1x（几乎等于纯人工）
- 预估开发时间 58 人天（11.6 周），效率极低

### 1.2 学习策略

采用**先学核心再边做边学**策略：
- 先花 5-6 周集中学习核心技术栈的核心概念（不求精通但求理解）
- 学习完成后开始开发，在开发过程中继续深入学习
- Superpowers 工作流从学习阶段就开始嵌入，不做单独培训

### 1.3 核心目标

1. 建立编程基础认知，能读懂 AI 生成的代码并解释大意
2. 能独立用 FastAPI 写 CRUD API、用 Vue3 写 SPA 页面
3. 熟练掌握 superpowers 工作流（brainstorming → plan → TDD → debug）
4. 能用 uiautomator2 在真机运行自动化脚本
5. 学习完成后 AI 加速效果从 1.1x 提升到 1.5-2x

### 1.4 关键决策

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 学习方案 | 分阶段渐进式（5层） | 零基础必须按依赖关系分层学习，每层验证后再进入下一层 |
| 学习节奏 | 2h/天业余投入 | 开发者实际情况 |
| Superpowers 嵌入 | 从 Layer 1 开始在学习练习中使用 | 不单独学工作流，而是在实战中自然掌握 |
| 学习材料 | 官方教程为主 + AI答疑为辅 | 避免"看视频不做练习"的陷阱，官方教程最权威 |
| 验收方式 | 每层有实操验收项目 | 必须通过才能进入下一层，防止"学完不懂" |

***

## 2. 学习分层架构

```
Layer 0: 编程思维基础（Python + JS 双语启蒙）  ← 1.5 周
         │
Layer 1: 后端核心（FastAPI + SQLAlchemy）       ← 1.5 周
         │
Layer 2: 前端核心（Vue3 + Vite + Pinia）        ← 1 周
         │
Layer 3: Superpowers 工作流精通                 ← 0.5 周
         │
Layer 4: 专业领域（uiautomator2 + ADB + Agent） ← 1 周
```

总学习时间：5-6 周 × 2h/天 ≈ 70-84 小时

每层学习时间控制在 1-1.5 周，每层结束有实操验收项目，必须通过才能进入下一层。

***

## 3. Layer 0 — 编程思维基础（Python + JS 双语启蒙）

### 3.1 目标

建立"程序是什么、怎么运行、怎么调试"的基本认知，能读懂简单代码并做小修改。

### 3.2 时间

1.5 周 × 2h/天 = 21 小时（D1-D11）

### 3.3 为什么需要这层

零基础直接学 FastAPI/Vue3 = 看不懂任何代码。必须先理解变量、函数、条件、循环、错误处理这些最基础的概念。

### 3.4 学习内容

| 天 | 主题 | 学习材料 | 实操练习 |
|---|------|---------|---------|
| D1 | Python 基础：变量、类型、print | [Python官方教程](https://docs.python.org/zh-cn/3/tutorial/introduction.html) Ch1-3 | 写一个脚本：输入姓名→输出问候语 |
| D2 | Python 条件与循环 | Python官方教程 Ch4 | 写一个脚本：猜数字游戏（1-100） |
| D3 | Python 函数与模块 | Python官方教程 Ch5-6 | 把猜数字游戏拆成多个函数 |
| D4 | Python 文件读写 + 错误处理 | Python官方教程 Ch7-8 | 写一个脚本：读配置文件→解析→报错处理 |
| D5 | Python 字典、列表、类基础 | Python官方教程 Ch9-10 | 写一个"项目管理器"：用字典存项目列表，能增删查 |
| D6 | JS 基础：变量、函数、DOM | [MDN JS入门](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript) Ch1-3 | 写一个HTML页面：按钮点击→显示文字 |
| D7 | JS 条件、循环、数组 | MDN JS入门 Ch4-5 | 写一个HTML页面：待办事项列表（增删） |
| D8 | JS async/await 基础 | MDN JS异步教程 | 写一个HTML页面：fetch请求→显示结果 |
| D9 | 综合练习 | — | 用Python写一个CLI小工具（自己选题），用JS写一个HTML小页面 |
| D10-11 | 缓冲 + 补弱 | — | 回顾不懂的部分，补充练习 |

### 3.5 验收标准

1. 能独立写一个 50 行以上的 Python 脚本（有函数、文件IO、错误处理）
2. 能独立写一个 HTML+JS 页面（有 DOM 操作、事件处理）
3. 能读懂 AI 生成的简单 Python/JS 代码并解释大意

***

## 4. Layer 1 — 后端核心（FastAPI + SQLAlchemy）

### 4.1 目标

理解 Web API 的工作原理，能用 FastAPI 写 CRUD API，理解 ORM 如何操作数据库。

### 4.2 时间

1.5 周 × 2h/天 = 21 小时（D12-D21）

### 4.3 学习内容

| 天 | 主题 | 学习材料 | 实操练习 |
|---|------|---------|---------|
| D12 | HTTP 基础 + FastAPI 入门 | [FastAPI官方教程](https://fastapi.tiangolo.com/zh/tutorial/) Part1-3 | 写一个 Hello World API + 路径参数 |
| D13 | FastAPI 请求体 + Pydantic | FastAPI官方教程 Part4-5 | 写一个用户注册API（接收JSON→验证→返回） |
| D14 | FastAPI CRUD 完整示例 | FastAPI官方教程 Part6-9 | 写一个"笔记本"API：创建/列表/详情/删除笔记 |
| D15 | SQLAlchemy 2.0 模型定义 | [SQLAlchemy官方教程](https://docs.sqlalchemy.org/) ORM部分 | 定义 3 张表（User, Note, Tag）+ 关系 |
| D16 | SQLAlchemy CRUD 操作 | SQLAlchemy官方教程 Query部分 | 用 ORM 实现"笔记本"的增删改查 |
| D17 | FastAPI + SQLAlchemy 整合 | FastAPI官方教程 SQL Databases | 把笔记本API从内存换成SQLite持久化 |
| D18 | 异步基础 + async SQLAlchemy | FastAPI官方教程 Async部分 | 把笔记本API改成async版本 |
| D19 | CORS + 中间件 + 配置管理 | FastAPI官方教程 Middleware/CORS | 给笔记本API加CORS、错误处理、.env配置 |
| D20 | **验收项目**：迷你 DroidBot | — | 用 FastAPI+SQLAlchemy 实现项目的 CRUD API（仅 projects 表），这是 DroidBot P1 的缩小版 |
| D21 | 缓冲 + 补弱 | — | 回顾不懂的部分 |

### 4.4 Superpowers 嵌入

从 D14 开始，练习项目使用 superpowers 工作流：

- **D14**：用 `superpowers:brainstorming` 思考"笔记本API"要什么功能
- **D17**：用 `superpowers:writing-plans` 写实现计划
- **D20**：用 `superpowers:test-driven-development` 写测试先，再写代码

### 4.5 验收标准

1. 能独立用 FastAPI 写一个有 CRUD + SQLite 的 API 项目
2. 能解释 ORM 和原生 SQL 的区别
3. 能读懂 P0 实现计划中的 FastAPI 代码并解释每一行
4. 能用 superpowers brainstorming 分析一个小需求

***

## 5. Layer 2 — 前端核心（Vue3 + Vite + Pinia）

### 5.1 目标

理解 SPA 的工作原理，能用 Vue3 写页面组件，理解状态管理和路由。

### 5.2 时间

1 周 × 2h/天 = 14 小时（D22-D28）

### 5.3 学习内容

| 天 | 主题 | 学习材料 | 实操练习 |
|---|------|---------|---------|
| D22 | Vue3 基础：模板语法、响应式 | [Vue3官方教程](https://cn.vuejs.org/guide/introduction.html) Ch1-3 | 写一个组件：计数器 + 双向绑定输入框 |
| D23 | Vue3 组件 + Props/Events | Vue3官方教程 Ch4-5 | 写一个"任务列表"组件：父组件传任务→子组件渲染 |
| D24 | Vue3 Composition API + computed/watch | Vue3官方教程 Ch6-7 | 给任务列表加过滤（已完成/未完成）+ 自动保存 |
| D25 | Pinia 状态管理 | [Pinia官方教程](https://pinia.vuejs.org/zh/) | 把任务列表的状态移到 Pinia store |
| D26 | Vue Router + Axios | [Vue Router官方教程](https://router.vuejs.org/zh/) | 给任务列表加2个页面（列表页/详情页）+ API请求 |
| D27 | Vite 项目搭建 + CSS 变量 | [Vite官方教程](https://cn.vitejs.dev/guide/) + DroidBot现有HTML | 搭建一个 Vue3+Vite 项目，用暗色主题CSS变量 |
| D28 | **验收项目**：项目管理前端 | — | 写一个"项目管理"Vue页面：列表+创建弹窗+删除确认，对接 Layer 1 的笔记本API |

### 5.4 Superpowers 嵌入

- **D22**：用 `superpowers:brainstorming` 思考"计数器"组件应该有什么功能
- **D28**：用 `superpowers:writing-plans` 写前端实现计划 + `ui-ux-pro-max` 设计暗色主题卡片

### 5.5 验收标准

1. 能独立用 Vue3 写一个有组件、路由、状态管理的 SPA
2. 能让前端对接后端 API（Axios 请求 → Pinia 存储 → 页面渲染）
3. 能读懂 P0 实现计划中的 Vue3 代码
4. 能用 superpowers writing-plans 规划一个前端功能

***

## 6. Layer 3 — Superpowers 工作流精通

### 6.1 目标

熟练掌握 brainstorming → writing-plans → TDD → executing-plans → code-review 的完整开发循环。

### 6.2 时间

0.5 周 × 2h/天 = 7 小时（D29-D31）

### 6.3 为什么这层短

Layer 1-2 已经在练习中逐步使用了 superpowers，这层只是**集中强化**和补全没覆盖的技能（systematic-debugging、code-review、verification-before-completion）。

### 6.4 学习内容

| 天 | 主题 | 学习材料 | 实操练习 |
|---|------|---------|---------|
| D29 | brainstorming 完整流程 | superpowers brainstorming skill | 用 brainstorming 设计一个"脚本详情页"功能 |
| D30 | writing-plans + executing-plans | superpowers 对应 skills | 用 writing-plans 写"脚本详情页"实现计划，用 executing-plans 执行 |
| D31 | TDD + systematic-debugging | superpowers 对应 skills | 用 TDD 给"脚本详情页"写测试；故意写一个bug用 systematic-debugging 定位 |

### 6.5 验收标准

1. 能用 brainstorming 独立分析一个需求（不依赖 AI 提问）
2. 能用 writing-plans 写出结构化的实现计划
3. 能用 TDD 写测试后再写代码
4. 遇到 bug 能用 systematic-debugging 而不是随机试错

***

## 7. Layer 4 — 专业领域（uiautomator2 + ADB + Agent）

### 7.1 目标

理解 Android 自动化测试的原理，能用 uiautomator2 写简单脚本，理解 Agent 循环的概念。

### 7.2 时间

1 周 × 2h/天 = 14 小时（D32-D38）

### 7.3 学习内容

| 天 | 主题 | 学习材料 | 实操练习 |
|---|------|---------|---------|
| D32 | ADB 基础 + 设备连接 | [ADB官方文档](https://developer.android.com/studio/command-line/adb) + pure-python-adb | 用ADB连接真机、查看设备信息、安装/卸载应用 |
| D33 | uiautomator2 基础 | [u2官方README](https://github.com/openatx/uiautomator2) + 示例 | 用u2连接真机、截图、点击、输入文字 |
| D34 | u2 定位元素 + 等待策略 | u2文档元素定位部分 | 写一个脚本：打开设置→找到WiFi→点击（用不同定位方式） |
| D35 | u2 完整脚本编写 | u2文档 + DroidBot设计文档 | 写一个完整脚本：打开APP→执行3个操作→截图→断言结果 |
| D36 | MCP 概念 + Agent 循环原理 | [MCP协议文档](https://modelcontextprotocol.io/) + DroidBot设计文档 | 理解Agent如何用MCP工具与设备交互，画一个Agent循环流程图 |
| D37 | LLM API调用基础 | OpenAI兼容接口文档 | 用Python调用一个LLM API，理解prompt/response结构 |
| D38 | **验收项目**：手动 Agent | — | 手动模拟Agent循环：写需求→LLM生成脚本→真机执行→截图→分析结果 |

### 7.4 验收标准

1. 能用 uiautomator2 在真机上运行一个 5 步以上的自动化脚本
2. 能解释 Agent 循环的每一步（需求→规划→生成→执行→分析→修正）
3. 能用 Python 调用 LLM API
4. 能解释 MCP 工具在 Agent 中的作用

***

## 8. 学习完成后的开发准备评估

### 8.1 能力对照

| 能力 | 对应 AI 加速效果 |
|------|---------------|
| 读懂 AI 生成的 FastAPI/Vue3/SQLAlchemy 代码 | AI 加速从 1x → 1.5x |
| 用 superpowers 工作流规范开发流程 | 减少返工，加速从 1.5x → 1.8x |
| 判断 AI 代码质量是否合格 | 减少修正时间，加速从 1.8x → 2x |
| 用 uiautomator2 在真机验证脚本 | P2/P4 不再完全靠AI，加速从 1x → 1.3x |

### 8.2 修正后的开发时间估算

**原计划**：39 人天 × 1.3 = 50 人天（10 周）

**学习完成后，AI 加速 1.5-2x**：
- 39 人天 / 1.7 × 1.2 缓冲 ≈ 28 人天（5.6 周）

### 8.3 总时间线对比

| 方案 | 总时间 |
|------|--------|
| 不学习直接开发（效率极低） | 58 人天 ≈ 11.6 周 |
| 先学习再开发（本方案） | 5-6 周学习 + 5.6 周开发 ≈ **10-11 周** |
| 熟练开发者 + AI（理论最优） | 24 人天 ≈ 4.8 周 |

**结论**：花 5-6 周学习反而比不学习更快完成项目（10-11周 vs 11.6周），且代码质量更高、个人技术成长可持续。

### 8.4 开发阶段继续学习策略

学习完成后进入开发阶段，采用"边做边学"策略深化理解：

- 每个子项目（P1-P6）开始前，先用 brainstorming 分析需求 → 会发现哪些知识还不够深入
- 开发过程中遇到不懂的代码 → 用 AI 解释（而不是直接让 AI 写）→ 学习后再自己写
- 每个子项目完成后 → 用 code-review 回顾代码 → 识别哪些模式可以复用到下一个子项目

***

## 9. 学习方法指南

### 9.1 每日学习循环（2小时分配）

```
0-40分钟：读官方教程/文档（输入）
40-70分钟：做实操练习（输出）
70-90分钟：用 AI 答疑（不理解的概念让AI解释）
90-120分钟：回顾 + 记笔记（巩固）
```

### 9.2 避坑指南

| 坑 | 症状 | 解法 |
|----|------|------|
| 只看不练 | 看完教程觉得都懂了，一写代码就懵 | 每个概念必须写一个最小练习验证理解 |
| 依赖AI代写 | 让AI写代码→自己看→以为懂了→换个场景就不会 | 先自己写（哪怕很烂），再让AI点评和改进 |
| 跳层学习 | Layer 0没学完就看FastAPI教程 | 严格执行验收标准，不通过不升级 |
| 追求完美 | 某个概念卡住就反复学，进度停滞 | 学到"能用"就往下走，开发阶段会自然深化 |
| 看太多视频 | 看了10小时视频教程，0小时写代码 | 视频最多看20%，实操至少60%，AI答疑20% |

### 9.3 Superpowers 学习使用方式

| Superpowers Skill | 首次使用时机 | 用法 |
|-------------------|-------------|------|
| brainstorming | Layer 1 D14 | 练习项目前先 brainstorming 分析需求 |
| writing-plans | Layer 1 D17 | 练习项目前写一个简单实现计划 |
| test-driven-development | Layer 1 D20 | 验收项目用 TDD 方式开发 |
| systematic-debugging | Layer 3 D31 | 集中学习 bug 定位方法论 |
| ui-ux-pro-max | Layer 2 D28 | 前端验收项目用 skill 设计 UI |
| verification-before-completion | 开发阶段 | 每个子项目完成前做验证检查 |
| code-review | 开发阶段 | 每个子项目完成后做代码评审 |

***

## 10. 学习资源汇总

### 10.1 Python

- [Python 官方教程（中文）](https://docs.python.org/zh-cn/3/tutorial/index.html)
- [Python 练习平台](https://www.practicepython.org/) — 简单练习题

### 10.2 JavaScript

- [MDN JavaScript 入门（中文）](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript)
- [MDN 异步编程教程](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous)

### 10.3 FastAPI

- [FastAPI 官方教程（中文）](https://fastapi.tiangolo.com/zh/tutorial/)
- [FastAPI SQL Databases 示例](https://fastapi.tiangolo.com/zh/tutorial/sql-databases/)

### 10.4 SQLAlchemy

- [SQLAlchemy 2.0 官方教程](https://docs.sqlalchemy.org/en/20/tutorial/index.html)
- [SQLAlchemy ORM 快速入门](https://docs.sqlalchemy.org/en/20/tutorial/orm_data_model.html)

### 10.5 Vue3

- [Vue3 官方教程（中文）](https://cn.vuejs.org/guide/introduction.html)
- [Vue3 示例项目](https://cn.vuejs.org/examples/)

### 10.6 Pinia + Vue Router + Vite

- [Pinia 官方教程（中文）](https://pinia.vuejs.org/zh/)
- [Vue Router 官方教程（中文）](https://router.vuejs.org/zh/)
- [Vite 官方教程（中文）](https://cn.vitejs.dev/guide/)

### 10.7 uiautomator2 + ADB

- [uiautomator2 GitHub](https://github.com/openatx/uiautomator2)
- [ADB 官方文档](https://developer.android.com/studio/command-line/adb)
- [pure-python-adb GitHub](https://github.com/Swind/pure-python-adb)

### 10.8 MCP + LLM

- [MCP 协议文档](https://modelcontextprotocol.io/)
- [OpenAI API 文档](https://platform.openai.com/docs/api-reference)

***

## 11. 验收项目详细说明

### 11.1 Layer 0 验收：CLI 工具 + HTML 页面

**Python CLI 工具**：
- 功能：读取 JSON 配置文件，解析并输出格式化信息
- 要求：有函数、文件 IO、错误处理、至少 50 行
- 示例：读取项目列表 → 按名称搜索 → 输出详情

**HTML+JS 页面**：
- 功能：待办事项管理页面
- 要求：有 DOM 操作、事件处理、数组遍历
- 示例：添加/删除/标记完成 → 本地存储

### 11.2 Layer 1 验收：迷你 DroidBot 后端

- 功能：项目的 CRUD API（仅 projects 表）
- 要求：FastAPI + SQLAlchemy + SQLite + Pydantic schemas
- 对应 DroidBot P1 的缩小版
- 必须通过 superpowers TDD 流程开发

### 11.3 Layer 2 验收：项目管理前端

- 功能：项目管理的 Vue 页面（列表 + 创建弹窗 + 删除确认）
- 要求：Vue3 + Pinia + Axios + 暗色主题
- 对接 Layer 1 验收项目的 API
- 必须通过 superpowers brainstorming + writing-plans 流程开发

### 11.4 Layer 3 验收：脚本详情页

- 功能：一个完整的"脚本详情页"功能
- 要求：前后端一起做（FastAPI API + Vue3 页面）
- 必须通过完整的 superpowers 流程（brainstorming → plan → TDD → execute → review）

### 11.5 Layer 4 验收：手动 Agent

- 功能：手动模拟 Agent 循环的完整流程
- 要求：写需求 → 调用 LLM 生成脚本 → 真机执行 → 截图 → 分析结果
- 不需要自动循环，手动执行每个步骤即可
- 验证理解了 Agent 的每个环节

***

## 12. 学习进度追踪

建议用一个简单的 markdown 文件追踪每日学习进度：

```markdown
# 学习进度日志

## Layer 0: 编程思维基础
- [ ] D1: Python 变量/类型/print → 问候语脚本 ✓/✗
- [ ] D2: 条件/循环 → 猜数字游戏 ✓/✗
...
- [ ] 验收项目: CLI 工具 + HTML 页面 ✓/✗

## Layer 1: 后端核心
- [ ] D12: FastAPI 入门 → Hello API ✓/✗
...
```

每天学习结束时记录：完成了什么、遇到什么困难、哪些概念还不清楚（下次优先答疑）。