<center><h1>react状态管理器redux-toolkit简单介绍</h1></center>

## Redux 简单回顾一下
<img src="https://redux.js.org/assets/images/ReduxDataFlowDiagram-49fa8c3968371d9ef6f2a1486bd40a26.gif" width="600"/>

- 用户点击或者发生某些事件触发 action ，例如dispatch({type: 'counter/increment'})
- state 和 action 通过 reducer函数产生新的 state
- store得到新的state通知所有订阅的 UI 进行更新
- 订阅store数据的 UI 组件检查它们需要的状态是否更改
- 每个看到其数据已更改的组件都会强制使用新数据重新渲染

## Redux Toolkit 诞生的目的
Redux Toolkit旨在成为编写Redux逻辑的标准方式。解决Redux三个常见问题：

- 配置store太复杂
- 添加很多包才能让 Redux 做任何有用的事情
- Redux 太多样板代码

## 使用 create-react-app
```javascript
# Redux + Plain JS template
npx create-react-app my-app --template redux

# Redux + TypeScript template
npx create-react-app my-app --template redux-typescript
```
## 现有项目
```javascript
npm install redux react-redux @reduxjs/toolkit -D
```

## Redux Toolkit 包括以下 API:
- configureStore(): 包装createStore以提供简化的配置选项和良好的默认值。它可以自动组合你的 slice reducer，添加你提供的任何 Redux 中间件，redux-thunk默认包含，并启用 Redux DevTools Extension。
- createReducer()：这使您可以为 case reducer 函数提供操作类型的查找表，而不是编写 switch 语句。此外，它自动使用该immer库让您使用普通的可变代码编写更简单的不可变更新，例如state.todos[3].completed = true.
- createAction()：为给定的动作类型字符串生成动作创建函数。该函数本身已toString()定义，因此可以使用它来代替类型常量。
- createSlice()：接受reducer函数的对象、切片名称和初始状态值，并自动生成切片reducer，并带有相应的动作创建者和动作类型。
- createAsyncThunk: 接受一个动作类型字符串和一个返回承诺的函数，并生成一个pending/fulfilled/rejected基于该承诺分派动作类型的 thunk
- createEntityAdapter: 生成一组可重用的 reducer 和 selector 来管理 store 中的规范化数据
- 重新选择库中的createSelector实用程序，重新导出以方便使用。

## 案例
### 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f1b1652dcbd471bb6864904c54cd291.gif#pic_center)

### counterSlice.ts
```javascript
import { createAsyncThunk, createSlice, PayloadAction } from '@reduxjs/toolkit'
import { fetchCount } from './counterAPI'

export interface CounterState {
  value: number
  status: 'idle' | 'loading' | 'failed'
}

const initialState: CounterState = {
  value: 0,
  status: 'idle'
}

export const incrementAsync = createAsyncThunk('counter/fetchCount', async (amount: number) => {
  const response = await fetchCount(amount)
  return response.data
})

export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  // 这里内置了immer，可以直接操作state
  // reducers是reduces 和 actions 组合在一起的对象
  reducers: {
    increment: state => {
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload
    }
  },
  //这里对异步操作incrementAsync异步状态的处理
  extraReducers: builder => {
    builder
      .addCase(incrementAsync.pending, state => {
        state.status = 'loading'
      })
      .addCase(incrementAsync.fulfilled, (state, action) => {
        state.status = 'idle'
        state.value += action.payload
      })
      .addCase(incrementAsync.rejected, state => {
        state.status = 'failed'
      })
  }
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions
export default counterSlice.reducer
```

### store.ts
```javascript
import { configureStore, getDefaultMiddleware } from '@reduxjs/toolkit'
import logger from 'redux-logger'
import rootReducer from './rootReducer'
import { useDispatch } from 'react-redux'

import CounterSlice from '@pages/counter/counterSlice'

const middleware = getDefaultMiddleware().concat(logger)

export const store = configureStore({
  reducer: {
  	counter: CounterSlice
  },
  middleware,
  devTools: process.env.NODE_ENV !== 'production'
})

export type AppDispatch = typeof store.dispatch
export type RootState = ReturnType<typeof store.getState>
export const useAppDispatch = () => useDispatch<AppDispatch>()
```

### Counter.tsx
```javascript
import React, { useState } from 'react'
import { useSelector } from 'react-redux'
import { useAppDispatch, RootState } from '@store'
import { decrement, increment, incrementByAmount, incrementAsync } from './counterSlice'

import styles from './Counter.module.less'

const Counter = () => {
  //useSelector简化redux的模板代码
  const { value } = useSelector((state: RootState) => state.counter)

  const dispatch = useAppDispatch()
  const [incrementAmount, setIncrementAmount] = useState('2')

  const incrementValue = Number(incrementAmount) || 0

  return (
    <div>
      <div className={styles.row}>
        <button className={styles.button} aria-label="Decrement value" onClick={() => dispatch(decrement())}>
          -
        </button>
        <span className={styles.value}>{value}</span>
        <button className={styles.button} aria-label="Increment value" onClick={() => dispatch(increment())}>
          +
        </button>
      </div>
      <div className={styles.row}>
        <input
          className={styles.textbox}
          aria-label="Set increment amount"
          value={incrementAmount}
          onChange={e => setIncrementAmount(e.target.value)}
        />
        <button className={styles.button} onClick={() => dispatch(incrementByAmount(incrementValue))}>
          Add Amount
        </button>
        <button className={styles.asyncButton} onClick={() => dispatch(incrementAsync(incrementValue))}>
          Add Async
        </button>
      </div>
    </div>
  )
}

export default Counter
```

### index.tsx
```javascript
import React from 'react'
import { createRoot } from 'react-dom/client'
import { HashRouter, Routes, Route } from 'react-router-dom'
import { Provider } from 'react-redux'
import { store } from '@store'
import App from './app'

import './assets/style/index.less'

const container = document.getElementById('root')
const root = createRoot(container)

root.render(
  <Provider store={store}>
    <HashRouter>
      <Routes>
        <Route key={'counter-demo'} path="/counter" element={<CounterDemo />} />
      </Routes>
    </HashRouter>
  </Provider>
)
```

## 结束语
- 👀 目前专注于前端
- ⚙️ 在react、react-native开发方面有丰富的经验
- 🔭 最近在学习安卓，有自己的开源安卓项目，集成react-native热更新功能
- ❤️ 思考、学习、编码和健身
- 如果文章对您有帮助，三连支持一下～O(∩_∩)O谢谢！


