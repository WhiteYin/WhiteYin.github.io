---
title: 《深入浅出React和Redux》第三章笔记
comments: true
toc: true
date: 2018-03-29 14:58:54
tags:
- react
categories:
- react
description: 记录flux和redux中各模块的作用
---
# 前言
这一章讲了flux和redux，以及react-redux库的作用。可以看书籍的配套代码，理解相关知识。
# Flux
## 各部分作用
1. Action
    1. ActionType定义动作种类
    2. Actions中导出函数，接收action承载的数据作为参数，调用Dispacher.dispach方法广播动作
2. Dispacher
    从Flux中导入Dispacher对象，导出一个实例
3. Store
    1. 存储对应组件的数据
    2. 复制EventEmitter的原型对象，定义getter、on、emit和removeListener
    3. 定义监听的事件名称，比如'change'
    4. 调用dispacher.register方法，获取token。该方法接收一个action，根据action.type处理不同逻辑并emit事件
4. View
    1. 从Store中取数据
    2. 装载时设置监听事件回调函数
    3. 卸载时取消事件监听函数
    4. 按钮等动作触发时调用Action中定义的方法

# Redux
## 各部分作用
1. Action
    1. ActionType定义动作种类
    2. Actions中导出函数，接收action承载的数据作为参数，返回action对象
2. Reducer
    导出一个函数，接收state和action作为参数。根据action.type返回不同的state。如果接收到未定义的action，原样返回state
3. Store
    1. 存储对应组件的数据
    2. 从redux中导入createStore和combineReducers来根据reducer函数和initialValue生成一个store对象，并导出
4. View
    1. 从store.getState()方法中获取state
    2. 装载时使用store.subscribe()设置监听事件回调函数
    3. 卸载时使用store.unscribe()取消事件监听函数
    4. 按钮等动作触发时调用store.dispach()，传入Action中导出的函数作为参数

# 容器组件和展示组件
容器组件的作用是与store交互，展示组件的作用是渲染。将两个任务分给两种组件能够保证组件功能单一。

原来的单个组件中，生命周期、props和store的交互都不做修改，但是返回值是一个展示组件。
原来用来渲染的state都当作展示组件的props。因此渲染组件需要保证propTypes一致。又因为渲染组件不保存状态，所以只需要重写render函数，返回相应的渲染元素即可。
最终导出的为容器组件。渲染组件对外不可见。

# context
使用context能够保证每一个组件都避免导入store，而是通过父级组件传递下来。但是为了防止子组件层间太深，需要使用context的功能。

1. 编写Provider组件，store属性，值为导入的store对象；
2. 重写getChildContext方法，返回值为context对象，其中包含store属性；
3. store必须为object类型
    ```
     Provider.propTypes = {
      store: PropTypes.object.isRequired
    }
    //与getChildContext对应
    Provider.childContextTypes = {
      store: PropTypes.object
    };
    ```
4. 组件中，构造函数和super方法的第二个参数都为context，这样组件可以通过this.context.store来调用store；
5. 同时组件还需要限制context的类型：
    ```
    container.contextTypes = {
        store:PropTypes.object
    }
    ```

# React-Redux
使用react-redux库可以同时解决上面两种问题。
1. 可以从react-redux中导入Provider组件
2. 组件中，需要定义两个函数：mapStateToProps和mapDispatchToProps
3. 渲染组件部分不需要修改，容器组件通过从react-redux中导入的connect函数生成
4. 这个函数接收2中的两个函数作为参数，返回一个函数，再以渲染组件为参数返回容器组件
5. mapStateToProps代替的是原容器组件从store中获取state的函数，内容没有变化，只是函数名改变。参数有两个，第一个为store.getState()得到的状态，第二个是组件的props；
6. mapDispatchToProps接收两个参数，第一个为store.dispach方法，第二个为组件的props。返回值是一个对象，属性为定义的函数名，属性值为一个函数，能够分发动作。

比较之后，区别主要在于：
1. 不需要手写Provider组件，且使用方法不同；
2. 不需要手写容器组件，有connect函数生成；
3. 不需要在容器组件的componentDidMount和componentWillUnmount生命周期中编写subscribe和unsubcribe方法。应该是已经封装好了；
4. 需要编写两个函数用来将原容器组件的state和dispach映射到渲染组件的props上。
