---
title: 06-React-Controlled and Un_ Components
mathjax: false
date: 2023-01-01 21:18:43
summary: 受控组件与非受控组件
categories: React
tags:
  - react
---

# 表单中的受控组件与非受控组件

受控与非受控: 实质还是状态与非状态属性的差别

# 非受控组件

指组件(也包括标签)是通过 `ref` 或 `event` 事件监听用户操作的, 即没有绑定到 state 属性上, 其变化只能在组件内使用

如果用户操作要引起组件的重新渲染，则需要在处理函数中更新其它的 `state` 来间接渲染，这样就可能导致其余 `state` 的不必要更新
- 比如搜索功能，`input` 标签中内容的变化如果没有绑定 `state` 的话，那么就需要额外创建一个 `search result` 的 `state` 来维护和展示
- 因此涉及到这种操作与重渲染，还是应当用 `state` 也就是受控组件的方式维护

## defaultValue 的由来

非受控组件不会直接引发重渲染，因此像 `input` 标签的 `value` 属性一旦是非受控的，那么其值(静态值作为默认或外部传入的值)是不会被更新的，因为重新渲染后新值都会被 原来的值覆盖

因此针对这种情况引入了 `defaultValue`，在组件初次挂在时完成一次渲染，之后不再渲染，保证了非受控组件支持默认值的功能

## Demo

```jsx
export default class App extends Component {
    username = React.createRef()
    render() {
        return (
            <div>
                <h3> 这些标签都被 React 重置了默认值 value 就是一个固定的传入属性, 不会被更新 </h3>
                <input type="text" ref={this.username} value="cjc"></input>
                <button onClick={() => {
                    console.log(this.username.current.value)
                }}> 登陆 </button>
                <button onClick={() => {
                    this.username.current.value = ""
                }}> 重置 </button>

                <h3> 给出了 defaultvalue 模拟原先的 HTML 交互 </h3>
                <input type="text" ref={this.username} defaultValue="cjc"></input>
                <button onClick={() => {
                    console.log(this.username.current.value)
                }}> 登陆 </button>
                <button onClick={() => {
                    this.username.current.value = ""
                }}> 重置 </button>

            <h3> 但是这个组件没法和其它组件交互, 因为输入框的值没有与 state 绑定导致组件不会更新 </h3>
            </div>
        )
    }
}

```

# 受控组件

组件中涉及用户的交互操作由 state 直接绑定，用户特定的操作会触发相应的渲染，更加的灵活，但相对而言性能需求也更大
- `input` 标签中，可以将 `value` 等属性与 `state` 进行绑定，但 `value` 的监听处理 `hook` 为 `onChange`

## Demo

```jsx
import React, { Component } from 'react'

export default class App extends Component {

    state = {
        todoItems: [],
        id : 3,
        todoname: ""
    }

    render() {
        const todoItems_view = this.state.todoItems.map(
            (item, index) => (
            <li key={item.id} onDoubleClick={() => this.handleTodoRemoveClick(index)}>
                <input type="checkbox" checked={item.checked} onChange={() => {this.handleChecked(index)}}/>
                <span dangerouslySetInnerHTML={
                    {
                        __html: item.todoname
                    }
                } style={{textDecoration: item.checked ? "line-through" : ""}}>
                </span>
                {/* {item.todoname} */}
            </li>
            )
        )
        return (
            <div>
                <div>
                    <input type="text" ref={this.todoInput} placeholder="Input you todo thing..." value={this.state.todoname} onChange={(event) => {
                        this.setState({todoname: event.target.value})
                    }}></input>
                    <button onClick={() => this.handleTodoAddClick()}> ADD </button>
                </div>
                <div>
                    <ul>
                        {todoItems_view}
                    </ul>
                    {/* 条件渲染 */}
                    {/* {this.state.todoItems.length === 0 ? <h3>暂无代办事项</h3> : null}  */}
                    {this.state.todoItems.length === 0 && <h3>暂无代办事项</h3>} 
                </div>
            </div>
        )
    }

    handleTodoAddClick = () => {
        
        const {todoItems, id, todoname} = this.state

        if(todoname.trim() === "") {
            alert("Wrong TodoName")
            return
        }

        this.setState({
            todoItems: [...todoItems, {id:id + 1, todoname: todoname, checked: false}],
            id: id + 1,
            todoname: ""
        })
    }

    handleTodoRemoveClick = (index) => {
        const {todoItems} = this.state
        const todoItems_ = [...todoItems]

        todoItems_.splice(index, 1)
        this.setState({
            todoItems: [ ...todoItems_ ],
        })
    }

    handleChecked = (index) => {
        const todoItems_ = [...this.state.todoItems]
        todoItems_[index].checked = !todoItems_[index].checked
        this.setState({todoItems: todoItems_})
    }
}
```