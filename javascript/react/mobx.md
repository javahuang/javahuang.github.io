# mobx

使用步骤：

1. **state** 定义 state 对象并把他变成 observable 对象
2. **action** 使用 actions 来更新 state
3. **derivation** 创建**派生**(derivations)来自动响应 state 的变化
   1. 使用 `computed` 派生出来数据
   2. 使用**反应**(reaction)处理 model 的副作用，Model side effects using reactions。
        > In short, reactions bridge the worlds of reactive and imperative programming. 弥补了反应式和响应式编程？
   3. 反应式 react 组件，Reactive React components（使用 `observer` 包装的组件）
   4. Custom reactions，使用 `autorun`、`reaction`、`when` 函数来创建

MobX uses a uni-directional data flow where actions change the state, which in turn updates all affected views.
mbox 使用单向数据流，动作改变状态，状态会更新所有受影响的视图

![mobx](./assets/mobx.png)

## 如何理解 reactivity

> MobX reacts to any existing **observable property** that is **read** **during** the execution of a **tracked function**.
> MobX对在执行跟踪函数期间读取的任何现有可观察属性做出反应。

- "reading" dereferencing an object's property. 即通过 "." 或者 "[]"来获取对象的属性，MobX 跟踪的是**属性访问**，而不是属性值
- "tracked function" 跟踪函数有如下形式
  - `computed` 表达式
  - 使用 `observer` 包装的 react 函数式组件的 rendering 行为
  - react class 组件的 `render()` 方法
  - 作为第一个参数传递给 `autorun`、`reaction`、`when` 的函数
- "during" 表示仅跟踪函数执行期间读取的那些可观察值

MobX will not react to:

- 在跟踪函数之外来读取 observables 的值
- 在异步调用代码块里面来读取 observables 的值
- 在跟踪函数里面来访问越界函数索引

    ```js
        // 如果一开始的时候 message.lines 的 length =0，则后续操作 message.likes 都不会触发 autorun 函数的执行
        autorun(() => {
        console.log(message.likes[0])
        })
    ```

## actions

### 使用函数来包装 action

- action.bound
    > The action.bound annotation can be used to automatically bind a method to the correct instance, so that this is always correctly bound inside the function.
- runInAction
    > Use this utility to create a temporarily action that is immediately invoked


## Reactions

The goal of reactions is to model side effects that happen automatically.Their significance is in creating consumers for your observable state and automatically running side effects whenever something relevant changes.

可以使用如下的三个函数来创建 reactions

### Autorun

`autorun(effect: (reaction) => void)`

autorun 的函数在初始化之后会立即执行一次

### Reaction

`reaction(() => value, (value, previousValue, reaction) => { sideEffect }, options?).`

sideEffect 初始化之后不会立即运行，只有当表达式（参数 1）返回新的值时才会执行

### When

- `when(predicate: () => boolean, effect?: () => void, options?)`
- `when(predicate: () => boolean, options?): Promise`

when observes and runs the given predicate function until it returns true. Once that happens, the given effect function is executed and the autorunner is disposed.

```js
async function() {
    await when(() => that.isVisible)
    // etc...
}
```

### Options

- **name** This string is used as a debug name(比如可以在 MobX 开发者工具显示)
- **fireImmediately**（reaction） 是否初始化的时候立即执行 sideEffect，默认值是 false
- **delay**（autorun,reaction） Number of milliseconds that can be used to throttle the effect function. 默认值是 0，不开启节流操作
- **timeout**（when）设置 when 的等待时间，超时之后会 reject/throw
- **onError** 默认情况下在 reaction 内部出现的异常只会被记录，但是并不会抛出去。这个参数可以覆盖默认行为，通常可以用来作为全局异常处理或者通过 configure 来完全禁止错误捕获
- **scheduler**（autorun，reaction） Set a custom scheduler to determine how re-running the autorun function should be scheduled. `{ scheduler: run => { setTimeout(run, 1000) }}`
- **equals**（reaction） Set to comparer.default by default. comparer 函数用来比较 data function 前后两次的值，只有返回 false 才会继续调用 effect function

## API

- `makeObservable` 通常用在 class 的 constructor 上，将 class 变成 observable
- **observable** 标记一个属性或者对象，表示该属性或者对象是 observable 的
  - `makeObservable()`
- **computed** 表示计算属性
- **action** 通常用在函数上，用来改变 state，通常不允许在 action 之外来改变 state
  - `flow` 包装 async / await 使用更简单，使用 generator function

    ```js
    // * instead async
    fetchProjects = flow(function* (this: Store) {
        // yield instead await
        const projects = yield fetchGithubProjectsSomehow()
    ```

- **Computed** 从其他 observables 里面来获取值，值延迟计算，值会被缓存，并且只有在他依赖的 observable 改变时才会重新计算
- `getDependencyTree` 获取依赖书
- `autorun()` 

```js
import { makeObservable, observable, computed, action, makeAutoObservable } from "mobx"

class Doubler {
    value

    constructor(value) {
        makeObservable(this, {
            value: observable,
            double: computed,
            increment: action
        })
        this.value = value
    }

    get double() {
        return this.value * 2
    }

    increment() {
        this.value++
    }
}

function createDoubler(value) {
    return makeAutoObservable({
        value, // observable
        get double() { // computed属性
            return this.value * 2
        },
        increment() { // autoAction
            this.value++
        }
    })
}
```

基础概念：

```js
import { makeObservable, observable, computed } from "mobx"

class TodoList {
    todos = []
    get unfinishedTodoCount() {
        return this.todos.filter(todo => !todo.finished).length
    }
    constructor(todos) {
        makeObservable(this, { // 将当前对象变成被观察者
            todos: observable, // 观察属性
            unfinishedTodoCount: computed // 计算属性
        })
        this.todos = todos;
    }
}
```

## [react integration](https://mobx.js.org/react-integration.html)

The `observer` HoC automatically subscribes React components to any `observables` that are used during rendering. As a result, components will automatically re-render when relevant observables change.
Reading observables deeply is fine, complex expression like `todos[0].author.displayName` work out of the box.
通过 observer 将组件和 observable 绑定，observable 里面相关 observable 的属性和对象如果发生了改变（支持多层嵌套），observer 将会 rerender。

有两种使用 observable 的方式，一种是通过 prop 的方式传入，一种是直接在组件内部通过 state 使用（不推荐）。两种方式，组件都需要使用 observer 包装。

[sample](https://codesandbox.io/s/minimal-observer-forked-dhjxr?file=/src/index.tsx)

注意事项：

 1. 不能将 observable class 的 observable 属性直接传递给通过 observer 包装的组件
 2. 将 observable class 的 observable 属性传递给子组件，如果该属性是对象，子组件必须经过 observer 包装，否则即使对象属性变化，该组件也不会 rerender
 3. 在回调组件里面需要使用 \<Observer>
 4. 无需将 `observer` 包装的组件使用 `React.memo` 进一步包装
 5. 当 observer 和其他高阶组件放在一起的时候，需要给 observer 放在最前面
 6. 可以将 `userEffect` 和 `useLocalObservable` 结合在一起

### react 性能优化

mobx 很快，甚至比 redux 还快 😹

- 将组件尽量拆分成尽快多的小组件，然后使用 `observer` 包装，小组件越多，越多的小组件需要 re-render
- 不要使用 array index 作为 key
- dereference values as late as possible 尽早取消引用值

    ```tsx
    // slower, 父组件会 reRender，因为在父组件里面访问了 observable 的属性
    <DisplayName name={person.name} />
    // faster,  直接传入 person displayName 的父组件不用 reRender
    <DisplayName person={person} />
    ```

## mobx-react-lite

This is a lighter version of mobx-react which **supports React functional components only** and as such makes the library slightly faster and smaller (only 1.5kB gzipped).

## 参考

[mobx-cn](https://cn.mobx.js.org/)

[mobx-react-devtools](https://github.com/mobxjs/mobx-react-devtools)

[mobx-react-lite](https://github.com/mobxjs/mobx-react-lite)