## Mount

### Component called
React 的渲染过程是指 React 调用组件来确定屏幕上显示的内容。在这个过程中，React 要求组件必须“纯粹”，也就是说，组件在渲染时应该只返回 JSX，而不应该改变任何已经存在的对象或变量。

### Evaluate local variables and Initialize hooks
在 render 方法中，所有组件的本地变量都会被计算，并且效果会被安排。在首次渲染时，某些 hooks 会被初始化。
- 对于 useState、useReducer 和 useRef 等 hooks，在组件首次渲染时，React 会保存它们的初始值，并在之后的所有渲染中忽略它们。
    - React 允许将一个函数作为参数传递给 useState 和 useReducer，以便获得初始状态。为了提高性能，React 只会在组件挂载时严格调用一次这个初始化函数。
- useMemo 返回的值是调用传递给它的函数后的结果。
- 在初始渲染时，React 返回（而不是调用）传递给 useCallback 的函数。
- 在使用 useDeferredValue 时，React 返回的值与您提供的值相同。

### Return React elements
组件返回 JSX (React 元素)。 如果它返回组件，则 React 将递归渲染这些组件，直到只剩下主机元素（表示 DOM 节点的元素）为止。

### Insert DOM nodes
在从返回的 React 元素中插入 DOM 节点时，这被称为提交(commit)，由渲染器 (react-dom) 处理。
在 DOM 节点被插入到文档后，可以通过各种方式来操作它们，例如添加事件监听器、更新属性和样式、动态调整布局等。对于某些 DOM 元素（例如表单控件）可能需要自动聚焦来提高用户体验。可以通过向元素添加 autoFocus 属性以及 onFocus 事件处理器来实现这一点。在浏览器将元素添加到 DOM 中时，将自动触发 onFocus 事件，从而实现自动聚焦的效果。

### Set DOM refs
Refs 提供了一种访问 DOM 节点的方式。在第一次渲染时，DOM 节点还没有被创建，因此 ref.current 为 undefined。在提交后，React 就会设置 ref.current 的值。
除了使用 useRef 对象来创建引用外，还可以将一个函数作为 ref 属性传递，即 ref 回调函数。当该组件被挂载时，React 将调用该函数，并将相应的 DOM 元素作为参数传递给它。

### useLayoutEffect setups
当 React 完成了 DOM 渲染后，在浏览器进行绘制之前，会运行 useLayoutEffect 中返回的同步更新函数(setup)。它们可以访问和修改 DOM，但需要注意不能阻止浏览器的渲染。也可以有一个可选的清除函数(cleanup function)。需要注意的是，如果组件返回了子组件，布局副作用将从子组件向上调用父组件。

### DOM Paint
在渲染完成并更新 DOM 后，React 会将更新发送给浏览器，然后浏览器会重新计算和绘制网页元素。

### useEffect setups
在一段短暂的延迟之后，React 将会运行 useEffect。useEffect 会在浏览器渲染后运行，这是因为大多数类型的工作不应该阻止浏览器更新屏幕。effect 设置函数可以选择性地返回一个清除函数。需要注意的是，如果组件返回子组件，则 effect 会从子组件向父组件依次调用。

# Update
在 React 中，当一个组件的状态或属性发生变化时，React 会自动更新组件，并根据更新后的状态重新渲染组件。

在 React 中，默认情况下，如果一个组件的状态或属性发生变化，React 会递归渲染所有的子组件。哪怕这些子组件的属性并没有发生变化，React 也会重新渲染它们。

在 React 中，重渲染（rerenders）的次数通常比挂载（mounting）和卸载（unmounting）的次数多，因为组件可能会由于各种原因而需要更新，例如用户操作、数据变化等。因此，一个好的 React 组件应该可以在任何时候进行重新渲染，而不会出现问题。

### Evaluate local variables and Run necessary hooks
在 React 中，组件被重新渲染时，React 会重新调用组件函数（function component）或 render 方法（class component）并传入新的 props 和 state。同时，如果组件中使用了一些React Hook（例如 useEffect、useLayoutEffect、useMemo、useCallback、useImperativeHandle等），React 也会对这些 Hook 进行重新评估。

这些 Hook 都具有依赖数组（dependency array），它们用于指定 Hook 在何时应该执行。当组件重新渲染时，React 会比较新的依赖值和上一次的依赖值（或者是一个空数组），如果它们不相等，则认为这个 Hook 的依赖发生了变化，需要重新执行 Hook。

具体来说，React 会使用 Object.is 方法来比较新旧依赖值的相等性。如果它们相等，则 React 会跳过当前 Hook，并返回上一次 Hook 的返回值。否则，React 会调用当前 Hook，评估新的依赖值，并返回新的返回值。

### Return React elements
在 React 中，组件渲染更新后，会返回更新后的 JSX（也称为React elements）。这个 JSX 可以包含任何的 React 元素，例如 HTML 元素、自定义组件、或者由 React 创建的可重用组件等等。

### Update DOM nodes
在React中，更新组件的 JSX（React元素）之后，还需要将这些变化提交给实际的DOM树，以便用户可以看到最新的UI。这个过程称为“DOM更新”。

### Unset DOM refs
在React更新DOM之前，React会将受影响的ref.current值设置为null。这是为了确保在组件更新过程中已经不再引用旧的DOM或React元素。因此，当我们在使用ref属性时，确保可以处理null的情况非常重要。

此外，ref属性的标识（identity）也会强制它进行更新。如果我们使用不稳定的ref回调（即在渲染中使用闭包定义的函数），React会在每次渲染后调用此回调，并将其设置为新的DOM元素或React元素引用。因此，如果我们需要确保回调函数只在挂载时调用一次，我们可以使用稳定的ref回调。稳定的ref回调是在初始化时创建的，与挂载状态无关，并且只被调用一次。

### useLayoutEffect cleanups
当组件重新渲染时，useLayoutEffect的清除函数将被调用，以便保持组件的状态和行为，以便后续的渲染或更新。

但是需要注意的是，useLayoutEffect的清除函数是在DOM更新之前运行的，这意味着其使用的状态和属性值是前一个渲染的状态和属性值。这很重要，因为它可以帮助我们确定在组件更新之前需要执行的操作，并在组件重新渲染时更新状态。

而且需要注意，useLayoutEffect的清除函数的执行顺序是从子组件依次向父组件执行的，这与useEffect的清理函数行为是不同的。这是因为useLayoutEffect是在DOM更新之前立即同步触发的，因此必须保证其清除函数在元素卸载时按正确的顺序执行。而对于useEffect，清理函数则是在组件重新渲染之后异步执行的，不需要考虑这个顺序问题。

总之，在使用useLayoutEffect() hook时，我们需要小心地处理其清理函数和前一个渲染的状态，以确保组件在更新时保持正确的状态和行为。同时，在处理父子组件之间的交互时，需要注意useLayoutEffect清理函数的执行顺序，以确保组件被正确卸载并按预期方式更新。

### Set DOM refs
在React中，当更新DOM完成后，React会立即将对应的DOM节点设置为其相应的ref.current值。这意味着我们可以通过ref属性来访问最新的DOM节点，并在需要时执行一些操作。
需要注意的是，如果使用了稳定的Ref回调函数，它将不会在组件更新时被调用。这是由于稳定的Ref回调函数只会在初次挂载时被调用，因此不会受到更新的影响。

### useLayoutEffect setups
当组件重新渲染时，useLayoutEffect的设置函数将被再次调用，以便更新组件的状态和行为。
如果useLayoutEffect设置函数使用了依赖数组，那么只有依赖项发生变化时，才会执行这些设置函数。如果useLayoutEffect没有依赖数组，那么每次组件更新时都会执行设置函数。这可以帮助我们避免在不必要的情况下运行一些操作，从而提高性能。

### DOM Paint
组件的渲染可能并不一定会有DOM的更新。如果渲染的结果与当前DOM树结构相同，浏览器就不需要进行任何改动，因此组件可能并不会产生可视的变化。

但是，当组件产生DOM更新时，React会立即将这些更新传递给浏览器进行更新和重绘。当浏览器收到DOM更新后，它将重绘屏幕，并更新呈现组件的区域。这个过程称为“重绘”或“回流”，因为浏览器需要重新计算文档的布局并重新绘制受影响的元素。

重绘可以将一些耗时的计算分配给浏览器，并且React也会尽可能地优化DOM更新，以减少重绘的次数和运行时间。例如，React会使用虚拟DOM来比较当前DOM树和新的DOM树，只更新其中有变化的部分。这种优化可以减少不必要的DOM更新和重绘，从而提高应用程序的性能和响应速度。

总之，在React中，组件的渲染和DOM更新可能会触发浏览器的重绘和回流过程。我们需要小心处理这些过程，避免不必要的计算和变更，并尽可能地使用React的优化来提高应用程序的性能。同时，我们需要理解React的更新机制，以便能够预测何时组件会产生DOM更新以及何时进行重绘和回流。

### useEffect cleanups
当组件重新渲染时，React会先运行上一次渲染产生的useEffect()的清除函数，以确保所有副作用操作都在组件被销毁前被清理。

useEffect()清除函数会在组件重新渲染时运行，并且使用的状态或属性值是上一次渲染产生的值。这意味着在清除函数内部使用的状态或属性值是“过时的”，即它们是前一个渲染的状态或属性值。因此我们需要小心处理，避免在清除函数中使用过时的状态或属性值，以确保正确地清除副作用。

同时，与useEffect()的设置函数类似，清除函数也是从子组件依次向父组件运行，并且会先运行子组件的清除函数。这是为了确保所有副作用操作都能得到正确的清理，并避免可能的资源泄漏或其他问题。

### useEffect setups
当组件重新渲染时，React会运行useEffect()的设置函数，并传入新的状态或属性值，以便我们能够对其进行处理。
如果useEffect()设置函数使用了依赖项数组，那么只有在依赖项发生变化时，才会运行这些设置函数。如果useEffect()没有依赖项数组，那么它将在每次组件重新渲染时都运行。

# Unmount

### useLayoutEffect cleanups
React会运行上一次渲染产生的useLayoutEffect()的清除函数，以确保所有副作用操作都在组件被销毁前被清理。
在组件卸载时，useLayoutEffect()的清除函数会被调用，并传入最终的DOM元素。这可以让我们执行一些清理操作，例如移除事件侦听器，取消动画等等。

### Unset DOM refs
当组件被卸载时，React会自动将该ref属性设置为null，以便释放对DOM节点的引用和内存占用。
无论是稳定的ref回调函数还是不稳定的ref回调函数，都会在卸载时被调用，并传入null作为参数。这可以让我们执行一些必要的清理操作，例如释放内存、取消订阅等等。

### Remove DOM nodes
当一个组件被卸载时，它所关联的DOM节点也会被从页面中移除。这意味着该组件所产生的影响、事件处理等等都将被撤销。
如果该组件包含了其他组件作为子组件，那么这些子组件也将被递归地卸载并移除其关联的DOM节点。

### useEffect cleanups
组件被卸载时，React会运行所有之前运行的useEffect()的清除函数，以确保任何副作用操作都在组件被销毁前被清理。
