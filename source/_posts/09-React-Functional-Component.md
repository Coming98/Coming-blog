---
title: 09-React-Functional Component
mathjax: false
date: 2023-01-02 17:05:25
summary: 函数式组件钩子函数的介绍
categories: React
tags:
  - react
---

# 函数式组件

最简单的 React 组件形式，默认没有声明周期，没有 state 等一系列类组件所具有的功能
- 支持 props 属性

```jsx
function DemoFunctionalComponent(props) {
    return (
        <div>
            <h1> Hello World </h1>
        </div>
    )
}
```

## hooks

函数式组件的钩子函数, hooks, 扩展函数式组件的功能
- 因为类组件存在一些缺陷: 
  - 高阶组件为了复用, 导致代码层级复杂; 
  - 生命周期复杂; 
  - 无状态组件若需要状态再改成类组件所需成本高

# 钩子函数

## useState

实现类组件中 state 的功能

### Quick Start

1、创建自定义状态：会以数组的形式返回状态变量，以及状态修改接口

```jsx
const [money, setMoney] = useState(0);
```

2、使用状态修改接口函数更新状态：
```jsx
const decrease = () => {
    setMoney(money - 1)
}
```

### 自定义 useState

useState 只是一个通用的定义 state 的接口，我们可以在 useState 的基础上进行二次封装，重新定义其维护的对象，以及选择性的暴露修改接口

```jsx
export function useMoney() {
    // 创建状态，得到状态及状态维护函数
    const [money, setMoney] = useState(0);

    const decrease = () => {
        setMoney(money - 1)
    }

    const increase = () => {
        setMoney(money + 1)
    }

    const reset = () => {
        setMoney(0)
    }

    return { money, decrease, increase, reset };
}
```

## useReducer

useState 是在 useReducer 的基础上的二次封装，useReducer 更加适合复杂逻辑，采用指令的形式是实现逻辑的处理
- 本质实现状态管理的分离

### Quick Start

1、首选需要准备指令处理函数 `xxxHandler`, 接收状态与指令名称

```jsx
function moneyReducerHandler(state, action) {
    switch (action) {
        case 'decrease': 
            return state - 1;
        case 'increase':
            return state + 1;
        case 'reset':
            return 0;
        default:
            throw new Error('Wrong Action Name!');
    }
}
```

2、随后通过 `useReducer` Hook 实现状态的声明以及指令分发接口的暴露：第一个参数为处理函数, 第二个参数为初始状态
```jsx
const [money, dispatch] = useReducer(moneyReducerHandler, 0)
```

3、最后手动实现功能与指令分发的匹配: 可以看出实现了状态更改接口的封装与保护

```jsx
const decrease = () => {
    dispatch('decrease');
}

const increase = () => {
    dispatch('increase');
}

const reset = () => {
    dispatch('reset');
}
```

4、将功能暴露出去供开发者使用
```jsx
return { money, decrease, increase, reset };
```

Tips: 配合 useContext 可以将 `money` 与 `dispatch` 传递出去给子组件使用

## useEffect

模拟 componentDidMount 与 componentUpdate 以及 componentWillUnmount 这三个生命周期

- 参数 1 是回调函数
- 参数 2 是对 state 变量的监控, 类型式列表, 其中的 state 状态变化后回调函数将执行


### 模拟 componetDidMount

只在组件加载时执行一次：第二个参数为空列表

```jsx
useEffect(() => {
    console.log("Hello")
}, [])
```

### 模拟 compontDidUpdate

在第二个参数中要传入监视的 state 变量；如果没有第二个参数，则表示每次组件渲染（所有来源）都会执行该函数

```jsx
useEffect(() => {
    moneyTimer = setInterval(() => {
        setCount(c => c + money);
    }, 1000)

}, [money]);
```

### 模拟 componentWillUnmount

useEffect 中定义清除函数（返回的一个函数）即可实现当组件销毁时执行目标代码

```jsx
useEffect(() => {
    moneyTimer = setInterval(() => {
        setCount((c) => {return c + money});
    }, 1000)

    return () => {
        clearInterval(moneyTimer);
    }
}, [money]);
```

## useLayoutEffect

useLayoutEffect: 会在 React 完成 DOM 更新后马上同步调用代码，阻塞页面渲染
- useEffect 会在整个页面渲染完才会调用代码
- 官方推荐优先使用 `useEffect()`
- 如果涉及 `DOM` 操作，则为了避免页面抖动应使用 `useLayoutEffect()`

## useRef

函数式组件重新渲染后，其中的局部变量会重新被初始化和赋值，这就导致了一些功能逻辑上实现的复杂性，因此使用 useRef 可以实现局部变量的缓存，使得重新渲染后仍然存储之前的状态值

### Quick Start

实质上是自定义选择变量是否要被重新渲染，后续类似的还有自定义选择对象是否被重新渲染以及在什么条件下才重新渲染 `useCallback`（类似 useEffect）

```jsx
import { useEffect } from 'react'

let timerId = useRef(null);
timerId.current = setInterval(() => {
    setTimerCount(c => c + 1);
}, 100);
```

### 实现组件引用
```jsx
const inputTag = useRef()

const handleChange = (event) => {
    console.log(inputTag.current.value);
    console.log(event.target.value)
}

<input type="text" ref={inputTag} onChange={handleChange}>

```


### 与 useState 的区别

1. useRef 定义的相当于是渲染之外的全局变量，该变量的变化不会引起组件的重新渲染，而 useState 会
2. useRef 变量的更改是同步实时的，但是 useState 变量的更改是异步的，需要等到组件重新渲染后查看更新

因此我认为 useState 更常用于状态的展示，useRef 应更常用于逻辑的处理

## useCallback

涉及到功能函数, 想要继承之前的内容可以使用 `useCallback`, 函数也可以依赖于某个状态而选择性更新

```jsx
const start = useCallback(() => {
    timerId.current = setInterval(() => {
        setTimerCount(c => c + timerStep);
    }, 100);
    setTimerInfo({title: '暂停', operation: 'pause'});
}, [timerStep])

const pause = useCallback(() => {
    clearInterval(timerId.current);
    timerId.current = null;
    
    setTimerInfo({title: '继续', operation: 'start'});
}, [])

const reset = useCallback(() => {
    if(timerId.current) {
        clearInterval(timerId.current);
        timerId.current = null;
    }
    setTimerCount(0);
    setTimerInfo({title: '开始', operation: 'start'});
}, [])
```

## useMemo

与 `useCallback` 类似：都会在第一次渲染的时候执行，之后会在其依赖的变量发生改变时再次执行，并且这两个 `hooks` 都返回缓存的值
- `useMemo` 会执行传入的函数并将结果返回, 如果两次结果一致, useMemo 将能避免复杂的函数逻辑再次执行

```jsx
const getCinemaList = useMemo(
    () => {
        cinemaList.filter(...)
    }, [cinemaList]
)

```

## useContext

实现组件间通信:

1、先定义全局生产者空间：
```jsx
const GlobalContext = React.createContext()
```

2、父组件中使用 `Provider` 暴露接口给待通信的子组件
```jsx
<GlobalContext.Provider value={{...}}>
    // 子组件
</GlobalContext.Provide>
```

3、子组件中通过 `useContext()` 获取生产者提供的接口
```jsx
const valueObj = useContext(GlobalContext);
```

应用：useReducer 的接口通常需要传递给子组件使用，因此通常与 useContext 配合使用

## Custom hooks

在外部将一些 hooks 整合封装并暴露必要的接口