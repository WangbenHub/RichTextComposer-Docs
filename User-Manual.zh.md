# Rich Text Composer — 用户手册 (中文)

本手册面向 **已拿到编译好的插件**、在虚幻编辑器里使用它的用户（蓝图 / 关卡 / UI 等）。**English:** [`User-Manual.en.md`](./User-Manual.en.md).

**适用引擎版本**：以 **Fab / 商城商品页** 及本包 **`RichTextComposer.uplugin`** 中的 **`EngineVersion`**（若有）为准。

---

## 目录

- [1. 安装与启用](#1-安装与启用)
- [2. 项目设置（Edit → Project Settings → Editor → Rich Text Composer）](#2-项目设置edit--project-settings--editor--rich-text-composer)
- [3. 快速上手：Display 与文档数据](#3-快速上手display-与文档数据)
- [4. 编辑富文本的几种方式](#4-编辑富文本的几种方式)
- [5. 模态编辑窗口：当前能做与暂不能做](#5-模态编辑窗口当前能做与暂不能做)
- [6. 运行时显示控件（Rich Text Composer Display）](#6-运行时显示控件rich-text-composer-display)
- [7. 打字机](#7-打字机)
- [8. 函数芯片：概念、候选类与运行时绑定](#8-函数芯片概念候选类与运行时绑定)
- [9. 子系统与点击事件](#9-子系统与点击事件)
- [10. 常见问题](#10-常见问题)

---

## 1. 安装与启用

### 1.1 从 Epic / Fab / 启动器安装（常见）

1. 在 **Epic Games Launcher** 的 **虚幻引擎** 页签中，安装好对应版本的引擎。
2. 在启动器的 **库 / Vault**（或 Fab 购买记录）中，将产品 **安装到引擎**（具体按钮文案以启动器为准）。
3. 打开目标工程，菜单 **Edit → Plugins**，搜索 **Rich Text Composer**（或 `Rich Text`），勾选 **Enabled**；若提示 **Restart**，请重启编辑器。

**为何这里仍要写「去 Plugins 勾选」？** 通过启动器 / Fab **安装到引擎**的插件，通常按引擎习惯 **按工程单独启用**：即每个工程第一次要在 Plugins 里打开，和 `.uplugin` 里的 `EnabledByDefault` 不是同一套流程。插件文件在引擎管理目录下；**不必**再把插件文件夹拷进工程 `Plugins`，除非你希望改为「仅本工程」的拷贝方式（见下）。

### 1.2 手动拷贝到工程 `Plugins`（源码 / ZIP / 工程独享）

适用于：你要 **改插件源码**、或只给某一个工程用、或从仓库 / 压缩包拿到 **`RichTextComposer` 文件夹**。

1. 将 **`RichTextComposer` 整个文件夹**（内含 `RichTextComposer.uplugin`、`Source` 等）放到 **`YourProject/Plugins/`** 下（无则新建 `Plugins`）。
2. 若来自其他机器，请先删除插件目录下的 **`Binaries`**、**`Intermediate`**（若有），再在 **本机** 打开工程，按提示 **编译**。
3. 本插件在 `RichTextComposer.uplugin` 中配置了 **`EnabledByDefault": true`**：放在 **`YourProject/Plugins/`** 下并成功编译后，多数情况下会 **随工程默认启用**，**通常不必**再到 **Edit → Plugins** 里勾一次。**最终以 Plugins 窗口里该插件是否勾选为准**（若你或版本控制曾关掉过它，需手动重新启用）。

### 1.3 如何确认已可用

- 在 **Plugins** 列表中能看到 **Rich Text Composer** 且为启用（**商城 / 启动器安装**或你曾手动核对时，以此为准）。
- 在含 **`Rich Text Composer Document`** 属性的资源上，详情里出现 **Edit**；或在 UMG 里加入 **Rich Text Composer Display** 后，其 **Document** 等属性可编辑。

---

## 2. 项目设置（Edit → Project Settings → Editor → Rich Text Composer）

以下均在 **`Edit → Project Settings → Editor` 分组下，页面显示名为 `Rich Text Composer`**。修改后按各条目说明生效（多数在下次打开模态窗口或预览时生效）。

### 2.1 Toolbar（工具栏）

| 设置项 | 作用 |
|--------|------|
| **Quick Color Palette** | 模态编辑器工具栏上的 **快捷颜色** 预设；用于 **正文 / 选区颜色** 等工具栏着色入口（与颜色选择器配合）。 |
| **Font Size Presets** | 模态编辑器里 **字号下拉** 的整数列表（如 10、12、14）。 |
| **Font Families** | 模态编辑器里 **字体下拉** 的字体资源路径（`FSoftObjectPath`，指向 `UFont`）。未配置时会使用插件内置的默认字体列表作为后备。 |

### 2.2 Modal（模态窗口）

| 设置项 | 作用 |
|--------|------|
| **Edit Surface Background Color** | 模态编辑窗口里 **正文编辑区背景色**。 |

### 2.3 Preview（预览）

| 设置项 | 作用 |
|--------|------|
| **Preview Wrap Text** | 在 **详情面板** 里对 `Rich Text Composer Document` 的 **内联预览** 是否 **自动换行**。 |

### 2.4 Caret（光标）

| 设置项 | 作用 |
|--------|------|
| **Caret Width Pixels** | 模态编辑器中 **插入点光标线宽**（像素，有上下限钳制）。 |
| **Caret Color** | 模态编辑器中 **光标颜色**。 |

### 2.5 Function Chips（函数芯片扫描）

| 设置项 | 作用 |
|--------|------|
| **Global Blueprint Function Libraries For Chip Scan** | 全工程 **Library** 型函数芯片的 **蓝图函数库** 类列表；插入对话框里作为 `[Library]` 候选来源之一。 |
| **Global Instance Classes For Chip Scan** | 全工程 **Instance** 型候选类（**不要**填蓝图函数库）；插入对话框里作为 `[Instance]` 候选来源之一。 |

与 **单份文档** 上的 **Bind Blueprint Function Libraries** / **Bind Instance Classes** 的合并规则见 [第 8.2 节](#82-插入对话框里能选到哪些类)。

---

## 3. 快速上手：Display 与文档数据

1. 打开任意 **Widget Blueprint**，在 **Palette** 中把 **`Rich Text Composer Display`** 拖入 **Hierarchy**。
2. 选中该控件，在 **Details** 中可直接编辑 **`Document`**：内容与在别处编辑 `Rich Text Composer Document` 属性相同；也可点 **`Document`** 旁的 **Edit** 打开 **模态编辑器**（见第 4、5 节）。
3. 运行时除使用设计时填好的文档外，通常还会调用 **`Set Document`**（蓝图节点名一般为 **Set Document**）用变量或网络数据 **替换整份文档**。
4. 若使用 **Instance** 型函数芯片，记得在运行时用 **第 8.4 节** 的节点绑定真实对象，否则该芯片可能不显示。

---

## 4. 编辑富文本的几种方式

| 方式 | 说明 |
|------|------|
| **A. 任意 UObject / 蓝图上的文档属性** | 属性类型为 **`Rich Text Composer Document`**，详情里 **Edit** 打开模态窗口；同一详情里可配 **Bind Blueprint Function Libraries** / **Bind Instance Classes**（仅影响该文档的芯片候选）。 |
| **B. UMG 里的 Rich Text Composer Display** | 在设计器 **Details** 中直接编辑 **`Document`**，或点 **Edit** 打开同一套模态编辑器；适合把文案和界面绑在一起。 |
| **C. 运行时** | 对 Display 调用 **`Set Document`**，或整体替换某蓝图里存的文档变量后再 **Set Document**，用于任务文本、聊天等动态内容。 |

以上三种方式编辑的都是 **同一种类型**（`FRichTextComposerDocument` 及其字段含义），但 **不一定是同一个文档实例**：例如 DataAsset 上的属性与某个 Display 上的 **`Document`** 默认是 **两份独立数据**；若要让界面与资产一致，需在运行时 **`Set Document`** 把一方拷贝给另一方，或让你的逻辑始终只维护一处再绑定到 Display。

---

## 5. 模态编辑窗口：当前能做与暂不能做

**当前版本在模态窗口中已支持（与工具栏相关项由 [第 2 节](#2-项目设置edit--project-settings--editor--rich-text-composer) 配置）：**

- 输入与编辑正文；**粗体 / 斜体**；**字号**（来自项目设置 **Font Size Presets**）；**字体**（来自 **Font Families** 及内置后备）；**文字颜色**（含 **Quick Color Palette**）。
- 插入与编辑 **超链接**、**图片**、**函数芯片**。
- 与选中范围相关的 **行内元素间距**（工具栏中与「文本与行内芯片之间空隙」相关的控制；影响当前选区所涉段落）。

**当前版本在模态窗口中尚未提供 UI 的：**

- **段落对齐**（左 / 中 / 右 / 两端对齐等）：数据层虽有段落对齐字段，但 **本插件的模态编辑器未提供对齐控件**；若要对齐，需通过 **其它能改到 `FRichTextComposerDocument` 的途径**（例如自建工具或 C++）写入，本手册不展开。

---

## 6. 运行时显示控件（Rich Text Composer Display）

### 6.1 详情 / 设计时可调属性

| 属性 | 说明 |
|------|------|
| **Document** | 当前显示的富文本文档；可在 UMG 设计器里直接改，或由 **`Set Document`** 在运行时替换。 |
| **Base Font Size** | 当某段文字的字号为 **0** 时使用的 **默认像素字号**（≥ 1）。 |
| **Use Internal Scroll Box** | 是否由本控件 **自带滚动框**；若为 false，请把本控件放进 **你自己的** `ScrollBox` 等外层容器。 |
| **Always Show Vertical Scrollbar** | 在启用内部滚动且内容可滚动时，是否 **始终显示** 垂直滚动条。 |
| **Auto Scroll To End On Document Change** | 在 **`Set Document`** 之后，下一帧是否 **滚到底**（适用于聊天、日志流）。需启用内部滚动。 |
| **Auto Scroll Only If Near Bottom** | 与上一项联用：为 true 时，仅当用户 **本来就在底部附近** 才自动滚到底，减少打断阅读。 |

另有 **`Set Use Internal Scroll Box`** 蓝图节点，可在运行时切换是否使用内部滚动。

### 6.2 蓝图可调用函数（名称以编辑器为准）

| 节点 / 函数 | 用途 |
|-------------|------|
| **Set Use Internal Scroll Box** | 运行时是否使用控件内置滚动框。 |
| **Set Document** | 替换整份显示文档。 |
| **Get Document** | 读取当前文档副本。 |
| **Set Base Font Size** | 运行时修改默认字号。 |
| **Set Line Height Layout Reference** | 打字机逐字揭示时，用 **参考文档** 固定行高布局，减少跳动。 |
| **Clear Line Height Layout Reference** | 清除上述参考。 |
| **Scroll To End** | 在启用内部滚动时滚到末尾。 |
| **Get Scroll Progress** | 使用 **内部滚动条** 时返回 **0～1** 的滚动比例；**未使用内部滚动**（或尚无有效滚动条）时为 **0**。 |
| **Get Rich Text Content Geometry** | 取内层富文本区域的 **FGeometry**（用于与其它 UI 对齐）。 |
| **Get Document Tail Local** | 文档「尾部」在内层坐标系下的 **插入点** 与 **最后片段** 矩形等信息。 |
| **Get Document Tail Absolute** | 同上，**绝对屏幕空间** 坐标。 |
| **Bind Runtime Object to Chip Class (Widget)** | 为 **Instance** 型芯片按 **芯片绑定类** 登记 **运行时对象**（仅本控件有效）。 |
| **Unbind Runtime Object for Chip Class (Widget)** | 取消本控件上某一绑定类的登记。 |
| **Clear All Chip-Class Objects (Widget)** | 清空本控件上 **全部** Instance 绑定。 |
| **Find Runtime Object for Chip Class (Widget)** | 查询本控件上某绑定类当前登记的对象（**不查**子系统兜底）。 |

---

## 7. 打字机

打字机按 **揭示步进** 把文档逐步写入目标 **Rich Text Composer Display**。有两种用法：**独立 UObject** 与 **Actor 组件**（组件内部持有一个 Typewriter 并转发 API）。

### 7.1 `URichTextComposerTypewriter`（对象）

常见用法：在蓝图里 **新建该类型对象**（例如 **Construct Object from Class**）并保存为变量，在 **Details** 里调 **Interval**、**Target Display** 等；也可只在事件图里用 **Play From*** 等节点并临时设置属性。

**可在蓝图 / 细节里配置的属性**

| 属性 | 说明 |
|------|------|
| **Interval** | 每一步揭示的 **时间间隔**（秒）。 |
| **Chars Per Tick** | 每一步揭示的 **字符数**（或等价单位，≥ 1）。 |
| **Reveal Hyperlink By Character** | 为 true 时超链接可见文字 **逐字** 揭示；为 false 时 **整段链接** 一步出现。 |
| **Typing Sound** | 打字 **音效** 资源。 |
| **Typing Sound Volume Multiplier** | 音效音量倍率。 |
| **Play Typing Sound** | 是否播放打字音效。 |
| **Typing Sound Min Interval** | 两次音效之间的 **最短间隔**（秒）；0 表示每一步都可播。 |
| **Target Display** | 接收揭示结果的 **Rich Text Composer Display**；也用于芯片解析所需的 **World** 等上下文。 |

**委托（Assignable）**

| 委托 | 说明 |
|------|------|
| **On Reveal** | 每揭示一步触发；参数为 **当前已揭示到的文档**（`FRichTextComposerDocument`）。 |
| **On Finished** | 揭示 **完全结束** 时触发。 |

**蓝图可调用 / 可查询函数**

| 函数 | 说明 |
|------|------|
| **Play From Document** | 从一份 **`Rich Text Composer Document`** 开始播放（使用已设的 **Target Display**）。 |
| **Play From String** | 从纯文本 **`FString`** 构建临时内容并开始播放。 |
| **Play From Text** | 从 **`FText`** 开始播放。 |
| **Pause** | 暂停揭示（计时器停表）。 |
| **Resume** | 从暂停处继续。 |
| **Skip To End** | **瞬间**揭示到全文末尾；会触发 **On Finished**。 |
| **Stop** | 停止计时；重置内部状态；若已设置 **Target Display**，会 **清除行高参考** 并把该 Display 的文档设为 **空文档**（界面被清空）。**不会**触发 **On Finished**（与 **Skip To End** 不同）。 |
| **Set Interval** | 修改 **Interval**；若当前 **正在播放且未暂停**，会 **立即** 用新间隔重建计时器。 |
| **Is Playing** | 是否正在播放（未结束且未停在「未播放」态）。 |
| **Is Paused** | 是否处于暂停。 |
| **Get Total Units** | 内部统计的 **总揭示单位数**（用于进度条等）。 |
| **Get Revealed Units** | 当前 **已揭示单位数**。 |

### 7.2 `URichTextComposerTypewriterComponent`（Actor 组件）

在 **Details** 中可配置与 7.1 对应的 **Interval**（显示名为 **Typing Interval (seconds)**）、**Characters Per Step**、**Reveal Hyperlink By Character**、音效相关四项；**`Target Display` 不在细节面板默认编辑**，请用 **`Set Target Display`**，或在 **`Play From Document` / `Play From String` / `Play From Text`** 的 **可选 Display 引脚** 传入目标控件（与 **Set Target Display** 等效）。

**委托：** **On Reveal**、**On Finished**（语义同 7.1）。

**蓝图 API 一览**

| 函数 | 说明 |
|------|------|
| **Set Target Display** | 指定要驱动的 **Rich Text Composer Display**。 |
| **Get Target Display** | 返回当前目标 Display。 |
| **Play From Document** | 参数：文档，**可选** Display（非空则同时设为本次目标）。 |
| **Play From String** | 同上，从字符串播放。 |
| **Play From Text** | 同上，从 `FText` 播放。 |
| **Stop** | 与 7.1 中 **Stop** 相同：清空目标 Display 文档、清行高参考等（见 7.1 表格）。 |
| **Skip To End** | 与 7.1 相同：瞬间到末尾并触发 **On Finished**。 |
| **Pause** / **Resume** | 暂停 / 继续。 |
| **Set Interval** | 写入组件上的 **Interval**；若内部 Typewriter 已创建，会 **同步** 到内部对象（正在播放且未暂停时与 7.1 相同，**立即** 重建计时器）。若尚未执行过任何 **Play***，内部对象可能尚不存在，此时新间隔会在 **首次 Play** 时一并应用。 |
| **Is Playing** / **Is Paused** | 查询状态。 |
| **Get Total Units** / **Get Revealed Units** | 查询进度。 |
| **Get Typewriter** | 取得内部 **`URichTextComposerTypewriter`** 对象（需进阶调试时使用）。 |

---

## 8. 函数芯片：概念、候选类与运行时绑定

### 8.1 Library 与 Instance 怎么选

| 类型 | 适用场景 | 推荐 |
|------|----------|------|
| **Library** | 不依赖「关卡里某一个具体对象」上的 **成员状态**；逻辑写在 **`UBlueprintFunctionLibrary` 子类**即可（**蓝图函数库** 或 **C++ 函数库类** 均可），例如表驱动、格式化字符串等。 | 能 Library 就 Library，预览与部署都简单。 |
| **Instance** | 必须用 **某个实例** 上的成员状态（如当前 UI 控制器、玩家组件）。文档里只存 **类 + 函数名**；运行时必须 **绑定对象**（见 8.4）。 | 仅当 Library 无法表达业务时再选。 |

### 8.2 插入对话框里「能选到哪些类」

以下 **四类** 合并为插入对话框的类列表；**同一 `UClass` 只保留第一次出现**的一条。顺序为：

1. 项目设置 **Global Blueprint Function Libraries For Chip Scan**  
2. 项目设置 **Global Instance Classes For Chip Scan**  
3. 当前文档 **Bind Blueprint Function Libraries**  
4. 当前文档 **Bind Instance Classes**  

对话框中先选 **类**（带 **`[Library]`** / **`[Instance]`** 标签），再选 **函数**。

### 8.3 对 UFUNCTION 签名的要求

函数需为 **`Blueprint Callable` 或 `Blueprint Pure`**，并满足插件对 **返回值**、**参数类型** 的约束；不符合时可能无法选入或运行时不生效。以 **插入对话框能否选到** 为首要自检方式。

### 8.4 运行时绑定：完整节点列表与选用建议

**解析顺序（Instance 芯片）：** 先在 **本块 Display** 上查找绑定类；若无，再到 **`Rich Text Composer Subsystem`** 的 **Game Fallback** 表查找。Library 芯片 **不使用** 下列绑定。

#### 在 `Rich Text Composer Display` 上（推荐用于「只服务本 UI」）

| 蓝图节点显示名 | 用途 |
|----------------|------|
| **Bind Runtime Object to Chip Class (Widget)** | 为某 **芯片绑定类** 登记 **运行时对象**。蓝图中 **Runtime object** 引脚传 **None**（不连对象 / 空引用）表示 **取消**该类在本控件上的登记。 |
| **Unbind Runtime Object for Chip Class (Widget)** | 取消本控件上该绑定类的登记。 |
| **Clear All Chip-Class Objects (Widget)** | 清空本控件上 **所有** Instance 绑定。 |
| **Find Runtime Object for Chip Class (Widget)** | 仅查本控件，**不查**子系统兜底。 |

#### 在 `Rich Text Composer Subsystem` 上（适合全局单例式上下文）

| 蓝图节点显示名 | 用途 |
|----------------|------|
| **Bind Runtime Object to Chip Class (Game Fallback)** | **Game Instance** 级兜底：Display 上 **没有** 该绑定类登记时使用。蓝图中 **Runtime object** 为 **None** 表示取消该类的子系统登记。 |
| **Unbind Runtime Object for Chip Class (Game Fallback)** | 取消子系统上该类的登记。 |
| **Clear All Chip-Class Objects (Game Fallback)** | 清空子系统上 **全部** Instance 绑定（**不影响**各 Display 上的登记）。 |
| **Find Runtime Object for Chip Class (Game Fallback)** | 仅查子系统表。 |

#### 蓝图函数库 `Rich Text Composer Binding Blueprint Library`

| 蓝图节点显示名 | 用途 |
|----------------|------|
| **Clear All Chip-Class Objects (Game Fallback Only)** | 只清 **子系统**；不动任何 Display。 |
| **Clear All Chip-Class Objects (Widget Only)** | 只清 **传入的 Display**；不动子系统。 |
| **Clear All Chip-Class Objects (Game Fallback + Widget)** | 先清子系统，再清传入的 Display；**Display 若为空则只清子系统**。 |

**建议：** 与某块 Widget 强相关的上下文 → 优先 **Display (Widget)**；全游戏共享、多块 UI 复用的上下文 → **Game Fallback**。两处可对 **同一绑定类** 同时登记：**运行时始终优先用 Display 上的对象**，子系统仅作补缺；适合「全局默认 + 某界面特例覆盖」。若不需要这种分层，保持 **二选一** 更易理解。

### 8.5 编辑器预览与运行的差异

详情预览与模态预览 **没有** 完整游戏环境与你在运行时绑定的对象，**Instance** 芯片常显示为占位或省略；**Library** 芯片通常正常。

---

## 9. 子系统与点击事件

**`Rich Text Composer Subsystem`**（**Game Instance Subsystem**）随游戏实例存在。在蓝图右键菜单中搜索 **Subsystem** 或 **Game Instance**，使用 **`Get Game Instance Subsystem`**（部分引擎版本界面文案可能为 **Get Subsystem from Game Instance** 等），类选 **`Rich Text Composer Subsystem`**。

**可绑定的多播委托**

| 委托 | 说明 |
|------|------|
| **On Hyperlink Clicked** | 超链接被点击；参数含显示文本、目标字符串、`EHyperlinkTargetType` 等。 |
| **On Image Clicked** | 与可点击图片芯片相关；参数含 **Gameplay Tag**、纹理路径等。 |

Instance 芯片在子系统上的绑定 API 见 [第 8.4 节](#84-运行时绑定完整节点列表与选用建议)。

---

## 10. 常见问题

**Q：拷贝到工程后编译失败，或一打开就提示要编译插件？**  
A：删掉插件内 **`Binaries` / `Intermediate`** 后重开工程；确认 **Visual Studio** 已按引擎要求安装 C++ 工作负载；引擎 **大版本** 需与插件发行说明一致（以 Fab / 商品页为准；`.uplugin` 未必单独写 `EngineVersion`）。

**Q：从商城安装后，在 Plugins 里找不到？**  
A：确认产品已安装到 **当前工程使用的引擎版本**；在 Plugins 窗口搜索 **`Rich Text`** 或 **`Composer`**。

**Q：Instance 芯片在游戏里不显示？**  
A：确认已对 **芯片里保存的绑定类** 调用 **Bind Runtime Object to Chip Class (Widget)** 或 **(Game Fallback)**；传入对象需为该类的 **实例或子类**。

**Q：模态里怎么没有段落对齐？**  
A：当前版本 **未在模态编辑器中提供对齐 UI**；见 [第 5 节](#5-模态编辑窗口当前能做与暂不能做)。

**Q：字体列表从哪来？**  
A：模态工具栏字体来自 **Project Settings → Rich Text Composer → Font Families**（及内置后备）；字号列表来自 **Font Size Presets**。见 [第 2 节](#2-项目设置edit--project-settings--editor--rich-text-composer)。
