# react-router
直接学的 v6，语法上相对于 v5 是有一些区别的，升级见: [https://github.com/remix-run/react-router/blob/main/docs/upgrading/v5.md](https://github.com/remix-run/react-router/blob/main/docs/upgrading/v5.md)  
`react-router`包含`react-router-dom`和`react-router-native`，web 开发只要引入`reacr-router-dom`即可

#### 安装

`npm install react-router-dom`

#### 加载路由配置

使用`useRoutes`，替代了`react-router-config`

#### 路由跳转

使用`useNavigate`，替代了`useHistory`

```javascript
import { useNavigate } from "react-router-dom";

function App() {
  let navigate = useNavigate();
  function handleClick() {
    navigate("/home");
  }
  function replace() {
    navigate("/home", { replace: true });
  }
  return (
    <div>
      <button onClick={handleClick}>go home</button>
      <button onClick={replace}>replace go home</button>
      <button onClick={() => navigate(-2)}>Go 2 pages back</button>
      <button onClick={() => navigate(-1)}>Go back</button>
      <button onClick={() => navigate(1)}>Go forward</button>
      <button onClick={() => navigate(2)}>Go 2 pages forward</button>
    </div>
  );
}

import { Navigate } from "react-router-dom";

function App() {
  return <Navigate to="/home" replace state={state} />;
}
```

#### 路由传参

使用`useParams()`获取路由传递的一些参数，使用`useSearchParams()`获取?后的参数

```javascript
// 路由传参
navigate(`/home?from=${from}`);
navigate({
  pathname: "/class",
  search: `?id=${id}&grade=${grade}`
})
// 获取参数
import { useSearchParams } from "react-router-dom";
const ToPages = () => {
  const [searchParams] = useSearchParams();
  const id = searchParams.get("id");
  const grade = searchParams.get("grade");
  return (<h1>id : {id}, grade : {grade}</h1>)}
}

// state传参，刷新会丢失参数
import { useNavigate, useLocation } from "react-router-dom";
const goTo = () => {
  navigate(`/class`, { state: {id, grade} } )
}

const ToPages = () => {
  const location = useLocation ();
  const { id, grade } = location.state;
  return (<h1>id : {id}, grade : {grade}</h1>)}
}
```

#### 重定向

使用`Navigate`，替代了`Redirect`

#### 嵌套路由

```javascript
// route.js 路由文件
// 懒加载
const lazyLoad = (path) => {
  const Comp = React.lazy(() => import(`../application/${path}`));
  return (
    <React.Suspense fallback={<>加载中...</>}>
      <Comp/>
    </React.Suspense>
  )
};

const routes = [
  {
    path: "/",
    element: <Home/>,
    children: [
      {
        index: true,
        element: <Navigate to="/recommend"/>
      },
      {
        path: "recommend",
        element: <Recommend/>
      },
      {
        path: "singers",
        element: lazyLoad('Singers')
      },
      {
        path: "rank",
        title: "rank",
        element: <Rank/>
      }
    ],
  },
  { path: "*", element: <NotFound/> },
];
export default routes;

// App.js
import React from 'react';
import { GlobalStyle } from  './style';
import { IconStyle } from './assets/iconfont/iconfont';
import routes from './routes/index.js';
import { useRoutes } from 'react-router-dom';

function App () {
  return (
    <>
      <GlobalStyle></GlobalStyle>
      <IconStyle></IconStyle>
      { useRoutes(routes) }
    </>
  )
}
export default App;

// index.js
// 入口文件
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import { HashRouter } from 'react-router-dom';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    // hash模式
    <HashRouter>
      <App />
    </HashRouter>
  </React.StrictMode>
);

```

#### 子路由展示

使用 `<Outlet/>` 展示子路由页面

```javascript
// Home.js
import React from "react";
import { Outlet } from "react-router-dom";
function Home(props) {
  const activeClassName = "selected";
  return (
    <>
      <div>Home</div>
      <Outlet />
    </>
  );
}

export default React.memo(Home);
```

#### 动态路由

```javascript
import { Routes, Route, useParams } from "react-router-dom";
<Routes>
  <Route path={"/class/:id/:grade"} element={<Class />} />
</Routes>;
const routes = [
  {
    path: "class/:id",
    element: <Class />,
  },
];

// 获取参数
let { id } = useParams();
```

<br>
  
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/nqmv09