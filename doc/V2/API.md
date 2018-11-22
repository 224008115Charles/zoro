### API

#### 快速接入

zoro核心只有6个api，你可以在很短的时间内上手框架，只需要如下简单的几步即可快速开始开发

```js
import zoro from '@opcjs/zoro'
import { Provider } from 'react-redux'

const app = zoro() // 创建redux app
app.use(...) // 注入插件
app.model(...) // 注入model
const store = app.start() // 启动redux app

ReactDom.render(
  document.getElementById('root'),
  <Provider store={store}>
    <APP>
  </Provider>
)
```

---

#### `const app = zoro(opt)`

* `opt.initialState` `<Object>` 初始化redux state，类似于redux preloadState
* `opt.extraMiddlewares` `<Array>` 添加额外的redux middleware
* `opt.extraEnhancers` `<Array>` 添加额外的redux enhancer
* `opt.onEffect(action)` `<Function>` 监听用户调用model effect，版本2.1.0支持异步阻塞
* `opt.onAction(action)` `<Function>` 监听用户调用reducer action，版本2.2.0之后不再包含effect
* `opt.onReducer(modelName, reducer)` `<Function>` 用于对reducer进行扩展，比如redux-undo

> 以redux-undo为例, 后期会考虑以plugin形式实现此功能
```js
import { undoable } from 'redux-undo'
const app = zoro({
  onReducer(modelName, reducer) {
    return undoable(reducer)
  },
})
```
* `opt.onSetup(params)` `<Function>` redux应用启动时执行
  * `params.put(action)` `<Function>` 触发一个effect或者reducer
  * `params.select(handler)` `<Function>` 获取当前state的值，handler参数为state，未传递handler，默认直接获取整个全局state
> 调用app.start()触发该事件，或者手动调用
app.start(false);
app.setup();
* `opt.onError(e)` `<Function>` 执行effect发生错误时触发，用于处理全局错误，比如请求错误提示

#### `app.model()` `<Function>`

注册model到应用，可以注册一个或者多个，多个时参数为数组

#### `app.use()` `<Function>`

注册plugin，可以注册一个或者多个，多个时参数为数组，use推荐在model之前调用

#### `app.start(setup)` `<Function>`

`setup` `<Boolean>` 是否立即启动初始设置，默认为true，传false，则不会自动调用setup()，用于用户自定义控制启动点

启动应用，并返回redux store

#### `app.setup()` `<Function>`

立即启动初始化，该函数必须在app.start(false)执行之后执行

#### `app.intercept` `<Object>` 建立拦截器
  * app.intercept.action() 建立action的拦截器，不包含effect，不支持异步
  * app.intercept.effect() 建立effect的拦截器，支持异步

```js
app.intercept.action(function(action, { store, NAMESPACE_DIVIDER }) {
  // 添加你的逻辑

  // 返回值为改变之后的action，类型需保持不变，不返回默认不改变action
})

app.intercept.effect(async function(action, { store, NAMESPACE_DIVIDER }) {
  await delay(1000)

  // 添加你的逻辑

  // 返回值为改变之后的action，类型需保持不变，不返回默认不改变action
})
```

#### `dispatcher` `<Object>`
dispatcher中定义了所有model的action触发器
```js
import { dispatcher } from 'roronoa-zoro'

dispatcher.modelNamespace.queryData(payload, meta, error)
```

#### `connectComponent(mapStateToData, mapDispatchToMethod)(componentConfig)`
```js
import { connectComponent, createDispatcher } from 'roronoa-zoro'

const model = createDispatcher('model')
const mapStateToData = state => ({ ... })

const config = connectComponent(mapStateToData)({
  methods: {
    handleQueryData(event) {
      model.queryData(...)
    },
  },
})

Component(config)

```

---

#### `model` `<Object>`

* `model.namespace` `<String>` model命名空间
* `model.state` `<Any>` model默认的state值
* `model.setup(params)` `<Function>` 启动时的初始化动作
  * `params.put(action)` 发起一个reducer或者effect动作，调用自身model方法时，无需跟model namespace
  * `params.select(handler)` `<Function>` 获取当前state的值，handler参数为本model中的state，未传递handler，默认直接获取该model整个state
  * `params.selectAll(handler)` `<Function>` 获取当前state的值，handler参数为全局state，未传递handler，默认直接获取整个全局state
* `model.reducers` `<Object>` 定义model的reducers
* `model.effects` `<Object>` 定义model的effects

#### `model reducer`

```js
export default {
  namespace: 'model',
  state: 0,
  reducers: {
    add(action, state) {
      return state + 1
    },
  },
}
```

#### `model effect`

```js
const delay = function(time) {
  return new Promise((resolve, reject) => {
    setTimeout(time, resolve)
  })
}

export default {
  namespace: 'model',
  state: 0,
  reducers: {
    add(action, state) {
      return state + 1
    },
  },
  effects: {
    // 异步effect
    async delayAdd(action, { put, select, selectAll }) {
      await delay(1000)
      put({ type: 'add' })
    },
    // effect中调用异步effect
    async delayToDelayAdd(action, { put }) {
      await delay(1000)
      await put({ type: 'delayAdd' })
    },
    // 同步effect
    noneDelay(action, { put }) {
      put({ type: add })
    },
  },
}
```
