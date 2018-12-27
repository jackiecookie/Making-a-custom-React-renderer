# Part-I

This is part one of the tutorial. This is the most important section of the tutorial where you'll learn about the Fiber, detailed description of its structure and fields, creating host configuration for your renderer and injecting the renderer into devtools.

这是教程的第一部分.也是学习Fiber的最重要的部分.这部分详细描述了Fiber的结构和字段,和如何为你自己的render创建host configuration,以及如何将你的render注入到devtools中。

In this section, we will create a React reconciler using [`react-reconciler`](https://github.com/facebook/react/tree/master/packages/react-reconciler) package. We are going to implement the renderer using Fiber. Earlier, React was using a **stack renderer** as it was implemented on the traditional JavaScript stack. On the other hand, Fiber is influenced by algebraic effects and functional ideas. It can be thought of as a JavaScript object that contains information about a component, its input, and its output.

在这个章节中，我们将会使用[`react-reconciler`](https://github.com/facebook/react/tree/master/packages/react-reconciler)创建React调度器(reconciler)。我们会使用Fiber实现自己的render。在早期,React使用传统的的JavaScript栈来实现了一个**栈renderer**.另外一方面,Fiber收到代数学和方法编程的影响,他可以借由一个包含着字段的JavaScript对象来描述一个组件,以及他的输入和输出。

Before we proceed further, I'll recommend you to read [this](https://github.com/acdlite/react-fiber-architecture) documentation on Fiber architecture by [Andrew Clark](https://twitter.com/acdlite?lang=en). This will make things
easier for you.

在我们更深入之前,我建议你可以阅读一下[这个](https://github.com/acdlite/react-fiber-architecture)由[Andrew Clark](https://twitter.com/acdlite?lang=en)写的关于Fiber结构的文档。这会让你学起来简单一些。

Let's get started!

让我们开始吧!

We will first install the dependencies.

首先我们先安装一下依赖

```
npm install react-reconciler@0.2.0 fbjs@0.8.16
```

Let's import the `Reconciler` from `react-reconciler` and also the other modules.

让我们从`react-reconciler`中引入`Reconciler`,和其他的一些模块.

```js
import Reconciler from 'react-reconciler';
import emptyObject from 'fbjs/lib/emptyObject';

import { createElement } from './utils/createElement';
```

Notice we have also imported `createElement` function. Don't worry, we will implement it afterwards.

你应该注意到了我们还引入了`createElement`方法.别担心,我们将会在稍后实现他.

We will create a reconciler instance using `Reconciler` which accepts a **host config** object. In this object we will define
some methods which can be thought of as lifecycle of a renderer (update, append children, remove children, commit). React will manage all the non-host components, stateless and composites.

我们将使用`Reconciler`创建一个reconciler实例它接受一个**host config**对象.在这个对象中我们可以定义一些方法，这些方法可以作为render的生命周期方法 (update, append children, remove children, commit).React将会管理所有的非客户端组件,无状态组件以及混合的组件.

```js
const WordRenderer = Reconciler({
  appendInitialChild(parentInstance, child) {
    if (parentInstance.appendChild) {
      parentInstance.appendChild(child);
    } else {
      parentInstance.document = child;
    }
  },

  createInstance(type, props, internalInstanceHandle) {
    return createElement(type, props, internalInstanceHandle);
  },

  createTextInstance(text, rootContainerInstance, internalInstanceHandle) {
    return text;
  },

  finalizeInitialChildren(wordElement, type, props) {
    return false;
  },

  getPublicInstance(inst) {
    return inst;
  },

  prepareForCommit() {
    // noop
  },

  prepareUpdate(wordElement, type, oldProps, newProps) {
    return true;
  },

  resetAfterCommit() {
    // noop
  },

  resetTextContent(wordElement) {
    // noop
  },

  getRootHostContext(rootInstance) {
    // You can use this 'rootInstance' to pass data from the roots.
  },

  getChildHostContext() {
    return emptyObject;
  },

  shouldSetTextContent(type, props) {
    return false;
  },

  now: () => performance.now(),

  useSyncScheduling: true,

  mutation: {
    appendChild(parentInstance, child) {
      if (parentInstance.appendChild) {
        parentInstance.appendChild(child);
      } else {
        parentInstance.document = child;
      }
    },

    appendChildToContainer(parentInstance, child) {
      if (parentInstance.appendChild) {
        parentInstance.appendChild(child);
      } else {
        parentInstance.document = child;
      }
    },
    
    removeChild(parentInstance, child) {
      parentInstance.removeChild(child);
    },

    removeChildFromContainer(parentInstance, child) {
      parentInstance.removeChild(child);
    },
  
    insertBefore(parentInstance, child, beforeChild) {
      // noob
    },
  
    commitUpdate(instance, updatePayload, type, oldProps, newProps) {
      // noop
    },
  
    commitMount(instance, updatePayload, type, oldProps, newProps) {
      // noop
    },
  
    commitTextUpdate(textInstance, oldText, newText) {
      textInstance.children = newText;
    },
  }
})
```

Let's break down our host config -

把我们的把host config一个个来看

**`createInstance`**

This method creates a component instance with `type`, `props` and `internalInstanceHandle`.

这个方法创建一个包含`type`, `props` 和 `internalInstanceHandle`组件实例。

Example - Let's say we render,

举例来说 - 我们来渲染下面的组件,

```js
<Text>Hello World</Text>
```  

`createInstance` will then return the information about the `type` of an element (' TEXT '), props ( { children: 'Hello World' } ), and the root instance (`WordDocument`). 

`createInstance`方法会返回包含了element的`类型(type)` (' TEXT '),props( { children: 'Hello World' } ),和root实例(`WordDocument`)的对象. 

**Fiber**

A fiber is work on a component that needs to be done or was done. Atmost, a component instance has two fibers, flushed fiber and work in progress fiber.

一个fiber工作在一个已经完成工作或者需要完成工作的组件上.大多数情况下,一个组件实例有两个fibers,已经完成的fiber和正在工作的fiber.

`internalInstanceHandle` contains information about the `tag`, `type`, `key`, `stateNode`, and the return fiber. This object (fiber) further contains information about -

`internalInstanceHandle`包含的信息包括`tag`, `type`, `key`, `stateNode`,和返回的fiber.这个fiber对象会包含的对象包括 - 

* **`tag`** - Type of fiber.

* **`tag`** - fiber的类型.

* **`key`** - Unique identifier of the child.

* **`key`** - 子元素的唯一标识符.

* **`type`** - function/class/module associated with this fiber.

* **`type`** - 和这个fiber相关的方法/类/模块.

* **`stateNode`** - The local state associated with this fiber.

* **`stateNode`** - 和这个fiber相关的state.

* **`return`** - The fiber to return to after finishing processing this one (parent fiber).
* **`child`** - `child`, `sibling` and `index` represents the **singly linked list data structure**.
* **`sibling`**
* **`index`**
* **`ref`** - The ref last used to attach this (parent) node.

* **`return`** - 在结束执行这个fiber(父fiber)之后返回的fiber.
* **`child`** - `child`, `sibling` 和 `index` 表示的 **单链表list数据结构**.
* **`sibling`**
* **`index`**
* **`ref`** - 引用指向最后一个引用的节点.

* **`pendingProps`** - This property is useful when a tag is overloaded.
* **`memoizedProps`** - The props used to create the output.
* **`updateQueue`** - A queue of state updates and callbacks.
* **`memoizedState`** - The state used to create the output.

* **`pendingProps`** - 当一个标签过载的时候使用的属性.
* **`memoizedProps`** - 创建输出值时候使用的属性.
* **`updateQueue`** - state更新和回调的队列.
* **`memoizedState`** - 创建输入值时使用的state.

* **`internalContextTag`** - Bit field data structure. React Fiber uses bit field data structures to hold a sequence of information about the fiber and it's subtree which is stored in an adjacent computer memory locations. A bit within this set is used to determine the state of an attribute. Collection of bit fields called flags represent the outcome of an operation or some intermediate state. React Fiber uses AsyncUpdates flag which indicates whether the subtree is async is or not.

* **`internalContextTag`** - Bit数据结构.React Fiber使用bit数据结构来保存fiber信息的顺序，他的subtree保存在相邻的计算机内存位置中.一个bit用来确定属性的state.bit字段的集合作为一个标识用来代表一些操作的结果或者一些中间状态(intermediate state).React Fiber使用AsyncUpdates标识符来标识subtree是否是异步的.

* **`effectTag`** - Effect
* **`nextEffect`** - Singly linked list fast path to the next fiber with side-effects.
* **`firstEffect`** - The first(firstEffect) and last(lastEffect) fiber with side-effect within the subtree. Reuse the work done in this fiber.

* **`effectTag`** - Effect
* **`nextEffect`** - 单向连接的list可以快速的连接到下一个受影响的fiber.
* **`firstEffect`** - subtree中第一个(firstEffect)和最后一个(lastEffect)受影响的fiber. 可以重复使用这个fiber中完成后返回的结果.

* **`expirationTime`** - This represents a time in the future by which the work should be completed.
* **`alternate`** - Pooled version of fiber which contains information about the fiber and is ready to be used rather than allocated on use. In computer graphics, this concept is abstracted in **double buffer** pattern. It uses more memory but we can clean up the pairs.

* **`expirationTime`** - 代表了任务在需要完成的过期时间。
* **`alternate`** - 保存了存有信息的fiber. 在计算机中,这个概念叫做**双缓存(double buffer)**模式.他使用了更多的内存但是我们可以轻松的清除他们.

* `pendingWorkPriority`
* `progressedPriority`
* `progressedChild`
* `progressedFirstDeletion`
* `progressedLastDeletion`

**`appendInitialChild`**

It appends the children. If children are wrapped inside a parent component (eg: `Document`), then we will add all the children to it else we 
will create a property called `document` on a parent node and append all the children to it. This will be helpful when we will parse the input component
and make a call to the render method of our component.

添加子元素.如果子元素在父组件中(比如: `Document`),那么我们会把子元素全部加到父组建中,否则的话我们会在父元素中创建一个属性叫做`document`然后把所有的子元素都增加进去.当我们转换传入的组件然后调用我们组件的渲染方法的时候他就会有用.


Example - 

举个例子 - 

```js
const data = document.render(); // returns the output
```

**`prepareUpdate`**

It computes the diff for an instance. Fiber can reuse this work even if it pauses or abort rendering a part of the tree.

计算实例的差异.Fiber可以重用这个结果哪怕tree的一部分已经暂停或者放弃渲染。

**`commitUpdate`**

Commit the update or apply the diff calculated to the host environment's node (WordDocument).

提交更新或者把计算好的差异应用到客户端的环境元素中(WordDocument).

**`commitMount`**

Renderer mounts the host component but may schedule some work to done after like auto-focus on forms. The host components are only mounted when there is no current/alternate fiber.

Renderer装载客户端组件或者可能需要在任务结束后安排一些操作,类似于form自动或者焦点.只有当current/alternate没有fiber的时候才会被加载.

**`hostContext`**

Host context is an internal object which our renderer may use based on the current location in the tree. In DOM, this object 
is required to make correct calls for example to create an element in html or in MathMl based on current context.

客户端上下文是一个内部的对象他标识了当前render所在树的位置.在DOM中,这个对象是为了能够做一个正确的调用而存在,基于上下文在html或者MathMl中创建一个元素.

**`getPublicInstance`**

This is an identity relation which means that it always returns the same value that was used as its argument. It was added for the TestRenderers.

这是一个实体当我们把他作为参数的时候他始终返回同一个值.它是为测试渲染的时候添加的.

**`useSyncScheduling`**
This property is used to down prioritize the children by checking whether the children are offscreen or not. In other words, if this property is true then the work in progress fiber has no expiration time

这个属性用于在检查子元素是否已经是否已经移除屏幕以后降低它的优先级.换句话来说,如果这个属性为true,那么正在工作的fiber没有过期时间。

**`resetTextContent`**
Reset the text content of the parent before doing any insertions (inserting host nodes into the parent). This is similar to double buffering technique in OpenGl where the buffer is cleared before writing new pixels to it and perform rasterization.

在进行插入操作之前(将客户端元素插入父元素中)重置父元素的text内容.这类似于OpenGl中的双缓存技术,在写入新的像素之前清除缓存然后执行格栅化(rasterization:画一些基本的元素).

**`commitTextUpdate`**
Similar to `commitUpdate` but it commits the update payload for the text nodes.

类似于`commitUpdate`但是他提交的是文本元素的更新内容.

**`removeChild and removeChildFromContainer`**
When we're inside a host component that was removed, it is now ok to remove the node from the tree. If the return fiber (parent) is container, then we remove the node from container using `removeChildFromContainer` else we simply use `removeChild`.

当我们在一个已经被移除的客户端组件中,我们就可以把他从树中移除.当返回的fiber(parent)是一个包含其它组件的组件,我们使用`removeChildFromContainer`.否则我们就可以简单使用`removeChild`.

**`insertBefore`**
It is a `commitPlacement` hook and is called when all the nodes are recursively inserted into parent. This is abstracted into a function named `getHostSibling` which continues to search the tree until it finds a sibling host node (React will change this methodology may be in next release because it's not an efficient way as it leads to exponential search complexity)

**`appendChildToContainer`**
If type of fiber is a `HostRoot` or `HostPortal` then the child is added to that container.

**`appendChild`**
Child is added to the parent.

**`shouldSetTextContent`**
If it returns false then schedule the text content to be reset.

**`getHostContext`**
It is used to mark the current host context (root instance) which is sent to update the payload and therefore update the queue of work in progress fiber (may indicate there is a change).

**`createTextInstance`**
Creates an instance of a text node.

### Note

* You should **NOT** rely on Fiber data structure itself. Consider its fields private.
* Treat 'internalInstanceHandle' as an opaque object itself.
* Use host context methods for getting data from roots.

> The above points were added to the tutorial after a discussion with [Dan Abramov](https://twitter.com/dan_abramov) regarding the host config methods and Fiber properties.

## Injecting third party renderers into devtools

You can also inject your renderer into react-devtools to debug the host components of your environment. Earlier, it wasn't possible for third party renderers but now using the return value of `reconciler` instance, it is possible to inject the renderer into react-devtools.

> Note - This wasn't supported in `react-reconciler` version 0.2.0. So you'll need to update it to the current beta version 0.3.0-beta.1

**Usage**

Install standalone app for react-devtools

```
yarn add --dev react-devtools
```

Run

```
yarn react-devtools
```

or if you use npm,

```
npm install -g react-devtools
```

then run it with

```
react-devtools
```

```js
const Reconciler = require('react-reconciler')

let hostConfig = {
  // See the above notes for adding methods here
}

const CustomRenderer = Reconciler(hostConfig)

module.exports = CustomRenderer
```

Then in your `render` method,

```js
const CustomRenderer = require('./reconciler')

function render(element, target, callback) {
  ... // Here, schedule a top level update using CustomRenderer.updateContainer(), see Part-IV for more details.
  CustomRenderer.injectIntoDevTools({
    bundleType: 1, // 0 for PROD, 1 for DEV
    version: '0.1.0', // version for your renderer
    rendererPackageName: 'custom-renderer', // package name
    findHostInstanceByFiber: CustomRenderer.findHostInstance // host instance (root)
  }))
}
```

We're done with the Part One of our tutorial. I know some concepts are difficult to grok solely by looking at code. Initially it feels agitating but keep trying it and it will eventually make sense. When I first started learning about the Fiber architecture, I couldn't understand anything at all. I was frustated and dismayed but I used `console.log()` in every section of the above code and tried to understand its implementation and then there was this "Aha Aha" moment and it finally helped me to build [redocx](https://github.com/nitin42/redocx). Its a little perplexing to understand but you will get it eventually.

If you still have any doubts, DMs are open. I'm at [@NTulswani](https://twitter.com/NTulswani) on Twitter.

[More practical examples for the renderer](https://github.com/facebook/react/tree/master/packages/react-reconciler#practical-examples)

[Continue to Part-II](./part-two.md)
