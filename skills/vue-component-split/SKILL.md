---
name: vue-component-split
description: Use when working on Vue single-file components that have grown too large, mix multiple concerns, contain repeated template blocks, hold independent interaction modules, or need focused child components without changing behavior. Apply when reviewing, refactoring, or implementing Vue 3 SFCs, especially with <script setup>, Composition API state, watchers, computed values, lifecycle hooks, forms, modals, tables, charts, uploaders, dashboards, or testable UI modules.
---

# Vue Component Splitting

Use this skill to decide whether a Vue single-file component should be split, and to split it without changing existing behavior.

Splitting is not only about reuse. It is mainly about readability and maintainability. A parent component should organize the page like a table of contents instead of holding every implementation detail in one large file.

## Split Signals

Consider extracting child components when a Vue file shows any of these signals.

### 1. Independent Interaction Modules

A visible area owns its own state, logic, or lifecycle.

Common signals:

- Modal state such as `visible`, `confirmLoading`, `onOk`, and `onCancel` is mixed into the page component.
- A chart handles resize listeners, loading state, and data transformation by itself.
- An upload module handles progress, validation, retry, and cleanup logic.

Recommended approach:

- Extract focused components such as `<EditModal />`, `<SalesChart />`, or `<FileUploader />`.
- Let child components own local interaction state unless the page must coordinate that state globally.
- Expose a small, clear API: use props for inputs and emits for outputs. Use `defineExpose` only for imperative actions such as opening a modal, focusing an input, or resetting an upload queue.

### 2. Repeated UI Fragments

The same page contains two or more nearly identical template blocks.

Recommended approach:

- Extract a reusable display component, for example `<StatCard title="Orders" :value="orderCount" />`.
- Keep props semantic and minimal.
- Avoid premature complex configuration objects unless the project already uses a stable config-driven component pattern.

### 3. Oversized Templates

After a template grows beyond roughly 150-200 lines, reading and locating related behavior becomes noticeably harder.

Recommended approach:

- Split by business sections or workflow steps, for example `<BaseInfo />`, `<ConfigPanel />`, `<FileUploader />`, or `<ApprovalFlow />`.
- Keep layout, orchestration, and data distribution in the parent component.
- Let child components own their templates, local interactions, and local side effects.

### 4. Complex Composition Logic

`<script setup>` exceeds roughly 100 lines, or a group of `watch`, `computed`, lifecycle hooks, and side effects clearly serves only one feature.

Recommended approach:

- If the logic is tightly bound to a visible UI area, prefer extracting a child component.
- If the logic is pure data processing, UI-independent, and likely reusable, extract a composable instead.
- Keep coordination logic that spans several child components in the parent, but do not keep child implementation details there.

### 5. Independent Test Needs

A module has important behavior that deserves dedicated unit or component tests.

Recommended approach:

- Extract it into an independent component, for example `<ShoppingCart />`.
- If the project already has a test setup, add or migrate the relevant tests.
- Prefer testing the extracted focused component instead of writing broad tests against a large parent component.

## When Not To Split

It is reasonable to keep code in the current component only when all of these are true:

- The template is under roughly 50 lines.
- The script logic is under roughly 30 lines.
- The section is purely presentational and only receives props to display.
- It will not be reused and has no independent interaction unit.
- The current work is clearly a disposable prototype.

## Decision Flow

Evaluate in this order:

1. Does the section have independent state, logic, or lifecycle? If yes, split it.
2. Is the template repeated two or more times? If yes, split it.
3. Is the template or script already hard to navigate? If yes, split it.
4. Does the module need targeted tests? If yes, split it.
5. If none of the above apply, keep it in the current component for now.

When a section feels messy, or finding related code requires a lot of scrolling, treat that as a split signal.

## Splitting Workflow

1. Read the component first. Identify business sections, state ownership, event flow, side effects, and existing naming conventions.
2. Choose boundaries that match user-visible modules or repeated UI patterns. Do not split mechanically by line count alone.
3. Design the child component API:
   - Use `props` for parent-to-child data.
   - Use `emit` for child-to-parent notifications.
   - Use `v-model` when the parent owns data and the child edits it.
   - Use `defineExpose` only for imperative actions.
4. Move the template and the minimum necessary logic into the child component.
5. Preserve behavior, especially loading state, validation, watchers, lifecycle cleanup, events, permissions, i18n keys, analytics, and accessibility attributes.
6. Update imports, local names, types, and tests.
7. Run the project's available typecheck, lint, unit tests, or component tests.

## Parent Component Target Shape

After splitting, the parent component should read like a page outline:

- Own page-level layout.
- Own page-level data fetching and cross-module coordination.
- Pass clear inputs to child components.
- Handle child events that have business meaning.
- Avoid retaining child implementation details.

## Child Component Target Shape

Each child component should have one clear responsibility:

- Own its local UI state and local side effects.
- Declare props and emits explicitly.
- Avoid importing parent-specific business context unless the project architecture already organizes components that way.
- Avoid premature generalization. Split for clarity first, then abstract further when real reuse appears.

## Quality Check

Before finishing, verify that:

- Rendered output and user flows are unchanged.
- State ownership is clearer than before.
- The parent component has fewer implementation details, not just more files.
- Child component APIs are small and understandable.
- Repeated template blocks were actually removed.
- Watchers, lifecycle hooks, subscriptions, timers, and event listeners are still cleaned up correctly.
- Tests were added or updated when behavior moved into an independent component.
