# React核心原理 - 完整教程

## 目录
1. [JSX原理](#jsx原理)
2. [虚拟DOM](#虚拟dom)
3. [Fiber架构](#fiber架构)
4. [Diff算法](#diff算法)
5. [Hooks原理](#hooks原理)
6. [调度机制](#调度机制)
7. [事件系统](#事件系统)

## JSX原理

### JSX转换
```jsx
// JSX代码
const element = (
  <div className="container">
    <h1>Hello, {name}</h1>
    <p>Welcome to React</p>
  </div>
);

// 转换后（React 17之前）
const element = React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Hello, ', name),
  React.createElement('p', null, 'Welcome to React')
);

// 转换后（React 17+）
import { jsx as _jsx } from 'react/jsx-runtime';

const element = _jsx('div', {
  className: 'container',
  children: [
    _jsx('h1', { children: ['Hello, ', name] }),
    _jsx('p', { children: 'Welcome to React' })
  ]
});
```

### createElement实现
```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === 'object'
          ? child
          : createTextElement(child)
      )
    }
  };
}

function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: {
      nodeValue: text,
      children: []
    }
  };
}

// 使用
const element = createElement(
  'div',
  { className: 'container' },
  createElement('h1', null, 'Hello'),
  createElement('p', null, 'World')
);
```

### JSX编译配置
```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-react', {
      runtime: 'automatic', // 使用新的JSX转换
      development: process.env.NODE_ENV === 'development',
      importSource: 'react' // 自定义JSX运行时
    }]
  ]
};

// 自定义JSX运行时
// jsx-runtime.js
export function jsx(type, props) {
  return {
    $$typeof: Symbol.for('react.element'),
    type,
    props
  };
}
```

## 虚拟DOM

### 虚拟DOM结构
```javascript
const vdom = {
  type: 'div',
  props: {
    className: 'container',
    children: [
      {
        type: 'h1',
        props: {
          children: 'Hello'
        }
      },
      {
        type: 'p',
        props: {
          children: 'World'
        }
      }
    ]
  }
};
```

### 渲染虚拟DOM
```javascript
function render(vdom, container) {
  // 创建真实DOM
  const dom = vdom.type === 'TEXT_ELEMENT'
    ? document.createTextNode('')
    : document.createElement(vdom.type);
  
  // 设置属性
  Object.keys(vdom.props)
    .filter(key => key !== 'children')
    .forEach(name => {
      dom[name] = vdom.props[name];
    });
  
  // 递归渲染子元素
  vdom.props.children.forEach(child => {
    render(child, dom);
  });
  
  // 添加到容器
  container.appendChild(dom);
}
```

### 虚拟DOM优势
```javascript
// 1. 批量更新
const updates = [];
updates.push({ type: 'UPDATE', node: node1 });
updates.push({ type: 'UPDATE', node: node2 });
// 一次性应用所有更新

// 2. 跨平台
function renderToString(vdom) {
  // 服务端渲染
}

function renderToNative(vdom) {
  // React Native
}

// 3. 性能优化
function shouldUpdate(oldVdom, newVdom) {
  // 对比虚拟DOM，决定是否更新
  return oldVdom !== newVdom;
}
```

## Fiber架构

### Fiber节点结构
```javascript
const fiber = {
  // 节点类型
  type: 'div',
  
  // 节点属性
  props: {
    className: 'container',
    children: []
  },
  
  // DOM引用
  stateNode: domElement,
  
  // 父节点
  return: parentFiber,
  
  // 第一个子节点
  child: childFiber,
  
  // 下一个兄弟节点
  sibling: siblingFiber,
  
  // 旧的Fiber节点
  alternate: oldFiber,
  
  // 副作用标记
  effectTag: 'PLACEMENT', // PLACEMENT, UPDATE, DELETION
  
  // 副作用链表
  nextEffect: null,
  
  // Hooks链表
  memoizedState: null,
  
  // 优先级
  lanes: 0
};
```

### Fiber工作流程
```javascript
// 1. 调度阶段（可中断）
function workLoop(deadline) {
  let shouldYield = false;
  
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }
  
  if (!nextUnitOfWork && wipRoot) {
    commitRoot();
  }
  
  requestIdleCallback(workLoop);
}

// 2. 执行工作单元
function performUnitOfWork(fiber) {
  // 处理当前fiber
  if (!fiber.stateNode) {
    fiber.stateNode = createDom(fiber);
  }
  
  // 创建子fiber
  const elements = fiber.props.children;
  reconcileChildren(fiber, elements);
  
  // 返回下一个工作单元
  if (fiber.child) {
    return fiber.child;
  }
  
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.return;
  }
}

// 3. 提交阶段（不可中断）
function commitRoot() {
  commitWork(wipRoot.child);
  currentRoot = wipRoot;
  wipRoot = null;
}

function commitWork(fiber) {
  if (!fiber) return;
  
  const domParent = fiber.return.stateNode;
  
  if (fiber.effectTag === 'PLACEMENT' && fiber.stateNode) {
    domParent.appendChild(fiber.stateNode);
  } else if (fiber.effectTag === 'UPDATE' && fiber.stateNode) {
    updateDom(fiber.stateNode, fiber.alternate.props, fiber.props);
  } else if (fiber.effectTag === 'DELETION') {
    domParent.removeChild(fiber.stateNode);
  }
  
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

### 双缓冲技术
```javascript
// current树：当前显示的Fiber树
let currentRoot = null;

// workInProgress树：正在构建的Fiber树
let wipRoot = null;

function render(element, container) {
  wipRoot = {
    stateNode: container,
    props: {
      children: [element]
    },
    alternate: currentRoot
  };
  
  nextUnitOfWork = wipRoot;
}

// 提交时切换
function commitRoot() {
  commitWork(wipRoot.child);
  currentRoot = wipRoot; // 切换
  wipRoot = null;
}
```

## Diff算法

### 三大策略
```javascript
// 1. Tree Diff：只对比同层节点
function diffTree(oldTree, newTree) {
  // 不跨层级对比
  diffChildren(oldTree.children, newTree.children);
}

// 2. Component Diff：同类型组件对比
function diffComponent(oldComponent, newComponent) {
  if (oldComponent.type === newComponent.type) {
    // 更新props
    updateComponent(oldComponent, newComponent.props);
  } else {
    // 替换组件
    replaceComponent(oldComponent, newComponent);
  }
}

// 3. Element Diff：使用key优化列表
function diffElements(oldElements, newElements) {
  const keyedOld = new Map();
  oldElements.forEach((el, i) => {
    keyedOld.set(el.key || i, el);
  });
  
  // 使用key快速定位
  newElements.forEach((newEl, i) => {
    const oldEl = keyedOld.get(newEl.key || i);
    if (oldEl) {
      updateElement(oldEl, newEl);
    } else {
      insertElement(newEl, i);
    }
  });
}
```

### 单节点Diff
```javascript
function reconcileSingleElement(returnFiber, currentFiber, element) {
  const key = element.key;
  let child = currentFiber;
  
  while (child !== null) {
    // key相同
    if (child.key === key) {
      // type相同，复用
      if (child.type === element.type) {
        deleteRemainingChildren(returnFiber, child.sibling);
        const existing = useFiber(child, element.props);
        existing.return = returnFiber;
        return existing;
      } else {
        // type不同，删除旧的
        deleteRemainingChildren(returnFiber, child);
        break;
      }
    } else {
      // key不同，删除
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }
  
  // 创建新fiber
  const created = createFiberFromElement(element);
  created.return = returnFiber;
  return created;
}
```

### 多节点Diff
```javascript
function reconcileChildrenArray(returnFiber, currentFirstChild, newChildren) {
  let resultingFirstChild = null;
  let previousNewFiber = null;
  let oldFiber = currentFirstChild;
  let lastPlacedIndex = 0;
  let newIdx = 0;
  let nextOldFiber = null;
  
  // 第一轮遍历：处理更新的节点
  for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
    if (oldFiber.index > newIdx) {
      nextOldFiber = oldFiber;
      oldFiber = null;
    } else {
      nextOldFiber = oldFiber.sibling;
    }
    
    const newFiber = updateSlot(returnFiber, oldFiber, newChildren[newIdx]);
    
    if (newFiber === null) {
      if (oldFiber === null) {
        oldFiber = nextOldFiber;
      }
      break;
    }
    
    if (shouldTrackSideEffects) {
      if (oldFiber && newFiber.alternate === null) {
        deleteChild(returnFiber, oldFiber);
      }
    }
    
    lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
    
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    
    previousNewFiber = newFiber;
    oldFiber = nextOldFiber;
  }
  
  // 新节点遍历完，删除剩余旧节点
  if (newIdx === newChildren.length) {
    deleteRemainingChildren(returnFiber, oldFiber);
    return resultingFirstChild;
  }
  
  // 旧节点遍历完，插入剩余新节点
  if (oldFiber === null) {
    for (; newIdx < newChildren.length; newIdx++) {
      const newFiber = createChild(returnFiber, newChildren[newIdx]);
      if (newFiber === null) continue;
      
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
    return resultingFirstChild;
  }
  
  // 第二轮遍历：处理移动的节点
  const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
  
  for (; newIdx < newChildren.length; newIdx++) {
    const newFiber = updateFromMap(
      existingChildren,
      returnFiber,
      newIdx,
      newChildren[newIdx]
    );
    
    if (newFiber !== null) {
      if (shouldTrackSideEffects) {
        if (newFiber.alternate !== null) {
          existingChildren.delete(
            newFiber.key === null ? newIdx : newFiber.key
          );
        }
      }
      
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
  }
  
  // 删除未匹配的旧节点
  if (shouldTrackSideEffects) {
    existingChildren.forEach(child => deleteChild(returnFiber, child));
  }
  
  return resultingFirstChild;
}
```

## Hooks原理

### useState实现
```javascript
let workInProgressHook = null;
let currentHook = null;

function useState(initialState) {
  // 获取当前hook
  const hook = updateWorkInProgressHook();
  
  if (!hook.memoizedState) {
    // 初始化
    hook.memoizedState = [
      typeof initialState === 'function' ? initialState() : initialState,
      null
    ];
    
    // 创建更新队列
    const queue = {
      pending: null,
      dispatch: null
    };
    
    hook.queue = queue;
    
    // 创建dispatch函数
    const dispatch = (action) => {
      const update = {
        action,
        next: null
      };
      
      // 添加到更新队列
      if (queue.pending === null) {
        update.next = update;
      } else {
        update.next = queue.pending.next;
        queue.pending.next = update;
      }
      queue.pending = update;
      
      // 触发更新
      scheduleUpdateOnFiber(currentFiber);
    };
    
    queue.dispatch = dispatch;
    hook.memoizedState[1] = dispatch;
  } else {
    // 更新
    const queue = hook.queue;
    let update = queue.pending;
    
    if (update !== null) {
      const first = update.next;
      let newState = hook.memoizedState[0];
      
      do {
        const action = update.action;
        newState = typeof action === 'function'
          ? action(newState)
          : action;
        update = update.next;
      } while (update !== first);
      
      hook.memoizedState[0] = newState;
      queue.pending = null;
    }
  }
  
  return hook.memoizedState;
}
```

### useEffect实现
```javascript
function useEffect(create, deps) {
  const hook = updateWorkInProgressHook();
  
  const nextDeps = deps === undefined ? null : deps;
  let destroy = undefined;
  
  if (currentHook !== null) {
    const prevEffect = currentHook.memoizedState;
    destroy = prevEffect.destroy;
    
    if (nextDeps !== null) {
      const prevDeps = prevEffect.deps;
      // 依赖未变化，跳过
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        pushEffect(HookPassive, create, destroy, nextDeps);
        return;
      }
    }
  }
  
  // 标记副作用
  currentFiber.effectTag |= PassiveEffect;
  
  hook.memoizedState = pushEffect(
    HookHasEffect | HookPassive,
    create,
    destroy,
    nextDeps
  );
}

function pushEffect(tag, create, destroy, deps) {
  const effect = {
    tag,
    create,
    destroy,
    deps,
    next: null
  };
  
  let componentUpdateQueue = currentFiber.updateQueue;
  
  if (componentUpdateQueue === null) {
    componentUpdateQueue = createFunctionComponentUpdateQueue();
    currentFiber.updateQueue = componentUpdateQueue;
    componentUpdateQueue.lastEffect = effect.next = effect;
  } else {
    const lastEffect = componentUpdateQueue.lastEffect;
    if (lastEffect === null) {
      componentUpdateQueue.lastEffect = effect.next = effect;
    } else {
      const firstEffect = lastEffect.next;
      lastEffect.next = effect;
      effect.next = firstEffect;
      componentUpdateQueue.lastEffect = effect;
    }
  }
  
  return effect;
}

// 执行副作用
function commitHookEffectList(unmountTag, mountTag, finishedWork) {
  const updateQueue = finishedWork.updateQueue;
  let lastEffect = updateQueue !== null ? updateQueue.lastEffect : null;
  
  if (lastEffect !== null) {
    const firstEffect = lastEffect.next;
    let effect = firstEffect;
    
    do {
      if ((effect.tag & unmountTag) !== NoHookEffect) {
        // 执行清理函数
        const destroy = effect.destroy;
        effect.destroy = undefined;
        if (destroy !== undefined) {
          destroy();
        }
      }
      
      if ((effect.tag & mountTag) !== NoHookEffect) {
        // 执行副作用
        const create = effect.create;
        effect.destroy = create();
      }
      
      effect = effect.next;
    } while (effect !== firstEffect);
  }
}
```

### useMemo和useCallback实现
```javascript
function useMemo(nextCreate, deps) {
  const hook = updateWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const prevState = hook.memoizedState;
  
  if (prevState !== null) {
    if (nextDeps !== null) {
      const prevDeps = prevState[1];
      if (areHookInputsEqual(nextDeps, prevDeps)) {
        return prevState[0];
      }
    }
  }
  
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}

function useCallback(callback, deps) {
  return useMemo(() => callback, deps);
}
```

## 调度机制

### 优先级系统
```javascript
const ImmediatePriority = 1;      // 立即执行
const UserBlockingPriority = 2;   // 用户交互
const NormalPriority = 3;         // 正常优先级
const LowPriority = 4;            // 低优先级
const IdlePriority = 5;           // 空闲时执行

function scheduleCallback(priorityLevel, callback) {
  const currentTime = getCurrentTime();
  const timeout = timeoutForPriorityLevel(priorityLevel);
  const expirationTime = currentTime + timeout;
  
  const newTask = {
    callback,
    priorityLevel,
    expirationTime,
    sortIndex: expirationTime
  };
  
  push(taskQueue, newTask);
  requestHostCallback(flushWork);
  
  return newTask;
}
```

### 时间切片
```javascript
const frameInterval = 5; // 5ms一帧

function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  let currentTask = peek(taskQueue);
  
  while (currentTask !== null) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // 时间片用完，让出控制权
      break;
    }
    
    const callback = currentTask.callback;
    if (typeof callback === 'function') {
      currentTask.callback = null;
      const didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
      
      const continuationCallback = callback(didUserCallbackTimeout);
      currentTime = getCurrentTime();
      
      if (typeof continuationCallback === 'function') {
        currentTask.callback = continuationCallback;
      } else {
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }
    } else {
      pop(taskQueue);
    }
    
    currentTask = peek(taskQueue);
  }
  
  return currentTask !== null;
}

function shouldYieldToHost() {
  const currentTime = getCurrentTime();
  return currentTime >= deadline;
}
```

## 事件系统

### 事件委托
```javascript
// React在根节点统一监听事件
function listenToAllSupportedEvents(rootContainerElement) {
  allNativeEvents.forEach(domEventName => {
    listenToNativeEvent(
      domEventName,
      false,
      rootContainerElement
    );
  });
}

function listenToNativeEvent(domEventName, isCapturePhaseListener, target) {
  let eventSystemFlags = 0;
  if (isCapturePhaseListener) {
    eventSystemFlags |= IS_CAPTURE_PHASE;
  }
  
  addTrappedEventListener(
    target,
    domEventName,
    eventSystemFlags,
    isCapturePhaseListener
  );
}
```

### 合成事件
```javascript
function createSyntheticEvent(Interface) {
  function SyntheticBaseEvent(
    reactName,
    reactEventType,
    targetInst,
    nativeEvent,
    nativeEventTarget
  ) {
    this._reactName = reactName;
    this._targetInst = targetInst;
    this.type = reactEventType;
    this.nativeEvent = nativeEvent;
    this.target = nativeEventTarget;
    this.currentTarget = null;
    
    for (const propName in Interface) {
      if (!Interface.hasOwnProperty(propName)) {
        continue;
      }
      const normalize = Interface[propName];
      if (normalize) {
        this[propName] = normalize(nativeEvent);
      } else {
        this[propName] = nativeEvent[propName];
      }
    }
    
    const defaultPrevented =
      nativeEvent.defaultPrevented != null
        ? nativeEvent.defaultPrevented
        : nativeEvent.returnValue === false;
    
    if (defaultPrevented) {
      this.isDefaultPrevented = functionThatReturnsTrue;
    } else {
      this.isDefaultPrevented = functionThatReturnsFalse;
    }
    this.isPropagationStopped = functionThatReturnsFalse;
    
    return this;
  }
  
  Object.assign(SyntheticBaseEvent.prototype, {
    preventDefault() {
      this.defaultPrevented = true;
      const event = this.nativeEvent;
      if (!event) return;
      
      if (event.preventDefault) {
        event.preventDefault();
      } else {
        event.returnValue = false;
      }
      this.isDefaultPrevented = functionThatReturnsTrue;
    },
    
    stopPropagation() {
      const event = this.nativeEvent;
      if (!event) return;
      
      if (event.stopPropagation) {
        event.stopPropagation();
      } else {
        event.cancelBubble = true;
      }
      this.isPropagationStopped = functionThatReturnsTrue;
    }
  });
  
  return SyntheticBaseEvent;
}
```

### 事件优先级
```javascript
function getEventPriority(domEventName) {
  switch (domEventName) {
    case 'click':
    case 'keydown':
    case 'keyup':
      return DiscreteEventPriority;
    
    case 'drag':
    case 'scroll':
    case 'mousemove':
      return ContinuousEventPriority;
    
    default:
      return DefaultEventPriority;
  }
}
```

## 最佳实践

1. **理解Fiber架构的可中断特性**
2. **合理使用key优化列表渲染**
3. **避免在render中创建新对象**
4. **使用React.memo减少不必要渲染**
5. **理解Hooks的闭包陷阱**
6. **合理使用useMemo和useCallback**
7. **避免在useEffect中创建无限循环**

---

**@author erik.zhou**
