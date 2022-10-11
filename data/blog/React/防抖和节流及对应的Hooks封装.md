---
title: 防抖和节流及对应React Hooks封装
date: '2021-02-22'
tags: ['JavaScript', 'React']
summary: 防抖和节流总结一下，再配合 ReactHook 封装试试
---

## Debounce

> debounce 原意`消除抖动`，对于事件触发频繁的场景，只有最后由程序控制的事件是有效的。

防抖函数，我们需要做的是在一件事触发的时候设置一个定时器使事件延迟发生，在定时器期间事件再次触发的话则清除重置定时器，直到定时器到时仍不被清除，事件才真正发生。

```js
const debounce = (fun, delay) => {
  let timer
  return (...params) => {
    if (timer) {
      clearTimeout(timer)
    }
    timer = setTimeout(() => {
      fun(...params)
    }, delay)
  }
}
```

如果事件发生使一个变量频繁变化，那么使用`debounce`可以降低修改次数。通过传入修改函数，获得一个新的修改函数来使用。

如果是`class`组件，新函数可以挂载到组件`this`上，但是函数式组件局部变量每次`render`都会创建，`debounce`失去作用，这时需要通过`useRef`来保存成员函数（下文`throttle`通过`useRef`保存函数），是不够便捷的，就有了将`debounce`做成一个`hook`的必要。

```jsx
function useDebounceHook(value, delay) {
  const [debounceValue, setDebounceValue] = useState(value)
  useEffect(() => {
    let timer = setTimeout(() => setDebounceValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])
  return debounceValue
}
```

在函数式组件中，可以将目标变量通过`useDebounceHook`转化一次，只有在满足`delay`的延迟之后，才会触发，在`delay`期间的触发都会重置计时。

配合`useEffect`，在`debounce value`改变之后才会做出一些动作。下面的`text`这个`state`频繁变化，但是依赖的是`debounceText`，所以引发的`useEffect`回调函数却是在指定延迟之后才会触发。

```jsx
const [text, setText] = useState('')
const debounceText = useDebounceHook(text, 2000)
useEffect(() => {
  // ...
  console.info('change', debounceText)
}, [debounceText])

function onChange(evt) {
  setText(evt.target.value)
}
```

上面一个搜索框，输入完成`1`秒（指定延迟）后才触发搜索请求，已经达到了防抖的目的。

<hr/>

## Throttle

> throttle 原意`节流阀`，对于事件频繁触发的场景，采用的另一种降频策略，一个时间段内只能触发一次。

节流函数相对于防抖函数用在事件触发更为频繁的场景上，滑动事件，滚动事件，动画上。

看一下一个常规的节流函数 (ES6)：

```js
function throttleES6(fn, duration) {
  let flag = true
  let funtimer
  return function () {
    if (flag) {
      flag = false
      setTimeout(() => {
        flag = true
      }, duration)
      fn(...arguments)
      // fn.call(this, ...arguments);
      // fn.apply(this, arguments); // 运行时这里的 this 为 App组件，函数在 App Component 中运行
    } else {
      clearTimeout(funtimer)
      funtimer = setTimeout(() => {
        fn.apply(this, arguments)
      }, duration)
    }
  }
}
```

（使用`...arguments`和 call 方法调用展开参数及 apply 传入 argument 的效果是一样的）

> 扩展：在`ES6`之前，没有箭头函数，需要手动保留闭包函数中的`this`和参数再传入定时器中的函数调用：
>
> 所以，常见的`ES5`版本的节流函数：
>
> ```js
> function throttleES5(fn, duration) {
>   let flag = true
>   let funtimer
>   return function () {
>     let context = this,
>       args = arguments
>     if (flag) {
>       flag = false
>       setTimeout(function () {
>         flag = true
>       }, duration)
>       fn.apply(context, args) // 暂存上一级函数的 this 和 arguments
>     } else {
>       clearTimeout(funtimer)
>       funtimer = setTimeout(function () {
>         fn.apply(context, args)
>       }, duration)
>     }
>   }
> }
> ```

如何将节流函数也做成一个自定义`Hooks`呢？**上面的防抖的`Hook`其实是对一个变量进行防抖的，从一个`不间断频繁变化的变量`得到一个`按照规则（停止变化delay时间后）`才能变化的变量**。我们**对一个变量的变化进行节流控制，也就是从一个`不间断频繁变化的变量`到`指定duration期间只能变化一次(结束后也会变化)`的变量**。

`throttle`对应的`Hook`实现：

(标志能否调用值变化的函数的`flag`变量在常规函数中通过闭包环境来保存，在`Hook`中通过`useRef`保存)

```js
function useThrottleValue(value, duration) {
  const [throttleValue, setThrottleValue] = useState(value)
  let Local = useRef({ flag: true }).current
  useEffect(() => {
    let timer
    if (Local.flag) {
      Local.flag = false
      setThrottleValue(value)
      setTimeout(() => (Local.flag = true), duration)
    } else {
      timer = setTimeout(() => setThrottleValue(value), duration)
    }
    return () => clearTimeout(timer)
  }, [value, duration, Local])
  return throttleValue
}
```

对应的在手势滑动中的使用：

```js
export default function App() {
  const [yvalue, setYValue] = useState(0)

  const throttleValue = useThrottleValue(yvalue, 1000)

  useEffect(() => {
    console.info('change', throttleValue)
  }, [throttleValue])

  function onMoving(event, tag) {
    const touchY = event.touches[0].pageY
    setYValue(touchY)
  }
  return <div onTouchMove={onMoving} style={{ width: 200, height: 200, backgroundColor: '#a00' }} />
}
```

这样以来，手势的`yvalue`值一直变化，但是因为使用的是`throttleValue`，引发的`useEffect`回调函数已经符合规则被节流，每秒只能执行一次，停止变化一秒后最后执行一次。

## 对值还是对函数控制

上面的`Hooks`封装其实对值进行控制的，第一个防抖的例子中，输入的`text`跟随输入的内容不断的更新`state`，但是因为`useEffect`是依赖的防抖之后的值，这个`useEffect`的执行是符合防抖之后的规则的。

可以将这个防抖规则提前吗? 提前到更新`state`就是符合防抖规则的，也就是只有指定延迟之后才能将新的`value`进行`setState`，当然是可行的。但是这里搜索框的例子并不好，对值变化之后发起的请求可以进行节流，但是因为搜索框需要实时呈现输入的内容，就需要实时的`text`值。

对手势触摸，滑动进行节流的例子就比较好了，可以通过设置`duration`来控制频率，给手势值的`setState`降频，每秒只能`setState`一次：

```js
export default function App() {
  const [yvalue, setYValue] = useState(0)
  const Local = useRef({ newMoving: throttleFun(setYValue, 1000) }).current

  useEffect(() => {
    console.info('change', yvalue)
  }, [yvalue])

  function onMoving(event, tag) {
    const touchY = event.touches[0].pageY
    Local.newMoving(touchY)
  }
  return <div onTouchMove={onMoving} style={{ width: 200, height: 200, backgroundColor: '#a00' }} />
}

//常规节流函数
function throttleFun(fn, duration) {
  let flag = true
  let funtimer
  return function () {
    if (flag) {
      flag = false
      setTimeout(() => (flag = true), duration)
      fn(...arguments)
    } else {
      clearTimeout(funtimer)
      funtimer = setTimeout(() => fn.apply(this, arguments), duration)
    }
  }
}
```

这里就是对函数进行控制了，控制函数`setYValue`的频率，将`setYValue`函数传入节流函数，得到一个新函数，手势事件中使用新函数，那么`setYValue`的调用就符合了节流规则。如果这里依然是对手势值节流的话，其实会有很多的不必要的`setYValue`执行，这里对`setYValue`函数进行节流控制显然更好。

> 需要注意的是，得到的新函数需要通过`useRef`作为“实例变量”暂存，否则会因为函数组件每次`render`执行重新创建。
