---
title: React-Native基础（4）：组件引用Ref
date: 2017-05-22 15:58:44
tags: react-native
categories : react-native
---

在典型的React数据流中，父组件与子组件的唯一交互方式就是通过Props。修改子组件，就需要使用新的Prop来重新渲染。但是在有些情况，需要在数据流之外，直接来执行子组件的修改操作。React提供了使用Ref方式来解决直接修改子组件或子DOM元素的问题。

- 添加Ref到DOM元素
`ref`是React组件一个特殊属性，使用`回调函数`作为属性，函数的接收组件的基本DOM元素作为它的参数。
```js
<Component ref={(el) => this.el = el }/>
```
定义的回调函数会在组件加载和卸载时立即调用。
在组件卸载时函数中元素的值为`null`。
一个简单的例子
```js
...
class CustomTextInput extends React.Component{
   constructor(props){
       super(props);
       this.focus = this.focus.bind(this);
   }
   
   focus(){
       // 获取文字焦点
       this.textInput.focus();
   }
   
   render(){
       // 使用ref回调函数存储text input 元素的引用
       return (
        <div>
          <input 
            type="text" 
            ref={(input) => { this.textInput = input; }} />
          <input
            type="button"
            value="Focus the text input"
            onClick={this.focus} />
        </div>
       );
   }
}

```

```js
// es6 箭头函数
ref={input => this.textInput = input}
```

- 添加Ref到Class组件

当为组件添加Ref回调函数，回调函数将使用自定义的组件作为它的参数。
例如：
```js
class AutoFocusTextInput extends React.Component{
    constructor(props){
        this.textInput.focus();
    }
    
    render(){
        return(
          <CustomTextInput
            ref={ (input) => {this.textInput = input; }}
            />
        );
    }
}
```
这个方式只能用于以`class`方式声明的`CustomTextInput`

```js
class CustomTextInput extends React.Component {
    // ...
}
```
- Ref和函数组件

因为函数式组件没有组件实例，所以不能够通过上面的方式来给函数组件添加ref。
通过两种方式来解决
    1. 将函数组件转为Class组件。
    2. 在函数组件内部使用`ref`，这种方式同样适应于class组件。
    
```js
function CustomTextInput(props){
    // 需要声明引用的textInput
    let textInput = null;
    function handleClick(){
        textInput.focus();
    }
    
    return (
      <div>
        <input 
          text="text"
          ref={ (input) => { textInput = input; }}
          />
        <input 
          text="button"
          value="Focus the text input"
          onClick={ handleClick }
          />
      </div>
    );
}

```
- 暴露DOM的Ref给父组件

在极少情况下，父组件需要访问子组件。因为破坏了组件的封装性，这种做法并不推荐，但是有时对于触发焦点和测量子DOM节点的位置或大小会有用。如果给子组件添加ref时，也并不是理想解决方案，因为ref回调函数的参数是子组件，而不是DOM节点元素，并且不适用函数式组件。
因此，在这些情况建议暴露子组件的特殊属性，子组件使用任意名称（如：`inputRef`）的函数类型的Prop，并把这个属性添加大DOM节点元素中，作为`ref`属性。就可以通过中间组件将父组件的ref函数传递给子组件中的DOM节点
```js
function CustomTextInput(props){
  return (
    <div>
      <input ref={props.inputRef}/>
    </div>
  );
}

class Parent extends React.Component{
    render (
      return (
        <CustomTextInput
          inputRef={el => this.inputElement = el}
          />
      );
    )
}
```
上面代码中父组件使用`inputRef`属性将ref回调函数传递给中间组件`CustomTextInput`，`CustomTextInput`又将函数传给`<input />`的ref属性。最终父组件获取到对`<input />`的引用。这种形式不仅适用于Class组件，同样也适用于函数组件。
这种形式的另一个好处就是：对于深层的DOM节点同样有效。假设父组件不需要DOM节点引用，但是祖父组件想要获取子组件中DOM节点元素的引用，可以通过让祖父组件指定inputRef属性，传递给父组件，父组件再把`inputRef`转给`CustomTextInput`。

```js
function CustomTextInput(props){
  return (
    <div>
      <input ref={props.inputRef}/>
    </div>
  );
}

class Parent extends React.Component{
    render (
      return (
        <div>
          My input: <CustomTextInput inputRef={props.inputRef} />
        </div>
      );
    )
}


class GrandParent extends React.Component{
  render() {
    return (
      <Parent
        inputRef={el => this.inputElement = el}
      />
    );
  }
}

```
> 综合考虑下，不建议暴露DOM节点。 因为破坏了组件的封装，而且需要子节点添加额外的属性。

- 说明

如果`ref`以内联函数形式定义，在更新时会被调用两次
    - 第一次为函数的参数为null，第二次为DOM元素
因为每次render时，都要创建一个新的函数实例，建议通过组件bind函数来定义ref回调函数。

- 适应场景

    1. 管理输入焦点，文字选择或者媒体播放时；
    2. 触发必要的动画时；
    3. 与第三方DOM库交互时；

避免使用在可用声明方式的情景下。如：通过定义`isOpen`的方式，避免暴露`Dialog`的`open`和`close`的方法。

- 避免过度使用

在使用`ref`时，一定要经过慎重思考，尤其是拥有State状态的组件。首先考虑使用Props

[Refs and the DOM](https://facebook.github.io/react/docs/refs-and-the-dom.html)