# 兼容性测试自动化Agent可搭建方案

> **文档版本**：v1.0
> **编制日期**：2025年
> **适用场景**：移动端与锁端交互工具App的兼容性测试自动化

---

## 摘要

本方案针对移动端与锁端交互工具App的兼容性测试场景，设计了一套完整的自动化Agent系统。该系统采用六层Agent架构，结合VLM（视觉语言模型）视觉感知能力，实现了跨环境、跨设备的智能化兼容性测试。与传统自动化脚本相比，本方案具备环境自适应、语义级验证、异常自愈等核心能力，能够有效覆盖8大兼容性风险场景，输出可量化的兼容性评分报告。

> **⚠️ 自动化测试范围说明**
> 
> 本方案**仅覆盖App端操作到锁端的蓝牙开锁流程**。
> 
> 以下操作属于**锁端物理操作**，不在App自动化测试范围内：
> - 密码面板输入（数字密码、临时密码）
> - 指纹解锁
> - 门铃呼叫响应
> - 其他需要在锁端物理接触的操作
> 
> 如需测试上述场景，需使用专用的锁端测试设备和方法。

---

## 目录

1. [方案概述](#1-方案概述)
2. [兼容性测试风险体系](#2-兼容性测试风险体系)
3. [Agent架构设计](#3-agent架构设计)
4. [核心模块详细设计](#4-核心模块详细设计)
5. [测试场景库设计](#5-测试场景库设计)
6. [兼容性环境矩阵](#6-兼容性环境矩阵)
7. [报告与评分体系](#7-报告与评分体系)
8. [部署与集成](#8-部署与集成)
9. [扩展与演进](#9-扩展与演进)

---

## 1. 方案概述

### 1.1 兼容性测试Agent的定位与目标

#### 1.1.1 问题背景

移动端与锁端交互工具App面临严峻的兼容性问题挑战：

- **碎片化严重**：Android版本从8.0到14共存，设备分辨率从720p到2K+各异
- **厂商定制分化**：华为、小米、OPPO、vivo等厂商ROM各具特性
- **IoT场景复杂**：蓝牙连接状态、网络环境变化直接影响用户体验
- **安全敏感度高**：开锁操作涉及用户财产安全，可靠性要求极高

#### 1.1.2 传统方案的局限性

| 维度 | 传统自动化脚本 | 兼容性测试Agent |
|------|---------------|-----------------|
| 环境适配 | 硬编码坐标/路径 | 环境感知自动适配 |
| 验证方式 | DOM结构检查 | 语义级视觉理解 |
| 异常处理 | 固定重试策略 | 智能决策与恢复 |
| 对比分析 | 无 | 跨环境基线对比 |
| 报告输出 | 截图+日志 | 多维评分+趋势分析 |

#### 1.1.3 目标定位

本方案旨在构建一套**智能化、可扩展、自进化**的兼容性测试系统，实现以下目标：

1. **全面覆盖**：覆盖8大兼容性风险，测试场景覆盖率达到95%以上
2. **智能决策**：基于ReAct循环实现测试策略自适应
3. **语义验证**：不仅检查"元素存在"，更验证"用户能理解"
4. **量化评估**：输出可量化的兼容性评分，支持趋势追踪
5. **快速迭代**：支持CI/CD集成，缩短测试周期50%以上

### 1.2 核心设计理念

#### 1.2.1 为什么用Agent而不是传统脚本？

```
┌─────────────────────────────────────────────────────────────────┐
│                    传统脚本 vs Agent 核心差异                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   传统脚本                          Agent                        │
│   ─────────                         ─────                        │
│   IF-THEN固定逻辑          →        感知-理解-决策-执行循环        │
│   坐标硬编码                →        视觉定位自适应                │
│   断言=DOM匹配             →        断言=语义理解+结构验证        │
│   失败=重试N次             →        失败=分析原因+策略调整         │
│   单流程执行               →        探索性测试+回归验证            │
│                                                                  │
│   关键洞察：兼容性测试的本质不是"执行步骤"，而是"理解状态+决策行动"  │
│             这正是Agent擅长的领域                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 1.2.2 六层架构设计哲学

本方案采用六层架构设计，每层职责清晰、解耦：

1. **感知层**：环境的眼睛，采集屏幕、设备、网络等原始状态
2. **理解层**：环境的头脑，解析布局、理解语义、构建状态机
3. **决策层**：测试的大脑，基于场景引擎和策略做出测试决策
4. **执行层**：测试的手脚，操作设备、执行动作、注入环境
5. **验证层**：测试的裁判，多维验证、评分计算、结果判定
6. **记忆层**：知识的沉淀，存储基线、积累经验、优化策略

### 1.3 技术栈选型

#### 1.3.1 核心技术栈

| 组件 | 选型 | 选型理由 |
|------|------|----------|
| 设备控制 | uiautomator2 | Android官方推荐，支持WiFi连接，生态成熟 |
| 视觉感知 | VLM (GPT-4V/Claude/Gemini) | 语义级理解能力强，支持多模态输入 |
| Agent框架 | LangChain/LangGraph | ReAct模式原生支持，易于扩展 |
| 测试编排 | 自研ScenarioEngine | 适配IoT场景，支持参数化配置 |
| 报告生成 | Markdown + HTML | 格式开放，便于集成和归档 |

#### 1.3.2 依赖环境

```yaml
# 最小运行环境
Python: ">=3.10"
Android SDK: ">=31"
uiautomator2: ">=2.17.0"
minicap:  # 截图服务
minitouch:  # 触控服务
```

---

## 2. 兼容性测试风险体系

### 2.1 风险分类总览

基于对锁端交互App的深入分析，我们识别出8大兼容性核心风险，并进一步细化为完整的风险分类体系：

```
┌─────────────────────────────────────────────────────────────────┐
│                    兼容性测试风险全景图                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │   P0 阻断   │    │   P1 严重   │    │   P2 一般   │          │
│  │             │    │             │    │             │          │
│  │ 🔴 开锁失败 │    │ 🟠 连接异常 │    │ 🟡 UI错位   │          │
│  │ 🔴 安全漏洞 │    │ 🟠 权限弹窗 │    │ 🟡 字体截断 │          │
│  │ 🔴 崩溃死机 │    │ 🟠 响应超时 │    │ 🟡 动画异常 │          │
│  │             │    │             │    │             │          │
│  └─────────────┘    └─────────────┘    └─────────────┘          │
│                                                                  │
│  ┌───────────────────────────────────────────────────────┐      │
│  │                      风险领域                           │      │
│  ├───────────────────────────────────────────────────────┤      │
│  │ 📱 设备兼容  │ 🔗 连接兼容  │ 🎨 界面兼容  │ 🌐 网络兼容│      │
│  │  分辨率/DPI  │ 蓝牙/WiFi   │  颜色/字体   │  弱网/离线│      │
│  │  系统版本    │ 固件版本    │  深色模式    │  延迟抖动 │      │
│  │  厂商ROM     │ 设备数量    │  动画性能    │  状态同步 │      │
│  └───────────────────────────────────────────────────────┘      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 详细风险清单

#### 2.2.1 P0级别风险（阻断性问题）

| 风险ID | 风险名称 | 风险描述 | 检测方法 | 判定标准 |
|--------|----------|----------|----------|----------|
| R-P0-01 | 开锁功能失效 | 在特定环境下开锁按钮无法点击或响应 | 视觉检测开锁按钮可点击状态 + 实际点击测试 | 开锁操作必须在3秒内完成，否则P0 |
| R-P0-02 | 蓝牙连接崩溃 | 蓝牙连接过程中App崩溃或ANR | 监控连接流程的ANR/Crash日志 | 任何导致App崩溃的蓝牙操作均为P0 |
| R-P0-03 | 权限拒绝死锁 | 权限弹窗被错误处理导致功能不可用 | 模拟权限拒绝场景，检测恢复能力 | 权限相关功能不可出现不可恢复状态 |
| R-P0-04 | 离线状态误判 | App在离线状态下错误显示已连接 | 网络断开后检查连接状态UI | 状态显示必须与实际网络状态一致 |

#### 2.2.2 P1级别风险（严重问题）

| 风险ID | 风险名称 | 风险描述 | 检测方法 | 判定标准 |
|--------|----------|----------|----------|----------|
| R-P1-01 | 蓝牙连接不稳定 | 特定设备蓝牙连接成功率低于95% | 多次连接测试，统计成功率 | 连接成功率<95%为P1 |
| R-P1-02 | UI元素遮挡 | 关键按钮被状态栏/导航栏遮挡 | 截图分析元素可见区域 | 任何关键操作元素被遮挡为P1 |
| R-P1-03 | 系统弹窗打断 | 权限/广告弹窗打断核心流程 | 监控流程中断次数 | 核心流程被弹窗打断超过1次为P1 |
| R-P1-04 | 响应超时 | 操作响应时间超过预期阈值 | 记录每个操作的响应时间 | 响应时间>P0阈值的150%为P1 |

> **说明**：R-P1-05 密码/指纹输入异常 属于锁端物理操作范畴，不在App自动化测试范围内。App仅覆盖蓝牙开锁流程。

#### 2.2.3 P2级别风险（一般问题）

| 风险ID | 风险名称 | 风险描述 | 检测方法 | 判定标准 |
|--------|----------|----------|----------|----------|
| R-P2-01 | 界面元素错位 | 按钮/文字在不同DPI下位置偏移 | 与基线截图对比，检测偏移量 | 偏移量>5dp为P2 |
| R-P2-02 | 字体截断溢出 | 长文本在特定宽度下被截断或换行异常 | 检测文本渲染完整性 | 文字截断影响理解为P2 |
| R-P2-03 | 动画帧率异常 | 动画在高/低配设备上帧率异常 | 采集GPU渲染数据 | 帧率<25fps或>60fps持续超3秒为P2 |
| R-P2-04 | 深色模式色差 | 深色模式下颜色显示异常 | 与预期深色配色对比 | 明显色差影响可读性为P2 |
| R-P2-05 | 横竖屏布局异常 | 横竖屏切换后布局混乱 | 旋转设备检测布局完整性 | 旋转后布局可读性下降为P2 |

### 2.3 风险优先级矩阵

```
                    影响范围
         ┌──────────┬──────────┬──────────┐
         │   局部   │   重要   │   全局   │
    ┌────┼──────────┼──────────┼──────────┤
 高 │频繁│   P1     │   P0     │   P0     │
    ├────┼──────────┼──────────┼──────────┤
频  │一般│   P2     │   P1     │   P1     │
    ├────┼──────────┼──────────┼──────────┤
率  │偶发│   P2     │   P2     │   P1     │
    └────┴──────────┴──────────┴──────────┘
         │   低      │   中      │   高     │
                    严重程度
```

---

## 3. Agent架构设计

### 3.1 六层架构详解

#### 3.1.1 架构总览图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         兼容性测试Agent六层架构                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        📚 记忆层 (Memory Layer)                   │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │  基线数据   │  │  测试经验   │  │  策略库    │              │   │
│  │  │  Baseline   │  │  History    │  │  Strategies │              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    ↑                                    │
│                                    │ 存储/读取                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        🎯 验证层 (Verification Layer)              │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │   │
│  │  │ 结构验证   │  │ 规则验证   │  │ 语义验证   │  │对比验证 │ │   │
│  │  │ Structure   │  │  Rules     │  │ Semantic   │  │Compare  │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    ↑                                    │
│                                    │ 验证结果                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        🔧 执行层 (Execution Layer)                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │ UI操作    │  │ 环境注入   │  │ 锁端模拟   │              │   │
│  │  │ uiautomator│  │ Env Inject │  │ BLE Mock   │              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    ↑                                    │
│                                    │ 执行指令                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        🧠 决策层 (Decision Layer)                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │ 场景引擎   │  │ 探索策略   │  │ 异常处理   │              │   │
│  │  │ Scenario    │  │ Exploration │  │ Recovery   │              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    ↑                                    │
│                                    │ 决策指令                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        💡 理解层 (Understanding Layer)             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │ 布局解析   │  │ VLM视觉    │  │ 状态机     │              │   │
│  │  │ Layout Parse│  │ Understanding│  │ State Machine│              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                    ↑                                    │
│                                    │ 理解结果                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        👁️ 感知层 (Perception Layer)               │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │   │
│  │  │ 屏幕截图   │  │ XML树      │  │ 性能数据   │  │设备状态 │ │   │
│  │  │ Screenshot  │  │ Hierarchy  │  │ Perf Data  │  │Device   │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ══════════════════════════════════════════════════════════════════════ │
│  │                     🔄 ReAct 循环控制                                │   │
│  ══════════════════════════════════════════════════════════════════════ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 各层职责定义

##### 感知层 (Perception Layer)

**职责**：环境的眼睛，采集一切可观测的原始数据

**输入**：
- 当前屏幕状态
- 用户操作指令

**输出**：
- 屏幕截图（二进制）
- XML布局树（JSON）
- 设备性能数据（CPU/内存/帧率）
- 蓝牙/WiFi状态
- 系统通知状态

**关键算法**：
```python
class PerceptionLayer:
    """
    感知层：负责从设备采集所有可观测状态
    """
    
    def perceive(self) -> PerceptionResult:
        """
        执行一次完整的感知采集
        """
        return PerceptionResult(
            screenshot=self._capture_screen(),      # 高分辨率截图
            hierarchy=self._dump_hierarchy(),       # 完整XML树
            device_state=self._get_device_state(),  # 设备状态
            bluetooth_state=self._get_bluetooth(),  # 蓝牙状态
            wifi_state=self._get_wifi(),            # WiFi状态
            performance=self._collect_perf(),       # 性能指标
            timestamp=time.time()
        )
    
    def _dump_hierarchy(self) -> dict:
        """
        使用uiautomator2获取当前界面的XML布局树
        包含所有元素的bounds、text、resource-id、enabled等属性
        """
        xml_str = self.device.dump_hierarchy()
        return self._parse_xml(xml_str)
```

##### 理解层 (Understanding Layer)

**职责**：环境的头脑，将原始数据转化为结构化的语义理解

**输入**：
- 感知层输出的原始数据

**输出**：
- 界面布局结构（元素层级、位置关系）
- 视觉语义理解（元素功能、状态含义）
- 状态机当前状态
- 关键元素定位

**关键算法**：
```python
class UnderstandingLayer:
    """
    理解层：负责解析布局、理解语义、构建状态机
    """
    
    def understand(self, perception: PerceptionResult) -> UnderstandingResult:
        """
        执行一次完整的理解分析
        """
        # 1. 布局解析：提取关键元素及其位置关系
        layout = self._parse_layout(perception.hierarchy)
        
        # 2. VLM视觉理解：语义级分析
        visual = self._vlm_analyze(
            perception.screenshot,
            prompt=self._build_vlm_prompt(layout)
        )
        
        # 3. 状态机更新：根据当前界面推断状态
        state = self._update_state_machine(layout, visual)
        
        # 4. 关键元素定位：综合XML和VLM结果
        elements = self._locate_key_elements(layout, visual)
        
        return UnderstandingResult(
            layout=layout,
            visual=visual,
            state=state,
            elements=elements,
            confidence=visual.confidence
        )
    
    def _build_vlm_prompt(self, layout: Layout) -> str:
        """
        构建VLM分析提示词，引导模型关注关键信息
        """
        return f"""
        请分析这张锁端App截图：
        
        1. **界面状态识别**：当前界面是连接页面/开锁页面/设置页面/错误页面？
        2. **关键元素检测**：
           - 开锁按钮位置和状态（是否可点击、是否有加载动画）
           - 连接状态指示器（蓝牙图标、WiFi状态）
           - 任何异常提示或错误信息
        3. **用户可读性评估**：文字是否清晰可见？按钮是否容易被用户发现？
        4. **潜在问题识别**：是否有任何可能导致用户困惑的元素？
        
        请用JSON格式输出分析结果。
        """
```

##### 决策层 (Decision Layer)

**职责**：测试的大脑，决定下一步做什么

**输入**：
- 理解层的分析结果
- 当前测试场景定义
- 记忆层的历史经验

**输出**：
- 下一个操作指令（点击、滑动、等待等）
- 测试策略调整
- 异常处理决策

**关键算法**：
```python
class DecisionLayer:
    """
    决策层：基于ReAct模式实现智能决策
    """
    
    def decide(self, understanding: UnderstandingResult, 
               scenario: TestScenario) -> Decision:
        """
        ReAct循环的决策步骤
        Thought → Action → Observation → ...
        """
        
        # 1. 思考：分析当前状态与目标的差距
        thought = self._reason(understanding, scenario)
        
        # 2. 决定行动：根据思考结果选择动作
        if self._should_explore(thought):
            # 探索性测试：尝试发现新问题
            action = self._explore_action(understanding)
        elif self._should_verify(thought):
            # 验证性测试：检查预期元素
            action = self._verify_action(understanding, scenario)
        elif self._should_handle_exception(thought):
            # 异常处理：应对意外情况
            action = self._recover_action(understanding)
        else:
            # 正常流程：按场景步骤执行
            action = self._proceed_action(understanding, scenario)
        
        # 3. 更新策略：根据结果调整后续决策
        self._update_strategy(action, understanding)
        
        return Decision(
            thought=thought,
            action=action,
            confidence=self._calculate_confidence(thought)
        )
    
    def _reason(self, understanding: UnderstandingResult,
                scenario: TestScenario) -> str:
        """
        思考过程：分析当前状态
        """
        context = f"""
        当前状态：{understanding.state}
        场景进度：{scenario.current_step}/{scenario.total_steps}
        关键元素：{list(understanding.elements.keys())}
        检测到的问题：{understanding.visual.issues}
        
        目标状态：{scenario.target_state}
        
        请分析：当前状态是否正常？距离目标还有多远？是否遇到异常？
        """
        
        return self.llm.reason(context)
```

##### 执行层 (Execution Layer)

**职责**：测试的手脚，执行具体的操作

**输入**：
- 决策层的指令
- 目标元素定位

**输出**：
- 操作执行结果
- 执行后的状态变化

**关键算法**：
```python
class ExecutionLayer:
    """
    执行层：负责执行具体操作和环境控制
    """
    
    def execute(self, decision: Decision, 
                understanding: UnderstandingResult) -> ExecutionResult:
        """
        执行决策指定的动作
        """
        
        # 1. 准备执行环境
        self._prepare_environment(decision)
        
        # 2. 执行具体操作
        if decision.action.type == "click":
            result = self._execute_click(
                decision.action.target,
                understanding.elements
            )
        elif decision.action.type == "input":
            result = self._execute_input(
                decision.action.target,
                decision.action.value,
                understanding.elements
            )
        elif decision.action.type == "swipe":
            result = self._execute_swipe(decision.action)
        elif decision.action.type == "wait":
            result = self._execute_wait(decision.action.duration)
        elif decision.action.type == "environment":
            result = self._set_environment(decision.action.env_config)
        
        # 3. 记录执行日志
        self._log_execution(decision, result)
        
        return result
    
    def _execute_click(self, target: str, 
                       elements: Dict[str, Element]) -> ExecutionResult:
        """
        执行点击操作，支持多种定位方式
        """
        element = elements.get(target)
        
        if element is None:
            raise ElementNotFoundError(f"未找到目标元素: {target}")
        
        # 获取元素坐标（支持百分比和绝对坐标）
        x, y = element.get_click_point()
        
        # 使用uiautomator2执行点击
        self.device.click(x, y)
        
        return ExecutionResult(
            success=True,
            action=f"click({x}, {y})",
            element=element,
            timestamp=time.time()
        )
```

##### 验证层 (Verification Layer)

**职责**：测试的裁判，多维度验证测试结果

**输入**：
- 执行前后的感知数据
- 测试场景的验证规则
- 基线数据（跨环境对比用）

**输出**：
- 结构验证结果
- 规则验证结果
- 语义验证结果
- 对比验证结果
- 综合评分

**关键算法**：
```python
class VerificationLayer:
    """
    验证层：四维验证体系
    """
    
    def verify(self, context: VerificationContext) -> VerificationResult:
        """
        执行完整的四维验证
        """
        
        # 1. 结构验证：检查元素是否存在、位置是否正确
        structure_result = self._verify_structure(context)
        
        # 2. 规则验证：检查是否符合业务规则
        rules_result = self._verify_rules(context)
        
        # 3. 语义验证：使用VLM检查用户是否能理解
        semantic_result = self._verify_semantic(context)
        
        # 4. 对比验证：与基线数据对比
        compare_result = self._verify_compare(context)
        
        # 5. 综合评分
        score = self._calculate_score(
            structure_result,
            rules_result,
            semantic_result,
            compare_result
        )
        
        return VerificationResult(
            structure=structure_result,
            rules=rules_result,
            semantic=semantic_result,
            compare=compare_result,
            score=score,
            issues=self._collect_issues(
                structure_result,
                rules_result,
                semantic_result,
                compare_result
            )
        )
    
    def _verify_semantic(self, context: VerificationContext) -> SemanticResult:
        """
        语义验证：使用VLM判断用户是否能正确理解当前状态
        这是与传统自动化最大的区别
        """
        prompt = f"""
        请评估当前界面是否能让普通用户正确理解：
        
        **验证要点**：
        1. 连接状态是否清晰？（已连接/连接中/连接失败）
        2. 开锁按钮是否明显？（位置、大小、颜色是否突出）
        3. 是否有任何可能误导用户的信息？
        4. 文字是否清晰可读？（大小、颜色、对比度）
        
        **评分标准**：
        - 100分：用户体验完美，无任何理解障碍
        - 80分：有轻微问题但不影响使用
        - 60分：存在明显问题，可能导致用户困惑
        - <60分：存在严重问题，用户可能做出错误操作
        
        请给出分数和具体问题描述。
        """
        
        vlm_result = self.vlm.analyze(context.screenshot, prompt)
        
        return SemanticResult(
            score=vlm_result.score,
            issues=vlm_result.issues,
            suggestion=vlm_result.suggestion
        )
```

### 3.2 层间协作关系

```
┌─────────────────────────────────────────────────────────────────┐
│                      Agent主循环 (ReAct Loop)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────┐      ┌─────────┐      ┌─────────┐               │
│    │  感知   │ ──── │  理解   │ ──── │  决策   │               │
│    │         │      │         │      │         │               │
│    │ 采集    │      │ 解析    │      │ 思考    │               │
│    │ 屏幕    │      │ 布局    │      │ 选择    │               │
│    │ XML树   │ ──── │ VLM理解 │ ──── │ 行动    │               │
│    │ 性能    │      │ 状态机  │      │ 策略    │               │
│    └─────────┘      └─────────┘      └─────────┘               │
│         │                                    │                 │
│         │ 观察结果                             │ 决策指令         │
│         ▼                                    ▼                 │
│    ┌─────────────────────────────────────────────────────┐       │
│    │                       执行                         │       │
│    │   执行操作 → 环境注入 → 锁端模拟 → 状态变化        │       │
│    └─────────────────────────────────────────────────────┘       │
│                              │                                    │
│                              ▼                                    │
│    ┌─────────────────────────────────────────────────────┐       │
│    │                       验证                         │       │
│    │   结构验证 → 规则验证 → 语义验证 → 对比验证        │       │
│    └─────────────────────────────────────────────────────┘       │
│                              │                                    │
│                              ▼                                    │
│    ┌─────────────────────────────────────────────────────┐       │
│    │                       记忆                         │       │
│    │   更新基线 → 记录经验 → 优化策略 → 沉淀知识        │       │
│    └─────────────────────────────────────────────────────┘       │
│                              │                                    │
│                              │ 循环继续/终止                       │
│                              ▼                                    │
│                      [是否达到目标？]                              │
│                         ↙           ↘                              │
│                       是              否                          │
│                        ↓               ↓                          │
│                    [完成测试]     [返回感知继续]                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 与传统自动化框架对比

| 维度 | Appium/UiAutomator | Selenium | 本方案Agent |
|------|-------------------|----------|-------------|
| **架构模式** | 客户端-服务端 | 纯客户端 | 六层Agent |
| **定位方式** | 坐标/XPath/CSS | XPath/CSS | VLM视觉+XML混合 |
| **验证方式** | 断言=值匹配 | 断言=值匹配 | 四维验证+语义理解 |
| **异常处理** | try-catch重试 | try-catch重试 | 智能决策+策略调整 |
| **环境适配** | 脚本参数化 | 响应式设计 | 环境感知自动适配 |
| **测试策略** | 预设流程 | 预设流程 | 探索性+验证性混合 |
| **报告能力** | 截图+日志 | 截图+日志 | 多维评分+趋势分析 |
| **学习能力** | 无 | 无 | 积累经验优化策略 |
| **适用场景** | 功能测试 | Web测试 | 兼容性+语义理解 |

---

## 4. 核心模块详细设计

### 4.1 DeviceController 模块

**模块职责**：封装uiautomator2，提供设备控制、环境注入、状态感知等能力

#### 4.1.1 完整接口设计

```python
"""
DeviceController: 设备控制核心模块
封装uiautomator2，提供锁端交互App专用接口
"""

import uiautomator2 as u2
import time
import json
from typing import Dict, Any, Optional, Tuple, List
from dataclasses import dataclass, field
from enum import Enum
import logging

logger = logging.getLogger(__name__)


class DeviceState(Enum):
    """设备状态枚举"""
    CONNECTED = "connected"
    DISCONNECTED = "disconnected"
    UNAUTHORIZED = "unauthorized"
    OFFLINE = "offline"


class BluetoothState(Enum):
    """蓝牙状态枚举"""
    OFF = "off"
    ON = "on"
    SCANNING = "scanning"
    CONNECTED = "connected"
    DISCONNECTED = "disconnected"


@dataclass
class EnvironmentConfig:
    """环境配置：定义兼容性测试的各种环境参数"""
    # 显示配置
    resolution: Tuple[int, int] = None          # (width, height)
    dpi: int = None                             # DPI值
    font_scale: float = 1.0                     # 字体缩放 (1.0=正常, 1.5=大)
    
    # 外观配置
    dark_mode: bool = False                     # 深色模式
    night_mode: bool = False                    # 护眼模式
    
    # 网络配置
    network_type: str = "wifi"                 # wifi/4g/3g/2g/offline
    network_quality: str = "good"               # good/poor/off
    
    # 蓝牙配置
    bluetooth_enabled: bool = True
    bluetooth_device: str = None                # 目标蓝牙设备地址
    
    # 系统配置
    auto_rotate: bool = True                   # 自动旋转
    screen_orientation: str = "portrait"        # portrait/landscape
    
    def to_dict(self) -> Dict[str, Any]:
        return {k: v for k, v in self.__dict__.items() if v is not None}


@dataclass
class DeviceInfo:
    """设备信息：采集设备环境参数"""
    device_id: str
    model: str
    manufacturer: str
    android_version: str
    sdk_version: int
    resolution: Tuple[int, int]
    dpi: int
    cpu_abi: str
    memory_total: int
    memory_available: int


class DeviceController:
    """
    DeviceController - 设备控制核心类
    
    职责：
    1. 设备连接与管理
    2. 屏幕截图与布局dump
    3. UI操作执行
    4. 环境状态注入
    5. 性能数据采集
    6. 锁端交互专用接口
    
    设计理念：
    - 所有操作统一入口，统一日志
    - 环境配置与操作分离
    - 支持设备状态快照与恢复
    """
    
    def __init__(self, device_id: str = None, host: str = None):
        """
        初始化设备控制器
        
        Args:
            device_id: 设备序列号（通过adb devices获取）
            host: 设备IP（WiFi连接时使用）
        """
        self.device_id = device_id or self._auto_detect_device()
        self.host = host
        
        # 初始化uiautomator2
        if host:
            self.device = u2.connect_wifi(host)
        else:
            self.device = u2.connect(self.device_id)
        
        # 状态管理
        self._current_env = EnvironmentConfig()
        self._env_snapshots: List[EnvironmentConfig] = []
        
        # 性能采样器
        self._perf_sampler = PerformanceSampler(self.device)
        
        # 锁端专用状态
        self._lock_connection_state = "disconnected"
        
        logger.info(f"DeviceController initialized: {self.device_id}")
    
    # ==================== 基础能力 ====================
    
    def screenshot(self, quality: int = 95) -> bytes:
        """
        获取屏幕截图
        
        Args:
            quality: 图片质量 (1-100)
            
        Returns:
            PNG格式的截图二进制数据
        """
        return self.device.screenshot(quality=quality)
    
    def dump_hierarchy(self) -> str:
        """
        获取当前界面的XML布局树
        
        Returns:
            XML格式的布局树字符串
        """
        return self.device.dump_hierarchy()
    
    def press(self, key: str):
        """
        按键操作
        
        Args:
            key: 按键名称 (home/back/menu/power/volume_up/volume_down)
        """
        self.device.press(key)
    
    def click(self, x: int, y: int):
        """
        点击指定坐标
        
        Args:
            x: X坐标
            y: Y坐标
        """
        self.device.click(x, y)
    
    def swipe(self, start_x: int, start_y: int, 
              end_x: int, end_y: int, duration: float = 0.5):
        """
        滑动操作
        
        Args:
            start_x, start_y: 起始坐标
            end_x, end_y: 结束坐标
            duration: 持续时间（秒）
        """
        self.device.swipe(start_x, start_y, end_x, end_y, duration)
    
    def input_text(self, text: str):
        """
        输入文本（当前焦点元素）
        
        Args:
            text: 要输入的文本
        """
        self.device.set_fastinput_ime(True)
        self.device.send_keys(text)
        self.device.set_fastinput_ime(False)
    
    # ==================== 环境注入 ====================
    
    def set_environment(self, config: EnvironmentConfig) -> bool:
        """
        设置环境配置（分辨率/DPI/深色模式等）
        
        设计说明：
        环境注入是兼容性测试的核心能力。通过动态调整环境参数，
        模拟不同设备/配置下的用户体验。
        
        Args:
            config: 环境配置对象
            
        Returns:
            设置是否成功
        """
        self._current_env = config
        
        # 保存当前环境快照
        self._env_snapshots.append(self._save_current_env())
        
        success = True
        
        # 1. 设置深色模式
        if config.dark_mode is not None:
            success &= self._set_dark_mode(config.dark_mode)
        
        # 2. 设置字体大小
        if config.font_scale is not None:
            success &= self._set_font_scale(config.font_scale)
        
        # 3. 设置网络状态
        if config.network_type is not None:
            success &= self._set_network(config.network_type, config.network_quality)
        
        # 4. 设置蓝牙状态
        if config.bluetooth_enabled is not None:
            success &= self._set_bluetooth(config.bluetooth_enabled)
        
        # 5. 设置屏幕方向
        if config.screen_orientation is not None:
            success &= self._set_orientation(config.screen_orientation)
        
        # 等待环境生效
        time.sleep(1)
        
        return success
    
    def _set_dark_mode(self, enabled: bool) -> bool:
        """
        设置深色模式
        """
        try:
            cmd = f"settings put secure ui_night_mode {'2' if enabled else '1'}"
            self.device.shell(cmd)
            logger.info(f"Dark mode set to: {enabled}")
            return True
        except Exception as e:
            logger.error(f"Failed to set dark mode: {e}")
            return False
    
    def _set_font_scale(self, scale: float) -> bool:
        """
        设置字体缩放
        
        Args:
            scale: 缩放比例 (0.85=小, 1.0=正常, 1.15=大, 1.3=特大)
        """
        try:
            cmd = f"settings put system font_scale {scale}"
            self.device.shell(cmd)
            logger.info(f"Font scale set to: {scale}")
            return True
        except Exception as e:
            logger.error(f"Failed to set font scale: {e}")
            return False
    
    def _set_network(self, network_type: str, quality: str = "good") -> bool:
        """
        设置网络状态
        
        Args:
            network_type: wifi/4g/3g/2g/offline
            quality: good/poor/off
        """
        try:
            if network_type == "offline":
                # 飞行模式
                self.device.shell("settings put global airplane_mode_on 1")
                self.device.shell("am broadcast -a android.intent.action.AIRPLANE_MODE --ez state true")
            else:
                # 关闭飞行模式
                self.device.shell("settings put global airplane_mode_on 0")
                self.device.shell("am broadcast -a android.intent.action.AIRPLANE_MODE --ez state false")
            
            logger.info(f"Network set to: {network_type}")
            return True
        except Exception as e:
            logger.error(f"Failed to set network: {e}")
            return False
    
    def _set_bluetooth(self, enabled: bool) -> bool:
        """
        设置蓝牙开关
        """
        try:
            cmd = f"service call bluetooth_manager 6 {'i32 1' if enabled else 'i32 0'}"
            self.device.shell(cmd)
            logger.info(f"Bluetooth set to: {enabled}")
            return True
        except Exception as e:
            logger.error(f"Failed to set bluetooth: {e}")
            return False
    
    def _set_orientation(self, orientation: str) -> bool:
        """
        设置屏幕方向
        """
        try:
            if orientation == "landscape":
                self.device.shell("settings put system user_rotation 1")
            else:
                self.device.shell("settings put system user_rotation 0")
            logger.info(f"Orientation set to: {orientation}")
            return True
        except Exception as e:
            logger.error(f"Failed to set orientation: {e}")
            return False
    
    def restore_environment(self) -> bool:
        """
        恢复到上一个环境快照
        """
        if not self._env_snapshots:
            logger.warning("No environment snapshot to restore")
            return False
        
        last_env = self._env_snapshots.pop()
        return self.set_environment(last_env)
    
    # ==================== 状态感知 ====================
    
    def get_device_info(self) -> DeviceInfo:
        """
        获取设备信息
        """
        # 使用uiautomator2的device_info
        info = self.device.info
        
        return DeviceInfo(
            device_id=self.device_id,
            model=info.get('model', ''),
            manufacturer=info.get('manufacturer', ''),
            android_version=info.get('version', ''),
            sdk_version=info.get('sdkInt', 0),
            resolution=(info['screenWidth'], info['screenHeight']),
            dpi=info.get('displayDpi', 0),
            cpu_abi=self.device.shell("getprop ro.product.cpu.abi").strip(),
            memory_total=self._get_memory_info()['total'],
            memory_available=self._get_memory_info()['available']
        )
    
    def _get_memory_info(self) -> Dict[str, int]:
        """
        获取内存信息
        """
        output = self.device.shell("dumpsys meminfo").split('\n')
        # 简化解析，实际使用时需要更完善的解析逻辑
        return {'total': 0, 'available': 0}
    
    def get_bluetooth_state(self) -> BluetoothState:
        """
        获取蓝牙状态
        
        锁端App核心关注点：蓝牙连接状态直接影响开锁功能
        """
        try:
            output = self.device.shell("dumpsys bluetooth_manager").lower()
            
            if "state=connected" in output:
                return BluetoothState.CONNECTED
            elif "state=scanning" in output:
                return BluetoothState.SCANNING
            elif "enabled=true" in output or "enabled= true" in output:
                return BluetoothState.ON
            else:
                return BluetoothState.OFF
        except:
            return BluetoothState.OFF
    
    def get_network_state(self) -> str:
        """
        获取网络连接状态
        """
        try:
            wifi = self.device.shell("dumpsys connectivity").lower()
            if "wifi: connected" in wifi or "wifi: state: connected" in wifi:
                return "wifi_connected"
            elif "mobile: connected" in wifi:
                return "mobile_connected"
            else:
                return "offline"
        except:
            return "unknown"
    
    # ==================== 锁端专用接口 ====================
    
    def get_lock_connection_status(self) -> Dict[str, Any]:
        """
        获取锁端连接状态
        
        专门针对锁端App设计，获取更详细的连接状态信息
        """
        # 尝试通过AccessibilityService或App内部状态获取
        # 这里提供一个框架，实际需要根据具体App实现
        
        # 1. 检查App进程是否存在
        app_running = self._is_app_running("com.example.lockapp")
        
        # 2. 获取蓝牙扫描结果
        bluetooth_devices = self._scan_bluetooth_devices()
        
        # 3. 检查BLE连接状态
        ble_connected = self._check_ble_connection()
        
        return {
            "app_running": app_running,
            "bluetooth_enabled": self.get_bluetooth_state() != BluetoothState.OFF,
            "bluetooth_state": self.get_bluetooth_state().value,
            "nearby_devices": bluetooth_devices,
            "ble_connected": ble_connected,
            "last_sync_time": self._get_last_sync_time(),
            "battery_level": self._get_lock_battery()
        }
    
    def _is_app_running(self, package: str) -> bool:
        """检查App是否运行"""
        try:
            output = self.device.shell(f"dumpsys activity activities | grep {package}")
            return package in output
        except:
            return False
    
    def _scan_bluetooth_devices(self) -> List[Dict[str, Any]]:
        """扫描附近的蓝牙设备"""
        # 实现蓝牙扫描逻辑
        return []
    
    def _check_ble_connection(self) -> bool:
        """检查BLE连接状态"""
        # 实现BLE连接检查逻辑
        return False
    
    def _get_last_sync_time(self) -> Optional[int]:
        """获取最后同步时间戳"""
        return None
    
    def _get_lock_battery(self) -> Optional[int]:
        """获取锁的电量"""
        return None
    
    # ==================== 性能采集 ====================
    
    def collect_performance(self) -> Dict[str, Any]:
        """
        采集性能数据
        """
        return self._perf_sampler.sample()
    
    def start_performance_monitoring(self, interval: float = 1.0):
        """
        开始性能监控
        """
        self._perf_sampler.start(interval)
    
    def stop_performance_monitoring(self):
        """
        停止性能监控
        """
        self._perf_sampler.stop()
    
    # ==================== 工具方法 ====================
    
    def _save_current_env(self) -> EnvironmentConfig:
        """
        保存当前环境状态
        """
        return EnvironmentConfig(
            resolution=self.get_device_info().resolution,
            dpi=self.get_device_info().dpi,
            bluetooth_enabled=self.get_bluetooth_state() != BluetoothState.OFF,
            network_type=self.get_network_state()
        )
    
    @staticmethod
    def _auto_detect_device() -> str:
        """
        自动检测连接的设备
        """
        import subprocess
        result = subprocess.run(['adb', 'devices'], capture_output=True, text=True)
        devices = [line.split('\t')[0] for line in result.stdout.strip().split('\n')[1:] if line]
        if devices:
            return devices[0]
        raise RuntimeError("No device found. Please connect a device or specify device_id")


class PerformanceSampler:
    """
    性能采样器：采集CPU/内存/帧率等性能数据
    """
    
    def __init__(self, device):
        self.device = device
        self._running = False
        self._thread = None
        self._samples = []
    
    def sample(self) -> Dict[str, Any]:
        """
        执行一次性能采样
        """
        return {
            "cpu_usage": self._get_cpu_usage(),
            "memory_info": self._get_memory(),
            "fps": self._get_fps(),
            "timestamp": time.time()
        }
    
    def _get_cpu_usage(self) -> float:
        """获取CPU使用率"""
        try:
            output = self.device.shell("top -n 1 -d 1")
            # 简化解析
            return 0.0
        except:
            return 0.0
    
    def _get_memory(self) -> Dict[str, int]:
        """获取内存使用情况"""
        try:
            output = self.device.shell("dumpsys meminfo")
            # 简化解析
            return {"total": 0, "used": 0, "free": 0}
        except:
            return {}
    
    def _get_fps(self) -> float:
        """获取当前帧率"""
        try:
            output = self.device.shell("dumpsys gfxinfo")
            # 简化解析，实际需要解析FrameStats
            return 60.0
        except:
            return 0.0
    
    def start(self, interval: float = 1.0):
        """开始采样"""
        import threading
        
        def _sample_loop():
            while self._running:
                self._samples.append(self.sample())
                time.sleep(interval)
        
        self._running = True
        self._thread = threading.Thread(target=_sample_loop)
        self._thread.start()
    
    def stop(self):
        """停止采样"""
        self._running = False
        if self._thread:
            self._thread.join()
    
    def get_samples(self) -> List[Dict[str, Any]]:
        """获取所有采样数据"""
        return self._samples.copy()
```

### 4.2 VisualPerception 模块

**模块职责**：封装VLM视觉感知能力，提供屏幕分析、跨环境对比、语义理解等能力

#### 4.2.1 完整接口设计

```python
"""
VisualPerception: VLM视觉感知模块
提供屏幕分析、跨环境对比、语义理解等能力
"""

import base64
import json
from typing import Dict, Any, List, Optional, Tuple
from dataclasses import dataclass, field
from enum import Enum
from io import BytesIO
from PIL import Image
import logging

logger = logging.getLogger(__name__)


class PerceptionLevel(Enum):
    """感知级别"""
    BASIC = "basic"          # 仅结构分析
    STANDARD = "standard"    # 标准感知
    DEEP = "deep"            # 深度语义理解


@dataclass
class ElementInfo:
    """元素信息"""
    type: str                 # 元素类型: button/text/image/icon
    text: str = None          # 文字内容
    bounds: List[int] = None  # 边界坐标 [x, y, width, height]
    clickable: bool = False
    visible: bool = True
    enabled: bool = True
    confidence: float = 1.0  # 识别置信度
    
    # VLM额外识别
    semantic: str = None      # 语义描述
    state: str = None         # 状态: normal/loading/disabled/error


@dataclass
class ScreenAnalysis:
    """屏幕分析结果"""
    elements: List[ElementInfo]  # 识别到的元素
    layout_type: str             # 页面类型: home/lock/settings/error
    status_indicators: Dict[str, Any]  # 状态指示器
    issues: List[str]            # 发现的问题
    readability_score: float     # 可读性评分
    actionability_score: float   # 可操作性评分
    overall_score: float          # 综合评分
    raw_response: Dict = None    # 原始VLM响应


@dataclass
class ComparisonResult:
    """对比结果"""
    similarity: float            # 相似度 (0-1)
    differences: List[Dict]      # 差异列表
    critical_differences: List[Dict]  # 关键差异（可能影响功能）
    minor_differences: List[Dict]      # 次要差异（仅视觉）
    visual_score: float          # 视觉一致性评分
    

class VisualPerception:
    """
    VisualPerception - VLM视觉感知模块
    
    核心能力：
    1. 屏幕截图分析：识别元素、理解语义
    2. 跨环境对比：检测不同环境下的视觉差异
    3. 语义验证：判断用户是否能正确理解
    4. 问题检测：自动发现UI异常
    
    设计理念：
    - VLM Prompt工程是关键
    - 结构和语义双重验证
    - 支持增量对比（与基线对比）
    """
    
    def __init__(self, api_key: str = None, model: str = "gpt-4o"):
        """
        初始化视觉感知模块
        
        Args:
            api_key: VLM API密钥
            model: 使用的模型 (gpt-4o/claude-3-opus/gemini-pro)
        """
        self.model = model
        self.api_key = api_key
        
        # 初始化VLM客户端（根据实际使用的模型选择）
        # self.vlm_client = OpenAIClient(api_key) / ClaudeClient() / GeminiClient()
        
        # 缓存管理
        self._analysis_cache: Dict[str, ScreenAnalysis] = {}
        self._baseline_cache: Dict[str, Image] = {}
        
        logger.info(f"VisualPerception initialized with model: {model}")
    
    # ==================== 核心分析能力 ====================
    
    def analyze_screen(self, 
                       screenshot: bytes,
                       context: Dict[str, Any] = None,
                       level: PerceptionLevel = PerceptionLevel.STANDARD) -> ScreenAnalysis:
        """
        分析屏幕截图
        
        Args:
            screenshot: 屏幕截图二进制数据
            context: 上下文信息（当前场景、前置条件等）
            level: 分析级别
            
        Returns:
            ScreenAnalysis: 屏幕分析结果
        """
        # 检查缓存
        cache_key = self._compute_cache_key(screenshot)
        if cache_key in self._analysis_cache:
            logger.debug("Using cached analysis result")
            return self._analysis_cache[cache_key]
        
        # 构建Prompt
        prompt = self._build_analysis_prompt(level, context)
        
        # 调用VLM分析
        response = self._call_vlm(screenshot, prompt)
        
        # 解析结果
        analysis = self._parse_analysis_response(response, screenshot)
        
        # 缓存结果
        self._analysis_cache[cache_key] = analysis
        
        return analysis
    
    def _build_analysis_prompt(self, 
                               level: PerceptionLevel,
                               context: Dict[str, Any] = None) -> str:
        """
        构建VLM分析Prompt
        
        Prompt工程是VLM能力的关键：
        - 明确分析目标
        - 结构化输出要求
        - 提供参考框架
        """
        
        base_prompt = """
        你是一位专业的移动App测试工程师，负责分析锁端交互App的界面。
        
        请仔细分析这张截图，从以下维度进行全面评估：
        """
        
        if level == PerceptionLevel.BASIC:
            prompt = base_prompt + """
            
            **分析维度（基础）**：
            1. 页面类型识别：当前是什么页面？（连接页/开锁页/设置页/错误页）
            2. 关键元素定位：开锁按钮、连接状态、返回按钮等
            3. 元素可见性：所有关键元素是否可见？
            
            **输出格式**（JSON）：
            {
                "layout_type": "页面类型",
                "key_elements": [
                    {"type": "button", "name": "开锁按钮", "position": "center-bottom", "visible": true}
                ],
                "issues": []
            }
            """
        
        elif level == PerceptionLevel.STANDARD:
            prompt = base_prompt + """
            
            **分析维度（标准）**：
            1. **页面类型识别**：当前是什么页面？
            2. **关键元素检测**：
               - 开锁按钮：位置、大小、是否可点击、是否有加载状态
               - 连接状态：蓝牙图标、WiFi状态、信号强度
               - 导航元素：返回、菜单等
            3. **状态识别**：是否有加载中/错误/成功等状态？
            4. **可读性评估**：
               - 文字是否清晰可读
               - 颜色对比度是否足够
               - 元素间距是否合理
            5. **问题识别**：是否有任何可能导致用户困惑的问题？
            
            **输出格式**（JSON）：
            {
                "layout_type": "页面类型",
                "elements": [
                    {
                        "type": "button",
                        "semantic": "开锁按钮",
                        "bounds": [x, y, width, height],
                        "state": "normal",
                        "clickable": true
                    }
                ],
                "status_indicators": {
                    "bluetooth": "connected",
                    "wifi": "good"
                },
                "readability_score": 95,
                "actionability_score": 90,
                "issues": []
            }
            """
        
        else:  # DEEP
            prompt = base_prompt + """
            
            **分析维度（深度）**：
            1. **页面语义理解**：
               - 当前页面想传达什么信息？
               - 用户的第一注意力应该在哪里？
               - 整体用户体验是否符合预期？
            
            2. **用户旅程分析**：
               - 如果用户想开锁，他们能找到按钮吗？
               - 如果连接失败，用户知道下一步该做什么吗？
               - 是否有任何可能让用户迷失的设计？
            
            3. **安全性评估**：
               - 开锁按钮是否足够醒目，不会被误触？
               - 是否有足够的确认机制防止误操作？
               - 错误提示是否清晰，不会让用户困惑？
            
            4. **跨场景一致性**：
               - 与其他页面相比，风格是否一致？
               - 关键操作的位置是否符合用户预期？
            
            5. **潜在问题**：
               - 列出所有可能影响用户体验的问题
               - 评估每个问题的严重程度
               - 提供修复建议
            
            **输出格式**（JSON）：
            {
                "page_semantic": "页面语义描述",
                "user_journey": {
                    "primary_action": "主要操作",
                    "findability": "可发现性评分",
                    "cognitive_load": "认知负担评估"
                },
                "security_assessment": {
                    "unlock_button_prominence": "醒目程度",
                    "confirmation_mechanism": "确认机制",
                    "error_clarity": "错误提示清晰度"
                },
                "issues": [
                    {
                        "severity": "critical/major/minor",
                        "description": "问题描述",
                        "location": "问题位置",
                        "suggestion": "修复建议"
                    }
                ],
                "overall_score": 85,
                "recommendations": ["建议1", "建议2"]
            }
            """
        
        # 添加上下文
        if context:
            prompt += f"""
            
            **当前测试上下文**：
            - 测试场景：{context.get('scenario', 'unknown')}
            - 测试环境：{context.get('environment', 'unknown')}
            - 前置条件：{context.get('precondition', 'none')}
            - 预期结果：{context.get('expected', 'none')}
            """
        
        return prompt
    
    def _call_vlm(self, screenshot: bytes, prompt: str) -> Dict:
        """
        调用VLM API
        
        实现说明：
        - 支持多种VLM后端
        - 自动处理图片编码
        - 实现重试和降级策略
        """
        # 将图片转为base64
        img_base64 = base64.b64encode(screenshot).decode('utf-8')
        
        # 根据model选择客户端
        if self.model.startswith("gpt-4"):
            return self._call_openai_vision(img_base64, prompt)
        elif self.model.startswith("claude"):
            return self._call_claude_vision(img_base64, prompt)
        elif self.model.startswith("gemini"):
            return self._call_gemini_vision(img_base64, prompt)
        else:
            raise ValueError(f"Unsupported model: {self.model}")
    
    def _call_openai_vision(self, img_base64: str, prompt: str) -> Dict:
        """调用OpenAI Vision API"""
        # 实现OpenAI API调用
        # response = openai.ChatCompletion.create(
        #     model="gpt-4o",
        #     messages=[{
        #         "role": "user",
        #         "content": [
        #             {"type": "text", "text": prompt},
        #             {"type": "image_url", "image_url": {"data": f"data:image/png;base64,{img_base64}"}}
        #         ]
        #     }]
        # )
        # return json.loads(response.choices[0].message.content)
        raise NotImplementedError("OpenAI API调用需要实际实现")
    
    def _call_claude_vision(self, img_base64: str, prompt: str) -> Dict:
        """调用Claude Vision API"""
        raise NotImplementedError("Claude API调用需要实际实现")
    
    def _call_gemini_vision(self, img_base64: str, prompt: str) -> Dict:
        """调用Gemini Vision API"""
        raise NotImplementedError("Gemini API调用需要实际实现")
    
    def _parse_analysis_response(self, response: Dict, screenshot: bytes) -> ScreenAnalysis:
        """解析VLM响应"""
        # 实现响应解析逻辑
        return ScreenAnalysis(
            elements=[],
            layout_type=response.get("layout_type", "unknown"),
            status_indicators=response.get("status_indicators", {}),
            issues=response.get("issues", []),
            readability_score=response.get("readability_score", 0),
            actionability_score=response.get("actionability_score", 0),
            overall_score=response.get("overall_score", 0),
            raw_response=response
        )
    
    # ==================== 跨环境对比能力 ====================
    
    def compare_screens(self,
                        baseline: bytes,
                        current: bytes,
                        context: Dict[str, Any] = None) -> ComparisonResult:
        """
        跨环境屏幕对比
        
        核心能力：检测同一功能在不同环境下的视觉差异
        
        Args:
            baseline: 基线环境截图
            current: 当前环境截图
            context: 对比上下文
            
        Returns:
            ComparisonResult: 对比结果
        """
        # 分析两张截图
        baseline_analysis = self.analyze_screen(baseline, context)
        current_analysis = self.analyze_screen(current, context)
        
        # 构建对比Prompt
        prompt = self._build_comparison_prompt(
            baseline, current, 
            baseline_analysis, 
            current_analysis
        )
        
        # 调用VLM对比
        response = self._call_vlm_for_comparison(baseline, current, prompt)
        
        # 解析结果
        return self._parse_comparison_response(response)
    
    def _build_comparison_prompt(self,
                                 baseline: bytes,
                                 current: bytes,
                                 baseline_analysis: ScreenAnalysis,
                                 current_analysis: ScreenAnalysis) -> str:
        """
        构建对比Prompt
        
        对比分析的关键：
        - 识别有意义的差异
        - 区分功能性差异和装饰性差异
        - 评估差异对用户体验的影响
        """
        return f"""
        请对比两张锁端App截图，分析它们之间的差异。
        
        **基线环境截图**：{baseline_analysis.layout_type}
        - 关键元素：{[e.semantic for e in baseline_analysis.elements]}
        - 状态：{baseline_analysis.status_indicators}
        
        **当前环境截图**：{current_analysis.layout_type}
        - 关键元素：{[e.semantic for e in current_analysis.elements]}
        - 状态：{current_analysis.status_indicators}
        
        **对比维度**：
        1. **功能一致性**：关键功能元素是否都存在？
        2. **位置一致性**：相同元素的位置是否相近？（允许合理偏移）
        3. **状态一致性**：相同元素的状态是否一致？
        4. **可读性一致性**：文字可读性是否保持？
        
        **差异分类**：
        - 关键差异：可能影响用户使用功能的差异（如按钮被遮挡）
        - 次要差异：仅视觉上的差异，不影响功能（如颜色细微变化）
        
        **输出格式**（JSON）：
        {{
            "similarity_score": 0.85,
            "differences": [
                {{
                    "type": "position/size/color/text/state",
                    "element": "元素名称",
                    "baseline": "基线描述",
                    "current": "当前描述",
                    "severity": "critical/major/minor",
                    "impact": "对用户的影响"
                }}
            ],
            "functional_impact": "功能影响评估",
            "visual_score": 90
        }}
        """
    
    def _call_vlm_for_comparison(self, baseline: bytes, current: bytes, prompt: str) -> Dict:
        """调用VLM进行对比分析"""
        # 类似于_call_vlm，但需要同时发送两张图片
        raise NotImplementedError("对比API调用需要实际实现")
    
    def _parse_comparison_response(self, response: Dict) -> ComparisonResult:
        """解析对比响应"""
        return ComparisonResult(
            similarity=response.get("similarity_score", 0),
            differences=response.get("differences", []),
            critical_differences=[d for d in response.get("differences", []) 
                                   if d.get("severity") == "critical"],
            minor_differences=[d for d in response.get("differences", []) 
                               if d.get("severity") == "minor"],
            visual_score=response.get("visual_score", 0)
        )
    
    # ==================== 基线管理 ====================
    
    def save_baseline(self, name: str, screenshot: bytes, metadata: Dict = None):
        """
        保存基线截图
        
        Args:
            name: 基线名称（如 "标准环境-开锁页"）
            screenshot: 基线截图
            metadata: 元数据（环境配置等）
        """
        self._baseline_cache[name] = screenshot
        logger.info(f"Baseline saved: {name}")
        
        # 同时保存到持久化存储
        self._persist_baseline(name, screenshot, metadata)
    
    def get_baseline(self, name: str) -> Optional[bytes]:
        """
        获取基线截图
        """
        if name in self._baseline_cache:
            return self._baseline_cache[name]
        
        # 尝试从持久化存储加载
        return self._load_baseline(name)
    
    def list_baselines(self) -> List[str]:
        """列出所有基线"""
        return list(self._baseline_cache.keys())
    
    def _persist_baseline(self, name: str, screenshot: bytes, metadata: Dict):
        """持久化基线到存储"""
        # 实现持久化逻辑（文件系统/数据库等）
        pass
    
    def _load_baseline(self, name: str) -> Optional[bytes]:
        """从存储加载基线"""
        # 实现加载逻辑
        return None
    
    def _compute_cache_key(self, screenshot: bytes) -> str:
        """计算截图的缓存key"""
        import hashlib
        return hashlib.md5(screenshot[:1000]).hexdigest()
    
    # ==================== 辅助能力 ====================
    
    def detect_element(self, screenshot: bytes, element_description: str) -> Optional[ElementInfo]:
        """
        检测特定元素
        
        Args:
            screenshot: 截图
            element_description: 元素描述（如"开锁按钮"、"蓝牙图标"）
        """
        prompt = f"""
        请在截图中找到"{element_description}"，返回其位置和状态。
        
        输出格式（JSON）：
        {{
            "found": true/false,
            "bounds": [x, y, width, height],
            "state": "normal/loading/disabled",
            "confidence": 0.95
        }}
        """
        
        response = self._call_vlm(screenshot, prompt)
        # 解析响应...
        return None
    
    def validate_text_readability(self, screenshot: bytes) -> Dict:
        """
        验证文字可读性
        
        检测：
        - 文字大小是否足够
        - 颜色对比度是否足够
        - 是否存在文字截断
        """
        prompt = """
        请评估截图中所有文字的可读性：
        
        1. 每种文字的大小是否适合阅读？
        2. 文字与背景的颜色对比度是否足够？
        3. 是否有文字被截断或换行异常？
        4. 深色模式下文字是否仍然清晰？
        
        输出格式（JSON）：
        {{
            "overall_readable": true,
            "issues": [
                {{"location": "位置", "problem": "问题", "severity": "high/medium/low"}}
            ],
            "recommendations": ["建议"]
        }}
        """
        
        response = self._call_vlm(screenshot, prompt)
        return response
```

### 4.3 TestScenario 模块

**模块职责**：定义和管理测试场景，提供参数化场景模板

#### 4.3.1 场景定义与实现

```python
"""
TestScenario: 测试场景模块
定义和管理锁端App的测试场景
"""

from typing import Dict, Any, List, Optional, Callable
from dataclasses import dataclass, field
from enum import Enum
import logging

logger = logging.getLogger(__name__)


class ScenarioType(Enum):
    """场景类型
    
    注意：PASSWORD类型（密码面板输入、指纹解锁等）属于锁端物理操作，
    不在App自动化测试范围内。本方案仅覆盖蓝牙开锁流程相关的场景。
    """
    CONNECTION = "connection"           # 连接相关
    UNLOCK = "unlock"                   # 蓝牙开锁流程
    # PASSWORD = "password"             # 密码输入 - 已禁用（锁端物理操作，不属于App自动化范围）
    OFFLINE = "offline"                # 离线状态
    ORIENTATION = "orientation"        # 横竖屏切换
    FIRMWARE = "firmware"              # 固件升级
    MULTI_LOCK = "multi_lock"          # 多锁管理
    SHARE = "share"                    # 分享权限
    NOTIFICATION = "notification"     # 通知适配
    GESTURE = "gesture"                # 手势导航


class ScenarioStatus(Enum):
    """场景执行状态"""
    PENDING = "pending"
    RUNNING = "running"
    PASSED = "passed"
    FAILED = "failed"
    BLOCKED = "blocked"
    SKIPPED = "skipped"


@dataclass
class ScenarioStep:
    """场景步骤"""
    step_id: str
    description: str                    # 步骤描述
    action: str                         # 操作类型: click/input/swipe/wait/verify
    target: str = None                  # 目标元素
    params: Dict[str, Any] = None       # 操作参数
    expected: Dict[str, Any] = None     # 预期结果
    timeout: float = 10.0              # 超时时间
    retry: int = 3                      # 重试次数
    
    def to_dict(self) -> Dict:
        return {
            "step_id": self.step_id,
            "description": self.description,
            "action": self.action,
            "target": self.target,
            "params": self.params,
            "expected": self.expected,
            "timeout": self.timeout,
            "retry": self.retry
        }


@dataclass
class ScenarioConfig:
    """场景配置"""
    name: str
    type: ScenarioType
    description: str
    preconditions: List[str] = None    # 前置条件
    steps: List[ScenarioStep] = None    # 执行步骤
    verification_rules: List[Dict] = None  # 验证规则
    parameters: Dict[str, Any] = None   # 场景参数（用于参数化）
    environment: Dict[str, Any] = None  # 所需环境配置
    tags: List[str] = None              # 标签（用于分类筛选）
    priority: str = "P1"               # 优先级 P0/P1/P2
    
    def to_dict(self) -> Dict:
        return {
            "name": self.name,
            "type": self.type.value,
            "description": self.description,
            "preconditions": self.preconditions or [],
            "steps": [s.to_dict() for s in (self.steps or [])],
            "verification_rules": self.verification_rules or [],
            "parameters": self.parameters or {},
            "environment": self.environment or {},
            "tags": self.tags or [],
            "priority": self.priority
        }


@dataclass
class ScenarioResult:
    """场景执行结果"""
    scenario_id: str
    status: ScenarioStatus
    start_time: float
    end_time: float = None
    duration: float = None             # 执行时长
    steps_results: List[Dict] = None    # 每步执行结果
    issues_found: List[Dict] = None     # 发现的问题
    score: float = None                 # 兼容性评分
    screenshots: List[str] = None       # 执行过程截图
    logs: List[str] = None             # 执行日志
    error: str = None                  # 错误信息
    
    def to_dict(self) -> Dict:
        return {
            "scenario_id": self.scenario_id,
            "status": self.status.value,
            "start_time": self.start_time,
            "end_time": self.end_time,
            "duration": self.duration,
            "steps_results": self.steps_results or [],
            "issues_found": self.issues_found or [],
            "score": self.score,
            "screenshots": self.screenshots or [],
            "logs": self.logs or [],
            "error": self.error
        }


class ScenarioLibrary:
    """
    场景库：管理和组织所有测试场景
    """
    
    def __init__(self):
        self._scenarios: Dict[str, ScenarioConfig] = {}
        self._load_builtin_scenarios()
    
    def _load_builtin_scenarios(self):
        """加载预置场景"""
        # 连接状态场景
        self.register(ScenarioConfig(
            name="蓝牙连接测试",
            type=ScenarioType.CONNECTION,
            description="测试App与锁的蓝牙连接功能",
            preconditions=["蓝牙已开启", "锁在有效范围内"],
            steps=[
                ScenarioStep("step_1", "打开App", "click", target="app_icon"),
                ScenarioStep("step_2", "等待设备扫描", "wait", params={"duration": 3}),
                ScenarioStep("step_3", "点击目标锁设备", "click", target="lock_device"),
                ScenarioStep("step_4", "等待连接完成", "wait", params={"duration": 5}),
                ScenarioStep("step_5", "验证连接状态", "verify", target="connection_status", 
                           expected={"state": "connected"})
            ],
            verification_rules=[
                {"type": "element_visible", "target": "connection_success_icon"},
                {"type": "bluetooth_state", "expected": "connected"},
                {"type": "no_error_popup"}
            ],
            environment={"bluetooth_enabled": True},
            tags=["P0", "蓝牙", "连接"],
            priority="P0"
        ))
        
        # 蓝牙开锁流程场景
        self.register(ScenarioConfig(
            name="蓝牙开锁流程测试",
            type=ScenarioType.UNLOCK,
            description="测试App蓝牙开锁完整流程（仅限App→锁端蓝牙通信）",
            preconditions=["已连接锁设备"],
            steps=[
                ScenarioStep("step_1", "确认已连接", "verify", target="connection_status",
                           expected={"state": "connected"}),
                ScenarioStep("step_2", "点击蓝牙开锁按钮", "click", target="unlock_button"),
                ScenarioStep("step_3", "等待开锁动画", "wait", params={"duration": 2}),
                ScenarioStep("step_4", "验证开锁结果", "verify", target="unlock_result",
                           expected={"success": True})
            ],
            verification_rules=[
                {"type": "element_clickable", "target": "unlock_button"},
                {"type": "operation_complete", "timeout": 10},
                {"type": "no_crash"}
            ],
            tags=["P0", "核心功能", "开锁"],
            priority="P0"
        ))
        
        # === 密码输入测试场景已禁用 ===
        # 原因：密码面板输入、指纹解锁等操作属于锁端物理操作，不在App自动化测试范围内
        # App自动化测试仅覆盖蓝牙开锁流程
        # 
        # 原场景代码：
        # self.register(ScenarioConfig(
        #     name="密码输入测试",
        #     type=ScenarioType.PASSWORD,
        #     description="测试密码输入功能在不同键盘下的表现",
        #     parameters={
        #         "test_passwords": ["123456", "abcdef", "混合密码@123"]
        #     },
        #     steps=[
        #         ScenarioStep("step_1", "进入密码输入页面", "click", target="password_input"),
        #         ScenarioStep("step_2", "输入密码", "input", target="password_field",
        #                    params={"password": "{{password}}"}),
        #         ScenarioStep("step_3", "确认输入", "click", target="confirm_button"),
        #         ScenarioStep("step_4", "验证输入结果", "verify", target="password_display")
        #     ],
        #     verification_rules=[
        #         {"type": "password_accurate", "expected": "{{password}}"},
        #         {"type": "no_input_error"}
        #     ],
        #     tags=["P1", "密码", "输入"],
        #     priority="P1"
        # ))
        
        # 离线状态场景
        self.register(ScenarioConfig(
            name="离线状态测试",
            type=ScenarioType.OFFLINE,
            description="测试网络断开时的App表现",
            preconditions=["App已打开"],
            steps=[
                ScenarioStep("step_1", "切换到离线模式", "environment", 
                           params={"network": "offline"}),
                ScenarioStep("step_2", "等待UI响应", "wait", params={"duration": 2}),
                ScenarioStep("step_3", "验证离线提示", "verify", target="offline_indicator",
                           expected={"visible": True}),
                ScenarioStep("step_4", "尝试开锁", "click", target="unlock_button"),
                ScenarioStep("step_5", "验证离线开锁", "verify", target="unlock_result",
                           expected={"offline_mode": True})
            ],
            environment={"network_type": "offline"},
            verification_rules=[
                {"type": "offline_indicator_visible"},
                {"type": "offline_unlock_supported"},
                {"type": "clear_offline_message"}
            ],
            tags=["P0", "离线", "网络"],
            priority="P0"
        ))
        
        # 横竖屏切换场景
        self.register(ScenarioConfig(
            name="横竖屏切换测试",
            type=ScenarioType.ORIENTATION,
            description="测试横竖屏切换时布局是否正常",
            preconditions=["已连接锁设备"],
            steps=[
                ScenarioStep("step_1", "确认竖屏状态", "verify", target="orientation",
                           expected={"portrait": True}),
                ScenarioStep("step_2", "切换到横屏", "environment", 
                           params={"orientation": "landscape"}),
                ScenarioStep("step_3", "验证横屏布局", "verify", target="layout",
                           expected={"readable": True}),
                ScenarioStep("step_4", "点击开锁按钮", "click", target="unlock_button"),
                ScenarioStep("step_5", "切换回竖屏", "environment",
                           params={"orientation": "portrait"}),
                ScenarioStep("step_6", "验证竖屏恢复", "verify", target="orientation",
                           expected={"portrait": True})
            ],
            verification_rules=[
                {"type": "layout_not_broken"},
                {"type": "all_elements_visible"},
                {"type": "interaction_works"}
            ],
            tags=["P2", "UI", "横竖屏"],
            priority="P2"
        ))
        
        # 固件升级场景
        self.register(ScenarioConfig(
            name="固件升级测试",
            type=ScenarioType.FIRMWARE,
            description="测试固件升级流程",
            steps=[
                ScenarioStep("step_1", "进入设置页面", "click", target="settings_button"),
                ScenarioStep("step_2", "点击固件升级", "click", target="firmware_upgrade"),
                ScenarioStep("step_3", "确认升级", "click", target="confirm_upgrade"),
                ScenarioStep("step_4", "等待升级完成", "wait", params={"duration": 60}),
                ScenarioStep("step_5", "验证升级结果", "verify", target="upgrade_result",
                           expected={"success": True})
            ],
            verification_rules=[
                {"type": "upgrade_progress_visible"},
                {"type": "no_upgrade_failure"},
                {"type": "auto_reconnect"}
            ],
            tags=["P1", "固件", "升级"],
            priority="P1"
        ))
        
        # 多锁管理场景
        self.register(ScenarioConfig(
            name="多锁管理测试",
            type=ScenarioType.MULTI_LOCK,
            description="测试多把锁的管理功能（蓝牙开锁场景）",
            parameters={
                "lock_count": 3
            },
            steps=[
                ScenarioStep("step_1", "查看锁列表", "click", target="lock_list"),
                ScenarioStep("step_2", "切换到第二把锁", "swipe", target="lock_item",
                           params={"direction": "left"}),
                ScenarioStep("step_3", "验证切换成功", "verify", target="current_lock",
                           expected={"id": "lock_2"}),
                ScenarioStep("step_4", "执行蓝牙开锁", "click", target="unlock_button"),
                ScenarioStep("step_5", "验证开锁目标", "verify", target="unlock_result",
                           expected={"target_lock": "lock_2"})
            ],
            verification_rules=[
                {"type": "lock_list_complete"},
                {"type": "switch_smooth"},
                {"type": "correct_lock_operated"}
            ],
            tags=["P1", "多锁", "管理"],
            priority="P1"
        ))
        
        # 分享权限场景
        self.register(ScenarioConfig(
            name="分享权限测试",
            type=ScenarioType.SHARE,
            description="测试临时密码分享功能",
            steps=[
                ScenarioStep("step_1", "进入分享页面", "click", target="share_button"),
                ScenarioStep("step_2", "设置权限", "input", target="permission_duration",
                           params={"value": "24h"}),
                ScenarioStep("step_3", "生成分享码", "click", target="generate_code"),
                ScenarioStep("step_4", "验证分享码", "verify", target="share_code",
                           expected={"valid": True, "length": 6})
            ],
            verification_rules=[
                {"type": "share_code_generated"},
                {"type": "permission_accurate"},
                {"type": "code_copyable"}
            ],
            tags=["P2", "分享", "权限"],
            priority="P2"
        ))
        
        # 通知适配场景
        self.register(ScenarioConfig(
            name="通知适配测试",
            type=ScenarioType.NOTIFICATION,
            description="测试各类通知的显示效果",
            steps=[
                ScenarioStep("step_1", "触发开锁通知", "click", target="unlock_button"),
                ScenarioStep("step_2", "等待通知", "wait", params={"duration": 3}),
                ScenarioStep("step_3", "检查通知栏", "verify", target="notification",
                           expected={"title": "开锁成功", "readable": True}),
                ScenarioStep("step_4", "点击通知", "click", target="notification_item"),
                ScenarioStep("step_5", "验证跳转", "verify", target="app_opened",
                           expected={"in_foreground": True})
            ],
            verification_rules=[
                {"type": "notification_shows"},
                {"type": "notification_clickable"},
                {"type": "jump_to_correct_page"}
            ],
            tags=["P2", "通知", "系统集成"],
            priority="P2"
        ))
        
        # 手势导航场景
        self.register(ScenarioConfig(
            name="手势导航兼容测试",
            type=ScenarioType.GESTURE,
            description="测试与系统手势导航的兼容性",
            steps=[
                ScenarioStep("step_1", "启用手势导航", "environment",
                           params={"gesture_nav": True}),
                ScenarioStep("step_2", "打开App", "click", target="app_icon"),
                ScenarioStep("step_3", "测试底部滑动", "swipe", target="screen",
                           params={"from": "bottom", "distance": 100}),
                ScenarioStep("step_4", "验证手势响应", "verify", target="gesture_area",
                           expected={"not_blocked": True}),
                ScenarioStep("step_5", "测试App内返回", "swipe", target="screen",
                           params={"from": "left", "action": "back"})
            ],
            environment={"gesture_navigation": True},
            verification_rules=[
                {"type": "app_gesture_not_conflict"},
                {"type": "edge_areas_accessible"},
                {"type": "swipe_back_works"}
            ],
            tags=["P2", "手势", "导航", "全面屏"],
            priority="P2"
        ))
        
        logger.info(f"Loaded {len(self._scenarios)} builtin scenarios")
    
    def register(self, scenario: ScenarioConfig):
        """注册场景"""
        self._scenarios[scenario.name] = scenario
    
    def get(self, name: str) -> Optional[ScenarioConfig]:
        """获取场景"""
        return self._scenarios.get(name)
    
    def list_by_tag(self, tag: str) -> List[ScenarioConfig]:
        """按标签筛选场景"""
        return [s for s in self._scenarios.values() if tag in (s.tags or [])]
    
    def list_by_priority(self, priority: str) -> List[ScenarioConfig]:
        """按优先级筛选"""
        return [s for s in self._scenarios.values() if s.priority == priority]
    
    def list_all(self) -> List[ScenarioConfig]:
        """列出所有场景"""
        return list(self._scenarios.values())
    
    def create_parameterized(self, name: str, params: Dict[str, Any]) -> ScenarioConfig:
        """
        创建参数化场景实例
        
        Args:
            name: 场景名称
            params: 参数值
            
        Returns:
            填充参数后的场景实例
        """
        base = self.get(name)
        if not base:
            raise ValueError(f"Scenario not found: {name}")
        
        # 深度复制并替换参数
        import copy
        scenario = copy.deepcopy(base)
        scenario.name = f"{name}_{params.get('instance_id', 'custom')}"
        
        # 替换步骤中的参数
        for step in scenario.steps:
            if step.params:
                for key, value in step.params.items():
                    if isinstance(value, str) and "{{" in value:
                        # 替换模板变量
                        for param_key, param_value in params.items():
                            value = value.replace(f"{{{{{param_key}}}}}", str(param_value))
                        step.params[key] = value
        
        return scenario
```

### 4.4 AgentController 模块

**模块职责**：Agent主控，实现ReAct循环、决策策略、异常恢复

#### 4.4.1 Agent主控实现

```python
"""
AgentController: Agent主控模块
实现ReAct循环、决策策略、异常恢复
"""

import time
import json
from typing import Dict, Any, List, Optional, Callable
from dataclasses import dataclass, field
from enum import Enum
import logging

logger = logging.getLogger(__name__)


class AgentState(Enum):
    """Agent状态"""
    IDLE = "idle"
    RUNNING = "running"
    PAUSED = "paused"
    COMPLETED = "completed"
    FAILED = "failed"


@dataclass
class Observation:
    """观察结果"""
    timestamp: float
    perception: Dict                 # 感知层结果
    understanding: Dict              # 理解层结果
    verification: Dict = None        # 验证层结果（上一个）
    
    def to_dict(self) -> Dict:
        return {
            "timestamp": self.timestamp,
            "perception": self.perception,
            "understanding": self.understanding,
            "verification": self.verification
        }


@dataclass
class Thought:
    """思考结果"""
    reasoning: str                   # 推理过程
    diagnosis: str                   # 问题诊断
    confidence: float                # 置信度
    alternatives: List[str] = None   # 备选方案
    
    def to_dict(self) -> Dict:
        return {
            "reasoning": self.reasoning,
            "diagnosis": self.diagnosis,
            "confidence": self.confidence,
            "alternatives": self.alternatives or []
        }


@dataclass
class Action:
    """Agent行动"""
    action_type: str                 # click/input/swipe/wait/environment/verify/recover
    target: str = None               # 目标元素
    params: Dict[str, Any] = None   # 行动参数
    reason: str = None               # 行动原因
    
    def to_dict(self) -> Dict:
        return {
            "action_type": self.action_type,
            "target": self.target,
            "params": self.params,
            "reason": self.reason
        }


@dataclass
class StepResult:
    """步骤执行结果"""
    action: Action
    observation: Observation
    thought: Thought
    success: bool
    error: str = None
    duration: float = None
    
    def to_dict(self) -> Dict:
        return {
            "action": self.action.to_dict(),
            "observation": self.observation.to_dict(),
            "thought": self.thought.to_dict(),
            "success": self.success,
            "error": self.error,
            "duration": self.duration
        }


@dataclass
class AgentMemory:
    """Agent记忆"""
    baseline: Dict = None           # 基线数据
    history: List[StepResult] = field(default_factory=list)  # 执行历史
    issues: List[Dict] = field(default_factory=list)         # 发现的问题
    scores: List[float] = field(default_factory=list)        # 评分历史
    learnings: List[str] = field(default_factory=list)       # 学习到的经验
    
    def add_step(self, step: StepResult):
        """添加步骤结果"""
        self.history.append(step)
        if not step.success:
            self.issues.append({
                "step": len(self.history),
                "action": step.action.to_dict(),
                "error": step.error,
                "timestamp": step.observation.timestamp
            })
    
    def add_score(self, score: float):
        """添加评分"""
        self.scores.append(score)
    
    def add_learning(self, learning: str):
        """添加学习经验"""
        self.learnings.append(learning)
    
    def to_dict(self) -> Dict:
        return {
            "baseline": self.baseline,
            "history_count": len(self.history),
            "issues_count": len(self.issues),
            "avg_score": sum(self.scores) / len(self.scores) if self.scores else 0,
            "learnings_count": len(self.learnings)
        }


class AgentController:
    """
    AgentController - Agent主控模块
    
    核心职责：
    1. ReAct循环控制
    2. 决策策略管理
    3. 异常恢复机制
    4. 记忆管理
    5. 评分计算
    
    设计理念：
    - 感知-理解-决策-执行-验证形成闭环
    - 每个决策都有推理过程（可解释AI）
    - 记忆支持学习和策略优化
    """
    
    def __init__(self, 
                 device_controller,
                 visual_perception,
                 scenario_library):
        """
        初始化Agent主控
        
        Args:
            device_controller: 设备控制器
            visual_perception: 视觉感知模块
            scenario_library: 场景库
        """
        self.device = device_controller
        self.vision = visual_perception
        self.scenarios = scenario_library
        
        # Agent状态
        self.state = AgentState.IDLE
        self.current_scenario = None
        self.current_step_index = 0
        
        # 记忆
        self.memory = AgentMemory()
        
        # 策略配置
        self.config = {
            "max_steps": 50,              # 最大步数
            "step_timeout": 30,           # 步骤超时
            "retry_on_failure": 3,        # 失败重试次数
            "exploration_rate": 0.1,      # 探索率
            "confidence_threshold": 0.7   # 置信度阈值
        }
        
        # 回调函数
        self.on_step_complete: Callable = None
        self.on_issue_detected: Callable = None
        
        logger.info("AgentController initialized")
    
    # ==================== 主循环 ====================
    
    def run_scenario(self, scenario_name: str, **kwargs) -> Dict:
        """
        执行测试场景
        
        ReAct循环：
        1. Observe: 感知当前状态
        2. Think: 思考分析
        3. Act: 执行行动
        4. 重复直到完成或失败
        """
        self.state = AgentState.RUNNING
        self.current_scenario = self.scenarios.get(scenario_name)
        
        if not self.current_scenario:
            raise ValueError(f"Scenario not found: {scenario_name}")
        
        # 参数化场景
        if kwargs:
            self.current_scenario = self.scenarios.create_parameterized(
                scenario_name, kwargs)
        
        logger.info(f"Starting scenario: {self.current_scenario.name}")
        start_time = time.time()
        
        # 如果场景需要特定环境，先设置环境
        if self.current_scenario.environment:
            self.device.set_environment(
                self._dict_to_env_config(self.current_scenario.environment))
        
        try:
            # 采集基线数据（用于跨环境对比）
            self._collect_baseline()
            
            # ReAct主循环
            for step_index in range(len(self.current_scenario.steps)):
                self.current_step_index = step_index
                
                step_result = self._execute_react_step(step_index)
                self.memory.add_step(step_result)
                
                # 回调
                if self.on_step_complete:
                    self.on_step_complete(step_result)
                
                # 检查问题
                if not step_result.success:
                    issue = self._analyze_issue(step_result)
                    self.memory.add_learning(f"Issue detected: {issue}")
                    
                    if self.on_issue_detected:
                        self.on_issue_detected(issue)
                    
                    # 根据策略决定是否继续
                    if not self._should_continue(issue):
                        break
                
                # 达到目标？
                if self._is_goal_reached():
                    break
            
            # 最终验证
            final_verification = self._run_final_verification()
            score = self._calculate_score(final_verification)
            self.memory.add_score(score)
            
            self.state = AgentState.COMPLETED
            
            return {
                "status": "success",
                "scenario": scenario_name,
                "duration": time.time() - start_time,
                "steps_executed": len(self.memory.history),
                "issues_found": len(self.memory.issues),
                "score": score,
                "memory": self.memory.to_dict()
            }
            
        except Exception as e:
            logger.error(f"Scenario execution failed: {e}")
            self.state = AgentState.FAILED
            
            return {
                "status": "failed",
                "scenario": scenario_name,
                "error": str(e),
                "duration": time.time() - start_time,
                "steps_executed": len(self.memory.history),
                "memory": self.memory.to_dict()
            }
    
    def _execute_react_step(self, step_index: int) -> StepResult:
        """
        执行单个ReAct步骤
        """
        step = self.current_scenario.steps[step_index]
        step_start = time.time()
        
        logger.info(f"Executing step {step_index + 1}: {step.description}")
        
        # 1. Observe - 感知
        observation = self._observe()
        
        # 2. Think - 思考
        thought = self._think(step, observation)
        
        # 3. Act - 执行
        action = self._decide_action(step, thought, observation)
        action_result = self._execute_action(action)
        
        # 4. 整合结果
        return StepResult(
            action=action,
            observation=observation,
            thought=thought,
            success=action_result.get("success", True),
            error=action_result.get("error"),
            duration=time.time() - step_start
        )
    
    # ==================== ReAct子步骤 ====================
    
    def _observe(self) -> Observation:
        """
        感知：采集当前状态
        """
        # 截图
        screenshot = self.device.screenshot()
        
        # 布局树
        hierarchy_xml = self.device.dump_hierarchy()
        
        # 设备状态
        device_info = self.device.get_device_info()
        bluetooth_state = self.device.get_bluetooth_state()
        network_state = self.device.get_network_state()
        
        # 性能数据
        performance = self.device.collect_performance()
        
        # VLM分析
        visual_analysis = self.vision.analyze_screen(
            screenshot,
            context={
                "scenario": self.current_scenario.name,
                "step": self.current_step_index
            }
        )
        
        perception = {
            "screenshot": screenshot,
            "hierarchy": hierarchy_xml,
            "device_info": device_info.__dict__,
            "bluetooth_state": bluetooth_state.value,
            "network_state": network_state,
            "performance": performance
        }
        
        understanding = {
            "layout_type": visual_analysis.layout_type,
            "elements": [e.__dict__ for e in visual_analysis.elements],
            "status_indicators": visual_analysis.status_indicators,
            "issues": visual_analysis.issues,
            "readability_score": visual_analysis.readability_score,
            "actionability_score": visual_analysis.actionability_score
        }
        
        return Observation(
            timestamp=time.time(),
            perception=perception,
            understanding=understanding
        )
    
    def _think(self, step, observation: Observation) -> Thought:
        """
        思考：分析当前状态，决定如何行动
        """
        context = f"""
        当前场景：{self.current_scenario.name}
        当前步骤：{step.description}
        步骤类型：{step.action}
        目标元素：{step.target}
        预期结果：{step.expected}
        
        当前界面：
        - 页面类型：{observation.understanding['layout_type']}
        - 可见元素：{[e['semantic'] for e in observation.understanding['elements']]}
        - 状态指示器：{observation.understanding['status_indicators']}
        
        检测到的问题：{observation.understanding['issues']}
        
        请思考：
        1. 当前状态是否正常？
        2. 如何执行这个步骤？
        3. 可能遇到什么问题？
        4. 如何验证结果？
        """
        
        # 使用LLM进行推理
        # reasoning = self.llm.reason(context)
        reasoning = self._rule_based_reasoning(step, observation)
        
        return Thought(
            reasoning=reasoning,
            diagnosis=self._diagnose(observation),
            confidence=self._calculate_confidence(observation),
            alternatives=self._generate_alternatives(step, observation)
        )
    
    def _rule_based_reasoning(self, step, observation: Observation) -> str:
        """
        基于规则的推理（简化版，LLM实现更强大）
        """
        action = step.action
        target = step.target
        
        # 查找目标元素
        elements = observation.understanding.get('elements', [])
        target_element = None
        for elem in elements:
            if target and (elem.get('semantic', '').lower() in target.lower() or
                          target.lower() in elem.get('semantic', '').lower()):
                target_element = elem
                break
        
        if action == "verify":
            if target_element:
                return f"目标元素'{target}'已找到，状态：{target_element.get('state', 'normal')}"
            else:
                return f"警告：未找到目标元素'{target}'，需要调整策略"
        
        elif action == "click":
            if target_element and target_element.get('clickable'):
                return f"目标按钮'{target}'可点击，执行点击"
            elif target_element:
                return f"目标元素'{target}'存在但可能不可点击，尝试点击"
            else:
                return f"未找到'{target}'，尝试查找替代元素"
        
        elif action == "input":
            if target_element:
                return f"找到输入框'{target}'，执行输入"
            else:
                return f"未找到输入框'{target}'"
        
        elif action == "wait":
            duration = step.params.get('duration', 3) if step.params else 3
            return f"等待{duration}秒"
        
        elif action == "environment":
            return f"需要设置环境：{step.params}"
        
        return f"执行{action}操作"
    
    def _diagnose(self, observation: Observation) -> str:
        """诊断当前状态"""
        issues = observation.understanding.get('issues', [])
        
        if not issues:
            return "状态正常，未发现明显问题"
        
        return f"发现{len(issues)}个问题：{'; '.join(issues[:3])}"
    
    def _calculate_confidence(self, observation: Observation) -> float:
        """计算置信度"""
        # 基于多种因素计算置信度
        elements = observation.understanding.get('elements', [])
        
        # 元素识别置信度
        if elements:
            avg_confidence = sum(e.get('confidence', 0) for e in elements) / len(elements)
        else:
            avg_confidence = 0.5
        
        # 可读性评分
        readability = observation.understanding.get('readability_score', 100) / 100
        
        # 综合置信度
        confidence = (avg_confidence * 0.4 + readability * 0.6)
        
        return min(1.0, max(0.0, confidence))
    
    def _generate_alternatives(self, step, observation: Observation) -> List[str]:
        """生成替代方案"""
        alternatives = []
        
        elements = observation.understanding.get('elements', [])
        element_types = [e.get('type') for e in elements]
        
        if step.action == "click":
            # 查找可替代的按钮
            for elem in elements:
                if elem.get('type') == 'button' and elem.get('clickable'):
                    if elem.get('semantic') != step.target:
                        alternatives.append(f"点击替代按钮：{elem.get('semantic')}")
        
        if step.action == "verify":
            # 检查目标是否存在
            if not any(step.target in e.get('semantic', '') for e in elements):
                alternatives.append(f"验证替代条件：页面类型={observation.understanding['layout_type']}")
        
        return alternatives[:3]  # 最多3个替代方案
    
    def _decide_action(self, step, thought: Thought, observation: Observation) -> Action:
        """
        决定行动
        """
        # 检查置信度
        if thought.confidence < self.config["confidence_threshold"]:
            logger.warning(f"Low confidence: {thought.confidence}, investigating...")
            
            # 低置信度：先探索
            return Action(
                action_type="wait",
                params={"duration": 1},
                reason="低置信度，等待状态稳定"
            )
        
        # 根据步骤类型决定行动
        if step.action == "click":
            return self._decide_click_action(step, observation)
        elif step.action == "input":
            return self._decide_input_action(step, observation)
        elif step.action == "swipe":
            return self._decide_swipe_action(step, observation)
        elif step.action == "wait":
            return Action(
                action_type="wait",
                params={"duration": step.params.get('duration', 3) if step.params else 3},
                reason="按步骤执行等待"
            )
        elif step.action == "verify":
            return Action(
                action_type="verify",
                target=step.target,
                params={"expected": step.expected},
                reason="按步骤执行验证"
            )
        elif step.action == "environment":
            return Action(
                action_type="environment",
                params=step.params,
                reason="设置环境参数"
            )
        
        # 默认：执行步骤定义的动作
        return Action(
            action_type=step.action,
            target=step.target,
            params=step.params,
            reason=f"执行步骤：{step.description}"
        )
    
    def _decide_click_action(self, step, observation: Observation) -> Action:
        """决定点击行动"""
        elements = observation.understanding.get('elements', [])
        
        # 查找目标元素
        for elem in elements:
            if step.target and (step.target in elem.get('semantic', '') or
                               elem.get('semantic', '') in step.target):
                if elem.get('clickable'):
                    return Action(
                        action_type="click",
                        target=elem.get('semantic'),
                        params={"bounds": elem.get('bounds')},
                        reason=f"点击目标元素：{elem.get('semantic')}"
                    )
        
        # 未找到，尝试第一个可点击元素
        for elem in elements:
            if elem.get('clickable') and elem.get('type') == 'button':
                return Action(
                    action_type="click",
                    target=elem.get('semantic'),
                    params={"bounds": elem.get('bounds')},
                    reason=f"点击替代按钮：{elem.get('semantic')}"
                )
        
        return Action(
            action_type="click",
            target=step.target,
            reason=f"尝试点击：{step.target}"
        )
    
    def _decide_input_action(self, step, observation: Observation) -> Action:
        """决定输入行动"""
        return Action(
            action_type="input",
            target=step.target,
            params={"text": step.params.get('password') or step.params.get('text', '')},
            reason=f"输入内容到：{step.target}"
        )
    
    def _decide_swipe_action(self, step, observation: Observation) -> Action:
        """决定滑动行动"""
        return Action(
            action_type="swipe",
            params=step.params or {"direction": "up"},
            reason=f"执行滑动：{step.params}"
        )
    
    def _execute_action(self, action: Action) -> Dict:
        """
        执行行动
        """
        try:
            if action.action_type == "click":
                bounds = action.params.get('bounds', [])
                if bounds:
                    x = bounds[0] + bounds[2] // 2  # 中心点X
                    y = bounds[1] + bounds[3] // 2  # 中心点Y
                    self.device.click(x, y)
                else:
                    # 使用VLM定位
                    observation = self._observe()
                    elements = observation.understanding.get('elements', [])
                    for elem in elements:
                        if elem.get('semantic') == action.target:
                            bounds = elem.get('bounds', [])
                            x = bounds[0] + bounds[2] // 2
                            y = bounds[1] + bounds[3] // 2
                            self.device.click(x, y)
                            break
            
            elif action.action_type == "input":
                text = action.params.get('text', '')
                self.device.input_text(text)
            
            elif action.action_type == "swipe":
                direction = action.params.get('direction', 'up')
                screen_info = self.device.get_device_info()
                w, h = screen_info.resolution
                
                if direction == "up":
                    self.device.swipe(w // 2, h * 3 // 4, w // 2, h // 4)
                elif direction == "down":
                    self.device.swipe(w // 2, h // 4, w // 2, h * 3 // 4)
                elif direction == "left":
                    self.device.swipe(w * 3 // 4, h // 2, w // 4, h // 2)
                elif direction == "right":
                    self.device.swipe(w // 4, h // 2, w * 3 // 4, h // 2)
            
            elif action.action_type == "wait":
                duration = action.params.get('duration', 3)
                time.sleep(duration)
            
            elif action.action_type == "environment":
                env_config = self._dict_to_env_config(action.params)
                self.device.set_environment(env_config)
            
            elif action.action_type == "verify":
                # 验证在验证层处理
                pass
            
            return {"success": True}
            
        except Exception as e:
            logger.error(f"Action execution failed: {e}")
            return {"success": False, "error": str(e)}
    
    # ==================== 辅助方法 ====================
    
    def _collect_baseline(self):
        """采集基线数据"""
        screenshot = self.device.screenshot()
        hierarchy = self.device.dump_hierarchy()
        visual = self.vision.analyze_screen(screenshot)
        
        self.memory.baseline = {
            "screenshot": screenshot,
            "hierarchy": hierarchy,
            "layout_type": visual.layout_type,
            "elements": [e.__dict__ for e in visual.elements],
            "timestamp": time.time()
        }
        
        logger.info("Baseline collected")
    
    def _run_final_verification(self) -> Dict:
        """运行最终验证"""
        # 执行场景定义的验证规则
        results = []
        
        for rule in self.current_scenario.verification_rules:
            result = self._verify_rule(rule)
            results.append(result)
        
        return {"rules": results}
    
    def _verify_rule(self, rule: Dict) -> Dict:
        """验证单条规则"""
        rule_type = rule.get('type')
        
        observation = self._observe()
        
        if rule_type == "element_visible":
            target = rule.get('target')
            elements = observation.understanding.get('elements', [])
            found = any(target in e.get('semantic', '') for e in elements)
            return {"rule": rule_type, "target": target, "passed": found}
        
        elif rule_type == "bluetooth_state":
            expected = rule.get('expected')
            actual = observation.perception.get('bluetooth_state')
            return {"rule": rule_type, "expected": expected, "actual": actual, 
                    "passed": expected == actual}
        
        elif rule_type == "no_error_popup":
            issues = observation.understanding.get('issues', [])
            error_keywords = ['error', '失败', '错误', 'warning']
            has_error = any(any(kw in issue.lower() for kw in error_keywords) 
                          for issue in issues)
            return {"rule": rule_type, "passed": not has_error}
        
        return {"rule": rule_type, "passed": True}
    
    def _calculate_score(self, verification: Dict) -> float:
        """计算兼容性评分"""
        rules = verification.get('rules', [])
        
        if not rules:
            return 100.0
        
        passed = sum(1 for r in rules if r.get('passed', True))
        base_score = (passed / len(rules)) * 70
        
        # 加上可读性和可操作性评分
        readability = self.memory.history[-1].observation.understanding.get(
            'readability_score', 100) if self.memory.history else 100
        actionability = self.memory.history[-1].observation.understanding.get(
            'actionability_score', 100) if self.memory.history else 100
        
        total_score = base_score + (readability * 0.15) + (actionability * 0.15)
        
        return min(100.0, total_score)
    
    def _analyze_issue(self, step_result: StepResult) -> Dict:
        """分析问题"""
        return {
            "step": self.current_step_index,
            "action": step_result.action.to_dict(),
            "error": step_result.error,
            "thought": step_result.thought.to_dict()
        }
    
    def _should_continue(self, issue: Dict) -> bool:
        """判断是否继续执行"""
        # 如果是关键问题，停止执行
        if "P0" in str(issue):
            return False
        
        # 否则根据重试策略决定
        step_result = self.memory.history[-1]
        retry_count = sum(1 for r in self.memory.history 
                         if r.action == step_result.action)
        
        return retry_count < self.config["retry_on_failure"]
    
    def _is_goal_reached(self) -> bool:
        """判断是否达到目标"""
        if not self.current_scenario.steps:
            return True
        
        # 所有步骤都执行成功
        return (self.current_step_index >= len(self.current_scenario.steps) - 1 and
                all(r.success for r in self.memory.history[-len(self.current_scenario.steps):]))
    
    @staticmethod
    def _dict_to_env_config(env_dict: Dict):
        """字典转EnvironmentConfig"""
        from .device_controller import EnvironmentConfig
        return EnvironmentConfig(**env_dict)
```

### 4.5 VerificationLayer 模块

**模块职责**：实现四维验证体系（结构/规则/语义/对比）

```python
"""
VerificationLayer: 验证层模块
实现四维验证体系
"""

from typing import Dict, Any, List, Optional
from dataclasses import dataclass
import time
import logging

logger = logging.getLogger(__name__)


@dataclass
class VerificationResult:
    """验证结果"""
    passed: bool
    score: float
    dimension: str                      # 验证维度
    details: Dict = None                # 详细结果
    issues: List[Dict] = None           # 发现的问题
    suggestion: str = None              # 修复建议
    
    def to_dict(self) -> Dict:
        return {
            "passed": self.passed,
            "score": self.score,
            "dimension": self.dimension,
            "details": self.details or {},
            "issues": self.issues or [],
            "suggestion": self.suggestion
        }


@dataclass
class ComprehensiveResult:
    """综合验证结果"""
    overall_passed: bool
    overall_score: float
    structure_result: VerificationResult = None
    rules_result: VerificationResult = None
    semantic_result: VerificationResult = None
    compare_result: VerificationResult = None
    all_issues: List[Dict] = None
    timestamp: float = None
    
    def to_dict(self) -> Dict:
        return {
            "overall_passed": self.overall_passed,
            "overall_score": self.overall_score,
            "structure": self.structure_result.to_dict() if self.structure_result else None,
            "rules": self.rules_result.to_dict() if self.rules_result else None,
            "semantic": self.semantic_result.to_dict() if self.semantic_result else None,
            "compare": self.compare_result.to_dict() if self.compare_result else None,
            "all_issues": self.all_issues or [],
            "timestamp": self.timestamp or time.time()
        }


class VerificationLayer:
    """
    VerificationLayer - 验证层
    
    四维验证体系：
    1. 结构验证 (Structure): 检查元素是否存在、位置是否正确
    2. 规则验证 (Rules): 检查是否符合业务规则
    3. 语义验证 (Semantic): 使用VLM判断用户是否能理解
    4. 对比验证 (Compare): 与基线数据对比
    
    设计理念：
    - 每个维度独立验证，输出独立评分
    - 综合评分采用加权平均
    - 所有问题统一收集，便于分析
    """
    
    def __init__(self, vision_module, device_controller):
        self.vision = vision_module
        self.device = device_controller
        
        # 权重配置
        self.weights = {
            "structure": 0.25,
            "rules": 0.25,
            "semantic": 0.30,
            "compare": 0.20
        }
        
        # 阈值配置
        self.thresholds = {
            "critical": 60,     # P0问题阈值
            "major": 75,        # P1问题阈值
            "minor": 90         # P2问题阈值
        }
    
    def verify(self, 
               observation: 'Observation',
               baseline: Dict = None,
               rules: List[Dict] = None) -> ComprehensiveResult:
        """
        执行完整的四维验证
        
        Args:
            observation: 当前观察结果
            baseline: 基线数据（用于对比验证）
            rules: 验证规则列表
            
        Returns:
            ComprehensiveResult: 综合验证结果
        """
        results = []
        all_issues = []
        
        # 1. 结构验证
        structure_result = self._verify_structure(observation)
        results.append(structure_result)
        all_issues.extend(structure_result.issues or [])
        
        # 2. 规则验证
        if rules:
            rules_result = self._verify_rules(observation, rules)
            results.append(rules_result)
            all_issues.extend(rules_result.issues or [])
        
        # 3. 语义验证
        semantic_result = self._verify_semantic(observation)
        results.append(semantic_result)
        all_issues.extend(semantic_result.issues or [])
        
        # 4. 对比验证
        if baseline:
            compare_result = self._verify_compare(observation, baseline)
            results.append(compare_result)
            all_issues.extend(compare_result.issues or [])
        
        # 计算综合评分
        overall_score = self._calculate_weighted_score(results)
        
        # 判断是否通过
        critical_issues = [i for i in all_issues if i.get('severity') == 'critical']
        overall_passed = len(critical_issues) == 0 and overall_score >= self.thresholds["major"]
        
        return ComprehensiveResult(
            overall_passed=overall_passed,
            overall_score=overall_score,
            structure_result=structure_result if 'structure_result' in dir() else None,
            rules_result=rules_result if 'rules_result' in dir() else None,
            semantic_result=semantic_result if 'semantic_result' in dir() else None,
            compare_result=compare_result if 'compare_result' in dir() else None,
            all_issues=all_issues,
            timestamp=time.time()
        )
    
    def _verify_structure(self, observation: 'Observation') -> VerificationResult:
        """
        结构验证：检查元素是否存在、位置是否正确
        """
        elements = observation.understanding.get('elements', [])
        
        passed_checks = []
        failed_checks = []
        
        # 检查关键元素是否存在
        key_element_types = ['unlock_button', 'connection_status', 'back_button']
        for elem_type in key_element_types:
            found = any(elem_type.lower() in e.get('semantic', '').lower() 
                       for e in elements)
            if found:
                passed_checks.append(f"元素'{elem_type}'存在")
            else:
                failed_checks.append(f"元素'{elem_type}'不存在")
        
        # 检查元素可见性
        invisible_elements = [e.get('semantic') for e in elements 
                             if not e.get('visible', True)]
        if invisible_elements:
            failed_checks.append(f"以下元素不可见：{invisible_elements}")
        
        # 检查元素位置（是否被遮挡）
        screen_info = self.device.get_device_info()
        for elem in elements:
            bounds = elem.get('bounds', [])
            if bounds:
                # 检查是否超出屏幕
                if bounds[0] < 0 or bounds[1] < 0:
                    failed_checks.append(f"元素'{elem.get('semantic')}'位置异常")
        
        # 计算评分
        total_checks = len(passed_checks) + len(failed_checks)
        score = (len(passed_checks) / total_checks * 100) if total_checks > 0 else 100
        
        return VerificationResult(
            passed=len(failed_checks) == 0,
            score=score,
            dimension="structure",
            details={
                "passed_checks": passed_checks,
                "failed_checks": failed_checks,
                "element_count": len(elements)
            },
            issues=[{"type": "structure", "description": f, "severity": "major"}
                   for f in failed_checks],
            suggestion="检查布局文件，确保关键元素正确渲染"
        )
    
    def _verify_rules(self, observation: 'Observation', 
                      rules: List[Dict]) -> VerificationResult:
        """
        规则验证：检查是否符合业务规则
        """
        passed_rules = []
        failed_rules = []
        
        for rule in rules:
            rule_type = rule.get('type')
            result = self._check_rule(rule_type, rule, observation)
            
            if result['passed']:
                passed_rules.append(rule)
            else:
                failed_rules.append({
                    **rule,
                    'actual': result.get('actual'),
                    'expected': result.get('expected')
                })
        
        # 计算评分
        total_rules = len(rules)
        score = (len(passed_rules) / total_rules * 100) if total_rules > 0 else 100
        
        return VerificationResult(
            passed=len(failed_rules) == 0,
            score=score,
            dimension="rules",
            details={
                "passed_rules": len(passed_rules),
                "failed_rules": len(failed_rules),
                "total_rules": total_rules
            },
            issues=[{"type": "rule_violation", "rule": f['type'], 
                    "description": f"规则{f['type']}未通过", "severity": "critical"}
                   for f in failed_rules],
            suggestion="检查业务规则实现，确保符合规范"
        )
    
    def _check_rule(self, rule_type: str, rule: Dict, 
                    observation: 'Observation') -> Dict:
        """检查单条规则"""
        if rule_type == "element_visible":
            target = rule.get('target')
            elements = observation.understanding.get('elements', [])
            found = any(target in e.get('semantic', '') for e in elements)
            return {"passed": found, "expected": True, "actual": found}
        
        elif rule_type == "bluetooth_state":
            expected = rule.get('expected')
            actual = observation.perception.get('bluetooth_state')
            return {"passed": expected == actual, "expected": expected, "actual": actual}
        
        elif rule_type == "no_error_popup":
            issues = observation.understanding.get('issues', [])
            error_keywords = ['error', '失败', '错误', 'warning']
            has_error = any(any(kw in issue.lower() for kw in error_keywords) 
                          for issue in issues)
            return {"passed": not has_error, "expected": "no error", "actual": issues}
        
        elif rule_type == "element_clickable":
            target = rule.get('target')
            elements = observation.understanding.get('elements', [])
            for elem in elements:
                if target in elem.get('semantic', ''):
                    return {"passed": elem.get('clickable', False),
                           "expected": True, "actual": elem.get('clickable')}
            return {"passed": False, "expected": True, "actual": "not found"}
        
        elif rule_type == "operation_complete":
            timeout = rule.get('timeout', 10)
            # 检查是否有加载中的状态
            elements = observation.understanding.get('elements', [])
            loading = any('loading' in e.get('state', '').lower() for e in elements)
            return {"passed": not loading, "expected": "no loading", "actual": loading}
        
        elif rule_type == "no_crash":
            # 检查App是否崩溃（通常表现为无法获取UI）
            hierarchy = observation.perception.get('hierarchy')
            return {"passed": hierarchy is not None and len(hierarchy) > 0,
                   "expected": "app running", "actual": "running" if hierarchy else "crashed"}
        
        return {"passed": True}
    
    def _verify_semantic(self, observation: 'Observation') -> VerificationResult:
        """
        语义验证：使用VLM判断用户是否能理解
        
        这是与传统自动化最大的区别：
        - 不仅检查元素存在
        - 更关注用户体验和可理解性
        """
        screenshot = observation.perception.get('screenshot')
        
        if not screenshot:
            return VerificationResult(
                passed=False,
                score=0,
                dimension="semantic",
                issues=[{"type": "no_screenshot", "description": "无截图数据"}]
            )
        
        # 使用VLM进行深度语义分析
        prompt = """
        请评估当前锁端App界面的用户体验质量：
        
        1. **连接状态可理解性**：用户能否清晰知道当前蓝牙连接状态？
        2. **开锁操作可发现性**：开锁按钮是否足够醒目，用户能否快速找到？
        3. **错误提示清晰度**：如果有问题，错误信息是否让用户知道如何解决？
        4. **整体视觉层次**：界面是否有清晰的视觉层次，关键信息是否突出？
        5. **潜在用户困惑点**：是否有任何可能导致用户困惑或误操作的设计？
        
        **评分标准**（0-100）：
        - 90-100：用户体验优秀，无任何理解障碍
        - 75-89：有轻微问题但不影响使用
        - 60-74：存在明显问题，可能导致部分用户困惑
        - <60：存在严重问题，可能导致用户做出错误操作
        
        请返回（JSON格式）：
        {{
            "score": 85,
            "strengths": ["优点1", "优点2"],
            "issues": [
                {"severity": "minor/major/critical", "description": "问题描述", "location": "位置"}
            ],
            "suggestions": ["改进建议"]
        }}
        """
        
        try:
            # 调用VLM分析
            vlm_result = self.vision._call_vlm(screenshot, prompt)
            
            score = vlm_result.get('score', 0)
            issues = vlm_result.get('issues', [])
            
            # 转换severity
            for issue in issues:
                if issue.get('severity') == 'critical':
                    issue['severity'] = 'critical'
                elif issue.get('severity') == 'major':
                    issue['severity'] = 'major'
                else:
                    issue['severity'] = 'minor'
                issue['type'] = 'semantic'
            
            return VerificationResult(
                passed=score >= self.thresholds["major"],
                score=score,
                dimension="semantic",
                details=vlm_result,
                issues=issues,
                suggestion='; '.join(vlm_result.get('suggestions', []))
            )
            
        except Exception as e:
            logger.error(f"Semantic verification failed: {e}")
            return VerificationResult(
                passed=False,
                score=0,
                dimension="semantic",
                issues=[{"type": "verification_error", "description": str(e)}]
            )
    
    def _verify_compare(self, observation: 'Observation', 
                        baseline: Dict) -> VerificationResult:
        """
        对比验证：与基线数据对比
        
        核心能力：检测同一功能在不同环境下的视觉差异
        """
        current_screenshot = observation.perception.get('screenshot')
        baseline_screenshot = baseline.get('screenshot')
        
        if not current_screenshot or not baseline_screenshot:
            return VerificationResult(
                passed=True,
                score=100,
                dimension="compare",
                details={"reason": "无可对比的基线数据"}
            )
        
        try:
            # 使用VLM进行对比分析
            comparison = self.vision.compare_screens(
                baseline_screenshot,
                current_screenshot,
                context={
                    "baseline_env": baseline.get('environment'),
                    "current_env": observation.perception.get('device_info', {}).get('model')
                }
            )
            
            # 分类问题
            critical_diffs = comparison.critical_differences
            major_diffs = [d for d in comparison.differences 
                          if d.get('severity') == 'major']
            minor_diffs = comparison.minor_differences
            
            issues = []
            for diff in critical_diffs:
                issues.append({
                    "type": "compare_critical",
                    "element": diff.get('element'),
                    "description": diff.get('impact'),
                    "severity": "critical"
                })
            for diff in major_diffs:
                issues.append({
                    "type": "compare_major",
                    "element": diff.get('element'),
                    "description": diff.get('description'),
                    "severity": "major"
                })
            for diff in minor_diffs:
                issues.append({
                    "type": "compare_minor",
                    "element": diff.get('element'),
                    "description": diff.get('description'),
                    "severity": "minor"
                })
            
            return VerificationResult(
                passed=len(critical_diffs) == 0,
                score=comparison.visual_score,
                dimension="compare",
                details={
                    "similarity": comparison.similarity,
                    "critical_count": len(critical_diffs),
                    "major_count": len(major_diffs),
                    "minor_count": len(minor_diffs)
                },
                issues=issues,
                suggestion="检查跨环境兼容性问题，确保功能一致性"
            )
            
        except Exception as e:
            logger.error(f"Compare verification failed: {e}")
            return VerificationResult(
                passed=True,
                score=100,
                dimension="compare",
                details={"error": str(e)}
            )
    
    def _calculate_weighted_score(self, results: List[VerificationResult]) -> float:
        """计算加权评分"""
        total_score = 0
        total_weight = 0
        
        for result in results:
            dimension = result.dimension
            if dimension in self.weights:
                total_score += result.score * self.weights[dimension]
                total_weight += self.weights[dimension]
        
        if total_weight == 0:
            return 100.0
        
        return total_score / total_weight
```

---

## 5. 测试场景库设计

### 5.1 场景分类总览

```
┌─────────────────────────────────────────────────────────────────┐
│                       测试场景分类体系                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    核心功能场景 (P0)                       │    │
│  ├─────────────────────────────────────────────────────────┤    │
│  │ 🎯 连接状态测试          │ 蓝牙配对、连接、断开、重连        │    │
│  │ 🎯 开锁流程测试          │ 蓝牙开锁、离线开锁、快捷开锁       │    │
│  │ ⚠️ 密码/指纹解锁         │ 【App自动化范围外】锁端物理操作     │    │
│  │ 🎯 离线状态测试          │ 网络断开、弱网、恢复重连          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    扩展功能场景 (P1)                       │    │
│  ├─────────────────────────────────────────────────────────┤    │
│  │ 📱 固件升级测试          │ 升级流程、断点续传、回滚           │    │
│  │ 📱 多锁管理测试          │ 添加删除、切换、权限管理           │    │
│  │ 📱 分享权限测试          │ 临时密码、限时权限、分享记录        │    │
│  │ 📱 通知提醒测试          │ 开锁通知、异常告警、电量提醒        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    兼容适配场景 (P2)                       │    │
│  ├─────────────────────────────────────────────────────────┤    │
│  │ 🖥️ 横竖屏切换          │ 布局适配、状态保持、功能正常        │    │
│  │ 🖥️ 手势导航兼容        │ 边缘防误触、手势响应、区域划分       │    │
│  │ 🖥️ 深色模式适配        │ 配色对比度、图标清晰度、文字可读    │    │
│  │ 🖥️ 字体大小适配        │ 文字截断、间距调整、布局自适应       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 场景参数化模板

```python
# === 密码输入测试模板已禁用 ===
# 原因：密码面板输入、指纹解锁等操作属于锁端物理操作，不在App自动化测试范围内
# 原模板代码：
# PASSWORD_INPUT_TEMPLATE = {
#     "name": "密码输入测试",
#     "parameters": {
#         "test_cases": [
#             {"password": "123456", "type": "数字", "length": 6},
#             {"password": "abcdef", "type": "字母", "length": 6},
#             {"password": "Ab1234", "type": "混合", "length": 6},
#             {"password": "123456789012", "type": "长密码", "length": 12},
#             {"password": "1", "type": "短密码", "length": 1}
#         ],
#         "keyboard_types": ["系统键盘", "第三方键盘"],
#         "input_methods": ["手动输入", "粘贴", "自动填充"]
#     }
# }

# 场景组合示例：完整兼容性测试
COMPATIBILITY_TEST_SUITE = {
    "name": "完整兼容性测试",
    "scenarios": [
        "蓝牙连接测试",
        "开锁流程测试", 
        "离线状态测试",
        "横竖屏切换测试",
        "深色模式测试"
    ],
    "environments": [
        {"resolution": "1080x1920", "dpi": 480, "android": "12"},
        {"resolution": "1440x3200", "dpi": 560, "android": "13"},
        {"resolution": "720x1280", "dpi": 320, "android": "11"}
    ]
}
```

---

## 6. 兼容性环境矩阵

### 6.1 维度定义

| 维度 | 分类 | 典型值 | 说明 |
|------|------|--------|------|
| **设备** | 分辨率 | 720x1280, 1080x1920, 1440x3200 | 主流手机分辨率 |
| | DPI | 320, 400, 480, 560 | 对应不同屏幕密度 |
| | 屏幕尺寸 | 5.5", 6.1", 6.7" | 对角线尺寸 |
| **系统** | Android版本 | 8.0, 10, 11, 12, 13, 14 | 系统版本 |
| | 厂商ROM | MIUI, EMUI, ColorOS, OriginOS | 厂商定制 |
| **配置** | 深色模式 | 开/关 | 夜间模式 |
| | 字体大小 | 小/正常/大/特大 | 系统字体 |
| | 手势导航 | 开/关 | 全面屏手势 |
| **网络** | WiFi | 强/弱 | 信号强度 |
| | 4G | 强/弱/不稳定 | 移动网络 |
| | 离线 | 断开 | 飞行模式 |
| | 蓝牙 | 开/关/异常 | 蓝牙状态 |

### 6.2 正交矩阵设计

```
┌─────────────────────────────────────────────────────────────────┐
│                    兼容性测试环境矩阵                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  维度组合策略：使用正交法减少测试组合数量                          │
│                                                                  │
│  ┌────────────┬────────────┬────────────┬────────────┐           │
│  │ 设备环境   │  系统环境  │  配置环境  │  网络环境  │           │
│  ├────────────┼────────────┼────────────┼────────────┤           │
│  │ A: 高端机  │ S: 最新系统│ D: 深色开  │ N: WiFi强  │           │
│  │ (2K/DPI560)│ (Android14)│ F: 字体大  │ 4G: 4G强   │           │
│  │ B: 主流机  │ S-1: 次新  │ N: 深色关  │ W: WiFi弱  │           │
│  │ (1080/DPI480│ (Android13)│ G: 手势开  │ L: 弱网    │           │
│  │ C: 入门机  │ S-2: 老系统│            │ O: 离线    │           │
│  │ (720/DPI320)│ (Android11)│            │ B: 蓝牙关  │           │
│  └────────────┴────────────┴────────────┴────────────┘           │
│                                                                  │
│  矩阵缩减策略：                                                  │
│  1. 全量组合：4×4×4×5 = 320种                                  │
│  2. 核心场景：优先级P0 × 必测环境 = 20种                        │
│  3. 扩展场景：P1 × 抽样 = 30种                                  │
│  4. 探索场景：P2 × 随机 = 20种                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 核心环境配置集

```python
# 核心环境配置集（必测）
CORE_ENVIRONMENTS = [
    {
        "name": "高端机-最新-深色-WiFi强",
        "resolution": (1440, 3200),
        "dpi": 560,
        "android": "14",
        "dark_mode": True,
        "font_scale": 1.0,
        "network": "wifi_strong",
        "bluetooth": True
    },
    {
        "name": "主流机-次新-深色-4G强",
        "resolution": (1080, 2400),
        "dpi": 480,
        "android": "13",
        "dark_mode": True,
        "font_scale": 1.0,
        "network": "4g_strong",
        "bluetooth": True
    },
    {
        "name": "入门机-老系统-正常-弱网",
        "resolution": (720, 1280),
        "dpi": 320,
        "android": "11",
        "dark_mode": False,
        "font_scale": 1.15,
        "network": "weak",
        "bluetooth": True
    }
]
```

---

## 7. 报告与评分体系

### 7.1 兼容性评分算法

```
┌─────────────────────────────────────────────────────────────────┐
│                    兼容性评分计算模型                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                         综合评分                                  │
│                            │                                     │
│            ┌──────────────┼──────────────┐                      │
│            ▼              ▼              ▼                      │
│     ┌──────────┐    ┌──────────┐    ┌──────────┐                │
│     │ 结构评分 │    │ 语义评分 │    │ 对比评分 │                │
│     │  (25%)  │    │  (30%)  │    │  (20%)  │                │
│     └────┬─────┘    └────┬─────┘    └────┬─────┘                │
│          │              │              │                        │
│          ▼              ▼              ▼                        │
│     元素存在      用户理解度      与基线差异                     │
│     位置正确      信息清晰度      视觉一致性                     │
│     可见可操作     无歧义         功能等价                      │
│                                                                  │
│  评分公式：                                                      │
│  Score = 0.25×结构 + 0.30×语义 + 0.25×规则 + 0.20×对比          │
│                                                                  │
│  评级标准：                                                      │
│  ┌─────────┬──────────┬────────────────────────┐                │
│  │  等级   │   分数   │       说明             │                │
│  ├─────────┼──────────┼────────────────────────┤                │
│  │   A     │ 90-100   │ 优秀，完全兼容         │                │
│  │   B     │ 75-89    │ 良好，有小问题         │                │
│  │   C     │ 60-74    │ 一般，需修复           │                │
│  │   D     │ <60      │ 差，严重问题           │                │
│  └─────────┴──────────┴────────────────────────┘                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 报告结构设计

```markdown
# 兼容性测试报告

## 执行概览
- 测试时间：2025-xx-xx
- 测试场景：5个核心场景
- 测试环境：3种设备配置
- 执行时长：xx分钟

## 评分汇总
| 环境 | 结构分 | 语义分 | 对比分 | 综合分 | 评级 |
|------|--------|--------|--------|--------|------|
| 高端机-最新 | 95 | 92 | 88 | 92 | A |
| 主流机-次新 | 88 | 85 | 82 | 85 | B |
| 入门机-老系统 | 75 | 78 | 70 | 75 | C |

## 问题清单
### 🔴 P0问题（阻断）
1. **[R-P0-01]** 开锁按钮在入门机老系统下被状态栏遮挡
   - 位置：锁端App首页
   - 复现：Android 11 + 720p分辨率
   - 建议：调整layout_marginTop值

### 🟠 P1问题（严重）
...

## 趋势分析
![趋势图](./trend.png)
```

---

## 8. 部署与集成

### 8.1 环境搭建步骤

```bash
# 1. 安装Python依赖
pip install uiautomator2 pillow openai

# 2. 安装ADB工具
# 下载Android Platform Tools

# 3. 连接设备
adb devices
adb forward tcp:7912 tcp:7912

# 4. 初始化uiautomator2
python -m uiautomator2.discover

# 5. 配置环境变量
export DEVICE_HOST=127.0.0.1:7912
export VLM_API_KEY=your_api_key
```

### 8.2 CI/CD集成

```yaml
# .gitlab-ci.yml 示例
stages:
  - test

compatibility_test:
  stage: test
  script:
    - python run.py --mode full --report output/report.html
  artifacts:
    reports:
      junit: output/junit.xml
    paths:
      - output/
  parallel:
    matrix:
      - DEVICE: high_end
      - DEVICE: mid_range
      - DEVICE: low_end
```

---

## 9. 扩展与演进

### 9.1 扩展路径

```
┌─────────────────────────────────────────────────────────────────┐
│                       Agent能力扩展路径                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  当前阶段：兼容性测试                                             │
│     │                                                          │
│     ▼                                                          │
│  扩展一：功能测试 ←──────────────────────────────────┐          │
│     │                                               │          │
│     ▼                                               │          │
│  扩展二：性能测试 ←──┐                               │          │
│     │               │                               │          │
│     ▼               │                               │          │
│  扩展三：安全测试 ←──┴───────────────────────────┐ │          │
│     │                                        │    │          │
│     ▼                                        ▼    ▼          │
│  扩展四：IoT通用测试              ←──── 综合测试平台            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 自学习机制

```python
# 场景自动生成示例
class ScenarioGenerator:
    """
    基于历史数据自动生成测试场景
    """
    
    def generate_from_history(self, history: List[TestResult]) -> List[ScenarioConfig]:
        """从历史测试结果中学习，生成新的测试场景"""
        
        # 1. 分析历史问题
        common_issues = self._analyze_common_issues(history)
        
        # 2. 生成针对性场景
        new_scenarios = []
        for issue in common_issues:
            scenario = self._create_scenario_for_issue(issue)
            new_scenarios.append(scenario)
        
        # 3. 优化现有场景
        for scenario in self._get_weak_scenarios(history):
            optimized = self._optimize_scenario(scenario)
            new_scenarios.append(optimized)
        
        return new_scenarios
```

---

## 附录

### A. 核心模块快速参考

| 模块 | 文件 | 主要类 | 职责 |
|------|------|--------|------|
| 设备控制 | device_controller.py | DeviceController | 设备操作、环境注入 |
| 视觉感知 | visual_perception.py | VisualPerception | 屏幕分析、VLM调用 |
| 场景管理 | test_scenario.py | ScenarioLibrary | 场景定义、参数化 |
| Agent主控 | agent_controller.py | AgentController | ReAct循环、决策 |
| 验证层 | verification_layer.py | VerificationLayer | 四维验证、评分 |

### B. 配置项参考

```python
# Agent配置
AGENT_CONFIG = {
    "max_steps": 50,
    "step_timeout": 30,
    "retry_on_failure": 3,
    "exploration_rate": 0.1,
    "confidence_threshold": 0.7
}

# VLM配置
VLM_CONFIG = {
    "model": "gpt-4o",
    "temperature": 0.1,
    "max_tokens": 2000
}

# 验证权重
VERIFICATION_WEIGHTS = {
    "structure": 0.25,
    "rules": 0.25,
    "semantic": 0.30,
    "compare": 0.20
}
```

---

*文档完*
