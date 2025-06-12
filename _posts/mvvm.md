---
name: jc-vue
title: 前端概念
date: 2025-06-13
tags: 
- vue
- kmp
categories: 前端
---



# Jetpack Compose vs Vue.js

## **声明式 UI 编程：组件 = UI + 状态**

| 概念        | Jetpack Compose          | Vue                    |
| ----------- | ------------------------ | ---------------------- |
| UI 构建方式 | 使用 Kotlin 编写 UI 函数 | 使用 HTML + 模板语法   |
| 编程风格    | 声明式                   | 声明式                 |
| 示例        | `Text("Hello")`          | `<p>{{ message }}</p>` |

**说明：**

- Jetpack Compose 用函数来“描述 UI”
- Vue 用模板语法“声明 UI”
- 二者都强调 **状态驱动 UI**，界面是数据的“映射”

## **响应式系统：状态变 → UI自动更新**

| 功能           | Jetpack Compose                                     | Vue                                    |
| -------------- | --------------------------------------------------- | -------------------------------------- |
| 响应式状态管理 | `mutableStateOf`、`remember`、`State`               | `reactive()`、`ref()`、Vue 2 是 `data` |
| 自动更新机制   | Compose 会重新组合（Recompose）                     | Vue 会重新渲染 DOM                     |
| 示例           | `val name by remember { mutableStateOf("冯宇杰") }` | `data() { return { name: "冯宇杰" } }` |

**说明：**

- 当 `State` 或 `ref` 的值发生变化，界面自动刷新
- 这就是所谓的“数据驱动视图”

## **组件化开发**

| 特点     | Jetpack Compose                | Vue                              |
| -------- | ------------------------------ | -------------------------------- |
| 组件形式 | Kotlin 函数（Composable）      | Vue 单文件组件（SFC）            |
| 状态隔离 | 每个 Composable 可维护自身状态 | 每个 Vue 组件有自己的 data()     |
| 重用方式 | 像函数一样调用                 | 像标签一样引用 `<MyComponent />` |

**说明：**

- Compose 使用 `@Composable` 函数来封装 UI 单元
- Vue 使用 `.vue` 文件封装组件
- 都支持属性（props）、事件传递、插槽等概念

## **事件绑定与双向绑定**

| 概念         | Jetpack Compose                              | Vue                      |
| ------------ | -------------------------------------------- | ------------------------ |
| 单向数据绑定 | 默认行为（状态 -> UI）                       | 默认行为（状态 -> 模板） |
| 双向绑定     | 手动实现（如 `TextField` 的 `onValueChange`) | `v-model`                |
| 事件处理     | `onClick {}` 等 Lambda                       | `@click="handler"`       |

**说明：**

- 两者都支持从 UI 控件中“获取输入”并更新状态
- Vue 封装了 `v-model` 实现双向绑定；Compose 则需要手动处理

## **生命周期感知**

| Jetpack Compose                                              | Vue                                          |
| ------------------------------------------------------------ | -------------------------------------------- |
| `LaunchedEffect`、`DisposableEffect` 等用于管理生命周期感知行为 | `created`、`mounted`、`beforeDestroy` 等钩子 |

**说明：**

- Vue 有传统的生命周期钩子
- Compose 用更现代的 `Effect` API 来响应组件进入/退出组合的生命周期

## **路由与页面跳转（导航）**

| Jetpack Compose    | Vue           |
| ------------------ | ------------- |
| Navigation Compose | Vue Router    |
| 声明式路由配置     | 是            |
| 参数传递           | `navArgument` |

## **状态管理和跨组件通信**

| Jetpack Compose            | Vue                         |
| -------------------------- | --------------------------- |
| ViewModel + State Hoisting | Vuex（Vue2）/ Pinia（Vue3） |
| 状态提升、状态共享         | 提倡单向流动                |

## 总结

| 共性概念     | 说明                                   |
| ------------ | -------------------------------------- |
| 响应式系统   | UI 与数据自动同步                      |
| 声明式编程   | 描述界面“应该长什么样”，不是“怎么去做” |
| 状态驱动     | 状态决定 UI，不手动操作 UI 元素        |
| 组件化       | UI = 可组合的模块                      |
| 单向数据流   | 数据从父组件流向子组件，事件反向冒泡   |
| 生命周期感知 | 在特定生命周期执行代码                 |

**Jetpack Compose ≈ Vue + Vuex + JSX + 响应式系统，只不过 Compose 是 Kotlin 函数式语法，Vue 是 HTML 模板 + JS。**



# 跨平台UI框架

- Flutter、RN、CMP（JC的扩展）
- Jetpack Compose(JC)：android UI 框架
- Compose Multiplatform(CMP)：JetBrains 开发的JC扩展，用于跨平台支持
- Kotlin Multiplatform(KMP): 用于编写**跨平台共享业务逻辑**。

KMP 可以作为 CMP 的插件和底层跨平台支撑。反过来看，也可以认为 CMP 作为 KMP 项目中的 UI 支持，它不是 KMP 本身的一部分，只是一个通过启用共享 UI 来补充 KMP 的 SDK。



