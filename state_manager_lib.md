# 状态管理库

## 为什么需要状态管理

组件之间需要共享一些状态

## Flux

Flux 其实是一种思想，跟 MVC，MVVM 类似。

Flux 将一个应用分成了 4 个部分： View Action Dispatcher Store

Flux 最大的特点就是 数据都是单向流动的

FLux 的缺点：

- 一个应用可以拥有多个 Store
- 多个 Store 之间可能有依赖关系
- Store 封装了数据还有处理数据的逻辑。

## Redux

所有的数据都放在一个 Store 里面，并且 store 里面的状态不能直接修改，且每次返回一个新的 state

Action 必须有一个 type 属性

reducer 必须是纯函数（无副作用，对于相同的输入，永远都只会有相同的输出，不会影响外部的变量，也不会被外部变量影响，不得改写参数）

### Redux vs Flux

| Redux      | Flux           |
| ---------- | -------------- |
| 单一数据源 | 多数据源       |
| state 只读 | 可以任意修改   |
| 纯函数修改 | 不一定是纯函数 |
| 单向数据流 | 单向数据流     |

### 中间件

koa 中的 中间件「洋葱模型」

同步 & 异步

Reducer 作为纯函数，肯定不能承担异步操作，那样会被外部 IO 干扰。Action 呢，是一个纯对象，放不了操作。想来想去，只能在 View 中发送 Action 的时候，加上异步操作了。

一般情况下，请求开始时，dispatch 一个请求开始 Action，触发 State 更新为 “正在请求” 状态，View 重新渲染，比如展现个 Loading。

请求结束后，如果成功，dispatch 一个请求成功 Action，隐藏掉 Loading，把新的数据更新到 State；如果失败，dispatch 一个请求失败 Action，隐藏掉 Loading，给个失败提示。

显然，用 Redux 处理异步，可以自己写中间件来处理，大多数人会选择一些现成的支持异步处理的中间件。比如 `redux-thunk` 或 `redux-promise` 。

#### redux-thunk

thunk 没有做太多的封装，把大部分自主权交给了用户

#### redux-promise

做了一些简化，成功失败手动 dispatch 被封装成自动了

封装少，自由度高，但是代码就会变复杂；封装多，代码变简单了，但是自由度就会变差。redux-thunk 和 redux-promise 刚好就是代表这两个面。

## Vuex

每一个 Vuex 里面有一个全局的 Store，包含应用的状态 State，这个 State 只是需要在组件中共享的数据，不用放所有的 State。这个 State 是单一的，和 Redux 类似，所以一个应用仅会包含一个 Store 实例。

State 不能直接改，需要通过一个约定的方式，这个方式在 Vuex 里面叫做 mutation，更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)

触发 mutation 事件的方式不是直接调用，而要通过 store.commit 方法

Vuex 加入了 Action 这个东西来处理异步

## react-redux

为了简单处理 Redux 和 React UI 的绑定，一般通过 react-redux 和 React 配合使用

Redux 将 React 组件分为容器型组件和展示型组件，容器型组件一般通过 connect 函数生成，它订阅了全局状态的变化，通过 mapStateToProps 函数，可以对全局状态进行过滤，而展示型组件不直接从 global state 获取数据，其数据来源于父组件。mapDispatchToProps 把 UI 组件的事件映射到 dispatch 方法

## Redux-saga

redux-saga 把异步获取数据这类的操作都叫做副作用（Side Effect），它的目标就是把这些副作用管理好，让他们执行更高效，测试更简单，在处理故障时更容易。

ES6 Generator

- 异步数据获取的相关业务逻辑放在了单独的 saga.js 中，不再是掺杂在 action.js 或 component.js 中。
- dispatch 的参数是标准的 action，没有魔法。
- saga 代码采用类似同步的方式书写，代码变得更易读。
- 代码异常/请求失败 都可以直接通过 try/catch 语法直接捕获处理。
- `*` 很容易测试，如果是 thunk 的 Promise，测试的话就需要不停的 mock 不同的数据。

## Dva

dva 首先是一个基于 redux 和 redux-saga 的数据流方案，然后为了简化开发体验，dva 还额外内置了 react-router 和 fetch，所以也可以理解为一个轻量级的应用框架。

dva 做的事情很简单，就是让这些东西可以写到一起，不用分开来写了。

```js
app.model({
  namespace: "count",
  state: {
    record: 0,
    current: 0,
  },
  reducers: {
    add(state) {
      const newCurrent = state.current + 1;
      return {
        ...state,
        record: newCurrent > state.record ? newCurrent : state.record,
        current: newCurrent,
      };
    },
    minus(state) {
      return { ...state, current: state.current - 1 };
    },
  },
  effects: {
    *add(action, { call, put }) {
      yield call(delay, 1000);
      yield put({ type: "minus" });
    },
  },
  subscriptions: {
    keyboardWatcher({ dispatch }) {
      key("⌘+up, ctrl+up", () => {
        dispatch({ type: "add" });
      });
    },
  },
});
```

## Mobx

_任何源自应用状态的东西都应该自动地获得_

MobX 允许有多个 store，而且这些 store 里的 state 可以直接修改，不用像 Redux 那样每次还返回个新的。

### Redux

```js
import React, { Component } from "react";
import { createStore, bindActionCreators } from "redux";
import { Provider, connect } from "react-redux";
// ①action types
const COUNTER_ADD = "counter_add";
const COUNTER_DEC = "counter_dec";
const initialState = { a: 0 };
// ②reducers
function reducers(state = initialState, action) {
  switch (action.type) {
    case COUNTER_ADD:
      return { ...state, a: state.a + 1 };
    case COUNTER_DEC:
      return { ...state, a: state.a - 1 };
    default:
      return state;
  }
}
// ③action creator
const incA = () => ({ type: COUNTER_ADD });
const decA = () => ({ type: COUNTER_DEC });
const Actions = { incA, decA };
class Demo extends Component {
  render() {
    const { store, actions } = this.props;
    return (
      <div>
        <p>a = {store.a}</p>
        <p>
          <button className="ui-btn" onClick={actions.incA}>
            增加 a
          </button>
          <button className="ui-btn" onClick={actions.decA}>
            减少 a
          </button>
        </p>
      </div>
    );
  }
}
// ④将state、actions 映射到组件 props
const mapStateToProps = (state) => ({ store: state });
const mapDispatchToProps = (dispatch) => ({
  // ⑤bindActionCreators 简化 dispatch
  actions: bindActionCreators(Actions, dispatch),
});
// ⑥connect产生容器组件
const Root = connect(mapStateToProps, mapDispatchToProps)(Demo);
const store = createStore(reducers);
export default class App extends Component {
  render() {
    return (
      <Provider store={store}>
        <Root />
      </Provider>
    );
  }
}
```

### Mobx

```js
import React, { Component } from "react";
import { observable, action } from "mobx";
import { Provider, observer, inject } from "mobx-react";
// 定义数据结构
class Store {
  // ① 使用 observable decorator
  @observable a = 0;
}
// 定义对数据的操作
class Actions {
  constructor({ store }) {
    this.store = store;
  }
  // ② 使用 action decorator
  @action
  incA = () => {
    this.store.a++;
  };
  @action
  decA = () => {
    this.store.a--;
  };
}
// ③实例化单一数据源
const store = new Store();
// ④实例化 actions，并且和 store 进行关联
const actions = new Actions({ store });
// inject 向业务组件注入 store，actions，和 Provider 配合使用
// ⑤ 使用 inject decorator 和 observer decorator
@inject("store", "actions")
@observer
class Demo extends Component {
  render() {
    const { store, actions } = this.props;
    return (
      <div>
        <p>a = {store.a}</p>
        <p>
          <button className="ui-btn" onClick={actions.incA}>
            增加 a
          </button>
          <button className="ui-btn" onClick={actions.decA}>
            减少 a
          </button>
        </p>
      </div>
    );
  }
}
class App extends Component {
  render() {
    // ⑥使用Provider 在被 inject 的子组件里，可以通过 props.store props.actions 访问
    return (
      <Provider store={store} actions={actions}>
        <Demo />
      </Provider>
    );
  }
}
export default App;
```
