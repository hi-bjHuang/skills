---
name: vue-component-split
description: Use when working on Vue single-file components that have grown too large, mix multiple concerns, contain repeated template blocks, hold independent interaction modules, or need focused child components without changing behavior. Apply when reviewing, refactoring, or implementing Vue 3 SFCs, especially with <script setup>, Composition API state, watchers, computed values, lifecycle hooks, forms, modals, tables, charts, uploaders, dashboards, or testable UI modules.
---

# Vue 组件拆分

使用这个 skill 判断 Vue SFC 是否应该拆分，并在拆分时保持现有行为不变。

拆分不只是为了复用，更是为了可读性和可维护性。主组件应该像目录一样组织页面，而不是把所有实现细节堆成一本厚书。

## 拆分信号

当一个 Vue 文件出现以下任意一种信号时，就应该考虑拆分出子组件。

### 1. 存在独立的交互模块

页面上某个区域拥有独立的 state、逻辑或生命周期。

常见信号：

- 弹窗的 `visible`、`confirmLoading`、`onOk`、`onCancel` 全部混在主组件里。
- 图表自己处理 resize 监听、加载状态、数据转换。
- 上传模块自己处理进度、校验、重试、清理。

推荐做法：

- 拆成聚焦组件，例如 `<EditModal />`、`<SalesChart />`、`<FileUploader />`。
- 子组件管理自己的局部交互状态，除非这些状态必须由页面统一协调。
- 对外暴露小而清晰的接口：输入用 props，输出用 emits；只有弹窗打开、输入框聚焦、上传队列重置这类命令式动作才使用 `defineExpose`。

### 2. 出现重复的 UI 片段

同一页面中出现两次或以上几乎相同的模板代码。

推荐做法：

- 抽成可复用展示组件，例如 `<StatCard title="订单" :value="orderCount" />`。
- props 保持语义化和最小化。
- 不要过早抽成复杂配置对象，除非项目里已经有稳定的配置式组件模式。

### 3. 模板行数过多

模板超过约 150-200 行后，阅读和定位问题会明显变困难。

推荐做法：

- 按业务区块或工作流步骤拆分，例如 `<BaseInfo />`、`<ConfigPanel />`、`<FileUploader />`、`<ApprovalFlow />`。
- 主组件负责布局、编排和数据分发。
- 子组件负责自己的模板、局部交互和局部副作用。

### 4. 存在复杂的组合式逻辑

`<script setup>` 超过约 100 行，或者某组 `watch`、`computed`、生命周期钩子、副作用明显只服务于某个功能。

推荐做法：

- 如果逻辑和某个可见 UI 区域强绑定，优先拆成子组件。
- 如果逻辑是纯数据处理、与 UI 无关、且可能复用，再抽成 composable。
- 跨多个子组件的协调逻辑可以留在主组件，但不要保留子组件内部实现细节。

### 5. 该部分需要单独测试

模块逻辑重要，需要专门编写单元测试或组件测试。

推荐做法：

- 拆成独立组件，例如 `<ShoppingCart />`。
- 如果项目已有测试体系，补充或迁移对应测试。
- 优先测试拆出的聚焦组件，避免只在复杂父组件里做大而散的测试。

## 可以不拆分的情况

只有同时满足以下条件时，才可以放心不拆：

- 模板不到约 50 行。
- 脚本逻辑不到约 30 行。
- 该部分纯展示，只接收 props 并显示。
- 不会复用，也没有独立的交互单元。
- 当前明确处于一次性原型验证阶段。

## 决策流程

按以下顺序判断：

1. 是否有独立的状态、逻辑或生命周期？如果有，拆分。
2. 模板是否重复两次或以上？如果是，拆分。
3. 模板或脚本是否已经难以导航？如果是，拆分。
4. 是否需要针对该模块单独测试？如果是，拆分。
5. 如果以上都不是，暂时保留在当前组件中。

当你觉得“这里有点乱”，或者需要滚动很久才能找到相关代码时，就把它当作拆分信号。

## 拆分工作流

1. 先阅读组件，识别业务区块、状态归属、事件流、副作用和现有命名风格。
2. 选择符合用户可见模块或重复 UI 模式的边界，不要只按行数机械拆分。
3. 设计子组件 API：
   - 父传子数据用 `props`。
   - 子通知父用 `emit`。
   - 父拥有、子编辑的数据用 `v-model`。
   - 命令式动作才用 `defineExpose`。
4. 移动模板和最小必要逻辑到子组件。
5. 保持行为不变，尤其是 loading、校验、watch、生命周期清理、事件、权限、i18n key、埋点和可访问性属性。
6. 更新 imports、局部命名、类型和测试。
7. 运行项目可用的 typecheck、lint、单元测试或组件测试。

## 主组件目标形态

拆分后，主组件应该像页面目录：

- 负责页面级布局。
- 负责页面级数据获取和跨模块协调。
- 向子组件传递清晰输入。
- 处理有业务意义的子组件事件。
- 不保留子组件内部实现细节。

## 子组件目标形态

每个子组件应该只有一个清晰职责：

- 管理自己的局部 UI 状态和局部副作用。
- 明确声明 props 和 emits。
- 不引入父组件专属业务上下文，除非项目架构已经这样组织。
- 不要过早泛化。先为清晰度拆分，等真实复用出现后再抽象。

## 质量检查

完成前检查：

- 渲染结果和用户流程没有变化。
- 状态归属比拆分前更清晰。
- 主组件减少了实现细节，而不是只增加了文件数量。
- 子组件 API 小而易懂。
- 重复模板确实被移除。
- watch、生命周期钩子、订阅、定时器和事件监听仍然正确清理。
- 行为迁移到独立组件后，相关测试已补充或更新。
