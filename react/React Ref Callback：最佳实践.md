# React Ref Callback：最佳实践

“Ref” 在 React 中有两个相关的含义，令人困惑。因此，在我们开始之前，让我们先搞清楚这一点。

1. `useRef` 钩子返回的“ref对象”是一个普通的JavaScript对象，它只有一个名为 `current` 的属性，您可以读取或将其设置为任何值。
2. 宿主元素——代表 DOM 元素的 JSX ——具有特殊的“ref属性”，可用于访问其对应的 DOM 元素。

这两个通常一起使用，“ref 对象”可以传递给“ref 属性”，React 将把对 DOM 元素的引用设置为其当前属性。

## ref Callback

除了 ref 对象之外，ref 属性还接受一个函数；即 ref 回调。它的唯一参数是对渲染后的 DOM 元素的引用。

像 effect 函数一样，React 在组件的生命周期中的特定时刻调用它。ref 回调在附加的 DOM 元素创建后立即被调用，并在元素被移除时再次使用 null 调用。

如果一个 ref 回调函数被定义为内联函数，React 将在每次渲染时调用它两次，首先是 null，然后是 DOM 元素。有关此的更多信息，请参见下面的 ref 回调注意事项。

> 被调用两次的内联引用回调可能看起来令人惊讶，我认为从 React 的角度来看，这种行为是有意义的，它确保了一致性。每个渲染都会创建一个函数的新实例。它可能是一个完全不同的函数，或者该函数可能关闭了在此期间可能已经改变的 `props` 或 `state`。因此，React需要清除旧的引用回调（用 `null` 调用它），然后设置新的（用 DOM 元素调用它）。这允许您有条件地附加引用，甚至在 React 元素之间交换它们。这可能会导致它被不必要地调用。大多数时候，这没什么大不了的，在某些情况下，这可能不是你想要的。您可以通过在 `useCallback` 中包装 ref 回调或将函数移出组件来避免此行为。

由于 ref 回调在渲染后被调用，因此可以安全地在其中执行副作用。

## 使用场景

在 React Docs 中关于 ref callback 的内容较少。也许是他们故意不去讨论它，因为它的使用场景非常少，访问 DOM 元素的场景并不多见。

ref callback 绝对是 React 的一个小众功能，你不会每天都需要它。尽管如此，它确实有一些合法的用例，否则就不会出现在 React 中！因此让我们看一些用例。

仅当您需要底层的 DOM 元素时，才需要使用 ref callback。那么 ref callback 何时有用？

> 当您想在 React 将 ref 附加或分离到 DOM 元素时执行操作时，请使用 ref callback。

据我所知，这归结为以下四种情况。

1. 当元素挂载或更新时，调用 DOM 元素实例方法执行副作用。
2. 当 DOM 元素的某个属性发生变化时，“通知” 并重新渲染。
3. 将 DOM 元素设置到 state 中，以便在渲染期间访问它。
4. 共享 DOM 元素，使用 DOM 元素执行多项操作。

现在，让我们具体来看看每一个场景。

### 1. 当 DOM 元素挂载时滚动到该元素

您可以在挂载时使用 ref callback 调用 DOM 元素实例方法，以执行诸如滚动或聚焦等 DOM 副作用。例如，自动滚动到列表中的最后一项：

```jsx
// On first render and on unmount there 
// is no DOM element so `el` will be `null` 
const scrollTo = (el) => {
  if (el) {
    el.scrollIntoView({ behavior: "smooth" });
  }
};

function List({ data }) {
  return (
    <ul>
      {data.map((d, i) => {
        const isLast = i === data.length - 1;
        return (
          <li
            key={d.name}
            // ref callback to scroll to the last list element
            ref={isLast ? scrollTo : undefined}
          >
            {d.name}
          </li>
        );
      })}
    </ul>
  );
}
```

记住，管理 DOM 是 React 的工作，避免执行 DOM 上的可变方法， 比如（`insert`,`remove`, `set`,`replace` 等），对于 `focus` 和 `scroll` 等非破坏性的操作则允许我们开发实现。

请注意，浏览器不允许在没有用户交互的情况下调用某些 DOM 元素方法，例如 [requestFullscreen](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/requestFullscreen)。当在 ref callback 中调用此类受保护的方法时，它们将不起作用。


### 2. 当 DOM 元素变化时的重新渲染

当您希望 React 知道某个 DOM 元素属性时，可以使用 ref callback。在 ref callback 中读取像滚动位置这样的 DOM 元素属性或调用提供有关该元素信息的方法（例如 `getBoundingClientRect()`），并将该信息设置到 state 中。

#### 测量 DOM 元素

这是一段直接来自（旧）React 文档的片段：[如何测量 DOM 节点](https://reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)。这是 ref callback 的一个很好的例子，所以将其复制到这里。

```jsx
 const [size, setSize] = useState();

 const measureRef = useCallback((node) => {
   setSize(node.getBoundingClientRect());
 }, []);

 return <div ref={measureRef}>{children}</div>;
```

在这个案例中，没有选择使用 `useRef`，因为当 ref 是一个对象时，它并不会把当前 ref 值的变化情况通知到我们。使用 callback ref 可以确保即便被测量的节点在子组件延迟显示 (比如为了响应一次点击)，我们依然能够在父组件接收到相关的信息，以便更新测量结果。注意到我们传递了 [] 作为 `useCallback` 的依赖列表。这确保了 ref callback 不会在再次渲染时改变，因此 React 不会在非必要的时候调用它。

### 3. 在 render 中访问 DOM 元素

如果在 ref 回调中将 DOM 元素设置到 `state`，它将触发新的渲染，因为这正是设置 `state` 的作用。但是它不会陷入无限渲染循环，因为 `setState` 是一个稳定的函数，因此 ref callback 仅在挂载和卸载时调用。

在这种情况下，为什么我们不使用 `useRef` ？答案是，因为不允许在渲染过程中访问 ref 对象。对于渲染中的 DOM 元素，必须通过 state 来访问。

#### React Portal

React portal 主要用于解决组件树和 DOM 树的结构之间不一致的问题。portal 将 DOM 树上不同位置上的组件连接到一起，最为常使用的场景就是将 Modal 弹窗覆盖整个视窗。

```jsx
// Assume an empty div with id 'modal' is in your HTML
 const modalEl = document.getElementById("modal");

 function Modal({ children, ...props }) {
   return ReactDOM.createPortal(
     <ModalBase {...props}>
       {children}
     </ModalBase>,
     modalEl
   );
 }
```

可以使用 `document.getElementById()` 来获取 DOM 元素，前提是你能保证它是存在的。或许你不想通过 HTML 来控制 Modal，而是希望能 portal 到一个 React 创建的 DOM 元素上。

这就需要在进行 render 时访问到相应的 DOM 元素，使用 ref callback 可以实现这个功能。

```jsx
 function Parent() {
   const [modalElement, setModalElement] = useState(null);

   return (
     <div>
       <div id="modal-location" ref={setModalElement} />
       {/* Imagine that the modal container and the
           Modal itself are farther apart in the component tree */}
       <Modal modalElement={modalElement}>Warning</Modal>
     </div>
   )
 }

 function Modal({ children, modalElement, ...props }) {
   return modalElement
     ? ReactDOM.createPortal(
         <ModalBase {...props}>{children}</ModalBase>,
         modalElement
       )
     : null;
```

在最开始，modalElement 的值是 `null`，所以需要在创建 portal 之前做一下判断。

### 4. 共享 DOM Ref

经常会出现不止一个消费者需要访问 DOM 元素。假设你想测量一个 `<div>` 的宽度，并将其交给另一个 React 之外的库来处理 DOM 内容。对于 React 来说，这样的元素就是一个黑盒。React 既不知道它的内部有什么，也不关心它是什么。这个元素完全交给另外一个库来管理。

一个典型的例子就是使用 D3 或 `@observable/plot` 创建的响应式图标。在下面的例子中，我们会使用 `@observable/plot` 创建一个 plot，并且使用 react-use-measure 来计算元素的宽度。使用 ref callback 将 DOM 元素传递给它们俩：

```jsx
 import useMeasure from "react-use-measure";
 import * as Plot from "@observablehq/plot";

 export function BoxPlot({ data }) {
   const [measureRef, { width, height }] = useMeasure({ debounce: 5 });

   const plotRef = useRef<HTMLDivElement | null>(null);

   useEffect(() => {
     const boxPlot = Plot.plot({
       width: Math.max(150, width),
       marks: [
         Plot.boxX(data),
       ],
     });

     plotRef.current.append(boxPlot);

     return () => boxPlot.remove();
   }, [data, width]);

   const initBoxPlot = useCallback((el: HTMLDivElement | null) => {
     plotRef.current = el;
     measureRef(el);
   }, []);

   return <div ref={initBoxPlot} />;
 }
```

## 总结
- ref callback 是一个传递给元素的 ref 属性的函数。React 会在组件挂载时调用它，这时的参数是 DOM 元素；当组件卸载的时候也会调用它，这时的参数是 `null`
- 当你设置不同的 ref callback 时，React 也会调用 ref callback
- 切换绑定和取消绑定 ref 到一个 DOM 元素，ref callback 可以让我们实现特定的操作
- ref callback 可以用来做以下事情
- 操作 DOM，比如在组件挂载的时候滚动或聚焦
- 在 React 获取 DOM 属性，比如宽度或滚动位置
- 在 React 控制的 DOM 元素上使用 Portal
- 非受控复合组件是一个很强大的模式
- 将 DOM 元素提供给多个消费者