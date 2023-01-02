---
title: 11-React-Redux
mathjax: false
date: 2023-01-02 19:10:55
summary: React 中的状态管理思想
categories: React
tags:
  - react
---


# Flux

FLUX 是一种架构思想，是一种模式

![](https://raw.githubusercontent.com/Coming98/pictures/main/202301021901680.png)

# Redux-toolkit

## Quick Start

1. 通过 createSlice 创建状态分片, 配置: initialState, action; 导出 actions 以及需要用的状态

```jsx
import { createSlice } from '@reduxjs/toolkit'

const initialState = {
    show: true,
}

export const slice = createSlice({
    name: 'tabbar',
    initialState,
    reducers: {
        showTabbar: (state) => {
            state.show = true
        },
        hideTabbar: (state) => {
            state.show = false
        }
    },
})

// Action creators are generated for each case reducer function
export const { showTabbar, hideTabbar } = slice.actions
export const selectShow = (state) => {
    console.log(1)
    return state.tabbar.show
}

export default slice.reducer
```

2. 通过 `configureStore` 以及配置的 reducer 创建 store 对象

```jsx
import { configureStore } from '@reduxjs/toolkit'
import tabbarReducer from './tabbarSlice'
import cityReducer from './citySlice'
import cinemaSlice from './cinemaSlice'

export const store = configureStore({
  reducer: {
      tabbar: tabbarReducer,
      city: cityReducer,
      cinema: cinemaSlice
  },
})
```

3. 引入 store 对象, 包装组件

```jsx
import { store } from './05-router/redux/store'
import { Provider } from 'react-redux'
const container = document.getElementById('root');
const root = createRoot(container);
root.render(<Provider store={store}><App /></Provider>);
```

4. 消费者中可以通过 dispatch 发出 action

```jsx
import { useDispatch } from 'react-redux'
import { loadCinemaList } from '../redux/cinemaSlice'
const dispatch = useDispatch()
dispatch(loadCinemaList())
```

5. 消费者可以通过 useSelector 获得状态

```jsx
import { useSelector } from 'react-redux'
import { selectCinemaList } from '../redux/cinemaSlice'
const cinemaList = useSelector(selectCinemaList)
```

## 异步 Trunk 创建

涉及到异步请求时, 我们要管理异步请求的状态并更新 store 中的状态


1. 通过 createAsyncThunk 创建异步 Trunk 并导出: 名字方便管理, 将处理正常时的结果进行返回
```jsx
import axios from "axios";
import { createAsyncThunk } from "@reduxjs/toolkit";

export const loadCinemaList = createAsyncThunk('cinema/loadCinemaList', async () => {
    const response = await axios({
        url: "https://m.maizuo.com/gateway?cityId=110100&ticketFlag=1&k=1237115",
        method: "get",
        headers: {
            'X-Client-Info': '{"a":"3000","ch":"1002","v":"5.2.0","e":"1649749646626811822145537"}',
            'X-Host': 'mall.film-ticket.cinema.list'
        }
    })
    return response.data.data.cinemas
})
```

2. 在状态切片的 `extraReducers` 属性中配置 asyncThunk 的状态管理: `fulfilled` 会获取处理结果

```jsx
export const cinemaSlice = createSlice({
    name: 'cinema',
    initialState,
    reducers: {
        setCinemaList: (state, action) => {
            state.cinemaList = action.payload.cinemaList
            
        },
    },
    extraReducers: {
        [loadCinemaList.pending] : () => {
            console.log("Pending...")
        },
        [loadCinemaList.fulfilled] : (state, {payload}) => {
            console.log("Success!")
            return { ...state, cinemaList: payload}
        },
        [loadCinemaList.rejected] : () => {
            console.log("Rejected!")
        },
    }
})
```

# Redux

仅仅是 Flux 的一种实现, 用于应用状态的管理, 用一个单独的状态向量树(State), 维护一整个应用的状态

主要有三大原则: 
1. state 以单一对象存储在 store 对象中
2. state 只读, 每次都返回一个新的对象
3. 使用纯函数(对外界没有副作用的函数) reducer 执行 state 更新

## Quick Start

`store` 对象用于统一接口, 然后 `reducer` 负责处理状态变化

1. Installation: `npm i redux`

2. Create `store` Object and actions:

```jsx
import {createStore} from 'redux'

const reducer = (prevState = {
    'headerName': null
}, action) => {
    switch(action.type) {
        case 'header':
            const state = {...prevState}
            state.headerName = action.headerName
            return state
        default:
            return prevState
    }
}

const headerStore = createStore(reducer);

export default headerStore
```

actions

```jsx
function setHeaderNameAction(headerName) {
    return {
        type: 'header',
        headerName: headerName
    }
}

export {setHeaderNameAction}
```

3. 订阅与发布
```jsx
import {setHeaderNameAction} from '../../../redux/actionCreator/HeaderAction'
import headerStore from '../../../redux/HeaderStore'

const setHeaderName = useCallback((headerName) => {
    return () => {
        headerStore.dispatch(setHeaderNameAction(headerName))
    }
}, [])

import headerStore from '../../redux/HeaderStore'
useEffect(() => {
    headerStore.subscribe(() => {
        setHeaderName(headerStore.getState().headerName)
    })
}, [])
```

## CombineReducers

如果不同的 action 所处理的属性之间没有联系, 我们可以把 Reducer 函数拆分, 不同的函数负责处理不同属性, 最终把它们合并成一个大的 Reducer 即可

```jsx
import {createStore, combineReducers} from 'redux'

const reducer = combineReducers({
    CityReducer,
    TabbarReducer,
})

const store = createStore(reducer)

// 返回状态
store.getState().CityReducer.XXX
```

## 异步请求的处理

## redux-trunk

通过 action 与 中间件 进行异步请求处理, 然后下发 action

中间件: 是用于处理异步请求的

```jsx
export default function thunkMiddleware({ dispatch, getState }) {
    return next => action => 
        typeof action === 'function' ? 
            action(dispatch, getState) :
            next(action);
}
```

这时如果 `action` 是一个函数的话, 那么就会执行, 并将 `dispatch` 与 `getState` 函数接口给出, 供异步函数处理完成后回调使用

### Quick Start


1. Installation: `npm i redux-thunk`

2. 应用中间件
```jsx
import reduxTrunk from 'redux-trunk'
import {applyMiddleware, combineReducers, createStore} from 'redux'
const store = createStore(reducer, applyMiddleware(reduxThunk));
```

3. 获取 `dispatch` 与 `getState` 接口

```jsx
function getCinemaListAction() {
    return (dispatch) => {
        axios({
            url: '',
            method: 'get',
            headers: {
                '',:''
            }
        }).then(res => {
            dispatch({
                type: 'change-list',
                payload: 'res.data.data.cinemas'
            })
        })
    }
}
```

## redux-promise

1. Installation: `npm i redux-promise`

2. 应用中间件
```jsx
import reduxPromise from 'redux-promise'
import {applyMiddleware, combineReducers, createStore} from 'redux'
const store = createStore(reducer, applyMiddleware(reduxThunk, reduxPromise)); // 支持多个中间件
```

3. 这时支持在 `dispatch` 时传入函数并支持 `promise` 返回

```jsx
function getCinemaListAction() {
    return axios({
        url: '',
        method: 'get',
        headers: {
            '',:''
        }
    }).then(res => {
        return {
            type: 'change-list',
            payload: 'res.data.data.cinemas'
        }
    })
}
```