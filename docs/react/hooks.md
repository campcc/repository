---
title: setState
order: 2
---

# setState

如果你不能很好地解释下面代码的输出结果，那可能你对 setState 的理解还不够

```js
import React from 'react';

export default class App extends React.Component {
  state = {
    count: 0,
  };

  increment = () => {
    console.log('increment setState前的count', this.state.count);
    this.setState({
      count: this.state.count + 1,
    });
    console.log('increment setState后的count', this.state.count);
  };

  triple = () => {
    console.log('triple setState前的count', this.state.count);
    this.setState({
      count: this.state.count + 1,
    });
    this.setState({
      count: this.state.count + 1,
    });
    this.setState({
      count: this.state.count + 1,
    });
    console.log('triple setState后的count', this.state.count);
  };

  reduce = () => {
    setTimeout(() => {
      console.log('reduce setState前的count', this.state.count);
      this.setState({
        count: this.state.count - 1,
      });
      console.log('reduce setState后的count', this.state.count);
    }, 0);
  };

  render() {
    return (
      <div>
        <button onClick={this.increment}>点我增加</button>
        <button onClick={this.triple}>点我增加三倍</button>
        <button onClick={this.reduce}>点我减少</button>
      </div>
    );
  }
}
```

输出结果，

```bash
increment setState前的count 0
increment setState后的count 0
triple setState后的count 1
triple setState后的count 1
reduce setState前的count 2
reduce setState前的count 1
```

如果你是一个熟手 React 开发，那么 increment 这个方法的输出结果想必难不倒你——正如许许多多的 React 入门教学所声称的那样，“setState 是一个异步的方法”，这意味着当我们执行完 setState 后，state 本身并不会立刻发生改变。 因此紧跟在 setState 后面输出的 state 值，仍然会维持在它的初始状态。
