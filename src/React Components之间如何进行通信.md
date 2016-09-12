# React Components之间如何进行通信
--------------
> 基于React开发的Web程序从本质上来讲都是一个个Component构成的。那么如何进行组件间的通信呢？  

## 前言
基于React的应用本质上是一堆Component的复合，那么在程序中不可避免的会遇到组件间的通信。我们根据组件的在组件树中所在的level可以把组件间通信分为如下四中形式：
*	父组件与子组件通信
*	子组件与父组件通信
*	兄弟组件间通信
*	任意组件间通信  

如果我们用树形图可以表示成如下：<sup>【1】</sup>
![Alt text](./1473424095806.png)  

通信，顾名思义是指信息的交换。学过通信的同学都知道信息交换是需要介质的，也就是说我们要解决组件间通信的问题，那么就是得找到组件间通信的介质。下面我们看看针对上述四种形式都有哪些解决方法。

## 父组件与子组件通信

父组件与子组件通信简而言之就是父组件能够通过通信介质传递信息给子组件。有React基础的人都知道Component通过props接受外来属性。显而易见，这里的props就可以当做父子组件通信的介质。 如下列示例： 
```javascript
class Child extends  React.Component {
  render() {
    return (
      <p>{this.props.count}</p>
    )
  }
}

class Parent extends React.Component {
  constructor(){
    super();
    this.state = {count:0}
  }

  clickHandle = () => {
    const count = this.state.count + 1;
    this.setState({count});
  }
  
  render() {
    return(
      <div>
        <Child count={this.state.count}/>
        <button onClick={this.clickHandle}>计数加1</button>
      </div>
    )
  }
}
```  
上述代码最终运行结果是点击父组件中的button，父组件state中的count加1，由于子组件count属性等于父组件state中的count，所以子组件显示的count也会加1。由此我们能够得出结论父组件与子组件通信可以通过props将父组件的state传递给子组件，这样父组件只用更新自身的state便能将改变通过props传递给子组件。    



##子组件与父组件通信
我们要实现子组件与父组件通信，首先要找到通信介质。上文讲到了父组件可以将自身的state传递给子组件来实现父组件向子组件传递信息的目的，那么换言之，父组件通过props将句柄传递给子组件供其调用那么就能达到子组件与父组件通信的目的。示例代码如下：
```javascript
class Child extends React.Component {

render() {
  return (
    <button onClick={this.props.clickHandle}>点我</button>
  );
}
}

class Parent extends  React.Component {
  clickHandle = () => {
    console.log('子组件调用父组件');
  }
  render() {
    return (
      <Child/>
    )
  }
}
```  
上述代码运行结果是点击子组件的button，父组件会进行log的打印。由此我们可以看到子组件与父组件的通信可以通过props将父组件的句柄传递给子组件，子组件在适当的时候调用该句柄便可以与父组件进行通信。  

## 兄弟组件
所谓的兄弟组件就是两个组件拥有共同的父组件。从上文的图中我们可以看到兄弟组件的情形其实就是上文中父组件与子组件通信以及子组件与父组件通信的结合体。那么这个时候通信的介质其实就是父组件，因为父组件能够充当连接两个子组件通信桥梁的作用。示例代码如下：  
```javascript
class Child2 extends React.Component {
  render() {
    return (
      <button onClick={this.props.clickHandle}>点我</button>
    )
  }
}

class Parent extends  React.Component {
  constructor() {
    super();
    this.state = {
      count: 0
    }
  }

  clickHandle = () => {
    const count = this.state.count + 1;
    this.setState({count});
  }
  
  render () {
    return (
      <div>
        <Child1 count={this.state.count}/>
        <Child2 clickHandle={this.clickHandle}/>
      </div>
    )
  }
}
```  
我们可以看到父组件将句柄传给了子组件2，将状态传递给子组件1。子组件2调用父组件的句柄改变父组件的状态，父组件通过props将状态传递给子组件2，这样就完成了子组件1、2的通信。我们能够得出结论兄弟组件的通信可以通过父组件分别通过props将句柄以及状态传递给不同的子组件，子组件调用句柄的同时也会拿到状态的子组件也会做相应的变化。  

## 任意组件