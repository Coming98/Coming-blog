---
title: 07-React-组件间通信
mathjax: false
date: 2023-01-01 22:30:01
summary: 组件间通信方式的整理
categories: React
tags:
  - react
---

# 父子状态通信

## props

组件关系必须是父子关系, 如果关系层次较深, 则实现较为复杂
- 父传子: 父组件通过设定 props 传递给子组件控制属性
- 子通父: 父组件通过设定 props 传递给子组件 callbasck 函数


```jsx
class Navbar extends Component {
    render() {
        const { handleSidebar } = this.props
        return (
            <div style={{background: "red"}}>
                <button onClick={
                    () => {handleSidebar()}
                }> Click </button>
                <span> Navbar </span>
            </div>
        )
    }
}

class Sidebar extends Component {
    render() {
        const { show } = this.props
        return(
            <div style={{background: "yellow", transition: "1.5", display: show ? " " : "none" }}>
                <ul>
                    <li> CJC </li>
                    <li> CJC </li>
                    <li> CJC </li>
                    <li> CJC </li>
                    <li> CJC </li>
                </ul>
            </div>
        )
    }
}

export default class App extends Component {
    
    state = {
        show: false
    }

    render() {
        return (
            <div>
                {/* 父传子 */}
                <Navbar handleSidebar={this.handleSidebar}></Navbar>
                <Sidebar show={this.state.show}></Sidebar>
            </div>
        )
    }

    handleSidebar = () => {
        this.setState({show: !this.state.show})
    }
}

```

## ref 引用

不推荐使用: ref 有点强取强夺, 子组件内定义状态与修改接口, 父组件通过 ref 获取子组件的引用进而获取其内置状态值

```jsx
class Field extends Component {

    state = { value: "" }
    
    clear() { this.setState({value: ""}) }

    render() {
        const {label, type } = this.props
        const { value } = this.state
        return (
            <div style={{background: "yellow"}}>
                <label>{label}</label>
                <input type={type} value={value} onChange={(event) => {this.setState({value: event.target.value})}}></input>
            </div>
        )
    }
}

export default class App extends Component {

    username = React.createRef()
    password = React.createRef()

    render() {
        return (
            <div>
                <h1> 登陆页面 </h1>
                <Field label="用户名" type="text" ref={this.username}></Field>
                <Field label="密码" type="password" ref={this.password}></Field>
                <button onClick={() => {
                    console.log(this.username.current.state.value, this.password.current.state.value)
                }}> 登陆 </button>
                <button onClick={() => {
                    this.username.current.clear()
                    this.password.current.clear()
                }}> 取消 </button>
            </div>
        )
   }
}
```

## 中间人模式

可用于同父子的兄弟组件的通信, 实质还是利用父传子, 子通父实现, 让父作为两个组件的中间人
- FileItem 与 App 通信: App 通过设定子组件 FileItem 的点击事件
- App 与 FilmDetail 通信: App 通过将 state.fileDetail 传递给 FileDetail 的 props

```jsx
export default class App extends Component {

    state = {
        filmList : [],
        filmDetail: ""
    }

    componentDidMount() {
        BScroll.use(MouseWheel)
        this.bs = new BScroll('.wrapper', {
            mouseWheel: true
        })
        axios.get('/test.json').then( res => {
            this.setState({filmList: res.data.data.films}, () => {
                this.bs.refresh()
            })
        })
    }


    render() {
        return (
            <div>
                <div className='wrapper'>
                    <div className='content'>
                        {
                            this.state.filmList.map( item => 
                                <FilmItem key={item.filmId} {...item} onClick={(detail) => {
                                    this.setState({filmDetail: detail})
                                }}></FilmItem>    
                            )
                        }
                    </div>
                </div>
                <FilmDetail value={this.state.filmDetail}></FilmDetail>
            </div>
        )
    }
}

class FilmItem extends Component {
    
    render() {
        const { name, poster, grade, synopsis, onClick } = this.props
        return (
            <div className='fileItem' onClick={() => onClick(synopsis)}>
                <img src={poster} alt={name}></img>
                <h4>{name}</h4>
                <div> 观众评分: {grade}</div>
            </div>
        )
    }
}

class FilmDetail extends Component {
    render() {
        return (
            <div className='filmDetail'>
                { this.props.value }
            </div>
        )
    }
}
```

# 发布订阅模式

## pubsub

```shell
yarn add pubsub-js
import PubSub from 'pubsub-js'
```

App 作为顶级 壳，不应该过多的涉及管理子组件之间的状态变化与通信问题，因此使用消息订阅与发布技术实现子组件之间的自发通信
- 不必将信息一级一级的传递下去，可以实现任一组件之间通信
- PubSub，历史悠久，应用广泛

### 订阅

在组件刚挂载上就要完成所有的订阅，一次性定义完成该组件所接收的所有消息类型

```jsx
componentDidMount() {
    this.updateHeaderData = PubSub.subscribe('updateHeaderData', (msgName, data) => {
        this.setState(data)
    })
}
```

### 发布

引入 `PubSub` 后调用其 `publish` 方法

```jsx
PubSub.publish('updateHeaderData', {data: activeName})
```

### 取消订阅

引入 `PubSub` 后调用其 `unsubscribe` 方法，一般在组件卸载时调用

```jsx
componentWillUnmount() {
    // 取消订阅
    PubSub.unsubscribe(this.updateHeaderData)
}
```

## Context 模式

Provider 中提供共享信息以及修改该信息的接口, 消费者修改或读取该信息实现通信

```jsx
export default class App extends Component {

    state = {
        filmList : [],
        detail: ""
    }

    componentDidMount() {
        BScroll.use(MouseWheel)
        this.bs = new BScroll('.wrapper', {
            mouseWheel: true
        })
        axios.get('/test.json').then( res => {
            this.setState({filmList: res.data.data.films}, () => {
                this.bs.refresh()
            })
        })
    }


    render() {
        return (
            // 包裹供应商
            <GlobalContext.Provider value={{
                detail: this.state.detail,
                setDetail: (detail) => this.setState({detail})
            }}>
                <div>
                    <div className='wrapper'>
                        <div className='content'>
                            {
                                this.state.filmList.map( item => 
                                    <FilmItem key={item.filmId} {...item} ></FilmItem>    
                                )
                            }
                        </div>
                    </div>
                    <FilmDetail value="pass"></FilmDetail>
                </div>
            </GlobalContext.Provider>
        )
    }
}

class FilmItem extends Component {
    
    render() {
        const { name, poster, grade, synopsis, setDetail } = this.props
        return (
            <GlobalContext.Consumer>
                {
                    (value) => (
                        <div className='fileItem' onClick={() => value.setDetail(synopsis)}>
                            <img src={poster} alt={name}></img>
                            <h4>{name}</h4>
                            <div> 观众评分: {grade}</div>
                        </div>
                    )
                }
            </GlobalContext.Consumer>
            
        )
    }
}

class FilmDetail extends Component {

    render() {
        return (
            <GlobalContext.Consumer>
                {
                    (value) => {
                        return (
                            <div className='filmDetail'>
                                { value.detail }
                            </div>
                        )
                    }
                }
            </GlobalContext.Consumer>
        )
        
    }
}
```

## 插槽

不推荐: 通过 this.props.children 获取组件之间的内容, 其本质是个数组, 可以通过索引改变展示顺序

```jsx
class Navbar extends Component {
    render() {
        return (
            <div style={{background: "red"}}>
                { this.props.children }
                <span> Navbar </span>
            </div>
        )
    }
}

export default class App extends Component {
    
    state = {
        show: false
    }

    render() {
        return (
            <div>
                {/* 将 Button 作为插槽避免了父子通信 */}
                {/* 但是这样就混淆代码了, 使得代码更加复杂 */}
                <Navbar>
                    <button onClick={() => this.setState({show: !this.state.show})}> Click </button>
                </Navbar>
                <Sidebar show={this.state.show}></Sidebar>
            </div>
        )
    }
}
```