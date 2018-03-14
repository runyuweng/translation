> 原文地址：[Business logic as a data structure](https://www.jsblog.io/articles/christianalfoni/business_logic_as_a_data_structure)
# 作为数据结构的业务逻辑
七年前我从事一个大型单页应用的开发。那个时候我刚刚接触JavaScript并且没有研究JavaScript工具和框架的经验，主要是因为他们当时还很少。这意味着一切都写成了vanilla风格。每一个DOM元素的创建或更新都是使用简单直接、命令式的JavaScript完成的。像这样：
```
const h1 = document.createElement('h1')
document.body.appendChild(h1)

window.eventHub.on('changed:title', function () {
  h1.innerText = window.state.title
})
```
通过手动使用命令式的JavaScript来管理你所有的UI在现在是不常见、落后的，并且这种情况是有着充分的理由的。使用这种方式开发绝对是一种痛苦的体验。幸运的是有很多创新。从模板到由虚拟dom实现的JSX语法，让我们能够以一种声明的方式去表达动态UI。
> “声明式的代码描述了程序是什么，然而命令式的代码描述了JavaScript应该怎么做”

在本文中，我们将看看我们每天使用哪些数据结构来声明式编写。然后，我们将看看在编写业务逻辑时是否可以获得这些好处。？？？？？？？？？？？？

# 声明式的代码
那么声明式的代码是什么意思？无论你如何解释总有人会说，“这并不完全准确”。但是这里有一个声明式代码的例子，以便我们可以开始准确地讨论我们即将要讨论的问题：

*html*
```
<div>
  <h1>Hello</h1>
</div>

```
这段代码说明的是我们正在描述的是WHAT，而不是HOW。这段代码命令式的HOW版本将会是这样：
```
const div = document.createElement('div')
const h1 = document.createElement('h1')
h1.innerText = 'Hello'
div.appendChild(h1)
```
虽然这是一个有点假的例子，但是它向我们显示了如何区分声明式和命令式代码。
# JavaScript和声明式的代码
我们在JavaScript中也有声明性代码。例如当你想定义一个对象：
```
{
  name: 'Jon',
  age: 13
}
```
这段代码命令式的版本会是这样：
```
const user = new Object()
user.name = 'jon'
user.age = 13
```
与此类似的还有lists：
```
['apple', 'banana']
```
这段代码命令式的版本会是这样：
```
const fruits = new Array()
fruits[0] = 'apple'
fruits[1] = 'banana'
```
就像我们的html示例中声明式代码是对命令式代码的一种抽象一样。这意味着声明式代码是命令式代码执行的一种配方。

但为什么甚至有麻烦做一个声明式的抽象？那么，当你想知道什么是汤时，你更喜欢阅读食谱，而不是阅读厨师为制作汤所做的一切记录。这与代码相同。声明式描述了**What**，而命令式的描述了**How**
????????????
> “你想知道什么是汤。 你是否阅读了厨师所做的记录，或者只是阅读食谱？#declarative”

# 什么是业务逻辑？
所以在我们谈论声明式业务逻辑之前，我们必须谈一谈论业务逻辑。当我说声明式业务逻辑时，我指的是两件事：
1. 更新应用的状态
2. 更新应用状态改变所带来的影响

无论使用什么库都会执行这些操作，它们对于任何应用都是必需的，在本文中我们将这些内容定义为**业务逻辑**。
????????????

使用Redux的例子是一个thunk中间件：
```
function getUser (dispatch) {
  dispatch({type: USER_LOADING})
  ajax.get('/user')
    .then(user => {
      dispatch({type: USER_LOADED, user})
    })
    .catch(error => {
      dispatch({type: USER_ERROR, error})
    })
}
```
或使用Mobx这种更简单的方法：
```
class User {
  isLoading = false
  data = null
  error = null
  get () {
    this.isLoading = true
    ajax.get('/user')
      .then(user => {
        this.isLoading = false
        this.data = user
      })
      .catch(error => {
        this.isLoading = false
        this.error = error
      })
  }
}
```
这些方法都不是声明式的，但它们确实拥有使应用程序工作所必须的命令式代码。

向这个业务逻辑添加一个声明层不是轻而易举的。有两大挑战：
1. 我们没有任何原生数据结构，不像HTML，我们有一种类型的浏览器为我们处理的XML
2. 业务逻辑通常是异步的，与同步的html不同

# 为业务逻辑创建数据结构
