

### 处理1

```bash
./react/packages/react-reconciler/src/ReactFiberWorkLoop.js
Attempted import error: 'unstable_flushAllWithoutAsserting' is not exported from 'scheduler' (imported as 'Scheduler').
```

增加

```javascript
/* && Scheduler.unstable_flushAllWithoutAsserting === undefined */
```



### 处理2

```bash
TypeError: Cannot read property 'hasOwnProperty' of undefined
Module../react/packages/shared/ReactSharedInternals.js
/Users/ruyi/source-code/react-code-read/react/packages/shared/ReactSharedInternals.js:16
```

修改

```javascript
const ReactSharedInternals =
  React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED || {}
```

### 处理3

```bash
Error: Internal React error: invariant() is meant to be replaced at compile time. There is no runtime version.
invariant
/Users/ruyi/source-code/react-code-read/react/packages/shared/invariant.js:21
```

增加注释

```javascript
/* throw new Error(
    'Internal React error: invariant() is meant to be replaced at compile ' +
      'time. There is no runtime version.',
  ); */
```

### 处理4

```bash
Error: This module must be shimmed by a specific build.
./react/packages/scheduler/src/SchedulerHostConfig.js
src/SchedulerHostConfig.js:10
```

增加

```javascript
// throw new Error('This module must be shimmed by a specific build.');
export * from './forks/SchedulerHostConfig.default' 
export * from './forks/SchedulerHostConfig.mock' 
```

### 处理5

```bash

```









## 参考文档

- [搭建react本地源码测试环境](https://zhuanlan.zhihu.com/p/38118887)

- [React 源码全方位剖析](http://www.sosout.com/2018/08/12/react-source-analysis.html)

