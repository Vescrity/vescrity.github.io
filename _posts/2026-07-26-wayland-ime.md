---
layout: post
title: Wayland 输入法协议现状分析
categories: [Wayland]
---

内容使用 AI 辅助产出。

## 输入法需要什么

输入法需要什么？
1. **获取按键**: 输入法当然需要知道用户输入了什么
1. **提交预编辑至应用**: 预编辑常常由应用自己显示，所以应用需要知道预编辑内容
1. **提交最终文本至应用**。用户从候选词中选词，输入法把最终文本发给应用
1. **获取输入的上下文**。输入法需要知道光标周围有什么文字、光标位置、当前输入框类型。以提供准确候选词。
1. **放置候选框**: 总归有个角色需要知道候选框/光标在哪里
1. **用户交互**: 候选词选择等等

---
## X11 & Wayland

![alt text](https://www.csslayer.info/wordpress/wp-content/uploads/2022/02/Fcitx-5-Wayland-input-method-under-KDE-1.png)
<img src="https://www.csslayer.info/wordpress/wp-content/uploads/2023/08/x11.png" style="background-color: white;" alt="x11" />
<img src="https://www.csslayer.info/wordpress/wp-content/uploads/2023/08/wayland.png" style="background-color: white;" alt="wayland" />

输入法 grab 键盘，使 compositor 将所有按键事件先发给输入法。然后，根据按键处理结果（是否过滤），输入法可将按键事件转发回 compositor，再由 compositor 转发给应用——当该按键与输入法引擎的逻辑无关时。

## 一、概览

### 1.1 输入法分层架构

输入法在 Wayland 下涉及三个角色

- **Application 侧**：通过 text-input 协议向 compositor 表达输入意图（激活、光标位置、内容类型等）。
- **Compositor 侧**：接收 text-input 状态并转发给 IM、接收 IM 编辑请求并转发给应用、路由键盘事件。
- **IM 侧**：通过 input-method 协议接收应用状态，发回预编辑或提交编辑结果。输入法框架（fcitx5 / ibus）运行在此层，后端挂载具体引擎（pinyin / rime 等）。

### 1.2 协议 (wayland-protocols)

| 协议 | 接口名 |
|------|--------|
| text-input-v1 | `zwp_text_input_v1` |
| text-input-v3 | `zwp_text_input_v3` |
| xx-text-input-v3 | `xx_text_input_v3` |
| input-method-v1 | `zwp_input_method_v1` + `zwp_input_method_context_v1` |
| input-method-v2 | `zwp_input_method_v2` |
| xx-input-method-v2 | `xx_input_method_v1` |

> text-input 和 input-method 是成对的——一个约束客户端行为，一个约束输入法行为。compositor 在中间做协议翻译。两份协议独立但配套演进。

关于 xx-text-input-v3: 2025-04

> This commit introduces an experimental text-input protocol as a functionally exact copy of text-input-v3.
>
> The goal of this is to arrive at an improved text-input-next protocol, without committing to backwards-compatible changes beforehand.

关于 xx-input-method-v2: 2025-05

> This commit introduces an experimental input-method protocol as an exact
copy of the fle describing the unofficial zwp_input_method_v2 from
squeekboard.
> 
> It's also supported by wlroots and smithay.
>
> This protocol is the counterpart to text-input-v3. It gives the
compositor a standard way to outsource the handling of the input method.

### 1.3 数据通路概览
整条输入法链路可以拆成四条独立的通路，各走各的协议通道：
- **状态通路（应用 → IM）**：应用通过 text-input 协议告诉 compositor 当前文本域的状态——光标在哪、周围有什么文字、内容类型（密码/邮箱/终端）。compositor 将这些状态翻译后通过 input-method 协议发给 IM。IM 据此决定显示什么候选词。
- **编辑通路（IM → 应用）**：IM 通过 input-method 协议发回编辑结果。compositor 通过 text-input 协议转发给应用。删除操作也走这条通路。
- **键盘通路（独立通道）**：IM 需要拦截物理按键来驱动自己的输入逻辑。按键通过 input-method-v2 的 `keyboard_grab_v2` 独立接口实现。v1 时代按键在协议里（`keysym` 事件），v2 将其拆出为独立通道。
- **popup 通路（候选框定位）**：IM 创建候选框 popup surface，应用通过 text-input 的 `set_cursor_rectangle` 提供光标区域，compositor 据此放置 popup。

## 二、协议详解

### 2.1 text-input-v1

**接口与工厂**：`zwp_text_input_v1` + `zwp_text_input_manager_v1`（`create_text_input`，无 seat 参数）。

**Requests（Client → Compositor，11 个）**：

| Request | 参数 | 作用 |
|---------|------|------|
| `activate` | seat, surface | 文本域获焦时激活 |
| `deactivate` | seat | 文本域失焦时去激活 |
| `show_input_panel` | — | 请求显示虚拟键盘 |
| `hide_input_panel` | — | 请求隐藏虚拟键盘 |
| `reset` | — | 重置输入状态（外部修改文本后调用） |
| `set_surrounding_text` | text, cursor, anchor | 设置光标周围文本 |
| `set_content_type` | hint, purpose | 设置输入域类型（密码/邮箱/URL…） |
| `set_cursor_rectangle` | x, y, width, height | 设置光标位置（surface 本地坐标） |
| `set_preferred_language` | language | 设置首选语言 |
| `commit_state` | serial | 提交当前状态，serial 标识状态 |
| `invoke_action` | button, index | 触发候选框动作 |

**Events（Compositor → Client，13 个）**：

| Event | 参数 | 作用 |
|-------|------|------|
| `enter` | surface | surface 获得键盘焦点（text-input focus 跟随 keyboard focus） |
| `leave` | — | surface 失去键盘焦点，client 应清除 preedit 状态 |
| `modifiers_map` | map | modifier 名称表（用于 keysym 的修饰键位掩码） |
| `input_panel_state` | state | 虚拟键盘可见性变化 |
| `preedit_string` | serial, text, commit | 设置 preedit 文本 |
| `preedit_styling` | index, length, style | preedit 内部样式，**独立事件** |
| `preedit_cursor` | index | preedit 内光标位置，**独立事件** |
| `commit_string` | serial, text | 提交最终文本 |
| `cursor_position` | index, anchor | 提交后调整光标位置 |
| `delete_surrounding_text` | index, length | 删除光标周围文本 |
| `keysym` | serial, time, sym, state, modifiers | 转发 XKB 按键事件 |
| `language` | serial, language | 设置输入语言 |
| `text_direction` | serial, direction | 设置文本方向 |

**serial 机制**：客户端在 `commit_state` 请求中提供 serial；输入法在 `preedit_string`、`commit_string`、`keysym` 事件中携带它以标识所基于的已知状态；客户端可忽略过期 serial 的事件。

**设计特点（被 v3 改进）**：
1. **按键事件暴露**：`keysym` 事件把 XKB keysym 发给客户端的 text_input handler。
2. **preedit 三段式**：`preedit_styling` → `preedit_cursor` → `preedit_string` 顺序发送，但三事件间无原子性保证。

### 2.2 text-input-v3

**接口与工厂**：`zwp_text_input_v3` + `zwp_text_input_manager_v3`（`get_text_input(seat)`）。

**Requests（Client → Compositor，8 个）**：

| Request | 参数 | 作用 |
|---------|------|------|
| `destroy` | —（destructor） | 销毁对象，同时 disable 所有 enable 过的 surface |
| `enable` | — | 请求在 enter 所得的 surface 上启用 text input |
| `disable` | — | 显式 disable 当前 surface 上的 text input |
| `set_surrounding_text` | text, cursor, anchor | 设置光标周围纯文本 |
| `set_text_change_cause` | cause | 告知 compositor 文本变化原因 |
| `set_content_type` | hint, purpose | 设置内容类型 |
| `set_cursor_rectangle` | x, y, width, height | 标记光标区域 |
| `commit` | — | 原子应用所有 pending state |

**Events（Compositor → Client，6 个）**：

| Event | 参数 | 作用 |
|-------|------|------|
| `enter` | surface | surface 获得键盘焦点（text-input focus 跟随 keyboard focus） |
| `leave` | surface | surface 失去键盘焦点，client 应清除 preedit 状态 |
| `preedit_string` | text, cursor_begin, cursor_end | 设置 preedit 文本（光标范围合并为参数） |
| `commit_string` | text | 提交最终文本 |
| `delete_surrounding_text` | before_length, after_length | 删除光标前后文本 |
| `done` | serial | 应用上述三事件的 pending state |

**serial 机制**：serial 是状态版本号，解决异步通信中的"你说的这个编辑是基于哪个版本的状态"这个问题。客户端提供或 compositor 统计一个递增计数，附在状态提交和编辑事件中，让双方能判断"这个操作还是不是基于当前状态"、过期的就忽略。

v3 中 serial 由 compositor 统计——每个 text-input 对象收到了多少次 `commit` 请求，计数就是 serial，在 `done` 事件中携带。v1 中 serial 由客户端在 `commit_state(serial)` 里自行提供，输入法在编辑事件中回传。

**双缓冲**：v3 引入了 pending/current 两份状态的概念。`set_surrounding_text`、`set_content_type` 等请求只修改 pending state，`commit` 请求将 pending 原子替换为 current。input-method 侧同理——`surrounding_text`、`content_type` 等 event 只修改 pending state，`done` event 将 pending 原子替换为 current。v1 没有这个机制，状态在 `commit_state` 时直接生效。

**done 事件的应用顺序（协议原文）**：
1. 用光标替换已有 preedit 串
2. 删除请求的 surrounding text
3. 插入 commit string，光标在其末尾
4. 计算要发送的 surrounding text
5. 在光标位置插入新 preedit 文本
6. 在 preedit 文本内放置光标

### 2.3 text-input 对比

| 维度 | v1 | v3 | xx-v3 |
|------|----|----|-------|
| 激活模型 | `activate(seat, surface)`, 配对 | `enable`, 不要求成对 | 同 v3 |
| 焦点事件 | `enter(surface)` / `leave`（无参数） | `enter(surface)` / `leave(surface)` | 同 v3 |
| requests 数 | 11 | 8 | 10（+ `set_available_actions`, `announce_supported_features`） |
| events 数 | 13 | 6 | 8（+ `move_cursor`, `perform_action`） |
| 按键转发 | `keysym`、`modifiers_map`、`language`、`text_direction` | 全部移除 | 同 v3，全部移除 |
| preedit 样式 | `preedit_styling` + `preedit_cursor` 独立事件 | 合并进 `preedit_string(text, cursor_begin, cursor_end)` | 同 v3 |
| 状态提交 | `commit_state(serial)`，serial 由客户端提供 | `commit`（无参），serial 由 compositor 统计 | 同 v3 |
| done 应用步骤 | — | 6 步 | 8 步（+ "Move the cursor and selection" + "Perform the requested action"） |
| delete 参数 | `index, length`（相对光标） | `before_length, after_length`（光标前后） | 同 v3，增加越界调整和字节边界处理规范 |
| destroy | 无 destructor | `destroy` destructor | 同 v3 |
| 双缓冲 | 未提及 | 明确所有状态双缓冲 | 同 v3，新增 events/requests 均双缓冲 |
| content_hint | 有 `default(0x7)` 和 `password(0xc0)` 组合值 | 无组合值 | 同 v3 |
| content_purpose | 9=date, 10=time, 11=datetime, 12=terminal | 9=pin, 10=date, 11=time, 12=datetime, 13=terminal | 同 v3 |
| 输入面板 | `show_input_panel`/`hide_input_panel`/`input_panel_state` | 全部移除 | 同 v3 |
| move_cursor | 无 | 无 | `move_cursor(cursor, anchor)` event，支持相对偏移 + BEGINNING/END 特殊值 |
| action 机制 | `invoke_action(button, index)` | 无 | `set_available_actions` request + `perform_action` event，基于 `action` 枚举（finish 等） |
| supported_features | 无 | 无 | `announce_supported_features` request，基于 `supported_features` 位域枚举（如 `move_cursor`） |
| enable 约束 | 无 | 至多一个 enabled per instance | 同 v3 + "Requests to enable another text input when some text input is active must be ignored"（更明确的冲突处理） |

xx-v3 在 v3 基础上新增的能力：光标移动（`move_cursor`）、候选操作协商（`available_actions` / `perform_action`）、能力发现（`supported_features`）。这些能力与 xx-input-method-v2 的新增接口形成配对——compositor 侧新增 `perform_action`、`move_cursor`、`set_available_actions`、`announce_supported_features` 正是为了桥接两端。

### 2.3.1 xx-text-input-v3 详解

**接口**：2 个 interface
- `xx_text_input_v3` — text input 主接口
- `xx_text_input_manager_v3` — 工厂，`get_text_input(seat)`

是 text-input-v3 的超集扩展，保持 v3 全部 requests/events 不变，新增 2 个 requests 和 2 个 events（见 §2.3 对比表），done 应用步骤从 6 步扩展到 8 步。

**新增枚举**：

`action`：
- `finish(0)` — 用户完成当前编辑域，触发对应 content_purpose 的动作（如搜索/发送/导航）
- `none(1)` — 默认占位值，不应显式使用

`supported_features`（位域）：
- `none(0x0)` — 无额外功能
- `move_cursor(0x1)` — 支持 move_cursor 事件

**done 应用顺序新增步骤**（在 v3 的第 3 步和第 4 步之间、第 6 步之后各插入一步）：
- 4. 移动光标和选区（`move_cursor` 双缓冲状态）
- 8. 执行请求的操作（`perform_action` 双缓冲状态）

**serial 处理**：xx-v3 的 done 事件中 serial 参数被忽略。

**与 v3 不变的部分**：所有 v3 原有请求/事件的签名和语义完全一致。`delete_surrounding_text` 的描述中增加了越界调整和字节边界处理的具体规范。

### 2.4 input-method-v1

**接口**：4 个 interface
- `zwp_input_method_v1`（per-seat 单例）→ `activate` 事件创建 `zwp_input_method_context_v1`
- `zwp_input_method_context_v1`（per-activation 临时对象，去激活后销毁）
- `zwp_input_panel_v1` + `zwp_input_panel_surface_v1`

**生命周期**：`activate` 事件创建 context 对象 → IM 通过 context 发送编辑请求 → `deactivate` 事件销毁 context。

**Input Method Context Requests（IM → Compositor，14 个）**：

| Request | 参数 | 说明 |
|---------|------|------|
| `destroy` | —（destructor） | 销毁 context |
| `commit_string` | serial, text | 提交文本插入 |
| `preedit_string` | serial, text, commit | preedit 文本 |
| `preedit_styling` | index, length, style | preedit 样式，应在 preedit_string 前发 |
| `preedit_cursor` | index | preedit 光标，应在 preedit_string 前发 |
| `delete_surrounding_text` | index, length | 删除周围文本，跟随 commit_string |
| `cursor_position` | index, anchor | 调整光标，跟随 commit_string |
| `modifiers_map` | map | — |
| `keysym` | serial, time, sym, state, modifiers | 转发按键事件 |
| `grab_keyboard` | new_id (wl_keyboard) | 抓取硬件键盘 |
| `key` / `modifiers` | 多参数 | 转发未消费的按键事件 |
| `language` | serial, language | 设置语言 |
| `text_direction` | serial, direction | 设置文本方向 |

**Events（Compositor → IM，6 个）**：

| Event | 参数 | 说明 |
|-------|------|------|
| `surrounding_text` | text, cursor, anchor | 光标周围文本 |
| `reset` | — | 重置 |
| `content_type` | hint, purpose | 内容类型（引用 text-input-v1 的枚举） |
| `invoke_action` | button, index | 候选框动作 |
| `commit_state` | serial | 文本输入 serial |
| `preferred_language` | language | 首选语言 |

**设计特点**：
- 与 text-input-v1 镜像配对：v1 的 `keysym` / `language` / `text_direction` 在两端都有。
- preedit 三段式（styling → cursor → string）与 text-input-v1 一致。
- 键盘 grab 返回 `wl_keyboard`，IM 通过 `key`/`modifiers` 主动转发未消费事件。

### 2.5 input-method-v2

**接口**：4 个 interface
- `zwp_input_method_v2`（per-seat 单例，自带 active/inactive 状态，**无独立 context 对象**）
- `zwp_input_popup_surface_v2`
- `zwp_input_method_keyboard_grab_v2`
- `zwp_input_method_manager_v2`

**生命周期**：`activate` event 将 v2 对象标记 active → `deactivate` 标记 inactive。popup 在 active 时可见，deactivate 时不可见。无独立的 context 对象，v2 对象自身持状态。

**Events（Compositor → IM，6 个）**：

| Event | 参数 | 说明 |
|-------|------|------|
| `activate` | — | 标记为 active，重置状态，popup 可见 |
| `deactivate` | — | 标记为 inactive，popup 不可见 |
| `surrounding_text` | text, cursor, anchor | 光标周围文本 |
| `text_change_cause` | cause | 文本变化原因（引用 zwp_text_input_v3.change_cause） |
| `content_type` | hint, purpose | 内容类型（引用 zwp_text_input_v3.content_hint/content_purpose） |
| `done` | — | 原子应用上述事件的 pending state |

**Requests（IM → Compositor，7 个）**：

| Request | 参数 | 说明 |
|---------|------|------|
| `commit_string` | text | 提交文本（无 serial） |
| `set_preedit_string` | text, cursor_begin, cursor_end | preedit 文本（光标范围合并为参数） |
| `delete_surrounding_text` | before_length, after_length | 删除光标前后文本 |
| `commit` | serial | 应用上述三请求的 pending state |
| `get_input_popup_surface` | id, surface | 创建 popup surface |
| `grab_keyboard` | new_id (zwp_input_method_keyboard_grab_v2) | 键盘 grab，返回独立 grab 对象 |
| `destroy` | —（destructor） | 销毁主对象及子对象 |

**Keyboard Grab**：`zwp_input_method_keyboard_grab_v2` 独立 interface，follows wl_keyboard v6。IM 无需主动转发未消费事件——按键以 grab 对象的 event 出现。包含 `repeat_info` event。

**Popup Surface**：`zwp_input_popup_surface_v2` 有一个 event：

| Event | 参数 | 说明 |
|-------|------|------|
| `text_input_rectangle` | x, y, width, height | 通知 IM 当前光标区域位置（surface 本地坐标） |

当且仅当 IM 处于 active 状态时 popup 必须可见。

### 2.6 input-method-v1 vs v2

| 维度 | v1 | v2 |
|------|----|----|
| context 对象 | 有（per-activation 临时创建） | 无（v2 对象自身） |
| preedit 样式 | `preedit_styling` + `preedit_cursor` | 合并进 `set_preedit_string(text, cursor_begin, cursor_end)` |
| 按键转发 | `keysym` + `key`/`modifiers`（IM 主动转发） | 移除，按键走 `keyboard_grab_v2` 独立通道 |
| `keysym` / `language` / `text_direction` | 有 | 全部移除 |
| delete 参数 | `index, length` | `before_length, after_length` |
| 状态提交 | 无 commit/done 双缓冲概念 | 明确双缓冲区：events 改 pending → done 应用，requests 改 pending → commit 应用 |
| popup 定位 | `input_panel_surface_v1`，`set_toplevel` 或 `set_overlay_panel`，无定位反馈 | `input_popup_surface_v2`，`text_input_rectangle` event 告知 IM 光标区域 |
| manager | 无 | `zwp_input_method_manager_v2`（`get_input_method(seat)` 创建） |
| 竞争处理 | 未明确 | `unavailable` event 处理同 seat 多 IM 竞争 |
| 外部枚举引用 | text-input-v1 | text-input-v3 |
| 序列/组事件 | 部分事件有顺序约束 | 所有状态概念上双缓冲 |

### 2.7 xx-input-method-v2

**接口**：4 个 interface（比 v2 少 keyboard_grab，比 v2 多 positioner）
- `xx_input_method_v1`
- `xx_input_popup_surface_v2`（大幅扩展）
- `xx_input_popup_positioner_v1`（新增）
- `xx_input_method_manager_v2`

**与 v2 的差异要点**：

| 维度 | input-method-v2 | xx-input-method-v2 |
|------|----------------|---------------------|
| 协议名称 | `input_method_unstable_v2` | `input_method_experimental_v2` |
| 主接口 | `zwp_input_method_v2` v1 | `xx_input_method_v1` v4 |
| 新增 events | — | `set_available_actions`、`announce_supported_features`、`announce_protocol_compat` |
| 新增 requests | — | `perform_action`、`move_cursor` |
| error 枚举 | `(wl_surface 已有其他 role)` | `surface_has_role` + `inactive`（操作需 active 状态） |
| protocol_compat | — | 新增枚举 `text_input_v3(0x0)` / `xx_text_input(0x1)`；`xx_text_input` 时 serial 应设为 0，text input client 以 best-effort 处理请求 |
| done 应用顺序 | 6 步 | 8 步（+ "Move the cursor and selection" + "Perform the requested action"） |
| popup | `text_input_rectangle` 简单定位 | `start_configure` + `ack_configure` + `reposition` + `repositioned` 完整 configure 序列 |
| positioner | 无 | `xx_input_popup_positioner_v1`（anchor/gravity/constraint_adjustment/offset/reactive） |
| keyboard grab | `zwp_input_method_keyboard_grab_v2` | 协议中未见此 interface |
| manager | `zwp_input_method_manager_v2`（仅 `get_input_method`） | `xx_input_method_manager_v2` + `get_positioner` |
| 引用枚举 | `zwp_text_input_v3.*` | `xx_text_input_v3.*`（新增 action/supported_features） |

**Popup configure 序列（xx-input-method-v2 独有）**：
1. client: `popup = input_method.get_popup(wl_surface, positioner)`
2. client: `wl_surface.commit()`
3. compositor: `popup.start_configure(width, height, anchor_x, anchor_y, anchor_width, anchor_height, serial)`
4. compositor: `input_method.done()`
5. client: `ack_configure(serial)`
6. client: `wl_surface.commit()`

**positioner 设计**：`xx_input_popup_positioner_v1` 提供了与 `xdg_positioner` 类似的定位规则集合——anchor、gravity、constraint_adjustment（slide/flip/resize）、offset、reactive。这让 compositor 在定位时能参考 IM 提供的放置偏好和约束规则，而不是完全自行决定。`ack_configure` 握手确保 compositor 在 IM 完成 surface 内容更新后才呈现 popup，避免闪屏。

### 2.8 text-input 与 input-method 的配对关系

两族协议的核心关系：**compositor 在中间做协议翻译**，将 text-input 侧的状态翻译为 input-method 侧的事件，然后将 input-method 的编辑请求反向翻译为 text-input 侧的事件。

**语义对应表**
| 语义 | text-input 侧（Client / Compositor） | input-method 侧（Compositor / IM） |
|------|---------------------------------------|-------------------------------------------------------|
| 激活 | `enable` → compositor | compositor → `activate` |
| 去激活 | `disable` → compositor | compositor → `deactivate` |
| 周围文本 | `set_surrounding_text` → compositor | compositor → `surrounding_text` |
| 内容类型 | `set_content_type` → compositor | compositor → `content_type` |
| 状态提交 | `commit`（无参）→ compositor | compositor → `done`（无参） |
| 预编辑 | compositor → `preedit_string` | `set_preedit_string` → compositor |
| 提交文本 | compositor → `commit_string` | `commit_string` → compositor |
| 删除文本 | compositor → `delete_surrounding_text` | `delete_surrounding_text` → compositor |
| 状态确认 | compositor → `done(serial)` | `commit(serial)` → compositor |
| 可用操作 * | `set_available_actions` → compositor | compositor → `set_available_actions` |
| 功能声明 * | `announce_supported_features` → compositor | compositor → `announce_supported_features` |
| 光标移动 * | compositor → `move_cursor` | `move_cursor` → compositor |
| 执行操作 * | compositor → `perform_action` | `perform_action` → compositor |

**版本配对证据（协议枚举引用）**：
| 配对 | 证据 |
|------|------|
| v1 ↔ v1 | input-method-v1 的 content_type event 引用 text-input-v1 的 content_hint/content_purpose 枚举 |
| v3 ↔ v2 | input-method-v2 的 content_type 引用 `zwp_text_input_v3.content_hint`/`content_purpose`；text_change_cause 引用 `zwp_text_input_v3.change_cause` |
| xx ↔ xx | xx-input-method-v2 引用 `xx_text_input_v3.content_hint`/`content_purpose`/`change_cause`/`action`/`supported_features` 枚举；`announce_protocol_compat` 声明配对的是 `xx_text_input`（serial 设为 0）还是 `text_input_v3`（serial 正常） |

**按键事件路由的演变**：
- input-method-v1：按键事件在 text-input（`keysym` event）和 input-method（`keysym` request + `grab_keyboard` + `key`/`modifiers`）两侧均有通路。
- input-method-v2：按键全部收敛到 `keyboard_grab_v2` 独立通道，text-input 协议不再涉及按键。

---

## 三、实现落地：Treeland waylib 的 compositor 侧实现

### 3.1 协议抽象层

waylib 通过 `WTextInput` 抽象基类屏蔽 text-input v1/v2/v3 的接口差异。三个子类对应三个协议版本，将 v1 的 `activate`/`deactivate` 信号、v3 的 `enable`/`disable` 请求等不同激活模型统一为 `enabled`/`disabled` 信号；将 v1 的三段式 preedit（styling + cursor + string）和 v3 的一卷式 preedit 统一为 v3 风格的 `handleIMCommitted` 调用。内部枚举（content_hint、change_cause 等）也做了 v1/v3 的统一映射。

### 3.2 桥接器

`WInputMethodHelper` 是 compositor 内部的状态路由中枢，核心数据结构只有三个关键字段：

| 成员 | 含义 |
|------|------|
| `enabledTextInput` | 当前激活的 text input |
| `activeInputMethod` | 当前连接的 IM |
| `activeKeyboardGrab` | IM 键盘抓取 |

工作流程是镜像对称的：
- **text-input → IM**：text input enable/commit → helper 取状态 → IM sendActivate/sendDone
- **IM → text-input**：IM commit（preedit/commit_string/delete）→ helper 反向调用 → text input sendPreeditString/sendCommitString/sendDone

两端"激活"状态通过这三个字段保持同步：text-input enable → 记下 enabledTextInput → IM sendActivate；text-input disable → 清空 → IM sendDeactivate。

### 3.3 键盘 Grab

compositor 在 wlroots 上安装自定义 keyboard grab，拦截物理按键转发给 IM。

### 3.4 Popup Surface

IM 调用 `get_input_popup_surface` 创建候选框。每次 text-input commit 或 enable 时，光标位置广播给所有活跃 popup。

### 3.5 Virtual Keyboard

`wp_virtual_keyboard_v1` 允许外部进程模拟物理键盘注入按键。waylib 监控虚拟键盘设备创建/销毁，维护设备列表供键盘 grab 的绕过判定使用。

## 四、缺口与展望

### 4.1 当前 text-input-v3 + input-method-v2 的缺口

**popup 定位信息不足**：input-method-v2 中 IM 创建 popup 时不向 compositor 提供放置偏好，compositor 完全自行决定位置。xx-input-method-v2 通过 positioner（放置偏好 + 约束规则 + reactive）+ configure 序列（ack_configure 同步呈现）补全。

**缺少 available actions 机制**：text-input-v3 没有将候选操作作为协议级别的可发现能力暴露。IM 无法知道应用支持哪些操作（如 `finish`）。xx-text-input-v3 通过 `set_available_actions` + `perform_action` 补全。

**xx 新增能力需要发现机制**：v3/v2 用隐式信号表达能力（应用不发 `set_surrounding_text` 就是 empty initial），对 v3/v2 自身够用。但 xx 新增了 `move_cursor` 等能力，IM 发送前需要知道 text-input 是否支持，隐式机制不再够用。`announce_supported_features` 是为 xx 新能力配的发现机制。

**缺少光标移动能力**：text-input-v3 只能通过 preedit + commit 间接影响光标，没有直接的光标移动/选区操作。xx-text-input-v3 通过 `move_cursor` event 补全。

### 4.2 xx 协议的补全

xx-text-input-v3 和 xx-input-method-v2 形成完整的配对，在以下方面补全 v3/v2 的不足：

| 功能 | xx-text-input-v3 侧 | xx-input-method-v2 侧 |
|------|-------------------|---------------------|
| available actions | `set_available_actions` request | `set_available_actions` event → IM |
| perform action | `perform_action` event → app | `perform_action` request ← IM |
| supported features | `announce_supported_features` request | `announce_supported_features` event → IM |
| protocol compat | — | `announce_protocol_compat` event |
| move_cursor | `move_cursor` event → app | `move_cursor` request ← IM |
| popup positioner | `set_cursor_rectangle`（不变） | `xx_input_popup_positioner_v1` + configure 序列 |
| keyboard grab | — | 协议中未见此 interface |

### 4.3 协议演进趋势

- **按键处理与文本输入分离**：按键事件从 text-input 和 input-method 协议中移除，集中到 keyboard grab 独立通道。
- **简化复合状态**：preedit 从三段式合并为一卷式，状态提交引入双缓冲模型。
- **放弃 X11 遗留语义**：`keysym`、`language`、`text_direction`、`modifiers_map` 等 X11 时代概念被移除。
- **popup 定位协商增强**：从 v1 compositor 自行决定（IM 连光标位置都不知道）→ v2 compositor 将光标区域反馈给 IM（但定位仍是 compositor 单方面决定）→ xx IM 提供放置偏好作为输入，compositor 做约束计算后通过 ack_configure 同步呈现。
- **光标控制精细化**：从 v3 只能通过 preedit/commit 间接影响 → xx 的 `move_cursor` 直接操作光标位置和选区
- **能力发现**：v3/v2 用隐式信号表达能力（不发 `set_surrounding_text` 就是空）。xx 新增 `move_cursor`、`perform_action` 等能力后，IM 发送前需要知道 text-input 是否支持，因此引入 `announce_supported_features` 和 `set_available_actions` 显式发现机制。

xx 协议代表了 Wayland 输入法协议的下一个方向，但目前仍处于实验阶段

---

参考：wayland-protocols, cs slayer 博客
