> 原文地址：[Business logic as a data structure](https://www.jsblog.io/articles/christianalfoni/business_logic_as_a_data_structure)
# 用数据结构表示的业务逻辑
七年前我从事一个大型单页应用的开发。那个时候我刚刚接触JavaScript并且没有研究JavaScript工具和框架的经验，主要是因为JavaScript工具和框架在当时还很少。这意味着一切都写成了vanilla风格。每一个DOM元素的创建或更新都是使用简单直接、命令式的JavaScript语句完成的。像这样：
```
const h1 = document.createElement('h1')
document.body.appendChild(h1)

window.eventHub.on('changed:title', function () {
  h1.innerText = window.state.title
})
```
通过使用命令式的JavaScript来管理你所有的UI在现在看来是非常少见的，这种情况的出现是有有原因的。使用这种方式开发对开发者来说绝对是一种非常差的体验。幸运的是在前端领域有很多的创新。从模板到由虚拟dom实现的JSX语法，让我们能够以一种声明的方式去表示动态UI。
> “声明式的代码描述了程序是什么，然而命令式的代码描述了JavaScript应该怎么做”

在本文中，我们将思考用哪些常见的数据结构可以编写声明式的代码，然后思考使用这种方式是否有利于我们业务逻辑的编写。

# 声明式的代码
那么声明式的代码是什么意思？无论你如何向人解释，总有人会说，“这并不完全准确”。这里有一个声明式代码的例子，可以让我们准确地了解我们即将要讨论的问题：

*html*
```
<div>
  <h1>Hello</h1>
</div>

```
以上代码显示了我们正在描述的是*what*，而不是*how*。它命令式的*how*版本将会是以下这样：
```
const div = document.createElement('div')
const h1 = document.createElement('h1')
h1.innerText = 'Hello'
div.appendChild(h1)
```
虽然这个例子有点假，但它向我们展示了如何区分声明式的代码和命令式的代码。
# JavaScript和声明式的代码
我们在JavaScript中也有声明性代码。例如当你想定义一个对象：
```
{
  name: 'Jon',
  age: 13
}
```
以上代码命令式的版本会是这样：
```
const user = new Object()
user.name = 'jon'
user.age = 13
```
与此类似的还有数组：
```
['apple', 'banana']
```
以上代码命令式的版本会是这样：
```
const fruits = new Array()
fruits[0] = 'apple'
fruits[1] = 'banana'
```
就像在html例子中声明式代码是对命令式代码的一种抽象一样。这意味着声明式代码是命令式代码的另一种存在形式。而声明式存在的意义是什么？举个例子，当你想知道汤里是什么的时候，你会更喜欢读食谱，而不是看一份厨师做汤时做的所有事情的记录。代码也是一样的。声明式描述了**what**，而命令式的描述了**how**。
> “你想知道汤里是什么。 你会阅读厨师做汤时做的所有事情的记录，还是只是阅读食谱？#declarative”

# 什么是业务逻辑？
所以在我们谈论声明式业务逻辑之前，我们不得不谈一谈论业务逻辑。当我提到业务逻辑时我指的是两件事：
1. 更新应用的状态
2. 执行应用状态改变所带来的影响（Run side effects）

无论你使用使用什么库都会做这些事，它们对于任何应用都是必需的同时这也就是在本文中我们所定义的**业务逻辑**。

使用[Redux](https://redux.js.org/introduction)的例子是一个thunk中间件：
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
或者使用[Mobx](https://mobx.js.org/)这种更简单的方法：
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
这些方法都不是声明式的，但它们确实包含使应用程序工作所必须的命令式代码。

向这个业务逻辑添加一个声明层并不简单。主要有两大挑战：
1. 我们没有任何原生数据结构，不像HTML，浏览器可以为我们解析这种声明式的代码
2. 业务逻辑通常是异步的，而html是同步

# 为业务逻辑创建数据结构
我们的数据结构需要描述执行的是什么逻辑，按什么顺序执行和在什么情况下执行。因此我总结了以下几点：
1. 它需要构造函数来运行，不像html我们只需要构造要显示的元素
2. 它需要处理结构的异步部分，不像html中的所有内容都是同步的
3. 它需要能处理条件语句，而html中不存在这种问题

由于我们需要运行多个函数，所以我们可以把这个数据结构命名为一个序列。我们现在的工作是在JavaScript中寻找能使用声明式的语法并能解决以上三个问题的数据结构。

## 定义一个序列
那么我们用什么来声明式地编写一系列函数呢？实在想不出比数组更合适的东西了：
```
[
  funcA,
  funcB,
  funcC
]
```
每个函数都将遵循命令式的逻辑编写。通过仅引用这些函数，我们保持了序列的声明性。
## 异步函数
有时我们希望我们的逻辑等待其他逻辑完成后执行。例如，我们需要从服务器拉取数据之后才能设置用户信息。在执行序列中的一个函数返回一个promise时可以显示它的异步特性。所以如果funcB看起来像这样：
```
function funcB () {
  return new Promise(resolve => setTimeout(resolve, 1000))
}
```
序列会像这样执行：
```
[
  funcA,
  funcB, // Holds 1000ms
  funcC
]
```
## 发散执行
我们的业务逻辑充满了if和switch语句。为了处理这个声明，我们需要一个不同于数组的数据结构。我们没有太多的选择，但是这有一个很好的选择。一个对象精美地描述了序列中可能的执行路径：
```
[
  funcA,
  funcB, {
    pathA: funcC,
    pathB: funcD
  },
  funcE
]
```

# 执行业务逻辑
所以这并不是简单的开箱即用。 在JavaScript中并不能理解这种数据结构。为了使它工作我们必须创建一个工具，让我们可以传递这个序列并且让它被执行。幸运的是我们有[Function-Tree](https://cerebraljs.com/docs/addons/index.html)。一个完全可以做到这一点的项目。

Cerebral项目使用它来运行它所定义的的**signals**，这里有一个它的介绍视频：[视频](https://www.youtube.com/embed/o2ULoHp22BE)

能够以声明的方式定义业务逻辑并不是要减少项目中的代码行数，也不是要让你更灵巧地写代码。开发人员花了大量的时间阅读代码是如何工作的，以了解它做了什么。对业务逻辑进行声明式的抽象将使你和你的同事在开发新需求或修复bug时更加轻松省力。

希望这给了大家一些灵感，如果想进一步探索声明式业务逻辑，可以查看更多关于**Cerebral**的信息。
