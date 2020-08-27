# rc-redux-model 👋

简体中文 | [English](./README.en.md)

![](https://img.shields.io/badge/author-彭道宽-important.svg)
![](https://img.shields.io/badge/team-SugarTurboS-critical.svg)
![](https://img.shields.io/badge/category-Redux-blue.svg)
![](https://img.shields.io/badge/category-React-yellowgreen.svg)
![](https://img.shields.io/badge/rc--redux--modal-v1.0.3-green.svg)
![](https://img.shields.io/badge/redux-^4.0.1-inactive.svg)
![](https://img.shields.io/badge/license-MIT-yellow.svg)

> ✍️ 提供一种较为舒适的数据状态管理书写方式，让你简洁优雅的去开发；内部自动生成 action, 只需记住一个 action，可以修改任意的 state 值，方便简洁，释放你的 ⌨️ CV 键～

## ✨ 特性

- 轻巧简洁，写数据管理就跟写 `dva` 一样舒服
- 异步请求由用户自行处理，内部支持 call 方法，可调用提供的方法进行转发，该方法返回的是一个 Promise
- 参考 `redux-thunk`，内部实现独立的中间件，所有的 action 都是异步 action
- 提供默认行为 action，调用此 action ，可以修改任意的 state 值，解决你重复性写 action 、reducers 问题
- 内置 `seamless-immutable` ，只需开启配置，让你的数据不可变
- 默认检测不规范的赋值与类型错误，让你的数据更加健壮

## ⏳ 前世今生

- [why rc-redux-model and what's rc-redux-model](https://github.com/PDKSophia/rc-redux-model/issues/1)
- [rc-redux-model design ideas and practices](https://github.com/PDKSophia/rc-redux-model/issues/2)

## 🧱 强调说明

**rc-redux-model 出发点在于解决繁琐重复的工作，store 文件分散，state 类型和赋值错误的问题，为此，对于跟我一样的用户，提供了一个写状态管理较为[舒服]的书写方式，大部分情况下兼容原先项目**~

- 为了解决[store 文件分散]，参考借鉴了 dva 写状态管理的方式，一个 model 中写所有的 `action、state、reducers`
- 为了解决[繁琐重复的工作]，提供默认的 action，用户不需要自己写修改 state 的 action，只需要调用默认提供的 `[model.namespace/setStore]` 即可，从而将一些重复的代码从 model 文件中剔除
- 为了解决[state 类型和赋值错误]，在每次修改 state 值时候，都会进行检测，如果不通过则报错提示

## ⛏ 安装

```bash
npm install --save rc-redux-model
```

## 🚀 使用

如有疑问，看下边的相关说明~ 同时对于如何在项目中使用，[👉 可以点这里](https://github.com/PDKSophia/rc-redux-model/issues/3)

#### 提供默认 action，无需额外多写 action/reducers

原先，我们想要修改 state 值，需要在 reducers 中定义好 action，但现在， `rc-redux-model` 提供默认的 action 用于修改，所以在 model 中，只需要定义 state 值即可

```js
export default {
  namespace: 'appModel',
  state: {
    value1: '',
  },
}
```

在页面中，只需要调用默认的 `[model.namespace/setStore]` 就可以修改 state 里的值了，美滋滋，不需要你自己在 action、reducers 去写很多重复的修改 state 代码

```js
this.props.dispatch({
  type: 'appModel/setStore',
  payload: {
    key: 'value1',
    values: 'appModel_state_value1',
  },
})
```

### 复杂且真实的例子

1. 新建一个 model 文件夹，该文件夹下新增一个 userModel.js

```js
import adapter from '@common/adapter'

const userModel = {
  namespace: 'userModel',
  openSeamlessImmutable: false,
  state: {
    classId: '',
    studentList: [],
    userInfo: {
      name: 'PDK',
    },
  },
  action: {
    // demo: 发起一个异步请求，修改 global.model的 loading 状态，异步请求结束之后，修改 reducers
    // 此异步逻辑，可自行处理，如果采用 call，那么会通过 Promise 包裹一层帮你转发
    fetchUserInfo: async ({ dispatch, call }) => {
      // 请求前，将 globalModel 中的 loading 置为 true
      dispatch({
        type: 'globalModel/changeLoadingStatus',
        payload: true,
      })
      let res = await call(adapter.callAPI, params)
      if (res.code === 0) {
        dispatch({
          type: 'userModel/setStore',
          payload: {
            key: 'userInfo',
            values: res.data,
          },
        })
        // 请求结束，将 globalModel 中的 loading 置为 false
        dispatch({
          type: 'globalModel/changeLoadingStatus',
          payload: false,
        })
      }
      return res
    },
  },
}

export default userModel
```

2. 聚集所有的 models，请注意，这里导出的是一个 **数组**

```js
// model/index.js
import userModel from './userModel'

export default [userModel]
```

3. 处理 models, 注册中间件

```js
// createStore.js
import { createStore, applyMiddleware, combineReducers } from 'redux'
import models from './models'
import RcReduxModel from 'rc-redux-model'

const reduxModel = new RcReduxModel(models)

const reducerList = combineReducers(reduxModel.reducers)
return createStore(reducerList, applyMiddleware(reduxModel.thunk))
```

4. 在页面中使用

请注意，这里的 action 都是异步 action，内部中间件的实现方式参考 `redux-thunk`，也就是说，我们 `dispatch` 一个 `action` 都是对应的一个方法，看代码 :

```js
class MyComponents extends React.PureComponent {
  componentDidMount() {
    // demo: 发起一个异步请求，修改 global.model的 loading 状态，异步请求结束之后，修改 reducers
    // 具体的请求，在 model.action 中自己写，支持 Promise，之前需要 callback 回调请求后的数据，现在直接 then 获取
    this.props
      .dispatch({
        type: 'userModel/fetchUserInfo',
      })
      .then((res) => {
        console.log(res)
      })
      .catch((err) => {
        console.log(err)
      })

    // demo1: 调用自动生成的默认action，直接修改 state.userInfo 的值 （推荐此方法）
    this.props.dispatch({
      type: 'userModel/setStore',
      payload: {
        key: 'userInfo',
        values: {
          name: 'setStore_name',
        },
      },
    })
    // demo2: 调用自动生成的默认action，直接修改 state.classId 的值 （推荐此方法）
    this.props.dispatch({
      type: 'userModel/setStore',
      payload: {
        key: 'classId',
        values: 'sugarTeam2020',
      },
    })
  }
}
```

## hooks ?

hooks 的出现，让我们看到了处理复杂且重复逻辑的曙光，那么问题来了，在 hooks 中能不能用 `rc-redux-model` ，我想说 : “想啥呢，一个是 react 的特性，一个是 redux 的中间件， 冲突吗？”

```js
// Usage with React Redux: Typing the useSelector hook & Typing the useDispatch hook
// https://redux.js.org/recipes/usage-with-typescript#usage-with-react-redux
import { useDispatch } from 'redux'

export function useFetchUserInfo() {
  const dispatch = useDispatch()
  return async (userId: string) => {
    // 这里我选择自己处理异步，异步请求完后，再把数据传到 reducer 中
    const res = await callAPI(userId)
    if (res.code === 200) {
      dispatch({
        type: 'userModel/setStore',
        payload: {
          key: 'userInfo',
          values: res.data,
        },
      })
    }
  }
}
```

## 🔥 相关说明

在使用之前，请了解几个知识点，然后再看`完整例子`即可快速上手使用 !!! [👉 如果你想了解它是怎么来的，点这里](https://github.com/PDKSophia/rc-redux-model/issues/1)

#### 如何定义一个 model 并自动注册 action 及 reducers ?

_每一个 model 必须带有 namespace、state_，action 与 reducers 可不写，如需开启 `immutable`，需配置 `openSeamlessImmutable = true`，一个完整的 model 结构如下

```js
export default {
  namespace: '[your model.namespace]',
  state: {
    testA: '',
    testB: false,
    testC: [],
    testD: {},
  },
}
```

`rc-redux-model` 会根据你的 state，每一个 state 的字段都会自动注册一个修改此 state 的 action，从而释放你键盘上的 ⌨️ CV 键， 例如 :

```
state: {
  userName: 'oldValue'
}
```

那么会自动为你注册一个 action，action 名以 `set${stateName}` 格式，如你的 stateName 为 : userName，那么会自动注册的 action 为 : `setuserName`

```
action: {
  setuserName: ({ dispatch, getState, commit, call, currentAction }) => {}
}
```

你只要在组件中调用此 action 即可修改 state 值 （📢 不推荐使用这种 action 进行修改 state 值，推荐使用 **setStore**）

```js
this.props.dispatch({
  type: 'userModel/setuserName',
  payload: {
    userName: 'newValue',
  },
})
```

问题来了，当 state 中的值很多(比如有几十个)，那么为用户自动注册几十个 action，用户在使用上是否需要记住每一个 state 对应的 action 呢？这肯定是极其不合理的，所以一开始是提供一个默认的 action ，用于修改所有的 state 值 ...

随之而来的问题是，如果只提供一个 action，那么所有修改 State 的值都走的这个 action.type，在 [redux-devtools-extension](https://github.com/zalmoxisus/redux-devtools-extension) 中，会看不到具体的相对信息记录(因为都是同一个 action)，最终，还是提供一个默认的 action，此 action 会根据用户提供的 `payload.key`，从而转发至对应的 action 中。

> ✨ 对外提供统一默认 action，方面用户使用；对内根据 key，进行真实 action 的转发

```js
this.props.dispatch({
  type: '[model.namespace]/setStore',
  payload: {
    key: [model.state.key]  // 你要修改的state key
    values: [your values] // 你要修改的值
  }
})
```

🌟 所有修改 state 的 action，**都通过 setStore 来发**，不必担心在 redux devtools 中找不到，此 action 只是会根据你的 key，转发对应的 action 而已

#### 如何发送一个 action ?

一个 action 由 type、payload 组成，type 的命名规则为 : `[model.namespace / actionName]`

```js
// 下边是 namespace = appModel ，actionName = fetchUserList 的例子
const action = {
  type: 'appModel/fetchUserList',
}
// 发起这个 action
this.props.dispatch(action)
```

请注意，这里的每一个 action 都是 function, 也就是说，处理 `同步action` 的思路跟处理 `异步action`是一样的，如果你不明白，[👉 请移步这里](https://github.com/PDKSophia/rc-redux-model/issues/2)

#### 异步请求由谁处理 ?

在 `model.action` 中，每一个 action 都是 function，它的回调参数为 :

- dispatch : store 提供的 API，你可以调用 `dispatch` 继续分发 action
- getState : store 提供的 API，通过该 API 你可以得到最新的 state
- currentAction : 当前你 `this.props.dispatch` 的 action，你可以从这里拿到 `type` 和 `payload`
- call : 替你转发请求，同时会使用 Promise 包裹，当然你可以自己写异步逻辑
- commit : 接收一个 action，该方法用于 dispatch action 到 reducers ，从而修改 state 值

> 可以自己处理异步，再通过调用默认提供的 [model.namespace/setStore] 这个 action 进行修改 state 值

#### 如何在组件中获取 state 值？

请注意，rc-redux-model 是一个中间件，并且大部分情况下，能够在你现有的项目中兼容，所以获取 state 的方式，还是跟你原来在组件中如何获取 state 一样

一般来讲，我们的项目都会安装 `react-redux` 库，然后通过 `connect` 获取 state 上的值（没什么变化，你之前怎么写，现在就怎么写）

```js
class appComponent extends React.Component {
  componentDidMount() {
    // 发起 action，将loading状态改为true
    this.props.dispatch({
      type: 'appModel/fetchLoadingStatus',
      payload: {
        loadingStatus: true,
      },
    })
  }

  render() {
    const { loadingStatus } = this.props.appModel
    console.log(loadingStatus) // true
  }
}

const mapStateToProps = (state) => {
  return {
    appModel: state.appModel,
    reportTaskInfo: state.reportModel.taskInfo, // 其他 model 的值
  }
}

export default connect(mapStateToProps)(appComponent)
```

如果很不幸，你项目中没安装 `react-redux`，那么你只能在每一个组件中，引入这个 store，然后通过 `store.getState()` 拿到 state 值了

但是这种方式的缺陷就是，你要确保你的 state 是最新的，也就是你改完 state 值之后，需要重新 `store.getState()` 拿一下最新的值，这是比较麻烦的

```js
import store from '@your_folder/store' // 这个store就是你使用 Redux.createStore API 生成的store

class appComponent extends React.Component {
  constructor() {
    this.appState = store.getState()['appModel']
  }
}
```

#### 数据不可变的(Immutable) ?

在函数式编程语言中，数据是不可变的，所有的数据一旦产生，就不能改变其中的值，如果要改变，那就只能生成一个新的数据。如果有看过 redux 源码的小伙伴一定会知道，为什么每次都要返回一个新的 state，如果没听过，[👉 可以看下这篇文章](https://juejin.im/post/6844904183426973703)

目前 rc-redux-model 内部集成了 `seamless-immutable`，提供一个 model 配置参数 `openSeamlessImmutable`，默认为 false，请注意，如果你的 state 是 Immutable，而在 model 中不设置此配置，那么会报错 !!!

```js
// 使用 seamless-immutable

import Immutable from 'seamless-immutable'

export default {
  namespace: 'appModel',
  state: Immutable({
    username: '',
  }),
  openSeamlessImmutable: true, // 必须开启此配置
}
```

#### 类型正确性 ？

不可避免，有时在 `model.state` 中定义好某个值的类型，但在改的时候却将其改为另一个类型，例如 :

```js
export default {
  namespace: 'userModel',
  state: {
    name: '', // 这里定义 name 为 string 类型
  },
}
```

但在修改此 state value 时，传递的确是一个非 string 类型的值

```js
this.props.dispatch({
  type: 'userModel/setStore',
  payload: {
    key: 'name',
    values: {}, // 这里name 变成了object
  },
})
```

这其实是不合理的，在 rc-redux-model 中，会判断 `state[key]` 中的类型与 payload 传入的类型进行比较，如果类型不相等，报错提示

所有修改 state 的值，前提是 : 该值已经在 state 中定义，以下情况也会报错提示

```js
export default {
  namespace: 'userModel',
  state: {
    name: '', // 这里只定义 state 中存在 name
  },
}
```

此时想修改 state 中的另一属性值

```js
this.props.dispatch({
  type: 'userModel/setStore',
  payload: {
    key: 'age',
    values: 18, // 这里想修改 age 属性的值
  },
})
```

极度不合理，因为你在 state 中并没有声明此属性， rc-redux-model 会默认帮你做检测

## API

每一个 model 接收 5 个属性，具体如下

| 参数                  | 说明                       | 类型    | 默认值 |
| --------------------- | -------------------------- | ------- | ------ |
| namespace             | 必须，且唯一               | string  | -      |
| state                 | 数据状态，必须             | object  | {}     |
| action                | action，非必须             | object  | -      |
| reducers              | reducer，非必须            | object  | -      |
| openSeamlessImmutable | 是否开启 Immutable，非必须 | boolean | false  |

## 提供的默认 Action

```js
 @desc 注册生成默认的action
 @summary 使用方式

 this.props.dispatch({
   type: '[model.namespace]/setStore',
   payload: {
     key: [model.state.key]
     values: [your values]
   }
 })
```

## Maintainers

[@PDKSophia](https://github.com/PDKSophia)

[@SugarTurboS](https://github.com/SugarTurboS)

## Contributing

PRs accepted.

## License

MIT © 2020 PDKSophia/SugarTurboS

---

This README was generated with ❤️ by [readme-md-generator](https://github.com/kefranabg/readme-md-generator)
