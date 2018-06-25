
从 react 组件开始 web component

react 的组件声明方式
```javascript
class DiyCard extends React.Component {
    constructor() {...}
    componentDidMount() {...}
    render() {...}
}
ReactDOM.render(<DiyCard />, document.getElementById('app'));
```

或者在 jsx 中直接使用 `<DiyCard />` 或者 `<DiyCard>...</DiyCard>`

而 web components 是 HTML 的标准实现,因此不需要 react, 不需要 jsx, 不需要babel, 用最简单的 js 就可以创建一个自定义组件。


## 有些概念
- custom elements
- shadow DOM
- template AND slot


### custom elements

HTML 本身已经很多标记,简单点的比如 `div`, `span`, `p`, 稍微复杂些的如 `img` 或者表单元素 `input`, `select`等, 更复杂的如 `video`, `audio`; 而 custom elements 允许我们自己创建自定义的 HTML 标记。

#### custom elements 的一些规则
1. 必须包含短横线, 比如 diy-card, diy-more-card, 甚至 diycard-, 而类似 diycard, diy_card 都是不允许的。
2. 必须闭合, 我们知道像 `<hr />`, `<img /> ` 是可以自闭和的,但是自定义的 HTML 标记必须要写闭合标记, `<diy-card></diy-card>`。

#### 创建 custom elements
custom elements 通过 `window.customElements.define` 方法进行定义, `window.customElements` 是 `CustomElementsRegistry` 的实例, 但是 `CustomElementsRegistry` 无法通过 new 关键字再次实例化。

`window.customElements.define(name, constructor, options)` 接受三个参数, name 为自定义的标记, constructor 为具体实现, options 目前只在继承 built-in element时用到。

以下是一个简单示例：

```javascript
class Card extends HTMLElement {
    constructor() {
        super(); // 必须要首先执行 super
    }
}
window.customElements.define('diy-card', Card);
```

这样就声明了一个自定义的 HTML 标记 <diy-card>,当然现在如果把它放到页面中,没有任何效果,如果需要往其中添加内容,我们需要了解 shadow DOM。

##### Autonomous custom element && Customized built-in element

在上面的例子中 Card 继承自 HTMLElement,这样声明的标签是最基础与标准的 element ,这样的声明就是 `Autonomous custom element`；

但其实同样可以继承 HTML 内置的元素,比如 `<p>` HTMLParagraphElement,或者是 `<ul>` HTMLUlistElement；这样的声明就是 Customized built-in element,这样需在在第三个参数中传入 `{extends: 'p'}` 或 `{extends: 'ul'}` 这样子；具体可以参阅该 [规范](https://html.spec.whatwg.org/multipage/indices.html#element-interfaces)

### shadow DOM
shadow DOM 可以在页面创建出一个独立的 DOM 树,通常挂载在某个元素上,该元素称作 shadow host。在 shadow DOM 中的样式可以与外部DOM进行隔离,并且id与类名也无需担心与外部冲突,同时,外部的样式也不会影响内部。接下来我们要在 custom elements 中应用 shadow DOM。

#### 创建 shadow DOM
通过 `element.attachShadow({mode: 'open'})` 来创建对应 element 的 shadow DOM,其中 mode 参数可以是 open 或 closed; 这两者的区别是 open 可以通过 element.shadowRoot 拿到 shadow DOM ,进而可以操作内部的元素, 而 closed 的 element.shadowRoot 则返回 null。

一旦 shadow DOM 创建，则剩下的工作与普通 DOM 操作无异。

```javascript
class Card extends HTMLElement {
    constructor() {
        super(); // 必须要首先执行 super

        // 这里的 this 就是指向 custom element 实例自身
        const shadowRoot = this.attachShadow({mode: 'open'}); // 创建shadow DOM

        shadowRoot.innerHTML = `<h5>Title here</h5> <p>paragrpha here</p>`; // 粗暴的demo
    }
}
window.customElements.define('diy-card', Card);
```

#### 通过 attribute 传递参数(props)

#### shadow DOM 中的样式

#### shadow DOM 中的事件


### life cycle
custom elements 的生命周期

### template 与 slot


#### 参考
https://jeremenichelli.io/2017/10/the-web-components-experience/

https://developers.google.com/web/fundamentals/web-components/shadowdom?hl=zh-cn

https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM
