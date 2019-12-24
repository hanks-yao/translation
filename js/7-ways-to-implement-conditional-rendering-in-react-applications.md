# [译]在React中实现条件渲染的7种方法

> 原文地址：<https://scotch.io/tutorials/7-ways-to-implement-conditional-rendering-in-react-applications>

借助React，我们可以构建动态且高度交互的单页应用程序，充分利用这种交互性的一种方法是通过**条件渲染**。

## <span id="contents">目录</span>

* [挑战](#challenge)
* [解决方法](#solution)
    1. [使用if…else语句](#mark1)
    2. [使用元素变量](#mark2)
    3. [使用switch语句](#mark3)
    4. [三元运算符](#mark4)
    5. [逻辑运算符&&](#mark5)
    6. [使用立即调用函数表达式（IIFE）](#mark6)
    7. [使用增强的JSX](#mark7)
* [性能问题](#performance)
* [总结](#conclusion)

**条件渲染**一词描述了根据某些条件渲染不同UI标签的能力。在React文档中，这是一种根据条件渲染不同元素或组件的方法。此概念通常被应用到如下情况中：

* 从API渲染外部数据
* 显示/隐藏元素
* 切换应用程序功能
* 实现权限级别
* 认证与授权

在本文中，我们将研究在React应用程序中实现条件渲染的7种方法。

## <span id="challenge">挑战</span>
首先，根据在组件state中`isLoggedIn`的值，我们希望能够在用户未登入时显示`Login`按钮，在用户登入时显示`Logout`按钮。

下图是我们初始组件的截图：
![screenshoot](https://user-gold-cdn.xitu.io/2019/12/20/16f2130b2b67cca8?w=1176&h=1240&f=png&s=67838)

代码如下：
``` javascript
import React, { Component } from "react";
import ReactDOM from "react-dom";
import "./styles.css";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      isLoggedIn: true
    };
  }
  render() {
    return (
      <div className="App">
        <h1>
          This is a Demo showing several ways to implement Conditional Rendering in React.
        </h1>
        <button>Login</button>
        <button>Logout</button>
      </div>
    );
  }
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

## <span id="solution">解决方法</span>
> 在代码段中，…表示某些与所解释要点没有直接联系的代码。

### <span id="mark1">1. 使用if…else语句</span>
使用`if…else`语句允许我们可以指出，如果条件为`true`，则执行特定的操作，否则将执行其他操作。使用示例，我将测试`if…else`通常用于在React中条件渲染的两种方法。

* 将条件渲染提取到函数中

在JSX中，我们可以将JS代码和HTML标签放在一起，以确保程序内具有惊人的交互性。为此，我们使用大括号`{}`并在其中编写我们的JS。但在括号内能做事情也是有限的。例如，下面代码的结果并不能实现我们想要的结果。

``` javascript
// index.js
...
render() {
  let {isLoggedIn} = this.state;
  return (
    <div className="App">
      <h1>
        This is a Demo showing several ways to implement Conditional Rendering in React.
      </h1>
      {
        if(isLoggedIn){
          return <button>Logout</button>
        } else{
          return <button>Login</button>
        }
      }
    </div>
  );
}
...
```

要了解更多相关信息，请访问[此链接](https://react-cn.github.io/react/tips/if-else-in-JSX.html)。

为了解决这个问题，我们将条件逻辑提取到一个函数中，如下所示：
``` javascript
// index.js
...
render() {
  let {isLoggedIn} = this.state;
  const renderAuthButton = ()=>{
    if(isLoggedIn){
      return <button>Logout</button>
    } else{
      return <button>Login</button>
    }
  }
  return (
    <div className="App">
      <h1>
        This is a Demo showing several ways to implement Conditional Rendering
        in React.
      </h1>
      {renderAuthButton()}
    </div>
  );
}
...
```

注意，我们将逻辑从JSX提取到函数`renderAuthButton`中。因此，我们只需要在JSX大括号内执行函数即可。

* 多个返回语句

在使用此方法时，组件必须尽可能的简单，以避免兄弟或父组件的重新渲染。因此，我们创建了一个名为`AuthButton`的新组件。
``` javascript
// AuthButton.js

import React from "react";

const AuthButton = props => {
  let { isLoggedIn } = props;
  if (isLoggedIn) {
    return <button>Logout</button>;
  } else {
    return <button>Login</button>;
  }
};
export default AuthButton;
```

`AuthButton`根据通过组件属性`isLoggedIn`传入的状态值，返回不同的元素和组件。因此，我们将其导入`index.js`并将状态值传入，如下所示：
``` javascript
// index.js
...
import AuthButton from "./AuthButton";

...
  render() {
    let { isLoggedIn } = this.state;
    return (
      <div className="App">
      ...
        <AuthButton isLoggedIn={isLoggedIn} />
      </div>
    );
  }
...
```

一定要避免下面的操作：
``` javascript
// index.js
...
render() {
  let { isLoggedIn } = this.state;
  if (isLoggedIn) {
    return (
      <div className="App">
        <h1>
          This is a Demo showing several ways to implement Conditional
          Rendering in React.
        </h1>
        <button>Logout</button>;
      </div>
    );
  } else {
    return (
      <div className="App">
        <h1>
          This is a Demo showing several ways to implement Conditional
          Rendering in React.
        </h1>
        <button>Login</button>
      </div>
    );
  }
}
...
```
虽然上述代码将实现相同的结果，但使得组件不必要的臃肿，同时由于不断重新渲染一个不变的组件而导致性能问题。

### <span id="mark2">2. 使用元素变量</span>
元素变量是上述的将条件渲染提取到函数中的一个扩展。元素变量只是保存JSX元素的变量。因此，我们可以在JSX外部根据条件将元素/组件赋值给这些变量，仅在JSX渲染变量即可。
``` javascript
// index.js
...
render() {
  let { isLoggedIn } = this.state;
  let AuthButton;
  if (isLoggedIn) {
    AuthButton = <button>Logout</button>;
  } else {
    AuthButton = <button>Login</button>;
  }
  return (
    <div className="App">
      <h1>
        This is a Demo showing several ways to implement Conditional Rendering in React.
      </h1>
      {AuthButton}
    </div>
  );
}
...
```
注意我们如何有条件地将值（组件）分配给`AuthButton`，然后我们只需要在JSX中渲染它即可。

### <span id="mark3">3. 使用switch语句</span>
如前所示，我们可以使用if…else语句根据设置的条件从组件返回不同的标签。使用switch语句也可以达到相同的效果，在该语句中我们可以为不同的条件指定标签。看看如下代码：
``` javascript
// AuthButton.js
import React from "react";

const AuthButton = props => {
  let { isLoggedIn } = props;
  switch (isLoggedIn) {
    case true:
      return <button>Logout</button>;
      break;
    case false:
      return <button>Login</button>;
      break;
    default:
      return null;
  }
};

export default AuthButton;
```
注意我们如何根据`isLoggedIn`的值返回不同的按钮。当存在两个以上可能的值或结果时，采用此方法更为合理。你也可以通过break语句取消，正如return语句自动终止执行一样。

注意：从组件返回`null`会使它隐藏自身/不显示任何内容。这是切换组件可见性的好方法。

### <span id="mark4">4. 三元运算符</span>
如果你熟悉三元运算符，那么你应该知道这是编写if语句的一种更简洁的方法。因此我们也许会这样写：
``` javacript
// index.js
...
render() {
  let { isLoggedIn } = this.state;
  return (
    <div className="App">
      <h1>
        This is a Demo showing several ways to implement Conditional Rendering
        in React.
      </h1>
      {isLoggedIn ? <button>Logout</button> : <button>Login</button>}
    </div>
  );
}
...
```
但在上例中，这种方法会使组件臃肿，笨重和难以理解，你可以将条件封装在纯函数组件中。如下所示：
``` javascript
// AuthButton.js
import React from "react";

const AuthButton = props => {
  let { isLoggedIn } = props;
  return isLoggedIn ? <button>Logout</button> : <button>Login</button>;
};

export default AuthButton;
```

### <span id="mark5">5. 逻辑运算符&&</span>
短路运算是一种用于确保在表达式运算过程中没有副作用的技术。逻辑&&帮助我们指定仅在一种情况下执行，否则将被完全忽略。这对于仅在特定条件为真时才需要执行的情况下是很有用的。

例如，如果是登录状态，我们只需显示`Logout`按钮，否则我们什么也不做。如下：
``` javascript
// index.js
...
render() {
  let { isLoggedIn } = this.state;
  return (
    <div className="App">
      <h1>
        This is a Demo showing several ways to implement Conditional Rendering
        in React.
      </h1>
      {isLoggedIn && <button>Logout</button>}
    </div>
  );
}
...
```

如果`isLoggedIn`为true，则将显示`Logout`按钮，否则将不显示任何内容。我们用相同方法就可以实现最终结果，如下所示。
``` javascript
// index.js
...
return (
  <div className="App">
    <h1>
      This is a Demo showing several ways to implement Conditional Rendering
      in React.
    </h1>
    {isLoggedIn && <button>Logout</button>}
    {!isLoggedIn && <button>Login</button>}
  </div>
);
...
```
这是基于`isLoggedIn`的值渲染正确的按钮。但是，**不建议这样做**，因为有更好、更清洁的方法可以达到相同的效果。而且，一旦组件稍大一些，这很容易使你代码看起来混乱和难以理解。

### <span id="mark6">6. 使用立即调用函数表达式（IIFE）</span>
还记的刚才说的JSX局限性吗，在其中无法执行所有JavaScript代码。嗯...这并不完全正确的，因为有很多方法可以绕过这种行为。一种方法是使用IIFE。
``` javascript
(function () {
  statements
})();
```
点击[链接](https://developer.mozilla.org/zh-CN/docs/Glossary/%E7%AB%8B%E5%8D%B3%E6%89%A7%E8%A1%8C%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F)了解更多

通过这种技术，我们能够直接在JSX内编写条件逻辑，但将其包装在匿名函数中，该匿名函数在运行该部分代码后立即被调用。请参见下面的示例：
``` javascript
//index.js
...
render() {
  let { isLoggedIn } = this.state;
  return (
    <div className="App">
      <h1>
        This is a Demo showing several ways to implement Conditional Rendering
        in React.
      </h1>
      {(function() {
        if (isLoggedIn) {
          return <button>Logout</button>;
        } else {
          return <button>Login</button>;
        }
      })()}
    </div>
  );
}
...
```
也可以使用箭头功能，通过更加简洁的方式编写该代码，如下所示：
``` javascript
// index.js
...
return (
  <div className="App">
    <h1>
      This is a Demo showing several ways to implement Conditional Rendering in React.
    </h1>
    {(()=> {
      if (isLoggedIn) {
        return <button>Logout</button>;
      } else {
        return <button>Login</button>;
      }
    })()}
  </div>
);
...
```

### <span id="mark7">7. 使用增强的JSX</span>
某些库公开了扩展JSX的功能，因此可以直接用JSX实现条件渲染。此类库之一是[JSX Control Statements](https://github.com/AlexGilleran/jsx-control-statements)。它是Babel插件，可在编译过程中将类似组件的控制语句转换为JavaScript对应的语句。请参阅下面的示例以了解如何实现。
``` javascript
// index.js
...
return (
  <div className="App">
    <h1>
      This is a Demo showing several ways to implement Conditional Rendering in React.
    </h1>
    <Choose>
      <When condition={isLoggedIn}>
         <button>Logout</button>;
      </When>
      <When condition={!isLoggedIn}>
         <button>Login</button>;
      </When>
    </Choose>
  </div>
);
...
```
但是，不建议使用这种方法，因为您编写的代码最终会转换为常规JavaScript条件。仅仅编写JavaScript可能总比对如此琐碎的事情增加额外的依赖要好。

## <span id="performance">性能问题</span>
作为通用规则，最好确保在实现条件渲染时：

* 请勿随意更改组件的位置，以防止不必要地卸卸和重载组件。
* 仅更改与条件渲染有关的标签，而不改变组件中没有变动的部分。
* 不要在render方法中使组件不必要的臃肿，从而导致组件延迟渲染。

## <span id="conclusion">总结</span>
我们已经成功研究了在React中实现条件渲染的7种方法。每种方法都有其自身的优势，选择哪种方法主要取决于实际情况。要考虑的事项包括：

* 条件渲染标签的大小
* 可能结果的数目
* 哪个会更直观和可读

通常，我建议：

* 当只有一个预期的结果时，**逻辑&&运算符**非常方便。
* 对于布尔型情况或只有两个可能结果的用例，可以使用**if…else**，**Element变量**，**三元运算符**和**IIFE**。
* 如果结果超过2个，则可以使用**Switch语句**，**提取成函数**或**提取成纯函数组件**。

但是，这些仅是建议，最终还是根据实际情况选择。