参考文档: 
> http://www.sosout.com/2018/08/12/react-source-analysis.html
> React@16.8.6原理浅析（概念介绍）https://juejin.im/post/5e22b7b06fb9a02fdb5a769d#heading-49

## 1. React
> React: 它本质上是含有诸多属性的JavaScript对象，它核心内容只涉及如何定义组件，具体的组件渲染（即输出用户界面），需要引入额外的渲染模块，渲染组件方式由环境决定，定义组件，组件状态管理，生命周期方法管理，组件更新等应该跨平台一致处理，不受渲染环境影响，这部分内容统一由调和器（Reconciler）处理，不同渲染器都会使用该模块。调和器主要作用就是在组件状态变更时，调用组件树各组件的render方法，渲染，卸载组件

## 2. createElement ReactElement
createElement 返回的是ReactElement对象
```
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    $$typeof: REACT_ELEMENT_TYPE, //唯一类型
    type: type,  //  html: string, 自定义: 大写字母开头的对象
    key: key,    // 遍历时要求的唯一key
    ref: ref,      // 允许我们访问 DOM 节点或在 render 方法中创建的 React 元素
    props: props,  // 属性
    // Record the component responsible for creating this element.
    _owner: owner,
   };
    return element；
}
```
判断是不是ReactElement
```
export function isValidElement(object) {
  return (
    typeof object === 'object' &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
}
```
## 3.ReactClass
ReactClass就是我们平时写的Component组件(类或函数)，例如上面的Button类。ReactClass实例化后调用render方法可返回DOM Element。

## 4.ReactComponent
```
import {Component, PureComponent} from './ReactBaseClasses';
```
```
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}
```
ReactComponent是基于ReactElement创建的一个对象，这个对象保存了ReactElement的数据的同时注入了一些方法，这些方法可以用来实现我们熟知的那些React的特性。

## 5.ReactRoot ReactDOM
```
// 新API，替代render
  hydrate(element: React$Node, container: DOMContainer, callback: ?Function) {
    return legacyRenderSubtreeIntoContainer( null, element, container, true, callback);
  },

  render( element: React$Element<any>, container: DOMContainer, callback: ?Function) {
    return legacyRenderSubtreeIntoContainer( null, element, container, true, callback);
  },
```
```
// 渲染子组件树到父组件
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>, //父组件，首次为null
  children: ReactNodeList, // 虚拟DOM树
  container: DOMContainer, // DOM根元素
  forceHydrate: boolean,   // 服务器端渲染标识 这里为false
  callback: ?Function,     // 回调函数
) {
  if (__DEV__) {
    topLevelUpdateWarnings(container);
  }

  // TODO: Without `any` type, Flow says "Property cannot be accessed on any
  // member of intersection type." Whyyyyyy.
  let root: Root = (container._reactRootContainer: any);
  if (!root) { // 初次渲染时初始化
    // Initial mount
    // 创建一个 FiberRoot对象 并将它缓存到DOM容器的_reactRootContainer属性
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(root._internalRoot);
        originalCallback.call(instance);
      };
    }
    // Initial mount should not be batched.
    unbatchedUpdates(() => {
      if (parentComponent != null) {
        root.legacy_renderSubtreeIntoContainer(
          parentComponent,
          children,
          callback,
        );
      } else {
        root.render(children, callback);
      }
    });
  } else {
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(root._internalRoot);
        originalCallback.call(instance);
      };
    }
    // Update
    if (parentComponent != null) {
      root.legacy_renderSubtreeIntoContainer(
        parentComponent,
        children,
        callback,
      );
    } else {
      root.render(children, callback);
    }
  }
   // 返回根容器fiber树的根fiber实例
  return getPublicRootInstance(root._internalRoot);
}
```
`legacyRenderSubtreeIntoContainer` 主要执行了以下的几个操作：
- root： 根
- unbatchedUpdates 执行`root.legacy_renderSubtreeIntoContainer`或`root.render`

根节点生成
```
function legacyCreateRootFromDOMContainer(
  container: DOMContainer,
  forceHydrate: boolean,
): Root {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // ... ...
  // Legacy roots are not async by default.
  const isConcurrent = false;
  return new ReactRoot(container, isConcurrent, shouldHydrate);
}
```
```
function ReactRoot(
  container: DOMContainer,
  isConcurrent: boolean,
  hydrate: boolean,
) {
  const root = createContainer(container, isConcurrent, hydrate);
  this._internalRoot = root;
}
```
root节点是`createContainer`生成的


## 6.  更新容器内容
从`legacyRenderSubtreeIntoContainer`函数里可以看出，无论怎样判断，最终都会到·`root.legacy_renderSubtreeIntoContainer`和`root.render`两个方法

updateContainer的源码很简单，通过computeExpirationForFiber获得计算优先级，然后丢给updateContainerAtExpirationTime，这里updateContainerAtExpirationTime其实相当于什么都没做，通过getContextForSubtree（这里getContextForSubtree因为一开始parentComponent是不存在的，于是返回一个空对象。注意，这个空对象可以重复使用，不用每次返回一个新的空对象，这是一个很好的优化）获得上下文对象，然后分配给container.context或container.pendingContext，最后一起丢给scheduleRootUpdate。

## 7. 开始更新 `scheduleRootUpdate`
这里enqueueUpdate是一个链表，然后根据fiber的状态创建一个或两个列队对象，再接下来调用调度器API：scheduleWork(...)来调度fiber任务，现在我们看一下如何处理更新的。

## 8. 处理更新 `scheduleWork`

至此首次渲染的执行流程为：
ReactDOM.render（渲染入口） => legacyRenderSubtreeIntoContainer（把虚拟的dom树渲染到真实的dom容器中） => DOMRenderer.updateContainer（更新容器内容） => scheduleRootUpdate（开始更新） => scheduleWork（处理更新） => commitWork（提交更新）
## 9. React Fiber
Fiber对象，每一个组件实例对应有一个fiber实例，此fiber实例负责管理组件实例的更新，渲染任务及与其他fiber实例的通信，这个先进的调和器叫做纤维调和器（Fiber Reconciler），它提供的新功能主要有：
一：把可中断的任务拆分成小任务；
二：可重用各分阶段任务，对正在做的工作调整优先次序；
三：可以在父子组件任务间前进后退切换任务，以支持React执行过程中的布局刷新；
四：支持 render 方法返回多个元素；
五：对异常边界处理提供了更好的支持；

![img](/Users/ruyi/ali/Typora/Res/Img/02-01.png)


## 10. ExpirationTime
>  https://www.processon.com/view/link/5deb5debe4b0085967670f6a


## 11. Fiber Reconciler
当已经完成对整个 fiber tree 的最后一个节点更新后，react 会开始不断向上寻找父节点执行完成工作（completeUnitOfWork），如果父节点是普通 DOM 节点会比对属性的变化放在 update queue 上，同时将子节点产生的 effect 链挂载在自己的 effect 链上。

fiber reconciler 将一个更新分成两个阶段（Phase）：Reconciliation Phase 和 Commit Phase，第一个阶段也就是我们上面所说的协调的过程，它的作用是找出哪些 dom 是需要更新的，这个阶段是可以被打断的；第二个阶段就是将找出的那些 dom 渲染出来的过程，这个阶段是不能被打断的。

## 12. 

## 13.

## 14. HOOK

```
 const hook: Hook = {
    memoizedState: null,

    baseState: null,
    queue: null,
    baseUpdate: null,

    next: null,
  };

```

