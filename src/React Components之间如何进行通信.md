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
![Alt text](http://od6g4gld9.bkt.clouddn.com/WechatIMG6.jpeg)  

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
其实上文中的三种情况都属于任意组件的情形，解决方案都也可以采用上述的解决方案。但是就有可能导致位于上层的组件需要通过props一层一层的传递下去，这样就导致了下层所有组件都和上层有耦合，而且书写起来及其麻烦。那么不通过props传递，我们还能够怎么去做呢？  
### context
context类似于props，但是context一旦在组件声明，那么该组件下所有的底层组件都可以调用context。示例代码如下：
```javascript
class Child1 extends React.Component {
  static contextTypes = {
    count: React.PropTypes.number
  }
  render() {
    return (
      <p>{this.context.count}</p>
    )
  }
}

class Child2 extends React.Component {
  static contextTypes = {
    increaseCount: React.PropTypes.func
  }
  render() {
    return (
      <button onClick={this.context.increaseCount}>点我</button>
    )
  }
}

class Container extends  React.Component {
  render () {
    return (
      <div>{this.props.children}</div>
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

  static childContextTypes = {
    increaseCount : React.PropTypes.func,
    count: React.PropTypes.number
  }

  getChildContext = () => {
    return {
      increaseCount: this.increaseCount,
      count: this.state.count
    }
  }

  increaseCount = () => {
    console.log(111)
    const count = this.state.count + 1;
    this.setState({count});
  }

  render () {
    return (
      <div>
        <Child1/>
        <Container>
          <Child2/>
        </Container>
      </div>
    )
  }
}
```  
从上面示例我们可以看到我们在Parent组件的中设置了childContextTypes，然后定义了一个函数getChildContext返回count以及increaseCount，然后在Parent组件的下层组件内先声明contextTypes，然后通过this.context.XXX进行调用。使用context就能够完成任意组件的通信，不过前提是任意组件必须有共同的顶层组件。其实context可以理解为活跃在定义它的组件树内的上下文，在这个组件树中的任意组件都可以先声明要调用的context的contextTypes，然后直接使用context即可。  

### 发布订阅模式

任意组件间的通信还可以通过发布订阅模式来实现。任意组件发布一个事件，在想要通信的组件内订阅这个事件做处理即可。示例代码如下：  
```javascript
const PubSub = {
  _events: [],
  sub (event, cb) {
    !this._events[event] && (this._events[event] = []);
    this._events[event].push(cb);
  },
  pub(event, ...args){
    if (this._events[event]) {
      this._events.forEach(function (cb) {
        cb(...args);
      })
    }
  }
};

class Comp1 extends React.Component {
  clickHandle = () => {
    PubSub.pub('text','发布!')
  }
  
  render() {
    return (
      <button onClick={this.clickHandle}>点我</button>
    )
  }
}

class Comp2 extends React.Component {
  constructor() {
    super();
    this.state = {
      text: '默认值'
    }
  }

  componentDidMount() {
    PubSub.sub('text',function (text) {
      this.setState({
        text
      });
    })
  }

  render() {
    return (
      <div>
        {this.state.text}
      </div>
    )
  }
}
```  
从上述代码可以看到我们在Comp2的componentDidMount的生命周期里订阅了一个名为text的事件，在Comp1中的按钮click事件发布了一个text的事件，Comp2在Comp1的点击后监听到text事件后会改变state中的text的值。上述笔者简单写了个发布订阅模型，在实际开发中推荐使用[PubSubJS](https://github.com/mroderick/PubSubJS) 或者 [EventEmitter](https://github.com/Olical/EventEmitter)。  

### redux
当然，我们还可以使用redux或者flux来实现组件间通信。redux是提供状态管理的容器，也采用发布订阅模式。具体使用及教程见未来文章，这里不再进行阐述。

##其他方法
除了上文中所提供的常用方法，组件间还可以通过ref，dom事件的冒泡机制，全局变量等方法进行通信<sup>【1】</sup>。由于这些不常用或者过于简单，这里便不再展开描述。

## 总结
从全文可知，我们要实现组件间的通信，不管采用什么方法，都要找到通信介质，通过通信介质来充当组件间联系的桥梁，这样才能够实现通信。在选择通信方法的时候，要根据实际项目以及需求来进行选择。