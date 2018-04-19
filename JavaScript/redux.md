## store
- dispatch(Action) view发出Action，dispatch在组件的事件响应中，触发reducer
- getState() 获取当前store快照，redux中的state与react中的state无关系，仅仅名字相同
- subscribe(listener) state变化即可自动执行listener
  - react中 render() 或setState()放入listener
  - subscribe返回unsubscribe可注销监听器
  
## react-redux
### 单纯用redux
- component => dispatch(Action) => reducer => subscribe => getState => component
### 使用react-redux，可省略3处代码(dispatch、subscribe、getState)
- component => actionCreator(data) => reducer => component
### connect 连接react和redux
- mapStateToProps
  - 返回需要合入props的state
  - 订阅store，state更新时自动执行listener
- mapDispatchToProps
  - 返回需要合入props的actionCreator
  - 定义用户的哪些操作应当做Action传给store
### Provider 用于在组件间传递数据
- 原理： 将store作为props，并将其声明为context的属性，子组件可以在声明了contextType之后通过this.context.store访问store
