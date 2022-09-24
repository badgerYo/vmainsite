---
title: React Portal - 弹出层的优秀解决方案
date: '2021-02-12'
tags: ['React']
summary: ''
---

对于需要使用弹出层的需求 ，`Portal`可以说是提供了一种完美的解决方案。相比于`React Native`中的实现更多的使用`Modal`或者绝对定位，`Portal`实在是简易友好得多。

## 场景

对话框，确认提示框，悬浮窗这些组件，一般都要做一个比当前视图层层级更高的`View`，但是现有的方案都很难跳出父容器的约束。如何破除这个约束？

> `Portal`提供了一种将子节点渲染到存在于父组件以外的 `DOM` 节点的优秀的方案

**子节点可以被渲染到父组件之外的`DOM`上**，这不就是解除了这个约束了吗？

> Portal：传送门。steam 上有一款名为 Portal2 的游戏，互相利用传送门进行位移的配合闯关游戏，传送的含义和这里颇为相似，不过这里是“传送”组件。

## 使用和效果

普通的 render 方法就是将组件内的元素挂载到最近的 DOM 节点中。而 Portal 则通过一个 API，将元素挂载到任意有效的 DOM 节点上。

**`ReactDOM.createPortal( children,container,key? )`**

具体参数类型如下：

<img src="https://img.imgdb.cn/item/601265d43ffa7d37b3b6489c.jpg" width=700 />

现在开始在代码中使用：

1. 在`public`目录下的`index.html`文件中，设置`<div id="modal"></div>`。后面 React 元素会被放到`modal`这个`div`里，这个`DOM`元素就是`container`。

   ```html
   <div id="root"></div>
   <div id="modal"></div>
   ```

2. 对该 API 做简单封装，构建`Modal`组件。因为`Portal API`是读`children`子组件的，所以`Portal`肯定是作为容器来用。

   > （这里直接操作操作了`children`，你也可以像<a href="https://zh-hans.reactjs.org/docs/portals.html">Portal 文档</a>一样，在其中多加一个`div`层级隔离）

   ```jsx
   const modalDivElement = document.getElementById('modal')

   export default function Modal({ children }) {
     return ReactDOM.createPortal(children, modalDivElement)
   }
   ```

   使用`createPortal API`，Modal 组件中的子元素`children`会被渲染到`modalDivElement`中，而不管你的`Modal`在何处使用。

3. 使用`Modal`组件。在红色背景`div`中嵌入了`Modal`, `Modal`中只有一个`div`字符串`zhangsan`。

   ```JSX
   export default function App() {
     return (
       <div style={{ backgroundColor: "#a00", width: `200px`, height: `100px` }}>
         <Modal>
           <div>zhangsan</div>
         </Modal>
       </div>
     );
   }
   ```

   现在来看看效果：

   <img src="https://img.imgdb.cn/item/60126ef73ffa7d37b3bc6c89.jpg">

   如果没有`Modal`层级，`zhanghsan`是应该在红色区域里面的。但是为什么加了`Modal`就在外面了？这时检查渲染结构，秘密就在这里：

   <img src="https://img.imgdb.cn/item/601270413ffa7d37b3bd458a.jpg" />

   可以看到，`Modal`中的元素被渲染到了`id="modal"`的这个`div`里。来回顾一下发生了什么，我们的代码组件结构是这样：

   ```html
         <div id="root">
           <div style={{ backgroundColor: "#a00", width: `200px`, height: `100px` }}>
             <Modal>
               <div>zhangsan</div>
             </Modal>
           </div>
         </div>
         <div id="modal"></div>
   ```

   但是实际的渲染结构却是这样（观察其中`Modal`子元素的变化）：

   ```html
         <div id="root">
           <div style={{ backgroundColor: "#a00", width: `200px`, height: `100px` }</div>
         </div>
         <div id="modal">
           <div>zhangsan</div>
         </div>
   ```

   实际渲染结构和代码结构并不一致了 ,是因为**`Modal`中的`Portal`将子元素方法放到了指定的`container`中。**子元素被跨层级渲染，就像被传送过去一样，这便是`Portal`。

## 制作一个 Modal 弹出框组件

上面只是演示了`Portal`传送子组件的效果，如果要做到弹出蒙层的效果，只需要在需要传送的`children`子元素上添加一个`div`包裹并添加样式，如果弹出层样式一致，可以直接`Modal`组件中，当然也可以在具体的`children`中实现并自定义样式。

实例：

<img src="https://img.imgdb.cn/item/601281633ffa7d37b3c8a5da.gif" width=400/>

例子代码：

```jsx
export default function App() {
  const [isShow, setIsShow] = useState(false)
  function click(e) {
    console.info('e', e)
    setIsShow(!isShow)
  }
  return (
    <div
      style={{
        backgroundColor: '#a00',
        width: `200px`,
        height: `100px`,
      }}
      onClick={click}
    >
      {isShow && (
        <Modal>
          <span>zhangsan</span>
        </Modal>
      )}
    </div>
  )
}
```

`Modal`组件：

```jsx
const modalDivElement = document.getElementById('modal')

export default function Modal({ children }) {
  const modalContent = (
    <div
      style={{
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        position: 'absolute',
        top: 0,
        left: 0,
        right: 0,
        bottom: 0,
        backgroundColor: `rgba(0, 0, 0, 0.5)`,
      }}
    >
      {children}
    </div>
  )
  return ReactDOM.createPortal(modalContent, modalDivElement)
}
```

如果`Modal`交互和内容切换较多，可以进一步封装后通过`ref`命令式的调用，动态传入`Modal`和子组件切换。

## Portal 事件冒泡

`Portal`中的事件冒泡是遵从`React`结构的，并不是实际渲染的`DOM`元素结构，也就是说上面的`root`节点可以获取到`modal`中冒泡出来的事件。

而，上面的页面中的`click`事件，可以获取点击红色区域触发`Modal`的事件，也能获取到`Modal`内部的点击`span`的事件：

<img src="https://img.imgdb.cn/item/601288e33ffa7d37b3cd654a.jpg"/>

这样，`Modal`中的交互控制并没有脱离当前的页面或组件，和没有使用`Portal`时的事件表现一致。
