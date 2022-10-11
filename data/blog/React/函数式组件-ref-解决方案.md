---
title: 函数式组件 ref 解决方案
date: '2021-02-12'
tags: ['React']
summary: ''
---

对于 React 中需要强制修改子组件的情况，React 提供了 Refs 这种解决办法，使得我们可以操作底层 DOM 元素或者自定的 class 组件实例。除此之外，文档（v17.0.1）对函数式组件另有描述:

`不能在函数式组件上使用ref属性，因为他们没有实例`。

在函数式组件和 Hooks 大面积普及的现在，这个特性没有完全对标 class 组件，令人难以置信。经过一阵探索，发现确实是有对应的解决方案的：

**`useImperativeHandle`**

结合 React.forward ， <a href="https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle">useImperativeHandle 文档  </a>应该就能明白是如何使用的。

简而言之就是可以在函数式组件上使用 ref，通过`useImperativeHandle`这个`hook`可以指定暴露给父组件的值和函数。

案例：

修改子组件`Counter`中的值， 达到重置`count`的目的：

<img src="https://img.imgdb.cn/item/6007946f3ffa7d37b335f942.gif" width="300"/>

```jsx
export default function App() {
  return (
    <div>
      <button>reset</button>
      <Counter />
    </div>
  )
}
/** -------------------------------------- */
function Counter() {
  const [count, setCount] = useState(0)
  function increment() {
    setCount(count + 1)
  }
  return (
    <div>
      <hr />
      <span>{count}</span>
      <button onClick={increment}>+1</button>
    </div>
  )
}
```

> 对于这个案例，将`count`这个 state 往上提一层到 App 组件中是比较合适的，但是在这里重点讨论**操作子组件**。

使用`useImperativeHandle`，修改代码：

```jsx
export default function App() {
  const counterRef = useRef()
  function reset() {
    counterRef.current?.resetCount()
  }
  return (
    <div style={{ padding: 10 }}>
      <button onClick={reset}>reset</button>
      <MyCounter ref={counterRef} />
    </div>
  )
}
/** -------------------------------------- */
function Counter(props, ref) {
  const [count, setCount] = useState(0)
  useImperativeHandle(ref, () => ({
    resetCount: resetCount,
  }))
  function resetCount() {
    setCount(0)
  }
  function increment() {
    setCount(count + 1)
  }
  return (
    <div>
      <hr />
      <span>{count}</span>
      <button onClick={increment}>+1</button>
    </div>
  )
}
const MyCounter = React.forwardRef(Counter)
```

重点是`useImperativeHandle`中定义了`resetCount`，以及使用`React.forward`获取 ref，在`App`组件中为`MyCounter`中定义`ref`属性，然后就可以在外部父组件中使用通过`ref`调用子组件的`resetCount`方法。

<img src="https://img.imgdb.cn/item/600799b23ffa7d37b3388dc4.gif" width="300"/>

到这里，实际上已经达到了和`class`中`ref`对等的效果。

### 函数式组件的 Ref 是什么

将 ref 设置到 HTMl 元素上，获取的是对应的 DOM 元素，如 span:

<img src="https://img.imgdb.cn/item/6007a2d13ffa7d37b33d3d83.jpg" width="300"/>

设置到 class 组件上，获取的是 class 组件实例：

<img src="https://img.imgdb.cn/item/6007a34b3ffa7d37b33d75e3.jpg" width="600"/>

设置到函数式组件上的时候，获取的是一个包含可变值或函数的对象，如上例的 Counter 组件：

<img src="https://img.imgdb.cn/item/6007a4c23ffa7d37b33e5e7c.jpg" width="300"/>

`React.createRef` 和 `useRef `都是创建了一个包含`current`属性的对象，绑定`ref`时，对应的属性和函数都在`current`对应的对象中。

查看对应的`TypeScript`类型，`React.createRef`创建的是`React.RefObject`类型，是**只读**的。

<img src="https://img.imgdb.cn/item/6007a6fc3ffa7d37b33f41de.jpg" width="400"/>

而`useRef`创建的是`React.MutableRefObject`，是**可读写**的。可以保存任何可变的值，使用方式类似于`class`组件的`this`实例变量。（又是和`class`组件对标的一个点）

> <a href="https://zh-hans.reactjs.org/docs/hooks-reference.html#useref">文档</a>描述 useRef 为可以在其`.current`属性中保存一个可变值的“盒子”。

<img src="https://img.imgdb.cn/item/6007a75c3ffa7d37b33f8c60.jpg" width="600"/>
