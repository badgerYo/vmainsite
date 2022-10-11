---
title: React Native Animated
date: '2020-01-13'
tags: ['ReactNative']
summary: ''
---

如果要在一个 React Native 应用中添加动画效果，首先考虑的就是官方提供的 Animated 。通过定义输入和输出的动画状态值，调用 start/stop 方法就可以执行起来，使用上可以说是非常友好方便。

#### _Animated 动画组件_

如果让某个界面元素拥有动画效果，那它应该使用如下的动画组件：

- Animated.View
- Animated.Text
- Animated.Image
- Animated.ScrollView
- Animated.FlatList
- Animated.SectionList

#### _动画类型_

Animated 　可以创建三种动画类型，直接调用就能创建一个动画对象，如 Animated.timing(...) ，动画对象可以调用 start() 启动。

- timing 　先加速至全速，再逐渐减速渐停 (使用最多)
- spring 　执行动画时会超出最终值然后弹回，适用于弹性的动态效果
- decay 　以一定的初始速度开始变化，然后变化速度越来越慢直至停下

例如：

```javascript
// 创建并启动一个 spring 类型的动画
Aimated.spring(this.state.value, { ...动画配置 }).start()
```

#### _动画值_

动画执行过程中的变量不是普通的变量，只能是动画值，有两种类型：

- Animated.Value()　表示单个值
- Animated.ValueXY()　表示向量(矢量)值. （可用于改变 x,y 向的坐标）

实例：

```javascript
this.state = {
  dynamicHeight: new Animated.Value(0), //初始化
}

this.state.dynamicHeight.setValue(10) //重新设定动画值
```

#### _计算动画值_

一些时候所用的动画值并不是可以直接设定的，或者多个动画值存在一定关系，转换后使用更加方便，也就是在使用之前需要进行计算，这时就需要用到一些常用计算方法。

例如: 一个长宽比为 3：2 的图片，要实现放大动画，为保持变化过程中的长宽比不变，这时就可以用一个公因数， 则 宽度 = 公因数 乘以 2 ，长度 = 公因数 乘以 3 ，在 state 中变化的动画值就是这个公因数，实际使用的动画值就是通过乘法 Animated.multiply() 得到的动画值。

```javascript
// - 宽度计算 -
// factor 是 state 中的变量 ，乘以 2 得到真实动画值后再将真实动画值设置到属性或样式上
const realAnimatedWidth = Animated.multiply(this.state.factor, new Animated.Value(2))
```

可用的计算方法：

- Animated.add() 　加
- Animated.subtract()　减
- Animated.multiply()　乘
- Animated.divide()　除
- Animated.modulo()　模（取余）

#### _组合类型_

当多个动画需要需要以一定规律进行（多个动画值变动）的时候，就需要用到组合动画，Animated 提供了几种方法将动画按照一定方式组合：

- Animated.delay(time) 　按照给定延迟后执行动画
- Animated.parallel(animations, config?)　同时执行多个动画
- Animated.sequence(animations)　顺序执行，一个动画结束后开始执行下一个
- Animated.stagger(time, animations)　按照给定延迟顺序启动多个动画，可能并行重叠。
- loop(animation, config?) 　无限循环一个指定动画

举例：

```javascript
//并行两个动画   同时进行一个 timing 动画和一个 spring 动画
Animated.parallel([
	Animated.timing(...),
	Animated.spring(...)
]).start()
```

#### _插值_

根据一个动画值的变化范围，获取另一个动画值的范围，需要使用插值。 插值函数 interpolate 可以将输入范围映射到输出范围，一般是使用线性插值，但也可以指定 easing 函数 [[查看 Easing 介绍](https://reactnative.cn/docs/animations/ 'Easing 介绍')]。

对于一些属性值是字符串的变量，比如 rotate ，从 '0deg' 到 '180deg' ，也应该使用插值来获取变化范围。

- AnimatdValue.interpolate(config)

例如：

```javascript
	// Y 轴上的偏移量根据 fadeAnim 的值变化
	// fadeAnim -> [0,1] translateY -> [150,0]
	style={{
		opacity: this.state.fadeAnim, // Binds directly
		transform: [{
			translateY: this.state.fadeAnim.interpolate({
				inputRange: [0, 1],
				outputRange: [150, 0]  // 0 : 150, 0.5 : 75, 1 : 0
			}),
		}],
	}}
```

<br />

#### _一个完整的简单动画的实现_

<br />

一个完整的简单动画可以分如下几步实现

1. 定义动画值变量并初始化

2. 定义动画配置和执行方法

3. 将动画值绑定到组件上，用于改变属性或样式

4. 触发动画执行的事件或操作

对应于下面的例子来看，会更容易理解上述每一步。

##### 案例 1

**[ 动画 1 -动画值控制高度]** 点击展开视图的动画，效果如下：

![点击展开-动画效果](https://user-gold-cdn.xitu.io/2019/11/8/16e4a94286ce0ddf?w=350&h=216&f=gif&s=74437)

通过将 Animated.View 的高度设置成动画值实现。 以上三步均在代码中标出

```javascript
export default class AnimatedView extends PureComponent {
  constructor(props) {
    super(props)
    this.state = {
      dynamicHeight: new Animated.Value(0), //1 - 构造方法中定义 dynamicHeight 动画变量
    }
  }

  startViewAnimation = () => {
    //2 - 配置动画和定义执行方法
    let targetHeight = 80
    if (this.state.dynamicHeight._value === 80) {
      //根据高度判断是否已展开
      targetHeight = 0 //已展开的话，则需要将 View 收起，高度回到初始值 0
    }
    Animated.spring(
      //定义弹性动画
      this.state.dynamicHeight, //改变的动画变量
      {
        toValue: targetHeight, //目标值，也就是动画值变化后的最终值
        duration: 300, //动画持续时长
      }
    ).start()
  }

  render() {
    return (
      <View style={{ flex: 1 }}>
        <View style={{ width: 200, position: 'absolute', top: 30 }}>
          <View>
            <TouchableOpacity
              onPress={this.startViewAnimation} // 4 - 触发动画执行的点击事件
              style={styles.button}
            >
              <Text>点击展开</Text>
            </TouchableOpacity>
          </View>

          <Animated.View
            style={{
              backgroundColor: '#eee',
              justifyContent: 'space-around',
              height: this.state.dynamicHeight,
            }} //3 - 将高度是设为动画值，即用动画值改变 style （height）
          ></Animated.View>
        </View>
      </View>
    )
  }
}
```

##### 案例 2

**[ 动画 2 - 动画值插值转换]** 点击跳动的心形图：

![跳动效果](https://user-gold-cdn.xitu.io/2019/11/8/16e4a94287034a61?w=286&h=115&f=gif&s=49714)

动画值 imageSize 从 0 到 1，映射成对应的实际图片大小，再设置到 Animated.Image 的宽高样式上。

```javascript
export default class HeartView extends PureComponent {
  constructor(props) {
    super(props)
    this.state = {
      imageSize: new Animated.Value(0), //动画初始值为 0，目标值为 1 。会经过插值转换后应用到 Animated.Image上
    }
  }

  startImageAnimation = () => {
    //动画开始前先将初始值设为0，否则第一次执行后this.state.imageSize的动画值一直是 1, 不再有效果

    this.state.imageSize.setValue(0)
    Animated.spring(this.state.imageSize, {
      toValue: 1,
      duration: 2000,
    }).start()
  }

  render() {
    //动画值从0-1，对其进行插值转换 ，映射成实际的图片大小
    //再将实际的图片大小设置给 Animated.Image 的样式

    const imageSize = this.state.imageSize.interpolate({
      inputRange: [0, 0.5, 1],
      outputRange: [30, 40, 30],
    })
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <TouchableOpacity onPress={this.startImageAnimation}>
          <Animated.Image
            source={ICON_HEART}
            style={{
              width: imageSize,
              height: imageSize, //应用到 Animated.Image 上
              tintColor: '#dc3132',
            }}
          />
        </TouchableOpacity>
      </View>
    )
  }
}
```

这些都是动画 Animated 的基础使用，另外动画部分还有关于 setNativeProps 和 LayoutAnimation 的部分另做总结。
