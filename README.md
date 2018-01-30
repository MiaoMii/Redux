# Redux

### Action
>Actions（一个包含｛type，payload（有效载荷）｝JavaScript对象）Action是把数据从应用传到store的有效载荷。是store的唯一数据来源


  必须要有一个字符串类型的`type`字段表示将要执行的动作。`payload`是这个动作携带的数据，Action需要通过`store.dispatch`方式来发送
``` javascript
{
	type : 'ADD_TODO',
	text : 'Build my first Redux app'
}
```
一般使用函数(Action Creator)，Action Creator是一个**pure function** ，最后会返回一个Action对象
``` javascript
	function addTodo(text) {
	return {
		type : 'ADD_TODO',
		text
	}
}
```
### Reducer（一个函数）
>用来处理Action触发的对状态树的更改

接收两个参数，`action`和 `oldState` 返回一个新的`state:(oldState,action ) => new State`
一个简单的reducer：
```javascript
	const initialState = {
  a: 'a',
  b: 'b'
};

function someApp(state = initialState, action) {
  switch (action.type) {
    case 'CHANGE_A':
      return { ...state, a: 'Modified a' };
    case 'CHANGE_B':
      return { ...state, b: action.payload };
    default:
      return state
  }
}
```

不要在reducer中做的事：
- **直接修改State** 
- **调用API** 
-  **调用非纯函数** 

因为Redux中只有一个Store，对应一个State状态，所以整个State对象就是由一个reducer函数管理的，如果所有的状态逻辑都放在一个reducer里面，会使得reducer越大越巨大，越来越难以维护。得益于纯函数的实现，我们可以让状态树上的每一个字段都有一个reducer函数来管理，就可以拆分成很小的reducer了：
```javascript
	function someApp(state = {},action{
		return {
			a : reducerA(state.a,action),
			b : reducerB(state.b,action)
		}
	}
```
Redux 提供了一个工具函数 `combineReducers`来简化这种reducer的合并：
```python
	import { combineReducers } from 'redux'
	const somrApp = combineReducers({
		a : reducerA,
		b : reducerB
	})
```
像`someApp`这种管理整个State的reducer，可以称作 **root reducer**
### Store
>连接Action 和 Reducer

Store的作用：
-  维持应用的state
-  提供`getState()`方法获取state
-  提供`dispatch(action)`方法更新state
-  通过`subscribe(listener)`注册监听
-  通过 `subscribe(listener)`返回的函数注销监听器。

创建一个Store很容易，将 root ruducer 函数传递给 `createStore`方法即可
```javascript
	import { createStore } from 'redux'
	import someApp from './reducers'
	let store = createStore(someApp)

```
现在我们拿到了`store.dispatch`,可以用来分发action了：
```javascript
	et unsubscribe = store.subscribe(()   => console.log(store.getState()));

// Dispatch
store.dispatch({ type: 'CHANGE_A' });
store.dispatch({ type: 'CHANGE_B', payload: 'Modified b' });

// Stop listening to state updates
unsubscribe();
```
### Date Flow
`store.dispatch(action)  -> reducer(state,action) -> store.getState()` 其实就构成了一个单向数据流
#####1、调用 `store.dispatch(action)`
Action是一个包含 `{  type,payload }`的对象，它描述“发生了什么”，比如
```javascript
	{type : 'LIKE_ARTICLE',articleID:1}
	{type : 'FETCH_USER_SUCCESS',response:{ id:3,name:'Mary'}}
```
你可以在任何地方调用`store.dispatch(action)`,比如组件内部，Ajax回调函数等
#####2、Action会触发给Store指定的 root reducer
root reducer会返回一个完整的状态树，State对象上的各个字段值可以由各自的reducer函数处理并返回新的值。
- rudecer函数接收`(state,action)`两个参数
- reducer函数判断`action.type`然后处理对应的action.payload数据来更新并返回一个新的state
```javascript
	//当前应用的State
	let State = {
	A: 'SHOW_ALL',
	B:[
	    {
	        text:'',
	        complish:false
	    }
	]
	}
	//将要执行的action
	let action = {
		type:'ADD_TODO',
		text:'Add todo'
	}
	// reducer 返回处理后的应用状态
	 let nextState = someApp(State, action);
```
#####3、Store会保存root reducer返回的状态树
新的State会替代旧的State,然后所有`store.subscribe(listener)`注册的回调函数会被调用，在回调函数里面可以通过`store.getState()`拿到新的State
```javascript
	function reducerA(state = [], action) {
   // 省略处理逻辑...
   return nextState;
 }

 function reducerB(state = 'SHOW_ALL', action) {
   // 省略处理逻辑...
   return nextState;
 }

 let todoApp = combineReducers({
   reducerA,
   reducerB
 })
```
当你触发action后，`combineReducers`返回的someApp会调用两个reducer
```javascript
	let nextA = reducerA(state.A, action);
    let nextB = reducerB(state.B, action);
```
然后会把两个结果集合并成一个state树
```javascript
	return {
		reducerA : nextA,
		reducerB : nextB
	}
```
参考文献:
[Redux中文文档](http://cn.redux.js.org/)

[Redux 基础 | React 入门教程](https://hulufei.gitbooks.io/react-tutorial/content/redux-basic.html)
