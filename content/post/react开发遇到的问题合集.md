---
title: "React开发遇到的问题合集"
date: 2019-06-26T10:19:17+08:00
categories:
- problem
- react
tags:
- runError
- audio
- 图片引用
- input
keywords:
- setState
- audio src
- 图片引用
- onChange
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

<!-- toc -->

# 1.Warning: setState(…): Cannot update during an existing state transition  
## 问题表现
当报这类错误时，说明你的props和states在渲染的时候更改了。错误代码如下所示：  
{{< codeblock "demo.jsx" "js" "" "js" >}}class LoggingButton extends React.Component {
constructor(props) {
    super(props);
    this.state = {
        singleJourney: false
    };
    this.handleButtonChange = this.handleButtonChange.bind(this);
  }

  handleButtonChange(value) {
        this.setState({
            singleJourney: value
        });
    }

  render() {
    return (
      <button active={!this.state.singleJourney} onClick={this.handleButtonChange(false)}>Retour</button>
      <button active={this.state.singleJourney} onClick={this.handleButtonChange(true)}>Single</button>
    );
  }
}
{{< /codeblock >}}  

## 解决方案
1. 修复方案一：  
{{< codeblock "demo.jsx" "js" "" "js" >}}class LoggingButton extends React.Component {
constructor(props) {
    //...
    this.handleButtonChangeRetour = this.handleButtonChange.bind(this, true);
    this.handleButtonChangeSingle = this.handleButtonChange.bind(this, false);
  }

  render() {
    return (
      <button active={!this.state.singleJourney} onClick={this.handleButtonChangeSingle}>Retour</button>
      <button active={this.state.singleJourney} onClick={this.handleButtonChangeRetour}>Single</button>
    );
  }
}
{{< /codeblock >}}   

2. 修复方案二：  
{{< codeblock "demo.jsx" "js" "" "js" >}}class LoggingButton extends React.Component {
  //...

  render() {
    return (
      <button active={!this.state.singleJourney} onClick={() => this.handleButtonChange(false)}>Retour</button>
      <button active={this.state.singleJourney} onClick={() => this.handleButtonChange(true)}>Single</button>
    );
  }
}
{{< /codeblock >}}  

## 注意事项 
关于方案二的箭头函数，在大多数情况下没什么问题，但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。因此官方建议构造器中绑定或使用 class fields 语法来避免这类性能问题。  
例如：  
{{< codeblock "demo.jsx" "js" "" "js" >}}class LoggingButton extends React.Component {
  // 此语法确保 `handleClick` 内的 `this` 已被绑定。
  // 注意: 这是 *实验性* 语法。
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
{{< /codeblock >}}   

# 2.直接更改audio的src属性，浏览器并没有相应更改src的播放路径  
## 问题表现  
浏览器无法识别对<source>的src属性做出的任何更改，如果直接在<audio>标签上更改src属性，浏览器将按照预期重新渲染/重置音频标签。错误代码如下所示：  
{{< codeblock "demo.jsx" "js" "" "js" >}}<audio controls controlsList="nodownload" >
    <source src={this.state.audioUrl} type="audio/ogg" />
    <track kind="captions" />
    您的浏览器不支持 audio 元素。
</audio>
{{< /codeblock >}}  

## 解决方案 
通过ref直接获取audio对象，然后在需要给这个audio对象赋值的地方提供音频路径。具体代码如下：  
{{< codeblock "demo.jsx" "js" "" "js" >}}export default class Audio extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      audioUrl: null,//接口返回的音频地址
    };
  }
 
  componentWillReceiveProps(nextProps) {
  //通过接口获取音频地址 
    if (nextProps.getAudioUrl) {
      this.setState({ audioUrl: nextProps.getAudioUrl });
    } else {
      this.setState({ audioUrl: null });
    }
  }
  //这个周期在render后执行，此时你的audio对象是存在的
  componentDidUpdate() {
    if (this.state.audioUrl) {
      let theAudio = this.audioValue;
      theAudio.src = '';
      theAudio.src = this.state.audioUrl;
      theAudio.load();
    }
  }
  render(){
    return({
         audioUrl&&
         <audio ref={(audio) => { this.audioValue = audio; }} controls preload="none" controlsList="nodownload" >
            <track kind="captions" />
            您的浏览器不支持 audio 元素。
         </audio>})
  }
 }
{{< /codeblock >}}   

# 3.如何引入本地静态图片和背景图片  
1. require引入，例如：  
{{< codeblock "demo.jsx" "js" "" "js" >}}// 引入图片
<img src={require("../images/demo.png")} />  

const bgGround={
    background: `url(${require("../images/demo.png")})`
}

<span style={bgGround}></span>
{{< /codeblock >}}  

2. 使用import方法，例如：  
{{< codeblock "demo.jsx" "js" "" "js" >}}import img from '../images/demo.png' //引入图片的路径
<img src={img} />     //变量的方式引入图片，记得用{}大括号来进行引用  

const bgGround={
    background: `url(${img})`
}

<span style={bgGround}></span>
{{< /codeblock >}}  

# 4.input元素中文输入下的onChange事件问题  
## 问题表现  
react中的onChange事件类似于原生的input事件，当你想要输入中文的“事件”时，每按一次键都会触发onChange事件，所以会发现输入框内已经存在了“shijian”的英文字母内容，这不是我们想要的结果。代码如下所示：
{{< codeblock "demo.jsx" "js" "" "js" >}}...
<input ref="inputTest" type="text" placeholder="test" value={this.state.val} onChange={this.inputValue}/>
...
inputValue(e){
    this.setState({
        val:e.target.value
    })
}
{{< /codeblock >}} 

## 解决方案  
1. 解决方案一：使用uncontrolled 组件的方式，抛弃onChange事件  
{{< codeblock "demo.jsx" "js" "" "js" >}}<input ref="inputTest" type="text" placeholder="test" onCompositionEnd={this.handleComposition} />
{{< /codeblock >}}  

2. 解决方案二：使用compositionEvent组合事件与onChange事件做兼容
{{< codeblock "demo.jsx" "js" "" "js" >}}<input ref="inputTest" type="text" placeholder="test" 
              onCompositionStart={this.handlingComposition} 
              onCompositionUpdate={this.handlingComposition} 
              onCompositionEnd={this.handleComposition} 
              onChange={this.inputValue}/>
...
handlingComposition(){
    this.isCompositionEnd = false;
}
handleComposition(e){
    this.isCompositionEnd = true;
}
inputValue(e){
    if(this.isCompositionEnd){
        this.setState({
            val:e.target.value
        })
    }
}
{{< /codeblock >}}   

    以上代码会存在一点小问题，需要确保`onCompositionEnd在onChange`事件`前`触发，一旦有的浏览器存在兼容问题，两者的执行顺序相反，会导致onChange事件永不触发，因此，最好在handleComposition函数中重复执行一次onChange中的逻辑，避免出现兼容问题。