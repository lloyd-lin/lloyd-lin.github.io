---
title: react-memorization
date: 2019-05-27 13:07:24
tags:
---


# React 官方文档学习篇 - You Probably Don't Need Derived State
## Derived state
通过props的改变而改变内部自己的state，我们称之为derived state  
举例， 通过props的改变去请求自己新的state从而改变样式
```
// 父组件
Page extends Component {
  state = {
    active: false
    text: "default"
  };
  highlightBtn() {
    this.setState({
      active: !this.state.active
    });
  }
  udpateText() {
    this.setState({
      text: `default:avtive is ${this.state.active}`
    });
  }
  render() {
    console.log("render page");
    const { active } = this.state;
    const clazz = classnames("btn", active ? "active" : "inActive");
    return (
      <>
        <TextInput text={this.state.text} />
        <div className="btn active" onClick={this.udpateText.bind(this)}>
          更新text
        </div>
        <div className={clazz} onClick={this.highlightBtn.bind(this)}>
          更新其他属性
        </div>
      </>
    );
  }
}
// 子组件
class TextInput extends Component {
  state = {
    text: this.props.text
  };

  render() {
    return <input onChange={this.handleChange} value={this.state.text} />;
  }

  handleChange = event => {
    this.setState({ text: event.target.value });
  };
  componentWillReceiveProps(nextProps) {
    // sample1 
    // 问题 父组件render将会无脑导致更新
    this.setState({ text: nextProps.text });
  }
}
```
存在问题
1. 非预期的state更新  
2. 因为props和state的改变更新过于频繁  

父类控制的数据称之为可控数据,内部数据称之为非可控数据  
最常见的问题就是将两种内容混淆,当约束被打破时最终出现这种问题  
通常本次修改内容已经触发自身state的变化，而外部无感知，因此外部render一定会覆盖内部的值导致内部值丢失

```
// 父组件
Page extends Component {
  // ...

  // 控制更新
  // 由于组件的state值和外部props的值越来越多，就算使用shouldComponentUpdate来判断也会因为逻辑过多而返回不可靠的true值，本身也有一定限制必须在触发更新时才能更新别的状态
  shouldComponentUpdate(nextProps, nextState) {
    if (nextState.text !== this.state.text) {
      return true;
    }
    return false;
  }
}
```

因此，shouldComponentUpdate更适合用来做性能上的优化而不是用来处理这种潜在的state

```
// 子组件
class TextInput extends Component {
  // ...

  // sample3 改进版
  componentWillReceiveProps(nextProps) {
    if (nextProps.text !== this.props.text) {
      this.setState({ text: nextProps.text });
    }
  }
}
```
采用子组件在componentWillReceiveProps改进可以实现防止被刷新，但是默认赋值动作又会收到影响
解决这些问题的根源在于  
对于任何数据，你需要一个自己控制自己数据源的组件，避免和其他组件重叠  

1. 纯props组件， 但是父组件需要记录数据
2. 只有初始化定义数据是用props，后续完全自己维护state，需要再外部使用key来辅助切换新内容时使用defaultProps重置对象初始值（reset最佳实践），如果由于组件初始化代价较大，纯粹的defaultProps复写也可以提取到getDerivedStateFromProps中去

PureComponent
```
// 子组件
export default class PureTextInput extends PureComponent {
  render() {
    return <input onChange={this.props.handleChange} value={this.props.text} />;
  }
}

// 父组件
Page extends Component {
  // ...

  handleChange = event => {
    this.setState({ text: event.target.value });
  };
  return (
      <>
        <PureTextInput
          text={this.state.text}
          handleChange={this.handleChange.bind(this)}
        />
        <div className="btn active" onClick={this.udpateText.bind(this)}>
          更新text
        </div>
        <div className={clazz} onClick={this.highlightBtn.bind(this)}>
          更新其他属性
        </div>
      </>
    );
  }
}
```

key辅助,直接强制生成新节点，不会进入比较周期(大多数情况的最佳实践)
```
// 父组件
Page extends Component {
  // ...

  udpateText() {
    this.setState({
      text: `default:avtive is ${this.state.active}`,
      id: new Date().getMilliseconds() // 设置一个id
    });
  }
  // 通过id保证TextInput的重新加载
  return (
      <>
        <TextInput text={this.state.text} key={this.state.id} />
        <div className="btn active" onClick={this.udpateText.bind(this)}>
          更新text
        </div>
        <div className={clazz} onClick={this.highlightBtn.bind(this)}>
          更新其他属性
        </div>
      </>
    );
}
```
其他方案： 缓存一个类似key的值/ 设定重置方法，开放ref

## Memoization
基于props而需要记录下来计算的展示结果，推荐使用记忆化技术
```
// 对于无效render会带来性能问题方法的优化
  filter = memorize((dataArr) => {
    let count = 0;
    dataArr.forEach((ele) => {
      if (ele%7 === 0) {
        count ++;
      }
    });
    return count;
  })
  render() {
    const calResult = this.filter(this.props.data);
    <div>可以被7整除的数字数量为{calResult}</div>
  }
```
性能明显提升, 对于memorize源码可以自行研究，主要是需要对比2次props/state是否严格相等，如果相等则直接用返回上一次的计算结果。