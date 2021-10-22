---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
---

# 如何构建可持续维护的<br/>B 端 Vue 项目

张乐聪 · 07akioni

<img src="/cover.jpeg" style="margin: auto;" />

---

# 关于我

- 张乐聪
  - 生产用户界面
- Naive UI 的作者
  - https://www.naiveui.com
- Github 07akioni
  - https://github.com/07akioni
- 知乎 松若章
  - https://www.zhihu.com/people/hrsonion

---

# 目录

- 引言
- Vue 相关可维护性问题与治理方案
- 通用可维护性问题与治理方案
- 补充 & 总结

---

# 目录

- **引言**
- Vue 相关可维护性问题与治理方案
- 通用可维护性问题与治理方案
- 补充 & 总结

---

# 引言 · 关键词

### 两个关键词

<br/>

<v-click>

#### `局部最优`

项目的当下利益，某人在项目中的利益

</v-click>

<v-click>

#### `全局最优`

项目生命周期内的利益，产研团队的总体效益

</v-click>

<v-click>

> 提升项目可维护性：平衡“全局最优”和“局部最优”

<br />

> 最终目的：持续交付有价值的产品

</v-click>

---

# 引言 · 关于可维护性

<v-click>

- 非系统讲解（能力不足），漫谈

</v-click>

<v-click>

- 聚焦工程层面
  - 偏重执行侧
  - 非工程层面的重要性动态变化

</v-click>

<v-click>

- B 端
  - 前端业务逻辑重
    - 业务状态流转复杂，前后端逻辑边界不明确
  - 生命周期久，资源受限，更容易出现可维护性的问题
    - 现金流业务 HC 多，资源多 💰💰💰
  - 我没做过 C 端

</v-click>

---

# 引言 · 维护成本高的问题与困境

<v-click>

### 问题

- Bug 修复困难
- Feature 开发困难
- 构建困难
- 理解困难
- 过度消耗工具人工作积极性

</v-click>

<br />

<v-click>

### 困境

<div style="display: flex;">

<div style="overflow: hidden; flex-basis: 0; flex: 1;">

- 每一步走的都很快
- 但是在泥潭中行走
- 局部最优的和不是全局最优
  - 缺乏衡量研发效能的有效手段
  - 评价标准驱使我们追求局部最优

</div>

<div style="overflow: hidden; flex-basis: 0; flex: 1;">

<img src="/tarpit.jpeg" />

</div>

</div>

</v-click>

---

# 引言 · 为什么维护成本高

### 不可能三角

- 速度、成本、工程质量
- 重要性：速度 > 成本 >> 工程质量

<br>
<img src="/triangle.png" width="300">

<v-click>

> 工程质量真的没意义吗？

</v-click>

---

# 引言 · 工程质量

### 意义

- 长期来看，可维护性非常大的影响着速度和成本
- 局部优化（短期利益）= 对
- 全局优化（长期利益）= 好
- （对于多数业务项目）可维护性看起来是技术问题，实际上技术问题只是皮
- 做好比做对更难

<br>

<v-click>

### 维护成本影响因素

- 理解能力（人）
- 熟悉度（人）
- 业务复杂度（项目）
- **工程质量（项目）**

</v-click>

---

# 目录

- 引言
- **Vue 相关可维护性问题与治理方案**
- **通用的可维护性问题与治理方案**
- 补充 & 总结

---

# 项目中容易出现的可维护性问题

### Vue 特性相关

- 隐式逻辑
- 其他

<br>

### 通用

- 工具链
- 开发模式

---

# Vue 特性相关的问题

### 一条经验法则

一切能被滥用的特性都会被滥用

### 避免滥用

以较低的成本提升项目可维护性

---

# Vue 特性相关的问题

### 相关特性

- 隐式逻辑
  - mixin
  - extends
  - global properties
  - eventBus
  - $attrs
- 其他
  - 滥用 scoped style
  - this

---

# 隐式逻辑

### 常见特点

- 看代码很难知道发生了什么
  - 例如：间接使用状态，不知道哪里来的（`console.log(window.xxx)`)
  - 例如：状态变更发生在八杆子打不着的文件（`window.xxx = '报错吧'`)
  - 例如：直接 extends 一个组件，新旧数据、方法在同一文件混用，相互依赖
- 极度依赖运行时行为
  - 例如：类型使用 `any` 或者打全局补丁，不跑起来不知道行不行

---

# mixin · 滥用

```js
const fooableMixin = {
  methods: {
    foo() {
      console.log("foo");
    },
  },
};
```

```js
import { fooableMixin } from "...";

export default {
  mixins: [fooableMixin],
  methods: {
    test() {
      this.foo;
    },
  },
};
```

看上去方便极了！

---

# mixin · 滥用

### 隐式依赖

```js
export default {
  mixins: [mixin1, createMixin2(), mixin3, mixin4, ...],
  methods: {
    test() {
      this.xxx; // 这个方法哪来的？
    },
  },
};
```

---

# mixin · 滥用

```js
const mixin1 = {
  methods: {
    foo() {
      console.log("foo");
    },
  },
};
const mixin2 = {
  methods: {
    foo() {
      console.log("bar");
    },
  },
};
defineComponent({
  mixins: [mixin1, mixin2],
  methods: {
    test() {
      this.foo; // 调用哪一个？如何 Debug？如何保证 mixin 之间没有冲突？
    },
  },
});
```

---

# mixin · 治理方式

- 在 Vue 3 中，在业务逻辑中**彻底禁止** mixin 的使用
  - 如果不得不用，也仅允许在同一个地方进行全局 mixin 的注册（例如全局监控、统计）
- composition api
  - composition api 可以完全代替 mixin

---

# Composition API

简单介绍一下组合式 API

组合式 API 可以将同一个逻辑关注点相关代码收集在一起

<img src="/vs.png" style="height: 50%;" />

<div class="mt-4" v-click>

> 大多数情况，Composition API 的功能都可以使用 Options API 进行实现，但是更加复杂、可维护性低、类型支持差

</div>

---

# mixin · 使用 composable 治理 1

```js
function useFoo1() {
  return {
    foo() {
      console.log("foo");
    },
  };
}
function useFoo2() {
  return {
    foo() {
      console.log("bar");
    },
  };
}
defineComponent({
  setup() {
    const { foo: foo1 } = useFoo1(); // 显式使用
    const { foo: foo2 } = useFoo2(); // 显式使用
    return {
      foo1, // 显式使用
      foo2, // 显式使用
    };
  },
});
```

---

# mixin · 使用 composable 治理 2

### 带 Props

```js
const useFooProps = {
  foo: String
} as const
function useFoo(props: ExtractPropTypes<typeof useFooProps>) {
  return {
    printFoo () {
      console.log()
    }
  }
}

defineComponent({
  props: {
    ...useFooProps // 显式使用
  },
  setup(props) {
    const { printFoo } = useFoo(props) // 显式使用
  },
});
```

---

# extends · 滥用

<v-click>

- extends 是灾难的起点
- 在 UI 编程中，组合优于继承

</v-click>

<v-click>

不知道有什么东西非 extends 不可的

</v-click>

<v-click>

extends 三部曲：

1. 接手的人看不懂现有的组件，使用 extends 继承现有的组件
2. 在新组组件打补丁
3. 使用新组件替换旧组件

</v-click>

---

# extends · 滥用

### 曾经

```
- component.vue (行数 > 1000)
- component-new.vue (行数 > 1000)
// 覆写了一部分 component.vue 的代码
// 增加了一部分 component.vue 没有的代码
// 依赖了一部分 component.vue 的代码
```

### 现在

- 没有人知道发生了什么
- 没有人知道它正不正确
- 改代码在两个文件之间跳跃
- IDE 不知道发生了什么
- 没人知道改完之后对不对
- 出错的时候很难找到问题在哪
- 搜索关键词的时候总能搜出两个完全一致的方法名

---

# extends · 治理方式

- 请直接禁用
  - extends 为当时开发节省的时间，全都在后续维护中被找回来了
- 鉴于其功能和 mixin 的重叠，使用 composable 可以完全代替 extends

---

# eventBus · 滥用

## 用途

一种集中进行事件分发和监听的方式

```js
Vue.prototype.$eventBus = new Vue(); // 只在 Vue2 管用

this.$eventBus.$on("xxx");
this.$eventBus.$emit("yyy");
```

<v-click>

- 和 mixin 一样，看起来是不是很有诱惑力？轻轻松松跨组件调用
- 然后写了几十个 `$on`、`$emit` 之后，你露出了困惑的表情
  - 这个事件从哪来？
  - 这个事件到哪去？
  - 这个监听器收到的值删了什么类型？
  - 我改这段代码会影响哪里？

</v-click>

---

# eventBus · 治理方式

- 禁用
  - 好消息是 Vue 3 已经移除了 `$on`、`$off` 和 `$once`，你不能再这么用了
- 寻找替代品
  - provide/inject
  - composable
  - 状态管理工具
  - https://v3.vuejs.org/guide/migration/events-api.html#event-bus
- 建议
  - 需要显式的使用和良好的类型支持

---

# global properties · 滥用

### 举例

全部变量是一种共享数据、方法的方式

```js
Vue.prototype.$xxx = x; // vue 2
createApp().globalProperties.$xxx = x; // vue 3
window.xxx = x;
```

<v-click>

全局变量在许多场景可能是唯一的交流方式，例如不同的包想进行数据沟通

</v-click>

<v-click>

### 特点

谁都能插一脚；难以控制修改；不同项目间可能产生冲突

</v-click>

<v-click>

一旦冲突，晚上就又要加班了
<img src="/cover.jpeg" style="margin: auto; height: 100px;" />

</v-click>

---

# global properties · 治理方式

- 集中管理
- 使用 Context
- 使用一个项目内的变量充当全局变量的作用

```js
export let something = x;
export function setSomething(value) {
  something = value;
}
// 不要 window.something
```

<v-click>

### 使用原则

- 如果你说不出理由**必须**要使用全局变量，就不要使用
- 即不要因为全局变量能行，所以使用；而是只有它能行，所以使用

</v-click>

---

# $attrs · 滥用

### this.$attrs 是什么

包含了父作用域中不作为组件 props 或自定义事件的 attribute 绑定和事件

### 如何滥用？

<v-click>

把 attrs 当成 props 使用

</v-click>

<v-click>

```js
const data = {
  key: store.key || this.$route.query.key || this.$attrs.key,
};
```

</v-click>

<v-click>

开发者省了个 props 的时间，后续维护者一脸懵逼

`$attrs` 不需要声明，没有确切类型，没人知道这是哪来的，干啥的

</v-click>



---

# $attrs · 治理方式

### 用正确的姿势使用 $attrs

不要当 $props 用

<v-click>

对这么用的人建议做 200 个俯卧撑

</v-click>

---

# scoped style · 滥用

### 背景

Vue 提供了非常友好的样式隔离方案，这种特性如何被滥用？

<v-click>

- 某些业务在实现某些定制需求时，有些组件被完整的复制了一份，然后修改
- 两份代码几乎一样的组件，样式产生了些许差异
- 最后定制需求的组件停止了迭代，没有人知道这个 fork 出来的组件是干什么的，是否随着原组件的迭代而不再能用

</v-click>

<v-click>

滥用特性就像耍杂技，你永远都想不到实现者能做出什么姿势

</v-click>

---

# scoped style · 治理方式

### DRY (Don't repeat yourself)

- 禁止复制粘贴式编程

---

# this · 问题

### `this` 的特性

它具有传染的特性，依赖了实例的其他属性、方法，对逻辑拆分（组合）具有较大的影响

<v-click>

如果真的有逻辑拆分（组合）的需求，在 Vue 2 中只能使用 mixin

`this` 本身不能说是多大的问题，可是在某些场景下它确实会对 UI 代码的编写模式产生不利影响

</v-click>

<v-click>

再贴一遍 Option API 与 Composition API 的对比图：

<img src="/vs.png" style="height: 250px;" />

</v-click>

---

# this · 治理方式

- Composition API
  - 从心智上接受
  - 使用 Composition API 对于需要拆分（组合）的逻辑进行剥离
  - 如果团队能力足够，可以尝试彻底脱离 Options API
    - 可以做到
  - 好处
    - 更好的类型支持
    - 更显式的使用数据、方法，程序结构清晰，IDE 支持好
    - 逻辑聚合的更细
      - 可以把相关逻辑的代码写在一起
      - 基于 `this` 的代码逻辑全部注册于 `this` 上，理解逻辑需要在 `data`、`methods` 间跳转

---

# 小结

No Silver Bullet

避免了这些，我们的代码就可维护了么？

答案是不，但是做了更可能可维护

No Silver Bullet

<v-click>

### 如何执行

- CR
  - 有知道什么是好代码的 Project Owner，并真的能对项目和模块质量把关
- 共识
  - 花时间在团队建立好代码的共识
- 程序化约束
  - 可以自动化检查的约束才有最高的可行性

</v-click>

---

# 工具链相关的问题

- 长链路
  - 过度依赖编译工具
  - 冗余技术栈
- 不可信链路
  - 自制不可靠的工具
- 无效益链路
  - 不分场合使用 monorepo
- 忽略基本盘
  - 工具真正被使用到的功能质量差

---

# 长链路 · 过度依赖编译工具

- 插入过多的 webpack 插件
  - 升级之后没测试资源
- hard to debug

---

# 长链路 · 冗余技术栈

- 在项目中无脑的加入所有 babel 转化
  - 开发时引入 regenerator，如果搞不好 sourcemap，断点代码完全不可读
  - <img src="/regenerator.png" style="height: 230px;" />
  - sourcemap 有啥搞不好的？
    - 有一个朋友的组，一位老兄在编译期提取所有中文字符串塞进 i18n
    - 从那之后他们的 sentry 就嗝屁了
    - 你以为好了不等于真的好了，细节坑人

---

# 长链路 · 冗余技术栈

- 在项目中无脑的加入所有 babel 转化
- 库开发的时候引入巨多的打包插件
  - tsc 能解决的尽量不要引入别的工具
  - 现在的前端应用框架（例如 umi）都会对 node_modules 中的库进行 babel 转化
- 同一个项目混用 sass + less，在 vue 中这样做并不难

---

# 不可信链路 · 自制不可靠的工具

### 自制一个有 `build:watch` 功能的工具

编写代码 => `build:watch` => `webpack` => 浏览器

- 代码错了？
- `build:watch` 错了？
- 每一次出错都要进行双倍检查、重试
- 额外思维负担

<br />

### 理想状态

- 要么可靠
- 要么不用
- 要么明确作为过渡

---

# 无效益链路 · 滥用 monorepo

### 不分场合使用 monorepo

- Bug as feature，幻影依赖既然能用，为什么要写到 package.json
- 源码引用产物引用换着来，人均黑魔法大师
- 第三方依赖版本不一致，就想让所有库都从 NPM 各自拉，等的越久茶越香
- 仓库内部依赖版本不一致，没几周就要上线，谁更新 common 库的依赖谁是傻子，又不是不能用
- 版本发布随心所欲，beta forever，semver 限制了开发自由
- 使用 yarn
  - 每次 setup 不超过五分钟说明代码不够企业级
  - hoist 是不是带来难以排查的 bug

---

# 忽略基本盘 · 底盘不牢的工具链

- 假设存在有某个工具（纯属虚构），它声称自己无所不包，赋能研发，面向未来
  - 工具 KPI：使用率高 > 设计听起来厉害 >> 使用者体验
  - `build` 命令产生错误产物
  - `install` 每次可以喝杯茶
  - 每次升级必须崩溃
  - 拉 oncall 才能搞懂功能
- 使用者体验差并不会导致使用率低、也不影响这个工具的设计听起来很厉害
- 这个工具帮你提供可维护性了吗？

技术选型不要盲目跟风。如无必要，尽量不要受场外因素影响

---

# 小结

### 减法更重要

在业务开发中，对于工具链的使用，减法远比加法重要

### 做减法的方式

1. 不使用无必要的工具
   - tsc 够用就不要 rollup、babel
   - 没有明确目的的时候不要过度兼容老浏览器
2. 使用稳定到配好就可以忽略其存在的工具
   - 你什么时候怀疑过 v8 出问题
   - 对性能或理念有代差距的工具，这条正相反，例如请尽量在尝试使用 esbuild 解决实际问题
3. 在项目起步阶段做好选择
   - 在开发资源紧张的情况下，现在不做 = 永远不做

---

# 开发模式的问题

- 鸵鸟策略（只要能跑，就不改）
  - 错误后推
    - 错误应该前置
    - 依赖永不更新
  - 项目是别人的的心态

---

# 鸵鸟策略 · 错误后推

### 错误应该前置

- 在编译器出错好过运行时
  - 不使用 TypeScript 开发多人维护的大型项目
  - 使用 `any` 糊弄 TypeScript 检查
- 对于有长期维护价值的项目，今天出错好过明天出错
  - hard code 越来越多导致容错率越来越低

---

# 鸵鸟策略 · 依赖永不更新

- 祖传 lock-file
  - 一个 lock-file 用 X 年
- 不升级到无法升级
  - case 1
    - npm7 使用了 lock-file v2，使用后 lock-file 的大量变更，没有信心升级
    - 谁升级，谁背锅
    - 项目被锁在 node12 + npm6 时代
  - case 2
    - TS 版本被锁死
    - 配套代码工具，例如 lint 等工具受到牵连
- 理想状态
  - 去掉 lock-file 之后项目可以能正确的跑起来

<img src="/lockdiff.png" style="position: fixed; top: 10vh; height: 400px; right: 100px;"/>

---

# 鸵鸟策略 · 项目是别人的的心态

- 小明被借调到某项目，实现 feature
  - feature 紧急
  - 小明的其他排期不会顺延
- 请问小明是应该
  - 了解项目架构，达成一个 80 分的开发，用 ❤️ 发电
    - 自己会多花点时间
      - 减少出现奇怪 bug 的可能
      - 让后面接手的开发者更容易维护
  - 糊一坨代码，达成一个 60 分的开发，赶紧赶下个项目的排期
    - 自己省几个小时
      - 让后面的人看不懂发生了啥（未知时间成本）
      - 有一定概率让后续维护者重构
      - 有一定概率出现一个价值数小时的珍稀 BUG

---

# 鸵鸟策略 · 治理方式 · 小结

<v-click>

- 不积累无必要的技术债
  - 例：TypeScript `any`

</v-click>

<v-click>

- 定期清理技术债
  - 例：lock-file

</v-click>

<v-click>

- 以尽可能**自动化**的方式落地代码质量审查
  - 例：建立 eslint 规则，而不是“代码规范”

</v-click>

<v-click>

- 培养项目的主导者
  - 频繁变动的人员会导致大量无效沟通
  - 不要重复 Brooks 法则（人月神话）：向进度落后的项目中增加人手，只会使进度更加落后

</v-click>

<v-click>

- 着眼于游戏规则
  - SCRUM 中的概念：不要着眼个人，着眼于游戏规则

</v-click>

---

# 目录

- 引言
- Vue 相关可维护性问题与治理方案
- 通用可维护性问题与治理方案
- **补充 & 总结**

---

# 一些补充

- 测试
  - 测重要且稳定的功能，不要盲目追求 TDD
  - 如果你不知道为什么要 TDD，不要 TDD
  - 概念最终要解决实际问题
- 研发效能衡量机制
  - 太难，没有银弹
  - 概念到实践断层
  - 实事求是

---

# 一些建议

- （对于多数业务项目）可维护性看起来是技术问题，实际上技术问题只是皮
- 确定立场
- 因地制宜
- 项目需要质量的闸门
  - 能落地为真实规则的，必须要落地为规则
  - CR，至少一个真正能控制项目质量的 owner
- 把事情做好比做对更难

---

# 书籍推荐

### 持续交付 2.0

<br />

<img src="/cd2.jpeg" style="height: 60%;" />
