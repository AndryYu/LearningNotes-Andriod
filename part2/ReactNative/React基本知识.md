## React Redux 数据流

dispatch -> actions -> reducer -> store -> provider -> container(connect props与store联系) -> components

applyMiddleware的作用是添加一个中间件；你可以把一个个中间件看成一截截的管道，他们组成了一条长的管道，只不过管道里流过的不是水，而是action产生的数据；action传递过来的数据被一个中间处理后传递给下一个中间件处理，最后再给reducer处理。每个中间件可以调用dispatch和getState方法。


```
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
```
connect参数的含义：
* mapStateToProps 这个函数允许我们将 store 中的数据作为 props 绑定到组件上。
* mapDispatchToProps 它的功能是，将 action 作为props 绑定到 MyComp 上。
