# react-redux
![7e392031194240a8b3f85b836f73ed72_tplv-k3u1fbpfcp-zoom-in-crop-mark_4536_0_0_0.jpeg](https://cdn.nlark.com/yuque/0/2022/jpeg/394019/1665567783422-56c1525d-4c86-4b06-930f-d13d6c8b193a.jpeg#clientId=udeecf136-6f5e-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&id=u47e5cb5c&margin=%5Bobject%20Object%5D&name=7e392031194240a8b3f85b836f73ed72_tplv-k3u1fbpfcp-zoom-in-crop-mark_4536_0_0_0.jpeg&originHeight=590&originWidth=1372&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37347&status=error&style=none&taskId=u811483d0-6e2d-48bd-bdda-cfca719afdd&title=)

#### state

`state`是状态对象

#### action

`action`是一个普通 js 对象，描述发生了什么，可以理解为一个行为指示器，比如:

```javascript
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }

```

#### reducer

`reducer`初始化 store，并串起`state`和`action`，传入 state 和 action，并返回新的 state

#### dispatch

组件通过 dispatch 改变 state，实际通过调用 reducer 方法来得到新的 state

#### combineReducers

将多个 reducer 合并成一个 reducer

#### connect

`connect` 其实是一个高阶函数，高阶函数就是指可以接收参数调用并返回另外一个函数的函数。这里 connect 通过接收 mapStateToProps 然后调用返回一个新函数，接着这个新函数再接收 App 组件作为参数，通过 mapStateToProps 注入 todos 和 filter 属性，最后返回注入后的 App 组件。

#### 中间件

`redux-thunk`中间件，我们可以把异步过程通过函数的形式放在 dispatch 的参数里：

```javascript
/// 下面的代码是固定写法 /// redux-thunk 中间件 需要引入的
const composeEnhancers =
  typeof window === "object" && window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__
    ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({})
    : compose;

/// 中间件都需要用这个方法
const enhancer = composeEnhancers(
  applyMiddleware(thunk) /// redux-thunk 中间件 需要引入的
);
/// 创建
const store = createStore(
  reducer,
  enhancer //使用Redux-thunk 中间件
);

// #store/actionCreator.js
export const getToList = () => {
  /// 这里返回一个函数,能获取到 dispatch 方法 这就是 redux-thunk 的作用，可以返回一个函数
  return (dispatch) => {
    axios
      .get("/api/todolist")
      .then((res) => {
        // alert(res.data);
        const data = res.data;
        dispatch({
          type: INIT_LIST,
          data,
        });
      })
      .catch((error) => {
        console.log("网络请求错误了---thunk----》");
      });
  };
};
```

### 使用

`Provider` 是 context 的封装 ，分配给所有需要 state 的子孙组件，方便全局访问 store

```javascript
import React from "react";
import { render } from "react-dom";
import { Provider } from "react-redux";
import { createStore } from "redux";
import todoApp from "./reducers";
import App from "./components/App";

let store = createStore(todoApp);

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

```javascript
// #store/index.js
import { createStore,compose,applyMiddleware } from 'redux';
import reducer from './reducer';

const store = createStore(reducer);
export default store;


// #store/reducer.js
import { INIT_LIST, ADD_TODO_LIST_VALUE } from "./constants";

/// 初始化数据
const defaultState = {
    todos: [{
      text: 'Eat food',
      completed: true
    }, {
      text: 'Exercise',
      completed: false
    }],
    toggleTodo: false,
  	filter: 'SHOW_ALL',
}
/// Reducer 可以接受state，但是不能修改State ！
export default (state = defaultState, action) => {
  switch (action.type) {
    	case INIT_LIST:
        return {
          ...state,
          todos: action.data
        };
      case ADD_TODO_LIST_VALUE:
        const newState = JSON.parse(JSON.stringify(state));///将原来的state 做一次深拷贝
        newState.todos.push(action.data);
        return newState;
      default:
        return state;
  }
}

// #store/actionCreator.js
import { ADD_TODO_LIST_VALUE } from "./constants"

export const addItemListAction = (data) => ({
      type: ADD_TODO_LIST_VALUE,
      data
})

// #store/constants.js
export const INIT_LIST = 'init_list';
export const ADD_TODO_LIST_VALUE = 'add_todo_list_value';
export const TOGGLE_TODO = 'add_todo_list_value';


```

```javascript
# 组件中使用
import React from "react";
import AddTodo from "./AddTodo";
import TodoList from "./TodoList";
import Footer from "./Footer";

import { connect, useSelector } from "react-redux";

// 省略了 VisibilityFilters 和 getVisibleTodos 函数...

class App extends React.Component {
  constructor(props) {
    super(props);

    this.toggleTodo = this.toggleTodo.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
    this.setVisibilityFilter = this.setVisibilityFilter.bind(this);
  }

  // 省略中间其他方法...

  render() {
    const { todos, filter } = this.props;
    // 通过useSelector直接获取state值
    const _todos = useSelector(state=> state.todos)

    return (
      <div>
        <AddTodo onSubmit={this.onSubmit} />
        <TodoList
          todos={getVisibleTodos(todos, filter)}
          toggleTodo={this.toggleTodo}
        />
        <Footer
          filter={filter}
          setVisibilityFilter={this.setVisibilityFilter}
        />
      </div>
    );
  }
}

const mapStateToProps = (state, props) => ({
  todos: state.todos,
  filter: state.filter,
  toggleTodo: state.toggleTodo
});

export default connect(mapStateToProps)(App);


import React from "react";
import { connect } from "react-redux";
import { addItemListAction } from "../store/actionCreator";

const AddTodo = ({ dispatch }) => {
  let input;

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault();
          if (!input.value.trim()) {
            return;
          }
          dispatch(addItemListAction(input.value));
          input.value = "";
        }}
      >
        <input ref={node => (input = node)} />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
};

export default connect()(AddTodo);
```

### 使用原则

- 单一数据源: 全局唯一 state
- state 只读，操作只可通过 action
- 使用纯函数执行修改，没有其他操作

参考:  
[Redux 的设计模式](https://juejin.cn/post/6924834203129348109)  
[React-Redux 与 Vuex 使用对比](https://juejin.cn/post/7096780506976485383)

<br>
  
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/wsct2f