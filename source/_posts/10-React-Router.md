---
title: 10-React-Router
mathjax: false
date: 2023-01-02 18:59:27
summary: React 路由相关介绍
categories: React
tags:
  - react
---

# React 路由

SPA，single page web application，单个页面 Web 应用，整个应用只有一个完整的页面（单页面多组件），点击页面中的连接不会刷新页面，只会做页面的局部更新
- 数据都需要通过 ajax 请求获取，并在前端异步展现

前端路由: 即 React 中的路由, 实质 path 与 组件的映射
- 点击链接，变换url; 检测到 url 变化提取 path; 根据 path 更新目标组件
- 当 path 变化为 `/demo` 时, 就会加载其绑定的组件, 如 `<Demo/>` 组件

后端路由: 即 Django 中的 urlpatterns, 实现的是 path 与函数的映射，用于处理客户端提交的请求
- 当后端服务器接收到一个请求时，根据请求路径找到匹配的路由，调用路由中的处理函数返回响应数据

# react-router-dom V6

用于实现一个 SPA 项目，基于 React 的项目基本都会用到此库

```shell
yarn add react-router-dom
npm i react-router-dom
```

- React-Router 自带 404 功能: 应用程序在渲染、加载数据或执行数据突变时抛出错误时, React Router都会捕捉到错误并呈现默认错误界面

## Quick Start

1. 导入相关包

```jsx
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";
```

2. 建立根路由

```jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
  },
]);
ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

3. 根据需求添加子路由

```jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
      },
    ],
  },
]);
```

4. 配置子路由组件在根组件中的位置: 默认使用 <Outlet /> 表示匹配的子路由组件

```jsx
import { Outlet } from "react-router-dom";

export default function Root() {
  return (
    <>
      {/* all the other elements */}
      <div id="detail">
        <Outlet />
      </div>
    </>
  );
}
```

## index 默认路由

当路由在父路由层级并未涉及到子路由时, `<Outlet />` 组件内容就是空的, 十分不好看, 因此可以在子路由配置中加入 index 路由表示默认匹配的子路由组件
- `index: true`: That tells the router to match and render this route **when the user is at the parent route's exact path**

1. 编辑子路由组件

```jsx
export default function Index() {
  return (
    <p id="zero-state">
      This is a demo for React Router.
      <br />
      Check out{" "}
      <a href="https://reactrouter.com">
        the docs at reactrouter.com
      </a>
      .
    </p>
  );
}
```

2. 进行路由配置

```jsx
import Index from "./routes/index";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    action: rootAction,
    children: [
      { 
        index: true, 
        element: <Index /> 
      },
      /* existing routes */
    ],
  },
]);
```

## 导航

### Link

导航区的 a 标签改为 Link 标签，to 属性指路由变化

```jsx
import { Outlet, Link } from "react-router-dom";
<ul>
    <li>
        <Link to={`contacts/1`}>Your Name</Link>
    </li>
    <li>
        <Link to={`contacts/2`}>Your Friend</Link>
    </li>
</ul>
```

### Navlink

导航栏常常需要高亮当前选中的导航选项, React-Router 封装了 Navlink 快速实现该需求

1. 用 Navlink 替换 Link: className 中可以传入一个函数, 函数中接收 isActive 与 isPending 属性, 根据该属性返回 className 名称

- isActive: 表示当前 url 对应的就是该 Link
- isPending: 表示相关数据正在加载, 还没完全显示该 Link 对应的组件

```jsx
import { NavLink } from "react-router-dom";
export default function Root() {
  return (
    <nav>
        {contacts.length ? (
        <ul>
            {contacts.map((contact) => (
            <li key={contact.id}>
                <NavLink
                to={`contacts/${contact.id}`}
                className={({ isActive, isPending }) =>
                    isActive
                    ? "active"
                    : isPending
                    ? "pending"
                    : ""
                }
                >
                </NavLink>
            </li>
            ))}
        </ul>
        ) : (
        <p>{/* other code */}</p>
        )}
    </nav>
  );
}
```



## 编程式导航

### redirect

重定向, 响应给浏览器的请求

```jsx
import { redirect } from "react-router-dom";

export async function action({ params }) {
  await deleteContact(params.contactId);
  return redirect("/");
}
```

### navigate

导航, 由用户交互, 主动发起的导航
- `navigate(-1)`: 返回浏览器历史记录中的一个条目

```jsx
import { useNavigate } from "react-router-dom";
export default function Edit() {
  return (
    <Form method="post" id="contact-form">
      <p>
        <button type="submit">Save</button>
        // type=button: 虽然看起来是多余的，但它是阻止按钮提交表单的 HTML 方式
        <button
          type="button"
          onClick={() => {
            navigate(-1);
          }}
        >
          Cancel
        </button>
      </p>
    </Form>
  );
}
```

## Error Page

当 React-Router 捕捉到错误时, 会从当前路由配置中寻找 errorElement 属性, 如果找不到则层层向上寻找, 直到使用默认的 errorElement 属性

1. 自定义 error page: `touch src/error-page.jsx`

```jsx
import { useRouteError } from "react-router-dom";

export default function ErrorPage() {
  const error = useRouteError();
  console.error(error);

  return (
    <div id="error-page">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <i>{error.statusText || error.message}</i>
      </p>
    </div>
  );
}
```

2. 配置到路由中

```jsx
import ErrorPage from "./error-page";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
  },
]);
```

3. 在任何子路由页面中可以通过 `throw new Error("error message!");` 来触发 errorElement 的加载
- Response 封装了更精细的控制

```jsx
export async function loader({ params }) {
  const contact = await getContact(params.contactId);
  if (!contact) {
    throw new Response("", {
      status: 404,
      statusText: "Not Found",
    });
  }
  return contact;
}
```

### 子路由中的全局错误响应

子路由中捕捉到错误后, 除非子路由有自己的 errorElement 否则都会到根路由处理; 而为每个子路由添加相同的 errorElement 太不优雅了, 因此 React-Router 出手了
- 允许无 path 的路由, 仅仅和 UI 渲染相关的路由, 这样向上寻找 errorElement 就不会都到根路由中了

```jsx
createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    action: rootAction,
    errorElement: <ErrorPage />,
    children: [
      {
        errorElement: <ErrorPage />,
        children: [
          { index: true, element: <Index /> },
          {
            path: "contacts/:contactId",
            element: <Contact />,
            loader: contactLoader,
            action: contactAction,
          },
          /* the rest of the routes */
        ],
      },
    ],
  },
]);
```


## 数据加载

一个路由的跳转往往意味着新数据的加载, React-Router 针对此进行了处理, 引入了 loader 属性与 useLoaderData 钩子函数

1. 配置异步加载请求函数: 如果路由中存在动态字段 (参数), 可以从函数中的 params 参数中获取; 属性名要与路由中动态字段的名称一致

```jsx
// root.jsx
import { getContacts } from "../contacts";
export async function loader() {
  const contacts = await getContacts();
  return { contacts };
}
// contact.jsx
import { getContacts } from "../contacts";
export async function loader({ params }) {
  const contacts = await getContacts(params.contactId);
  return { contacts };
}
```

2. 配置相关路由的 loader 属性

```jsx
// 防止重名
import Root, { loader as rootLoader } from "./routes/root";
import Contact, { loader as contactLoader } from "./routes/contact";
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
        loader: contactLoader,
      },
    ],
  },
]);
```

3. 在目标组件中获取加载的数据

```jsx
import { useLoaderData } from "react-router-dom";
export default function Root() {
  const { contacts } = useLoaderData();
  return (
    <>
      <div id="sidebar">
        <nav>
          {contacts.length ? (
            <ul>
              {contacts.map((contact) => (...))}
            </ul>
          ) : (
            <p> <i>No contacts</i> </p>
          )}
        </nav>

        {/* other code */}
      </div>
    </>
  );
```

## 请求发布

与数据加载对应的就是用户操作触发的 (GET/POST) 请求, React-Router 针对该类请求也进行了处理, 引入了 action 属性与 Form 组件
- React-Router Action 触发后会自动验证 loader 数据, 并更新相关的 useLoaderData Hook, 从而触发组件的更新

1. 在组件中引入 Form 组件并编写异步的 action 请求处理函数 (与服务器进行交互)

```jsx
import { Form } from "react-router-dom";
import { getContacts, createContact } from "../contacts";

export async function action() {
  const contact = await createContact();
  return { contact };
}

export default function Root() {
  return (
    <Form method="post">
        <button type="submit">New</button>
    </Form>
  );
}
```

2. 在相关路由中配置 action 属性

```jsx
import Root, { action as rootAction } from "./routes/root";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    loader: rootLoader,
    action: rootAction,
    children: [
      {
        path: "contacts/:contactId",
        element: <Contact />,
      },
    ],
  },
]);
```

### 带参数的请求

常用的应用场景中 Form 触发请求都带有参数

1. action 接收 request 与 params 属性: request 获取 form 中的数据, params 获取 url 中的动态字段

- formData 中的属性名与 form 组件中的 name 属性对应, 通过 get 方法获取属性值: `formData.get(name)`
- 通过 ` Object.fromEntries` 可以快速将 formData 封装为一个对象
- redirect helper just makes it easier to return a response that tells the app to change locations.

```jsx
import { Form, useLoaderData, redirect } from "react-router-dom";
import { updateContact } from "../contacts";

export async function action({ request, params }) {
  const formData = await request.formData();
  const updates = Object.fromEntries(formData);
  await updateContact(params.contactId, updates);
  return redirect(`/contacts/${params.contactId}`);
}
```

2. 配置相关组件的 action 即可

3. 获取 url 中的 GET 参数可以通过 request 对象

```jsx
const url = new URL(request.url);
const q = url.searchParams.get("q");
```

### 多请求配置

一个页面中完全可以存在多个 (GET/POST) 请求出口 (Form), 类似于传统 form 标签:
- Form 默认的 action 是提交给当前路由的, 触发当前路由的 action 属性对应的钩子函数
- Form 也支持指定 action 属性, 触发目标路由的 action 属性对应的钩子函数

1. 指定 Form 的 action
```jsx
// contact.jsx
<Form
  method="post"
  action="destroy" // 可以是相对路径, 自动在当前路由的基础上拼接
  onSubmit={(event) => {
    if (
      !confirm(
        "Please confirm you want to delete this record."
      )
    ) {
      event.preventDefault();
    }
  }}
>
  <button type="submit">Delete</button>
</Form>
```

2. 创建响应的钩子函数

```jsx
// destroy.jsx
import { redirect } from "react-router-dom";

export async function action({ params }) {
  await deleteContact(params.contactId);
  return redirect("/");
}
```

3. 完善路由配置

```jsx
import { action as destroyAction } from "./routes/destroy";

const router = createBrowserRouter([
  {
    path: "/",
    /* existing root route props */
    children: [
      /* existing routes */
      {
        path: "contacts/:contactId/destroy",
        action: destroyAction,
      },
    ],
  },
]);
```

## 编程式请求发布

React-Router 的 Form 表单请求的发布不仅仅可以和 Button 绑定, 还可以和事件绑定, 这就需要编程式的请求发布了: `useSubmit`
- event.currentTarget.form: 表示的是 input 组件的父组件

```jsx
import { useSubmit } from "react-router-dom";

export default function Root() {
  const { contacts, q } = useLoaderData();
  const submit = useSubmit();
  return (
    <>
        <Form id="search-form" role="search">
            <input
                id="q"
                aria-label="Search contacts"
                placeholder="Search"
                type="search"
                name="q"
                defaultValue={q}
                onChange={(event) => {
                    submit(event.currentTarget.form);
                }}
            />
        </Form>
    </>
  );
}
```

这样可以实现输入的即时性反馈, 但是在历史记录中却会存在大量的误判, 这样的需求可以在事件处理函数中通过逻辑判断解决:

![](https://raw.githubusercontent.com/Coming98/pictures/main/202301061249957.png)

```jsx
onChange={(event) => {
    const isFirstSearch = q == null;
    submit(event.currentTarget.form, {
        replace: !isFirstSearch,
    });
}}
```

## 全局加载 UI

在实际的重定向或路由间跳转请求中, 可能因为异步的数据加载导致页面出现卡顿, 因此可以使用 useNavigation 钩子获取当前的导航状态, 根据状态切换样式, 让用户体验更加舒适
- navigation.state: 返回的状态有 idle(), submitting(), loading(数据正在加载)
- navigation.location: will show up when the app is navigating to a new URL and loading the data for it; It then goes away when there is no pending navigation anymore; 因此在加载中时可以设定加载样式

TODO: 通通用 navigation.state 解决不行吗？ 可能 location 更精细的控制搜索处的 pending 样式？

```jsx
import { useNavigation } from "react-router-dom";
export default function Root() {
  const { contacts } = useLoaderData();
  const navigation = useNavigation();

  // 
  const searching = navigation.location && new URLSearchParams(navigation.location.search).has("q");

  return (
    <>
      <div
        id="detail"
        className={
          navigation.state === "loading" ? "loading" : ""
        }
      >
        <Outlet />
      </div>
    </>
  );
}
```

## Mutations Without Navigation
 
TODO: useFetcher & Optimistic UI

# Refs

- [Getting Start](https://reactrouter.com/en/main/start/tutorial#mutation-discussion)



# TODO: 路由懒加载

路由条目太多, 导致加载拥塞; 路由懒加载将实现按需加载路由

```jsx
const LazyLoad = (path) => {
    const Comp = React.lazy(() => import(`../views/${path}`))
    return (
        <React.Suspense fallback={<> Loading... </>}>
            <Comp/>
        </React.Suspense>
    )
}
```

```jsx
<Route path="/cinema" element={LazyLoad("Cinema")} />
```


# 常见问题

## 静态样式丢失问题

样式引入时不能以当前相对路径来引入，路由机制会导致资源找不到
```html
<link rel="stylesheet" href="./css/bootstrap.css">
```

解决方案：
1. `href="%PUBLIC_URL%/css/bootstrap.css"` 使用 `%PUBLIC_URL%` 构造绝对路径引入 public 下资源文件
2. `href="/css/bootstrap.css"` 使用 `/` 以根路径形式访问
3. 使用 `<HashRouter>` 锚路由被 `#` 隔离了正常的 url 不会对相对路径产生影响了（不推荐）