# 整理 `React` 相关的面试题

### 🔴 `React` 中有对状态管理有做进一步封装吗？

来自：`百安居`

<details>

<summary>答案：</summary>

`React` 本身除了 `useContext` 和 `useReducer` 之外，没有内置的复杂状态管理方案，但它的生态系统中有很多流行的状态管理库，为复杂组件间的状态管理提供了进一步的封装和优化。

#### 主要的解决方案和封装

1. **`Context API`：**

- `React` 提供了 `Context API`，可以不用通过每一层组件传递 `props` 的情况下，全局共享状态。
- 但当应用程序变得复杂时，单靠 `Context API` 管理状态会变得繁琐，并且可能导致性能问题，特别是组件不必要的重新渲染。

2. **`React Redux`：**

- 提供了一种可预测的方式来管理和集中应用的状态。
- 通过中间件（如 `redux-thunk` 或 `redux-saga`），`Redux` 可以处理副作用。
- 此外，`React-Redux` 可以更高效地将 `Redux` 与 `React` 组件结合起来。

> `Redux Toolkit`：`Redux` 的封装，简化了传统 `Redux` 的配置，减少了样板代码，并提供了处理异步逻辑的 `createAsyncThunk` 工具。

3. **`Recoil`：**

- `Facebook` 开发的一个状态管理库，旨在与 `React` 的并发模式无缝工作。
- 它专注于高效地管理全局和派生状态，允许更细粒度的响应式更新。
- 只有使用了特定状态的组件会在该状态变化时重新渲染。

4. **`Zustand`：**

- 一个小巧、快速、可扩展的状态管理库
- 提供了一个简单的 `API` 来管理全局和局部状态，并避免不必要的重新渲染。
- 相比 `Redux` 更简洁，适用于小到中型项目。

5. **`Jotai`：**

- 另一个轻量级的状态管理库，基于 `React` 的 `Context API` 构建
- 提供了一种更结构化的方式来管理 `atoms`（状态单元）。
- 它使不同状态之间的依赖关系更加显式化，类似 `Recoil`，可以做到细粒度的更新。

6. **`MobX`：**

- 专注于简洁和响应式编程，允许状态自动派生和更新，减少手动将状态连接到组件的需求。
- `React` 组件可以观察状态的变化，`MobX` 确保只进行必要的最少量的重新渲染。

7. **`React Query`：**

- 虽然 `React Query` 不是纯粹的状态管理库，但它是管理服务器状态（如 `API` 数据）的利器
- 简化了数据获取逻辑、缓存、同步和更新等操作，特别适合处理异步数据。

#### 总结:

`React` 的核心功能可以通过不同的状态管理解决方案得到扩展，如 `Redux`、`Recoil`、`MobX` 等。这些解决方案根据项目的复杂性，为状态管理提供了不同的优化，通常在管理大规模应用时提升性能并简化代码组织。具体使用哪一个取决于项目的需求。

</details>

### 🧑‍💻 `React` 中如何在父组件获取子组件的方法？

来自：`百安居`

<details>

<summary>在 React 中，父组件可以通过以下几种方式获取子组件的方法：</summary>

#### 一、使用 `refs`

1. 在父组件中创建一个 `ref`：

```tsx
const RefParentCom: FC = () => {
  const comRef = useRef();
  return (
    <div>
      <p>Ref parent component</p>
      <p>
        <button onClick={() => comRef.current?.callChildMethod()}>
          click me
        </button>
      </p>
      <hr />
      <RefSubCom ref={comRef} />
    </div>
  );
};
```

2. 在子组件中暴露需要被父组件调用的方法：

```tsx
const RefSubCom = forwardRef<SubRefInstance>((_, ref) => {
  useImperativeHandle(ref, () => ({
    callChildMethod: () => console.log("Ref child method called"),
  }));
  return <p>Ref children component</p>;
});
interface SubRefInstance {
  callChildMethod: () => viod;
}
```

完整实例：https://codepen.io/levi0001/pen/oNKBwwa

#### 二、通过 `context` 父组件调用子组件的方法

1. 创建一个 `context`，并包裹在父子组件最外层：

```tsx
const ConnContext = createContext<ConnInstance>({} as ConnInstance);
const App: FC = () => {
  return (
    <ConnContext.Provider value={{}}>
      <ParentComponent>
        <SubComponent />
      </ParentComponent>
    </ConnContext.Provider>
  );
};
interface ConnInstance {
  callChildMethod?: () => void;
}
```

2. 子组价获取 `context` 并绑定方法

```tsx
const SubComponent: FC = () => {
  const conn = useContext(ConnContext);
  useEffect(() => {
    conn.callChildMethod = () => console.log("call sub method");
  }, [conn]);
  return <p>Sub children component</p>;
};
```

3. 父组价通过 `context` 调用子组件的方法

```tsx
const ParentComponent: FC<PropsWithChildren> = ({ children }) => {
  const conn = useContext(ConnContext);
  return (
    <div>
      <p>Parent children component</p>
      <p>
        <button onClick={() => conn?.callChildMethod()}>click me</button>
      </p>
      <hr />
      {children}
    </div>
  );
};
```

完整实例：https://codepen.io/levi0001/pen/YzmNQvb

</details>

### 🔴 `useCallback` 使用过没？

来自：`百安居`

<details>

<summary>答案：</summary>

`useCallback` 是 `React` 中的一个 `Hook`。它用于返回一个 `memoized` 回调函数，在依赖项不变的情况下，多次渲染之间始终返回相同的函数实例。这有助于避免在组件重新渲染时，因为回调函数的重新创建而导致子组件不必要的重新渲染。

**使用场景**

1. 当把回调函数传递给子组件时，如果这个回调函数在每次父组件渲染时都重新创建，可能会导致子组件的性能问题。使用 `useCallback` 可以避免这种情况。
2. 在优化性能时，对于一些复杂的计算或可能频繁触发重新渲染的场景，使用 `useCallback` 可以确保只有在必要的时候才重新计算回调函数。

**示例用法**

```tsx
import { FC, useCallback } from "react";

const MyComponent: FC = () => {
  const handleClick = useCallback(() => {
    console.log("Button clicked");
  }, []);

  return <button onClick={handleClick}>Click me</button>;
};

export default MyComponent;
```

在这个例子中 `handleClick` 回调函数在组件初次渲染时创建一次，因为依赖项数组为空。如果有依赖项，只有当依赖项发生变化时，才会重新创建回调函数。

</details>

### 🔴 函数组件和类组件处理重复渲染有什么区别？

来自：`百安居`

<details>

<summary>答案：</summary>

**函数组件**

1. 利用 `React.memo`：将函数组件包裹在 `React.memo` 中来实现浅比较 `props` 的方式来减少重复渲染。当组件的 `props` 没有变化时，组件不会重新渲染。
2. 依赖优化：在使用 `useEffect`、`useMemo` 和 `useCallback` 等 Hook 时，可以通过精确指定依赖项数组来控制何时触发副作用和计算新的值，从而避免不必要的重复渲染。

`React.memo` 默认是浅比较，可以通过第 2 个参数进行深度检查：

```tsx
import { FC, memo } from "react";

const MyComponent: FC<MyComProps> = ({ value }) => <>component: {value}</>;

interface MyComProps {
  value: string;
}

export default memo(MyComponent, (prev, next) => prev.value !== next.value);
```

**类组件**

1. `shouldComponentUpdate`：重写类组件的 `shouldComponentUpdate` 方法来进行更细粒度的控制，决定是否进行重新渲染。该方法接收新的 `props` 和 `state` 作为参数，通过比较它们与当前的 `props` 和 `state`，返回一个布尔值来决定是否重新渲染组件。
2. `PureComponent`：类组件可以继承 `React.PureComponent`，它会对 `props` 和 `state` 进行浅比较来决定是否重新渲染组件。但只进行浅比较，不够灵活。

总的来说，函数组件在处理重复渲染时更加简洁和灵活，可以通过 `hook` 和 `React.memo` 等方式进行优化。而类组件则需要通过重写特定方法或继承特定类来实现类似的效果，相对来说较为复杂。

</details>

### 🧑‍💻 封装的按钮权限组件怎么实现？

来自：`百安居`

<details>

<summary>答案：</summary>

根据传递的 `props` 检查对应的状态，给出对应的视图

1. 创建按钮组件通过 `props` 判断权限

```tsx
const CustomButton: FC<PropsWithChildren<CustomButtonProps>> = ({
  children,
  permissionKey,
}) => {
  const group = ["add", "edit"];
  return group.includes(permissionKey) ? (
    <button>{children}</button>
  ) : (
    <button disabled>{children}</button>
  );
};
```

2. 提供不同的 `key` 使用组件

```tsx
const App: FC = () => (
  <>
    <CustomButton permissionKey="add">Add</CustomButton>{" "}
    <CustomButton permissionKey="disable">Del</CustomButton>
  </>
);
```

完整演示：https://codepen.io/levi0001/pen/abepYKK

**总结**

在实际应用中，可以根据具体的权限管理方案来获取用户权限信息，比如从后端获取用户角色信息后进行判断，或者使用状态管理库来存储和管理权限状态。同时，可以根据需要扩展组件的功能，如添加不同的按钮样式、处理点击事件等。

</details>

### 🔴 数据什么时候定义在组件里面，什么时候定义在状态管理里面？

来自：`百安居`

<details>

<summary>一般来说，可以从以下几个方面考虑数据定义的位置：</summary>

**定义在组件里面：**

- 当数据仅在特定组件内部使用，并且不会被其他组件共享或影响多个组件状态时，可以定义在组件内部。
- 如果数据的生命周期与组件的生命周期紧密相关，随着组件的创建而创建，销毁而销毁，适合放在组件里。
- 对于一些临时的、局部的、快速变化且不需要在多个地方同步的数据，可以放在组件内。

**定义在状态管理里面：**

- 当数据需要在多个组件之间共享和同步时，应该放在状态管理中。比如用户的登录状态、购物车中的商品信息等，这些数据可能会被多个不同的组件访问和修改。
- 如果数据的变化会引起多个组件的状态更新，为了更好地管理这种复杂的状态变化，将数据放在状态管理中可以集中处理状态的更新逻辑。
- 对于一些全局的、持久化的数据，如应用的配置信息等，适合放在状态管理中，以便在整个应用中随时访问和修改。

</details>

### 🔴 方法什么时候写在父组件中，什么时候写在子组件中？

来自：`百安居`

<details>

<summary>以下是关于方法写在父组件和子组件中的考虑因素：</summary>

**写在父组件中：**

- 当方法的逻辑主要涉及多个子组件的协调或者对多个子组件的状态进行统一管理时，适合写在父组件中。例如，一个页面有多个子组件，父组件需要根据某个条件同时控制这些子组件的显示或隐藏，此时控制显示隐藏的方法就可以写在父组件中。
- 如果方法是与整个应用的全局状态或业务逻辑相关，而不是特定于某个子组件的功能，通常放在父组件中。比如，在一个电商应用中，父组件可能有一个方法用于处理购物车的总价计算，这个计算可能涉及多个子组件中的商品信息。

**写在子组件中：**

- 当方法的逻辑完全是为了实现子组件自身的特定功能，并且与其他组件没有直接关系时，应该写在子组件中。例如，一个子组件是一个输入框，它有一个方法用于验证输入内容的格式是否正确，这个方法就适合写在子组件中。
- 如果方法只影响子组件自身的状态变化和行为，不涉及到父组件或其他子组件的状态管理，那么可以放在子组件内。比如，一个子组件是一个下拉菜单，它有一个方法用于展开或收起菜单的功能，这个方法就可以放在子组件中。

</details>

### 🔴 `react` 用过哪些 `hooks`？

来自：`万云科技`、`异联信息`

<details>

<summary>答案：</summary>

**基础状态管理：**

1. `useState`：用于在函数组件中添加状态，可以接收并管理各种数据类型的状态。
2. `useReducer`：用于处理复杂的状态逻辑，它接收一个 `reducer` 函数和初始状态作为参数。

> 此类 `hooks` 都返回当前状态和一个 `dispatch` 函数来触发状态更新。

**副作用处理：**

1. `useEffect`：用于处理组件挂载、更新和卸载时的副作用操作，比如发送网络请求、订阅事件、手动修改 DOM 等。
2. `useLayoutEffect`：与 `useEffect` 类似，但它会在浏览器渲染之前同步执行副作用操作，适合处理涉及布局的副作用。

此类 `hooks` 都接受 2 个参数：

- 参数 1：函数用于执行回调
- 参数 2：依赖项数组，只有当依赖项发生变更才触发更新

关于依赖项：

- 不提供：随每次渲染执行回调
- 空数组：仅首次渲染执行回调
- 带有状态的依赖项数组：只有状态变更才执行回调

回调返回函数：

- 执行前都会先执行返回的函数，`React` 中此类 `hooks` 采用先出后进的原则

**上下文和引用类：**

1. `useContext`：用于在函数组件中访问 `React` 的上下文（`Context`），可以方便地在组件树中传递和共享数据，避免通过层层传递 `props`。
2. `useRef`：用于创建一个可变的引用，可以在组件的整个生命周期中保持对某个值的引用，而不会引起组件的重新渲染。

**性能优化类：**

1. `useMemo`：用于缓存计算结果，只有当依赖项发生变化时才重新计算。可以避免不必要的计算，提高性能。
2. `useCallback`：与 `useMemo` 类似，用于缓存函数，只有当依赖项发生变化时才重新创建函数。可以避免不必要的函数重新创建，特别是在将函数作为 `props` 传递给子组件时。
3. `useTransition`：用于处理 “并发模式”（`Concurrent Mode`）的 `hook`，主要用于管理并发更新，使用户界面保持响应。

</details>

### 🔴 `React` 为什么用 `function` 组件代替 `class` 组件？

来自：`gate.io`

<details>

<summary>在 React 中，越来越倾向于使用函数组件代替类组件，主要有以下几个原因：</summary>

#### 一、简洁性

**1. 代码更简洁**

函数组件通常比类组件更简洁明了。它们以函数的形式定义组件，没有类的复杂语法和结构，减少了代码的行数和复杂度。

例如，一个简单的展示用户信息的组件，用函数组件可以这样写：

```tsx
const UserInfo: FC = ({ user }: { user: any }) => {
  return <div>Hello, {user.name}!</div>;
};
```

而用类组件则需要更多的代码：

```tsx
class UserInfo extends React.Component {
  render() {
    const { user } = this.props;
    return <div>Hello, {user.name}!</div>;
  }
}
```

**2. 易于理解和维护**

- 函数组件的简洁性使得代码更易于理解和维护。开发者可以更快地阅读和理解函数组件的逻辑，减少了在复杂的类结构中寻找特定功能的时间。
- 对于新加入项目的开发者来说，理解函数组件的代码通常比理解类组件更容易，提高了团队的开发效率。

#### 二、性能优化

**1. 减少不必要的重新渲染**

- `React` 中的函数组件在某些情况下可以更好地利用 `React` 的优化机制，减少不必要的重新渲染。
- 函数组件可以使用 `React` 的钩子（`hooks`）来管理状态和副作用，通过使用 `useMemo` 和 `useCallback` 等钩子，可以缓存计算结果和函数引用，避免在每次渲染时都重新计算和创建新的函数，从而提高性能。

例如：

```tsx
const MyComponent: FC = ({ data }: { data: any }) => {
  const memoizedValue = useMemo(() => expensiveComputation(data), [data]);
  return <div>{memoizedValue}</div>;
};
```

**2. 更好的性能优化工具**

- `React` 团队在不断优化函数组件的性能，并提供了更多的性能优化工具和技术。例如，`React` 的新特性 `React.memo` 可以对函数组件进行浅比较，只在 `props` 发生变化时才重新渲染组件，进一步提高了性能。
- 相比之下，类组件的性能优化相对较为复杂，需要手动管理生命周期方法和状态更新，容易出现错误和性能问题。

#### 三、更好的逻辑复用

**1. 自定义钩子**

- 函数组件可以使用自定义钩子来封装和复用逻辑。自定义钩子是一个函数，它可以在多个函数组件中调用，实现逻辑的复用。

例如，可以创建一个用于获取用户数据的自定义钩子：

```tsx
import { useState, useEffect } from "react";

const useUserData: () => null | { name: string } = () => {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    fetchUserData().then((data) => setUserData(data));
  }, []);

  return userData;
};

const MyComponent: FC = () => {
  const userData = useUserData();
  return <div>{userData ? userData.name : "Loading..."}</div>;
};
```

> 这样，多个组件可以共享获取用户数据的逻辑，提高了代码的可维护性和可复用性。

**2. 组合优于继承**

- 函数组件更符合组合优于继承的原则。通过组合不同的函数组件和自定义钩子，可以轻松地构建复杂的用户界面，而不需要依赖类的继承关系。
- 继承在某些情况下可能会导致代码的复杂性增加，并且难以理解和维护。函数组件的组合方式更加灵活，可以根据具体需求自由地组合和拆分组件，提高了代码的可扩展性和可维护性。

#### 四、与现代开发工具和技术的兼容性

**1. 更好地支持 `TypeScript`**

- 函数组件与 `TypeScript` 的结合更加自然和方便。`TypeScript` 可以更好地推断函数组件的类型，提供更强大的类型检查和智能提示，减少类型错误的发生。
- 相比之下，类组件在与 `TypeScript` 结合时可能需要更多的类型定义和手动处理，增加了开发的复杂性。

**2. 与新的 `React` 特性和库的兼容性更好**

- `React` 不断推出新的特性和优化，函数组件通常更容易适应这些变化。例如，`React` 的并发模式和 `Suspense` 等新特性在函数组件中使用更加方便和自然。
- 同时，许多新的 `React` 库和工具也更倾向于支持函数组件，使得开发者可以更轻松地使用这些工具来构建应用程序。

#### 总结

综上所述，`React` 使用函数组件代替类组件具有简洁性、性能优化、更好的逻辑复用和与现代开发工具的兼容性等优势。函数组件的出现使得 `React` 开发更加高效、灵活和可维护，成为现代 `React` 开发的主流方式。

</details>

### 🔴 谈谈 `React`（组件）性能优化？

来自：`异联信息`

<details>

<summary>以下是常用的 React 性能优化策略：</summary>

`React` 性能优化的关键在于减少不必要的渲染、提升数据更新效率，并最大化组件的复用。

**1. 组件更新优化**

- **使用 `React.memo`**：通过将函数组件包裹在 `React.memo` 中，使组件在接收到相同的 `props` 时不重新渲染，从而提升性能。
- **使用 `shouldComponentUpdate`**：对于类组件，通过 `shouldComponentUpdate` 方法控制组件是否应该更新，这样在 `props` 或 `state` 不变时可以避免无意义的渲染。

备注：

- 关于组件重复渲染详细见：函数组件和类组件处理重复渲染有什么区别？ [[查看](#-函数组件和类组件处理重复渲染有什么区别)]
- `memo` 和 `useMemo` 虽然可以避免组件重复渲染，但滥用不但无法提升性能还会导致额外的计算，最好的方式建议优先考虑组合组件而不是变体组件

**2. 状态管理优化**

- **减少不必要的状态提升**：尽量将状态局限在最小作用域，避免将状态提升到不必要的父级，以减少重渲染的子组件数量。
- **使用 `Context API` 的优化方式**：`Context API` 可能导致不必要的子组件渲染，可以通过拆分 `Context` 或使用 `useContext` 时搭配 `React.memo` 优化性能。

备注：

- `useContext` 优先考虑是下放到叶子节点，其次考虑组合而大于变体，最后才是 `memo` 或 `useMemo`

**3. 避免匿名函数和对象**

- **在 `props` 中避免传递匿名函数**：每次渲染时匿名函数都会创建新实例，影响性能。可以使用 `useCallback` 缓存事件处理函数，以防止子组件重新渲染。
- **缓存复杂对象**：通过 `useMemo` 缓存复杂计算对象，避免在父组件渲染时每次生成新对象。

**4. 数据加载与异步操作优化**

- **数据懒加载**：对于数据较多的场景，使用懒加载或分页加载减少首屏加载量。
- **异步操作优化**：在合适时机处理异步操作，避免阻塞渲染流程。常用的方法包括在数据加载前显示加载组件或骨架屏。

**5. 合适的渲染模式**

- **虚拟列表（`Virtual List`）**：对于长列表，采用 `react-window` 或 `react-virtualized` 实现懒加载长列表，仅渲染当前可见区域的列表项。
- **避免频繁重渲染**：在使用动画或事件处理时，频繁的 `DOM` 操作可以通过 `requestAnimationFrame` 或节流、防抖等技巧控制更新频率，减少过于频繁的渲染消耗。

**6. 减少重渲染的方法**

- **组件拆分**：将复杂组件拆分为多个较小的子组件，使每个子组件独立管理状态和渲染逻辑，从而减小重渲染的范围。
- **使用 `Fragment` 减少 `DOM` 节点**：利用 `React.Fragment` 包裹多个子元素，减少不必要的 `DOM` 包裹节点，减轻 `DOM` 操作负担。

**7. 编译优化**

- **使用生产模式的构建**：在生产环境中，确保启用 `React` 的生产模式，避免开发模式的额外检查和日志消耗。[[查看](#-在生产环境中怎样启用-react-的生产模式)]
- **代码分离（`Code Splitting`）**：利用 `React.lazy` 和 `Suspense` 实现代码分离，将非首屏代码分离至按需加载，减小初始加载体积。

通过结合这些方法，`React` 应用可以在用户体验和数据更新上获得显著的性能提升。

</details>

### 🔴 在生产环境中，怎样启用 `React` 的生产模式？

来自：`问题衍伸`

<details>

<summary>要在生产环境中启用 React 的生产模式，可以通过以下几种常见方法：</summary>

**方法一：使用 `Create React App` 创建项目时的默认设置**

如果项目是使用 `Create React App` 创建的，那么在构建项目用于生产环境时，它会自动启用 `React` 的生产模式。只需运行以下任意命令来创建生产环境的构建：

```bash
# npm
npm run build

# yarn
yarn build
```

`Create React App` 会对代码进行优化处理，包括启用 `React` 的生产模式，压缩代码、去除开发环境相关的调试代码等，生成的构建版本可直接部署到生产服务器上。

**方法二：手动设置环境变量（对于非 `Create React App` 项目）**

1. 对于使用 `Webpack` 进行打包的项目：

在项目的 `Webpack` 配置文件（通常是 `webpack.config.js`）中，需要设置 `DefinePlugin` 来定义环境变量。

```js
const webpack = require("webpack");

module.exports = {
  //...其他Webpack配置项

  plugins: [
    new webpack.DefinePlugin({
      "process.env.NODE_ENV": JSON.stringify("production"),
    }),
  ],
};
```

这里通过 `DefinePlugin` 将 `process.env.NODE_ENV` 设置为 `production`，这样 `React` 就会识别到处于生产环境，从而启用生产模式。

2. 对于使用其他打包工具或自定义构建流程的项目：

同样需要确保在运行时将 `process.env.NODE_ENV` 环境变量设置为 `production`。例如，在运行项目的脚本命令中，可以这样设置（以 `Linux/macOS` 系统为例）：

```bash
NODE_ENV=production node index.js
```

这里假设 `index.js` 是项目的入口文件，通过在运行命令前设置 `NODE_ENV` 环境变量为 `production`，`React` 会进入生产模式。

当 `React` 处于生产模式时，它会进行一系列优化，比如对代码进行更严格的错误检查（只抛出关键错误以避免影响用户体验）、优化渲染性能等，有助于提升应用在生产环境中的运行效果。

</details>

### 🔴 说说 `react state` 的更新问题？

来自：`字节`

<details>

<summary>答案：</summary>

**1, 异步更新问题**

在 `React` 中，`setState` 方法在大多数情况下是异步更新的。这意味着当你调用 `setState` 后，状态不会立即更新，`React` 会将多个 `setState` 调用合并为一个更新操作来提高性能。例如，在以下代码中：

```js
this.setState({ count: this.state.count + 1 });
console.log(this.state.count);
```

你可能期望 `console.log` 输出更新后的 `count` 值，但实际上它输出的可能是更新前的值，因为 `setState` 的更新操作还没有完成。

解决方案：如果需要在状态更新后执行某些操作，可以传递一个回调函数给 `setState`。这个回调函数会在状态更新完成后被调用。例如：

```js
// class component
this.setState({ count: this.state.count + 1 }, () => {
  console.log(this.state.count);
});

// function component (not recommended)
setState((count) => {
  const num = count + 1;
  console.log(num);

  return num;
});

// effect subscript recommended
setState(count + 1);
useEffect(() => {
  console.log(count);
}, [count]);
```

**2. 不可变数据原则问题**

`React` 推荐遵循不可变数据原则来更新状态。如果直接修改状态对象或数组（例如，直接修改 `this.state.obj.key = 'newValue'` 或者 `this.state.array.push('newItem')`），`React` 可能无法正确检测到状态的变化，从而导致组件不能正确地重新渲染。

解决方案：对于对象，可以使用 `Object.assign` 或者扩展运算符（`...`）来创建一个新的对象进行更新。例如，要更新一个对象中的属性：

```js
this.setState((prevState) => ({
  obj: Object.assign({}, prevState.obj, { key: "newValue" }),
}));

// 或者使用扩展运算符
this.setState((prevState) => ({
  obj: { ...prevState.obj, key: "newValue" },
}));

// 函数组件和上述基本一致，但拿到的是更新的状态本身
setState((obj) => Object.assign({}, obj, { key: "newValue" }));
setState((obj) => ({ ...obj, key: "newValue" }));
```

对于数组，可以使用数组的非变异方法（如 `concat`、`filter`、`map` 等）来更新。例如，要在数组中添加一个元素：

```js
this.setState((prevState) => ({
  array: prevState.array.concat(["newItem"]),
}));

// 函数组件和上述基本一致，但拿到的是更新的状态本身
setState((array) => array.concat(["newItem"]));
setState((array) => [...array, "newItem"]);
```

**3. 更新顺序问题**

当在一个组件中有多个 `setState` 调用，并且这些调用相互依赖时，更新顺序可能会导致意外的结果。例如，在一个组件的方法中：

```js
this.setState({ count: this.state.count + 1 });
this.setState({ doubleCount: this.state.count * 2 });
```

这里期望 `doubleCount` 是更新后的 `count` 的两倍，但由于 `setState` 是异步的，实际上 `doubleCount` 可能是基于旧的 `count` 值计算的。

解决方案：可以使用 `prevState` 参数来确保基于正确的前一个状态进行更新。例如：

```js
this.setState((prevState) => ({
  count: prevState.count + 1,
}));
this.setState((prevState) => ({
  doubleCount: prevState.count * 2,
}));

// 或者将多个相关的状态更新合并到一个setState调用中
this.setState((prevState) => ({
  count: prevState.count + 1,
  doubleCount: (prevState.count + 1) * 2,
}));

// function component
setCount((count) => count + 1);
useEffect(() => {
  setDoubleCount(count * 2);
  return () => {
    console.log(count); // subscript previously count
  };
}, [count]);
```

> 在函数组件中可以在 `useEffect` 的 `memorize` 函数中通过 `return` 的方法中订阅状态更新前的值

**4. 性能问题（频繁更新）**

如果在短时间内频繁地调用 `setState`，可能会导致组件过度渲染，降低应用程序的性能。例如，在一个 `onScroll` 或 `onMouseMove` 事件处理函数中频繁地更新状态，会触发大量不必要的重新渲染。

解决方案：可以使用防抖（`Debounce`）或节流（`Throttle`）技术来限制 `setState` 的调用频率。另外，还可以使用 `shouldComponentUpdate`（在类组件中）或者 `React.memo`（在函数组件中）来优化组件的重新渲染，避免不必要的更新。例如，使用 `React.memo`：

```jsx
const MyComponent = React.memo((props) => {
  // 组件内容
  return <div>{props.value}</div>;
});
```

</details>
