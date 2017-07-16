---
title: React-Native基础（2）：聊聊Props与States
date: 2017-05-18 14:43:29
tags: react-native
categories : react-native
---

React-Native 基于状态实现对DOM控制和渲染。组件状态分为两种：一种是组件间的状态传递、另一种是组件的内部状态，这两种状态使用props和state表示。props用于从父组件到子组件的数据传递。组件内部也有自己的状态：state，这些状态只能在组件内部修改。
## Props
在React-Native中大多数组件创建时可以带有不同的参数，这些参数被称为`props`。
如：React-Native的常用组件`Image`。当创建`Image`时，可以用`source`属性来指定要显示的图片资源。
```js
import React , { Component } from 'react'
import { AppRegistry , Image , } from 'react-native'

class Bananas extends Component {
    render (){
        let pic = {
            uri : 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg'
        }
        return (
            <Image source={pic} style={{ width : 193 , height : 110}}/>
        );
    }
}

AppRegistry.registerComponent('Bananas', () => Bananas);
```
自定义的组件也可以使用`Props`，根据`Props`名称引用指定的属性值。如在组件`Greeting`的`render`方法中用`this.props.name`应用组件的`name`属性值。

```js
import React , { Component } from 'react'
import { AppRegistry , Text , View} from 'react-native'

class Greeting extends Component {
  render() {
    return (
      <Text>Hello {this.props.name}!</Text>
    );
  }
}

class LotsOfGreetings extends Component {
  render() {
    return (
      <View style={{alignItems: 'center'}}>
        <Greeting name='Rexxar' />
        <Greeting name='Jaina' />
        <Greeting name='Valeera' />
      </View>
    );
  }
}

AppRegistry.registerComponent('LotsOfGreetings', () => LotsOfGreetings);

```
- 默认属性

使用`static`定义组件的默认属性值

```js
import React , { Component } from 'react'
import { AppRegistry , Text , View} from 'react-native'
class Greeting extends Component {
  static defaultProps = {
      name : 'alex' , s
  }
  render() {
    return (
      <Text>Hello {this.props.name}!</Text>
    );
  }
}
...
```
- 约束与检查

使用`PropTypes` 对`Props`值的类型进行约束
```js
import React , { Component , PropTypes , } from 'react'
import { AppRegistry , Text , View , } from 'react-native'
class Greeting extends Component {
  ...
  static propTypes = {
      name : PropTypes.string , 
      age : PropTypes.num , 
      greet : PropTypes.func.isRequired ,
  }
  render() {
    return (
      <Text>Hello {this.props.name}!</Text>
    );
  }
}
```
如果必须使用指定属性值，通过`PropTypes.{type}.isRequired`来约束。

- 扩展语法...

传入对象的属性会被复制到组件内，可以多次使用，可以与其他属性组合使用，后面的属性值覆盖之前的属性。[JSX 展开属性](http://reactjs.cn/react/docs/transferring-props.html)
```js
import React , { Component , PropTypes , } from 'react'
import { AppRegistry , Text , View , } from 'react-native'
class Greeting extends Component {
  ...
  render() {
    return (
      <Text>Hello {this.props.name}</Text>
    );
  }
}

...
<Greeting {...this.props} name='alex' />
...
```
- Props属性解构
通过解构Props属性，直接引用解构后的属性。
```js
import React , { Component , PropTypes , } from 'react'
import { AppRegistry , Text , View , } from 'react-native'
class Greeting extends Component {
  ...
  render() {
    let {name , ...props} = this.props;
    return (
      <Text>Hello {name}</Text>
    );
  }
}
```
## State
控制组件有两种类型数据：`Props`和`State`。`Props`是由父元素设定的固定属性值，在组件整个生命周期中是不可变，使用`State`来更新数据，刷新UI。
通常在组件构造方法中初始化State，调用`setState()`方法更新State数据。
简单的文字闪烁的例子，文字的内容在组件创建时使用Props设置为了固定值，文字的闪烁状态则由`State`来控制。

```js
import React, { Component } from 'react';
import { AppRegistry, Text, View } from 'react-native';
class Blink extends Component {
  constructor(props) {
    super(props);
    this.state = {showText: true};

    // 每秒修改State
    setInterval(() => {
      this.setState({ showText: !this.state.showText });
    }, 1000);
  }

  render() {
    let display = this.state.showText ? this.props.text : ' ';
    return (
      <Text>{display}</Text>
    );
  }
}

class BlinkApp extends Component {
  render() {
    return (
      <View>
        <Blink text='I love to blink' />
        <Blink text='Yes blinking is so great' />
        <Blink text='Why did they ever take this out of HTML' />
        <Blink text='Look at me look at me look at me' />
      </View>
    );
  }
}

AppRegistry.registerComponent('BlinkApp', () => BlinkApp);

```
- 不要直接修改State属性值
赋值的形式唯一的地方只能在构造器中执行
```js
class ConcreateCompnent extends Component {
    constructor(props){
        super(props);
        this.State = {...};
    }
}
```
使用`setState()`函数修改属性
```js
// 错误
this.state.name = 'lucky';
// 正确
this.state.setState={ {name : 'lucky'} };
```

- 异步更新State

为了性能，React会批处理连续多个`setState`操作。
因为`this.props`和`this.state`可能执行一步更新，所以调用`this.props`或`this.state`时的值并不正确。
```js
// 错误
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
建议`setState`接收函数的形式来更新state。因为函数的形式会使用前一个`state`作为第一个参数，同时更新的`props`属性作为第二个参数。
```js
// 正确
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

- 合并更新State

调用setState方法时，React会合并提供的state值。
```js
class ConcreateComponent extends Component {
    constructor(props){
        super(props);
        this.state = {
            posts:[] ,
            comments : [] , 
        }
    }
    
    componentDidMount(){
        fetchPosts().then( response => {
            this.setState({
                posts : response.posts
            });   
        });
        
        fetchComment().then( response => {
            this.setState({
                comments : response.comments
            })
        })
    }
}

```
- 数据流向

React中的数据流是单向的，只会从父组件传递到子组件。属性props（properties）是父子组件间进行状态传递的接口，React会向下遍历整个组件树，并重新渲染使用这个属性的组件。



