# Hooks
## Write your own hooks

Chúng ta có thể tự viết Hooks để thực hiện chia sẻ stateful logic giữa các component.
Một custom hook là một hàm Javascript bắt đầu bằng 'use' và có thể gọi các Hooks khác.

1. Tạo Hook

Ví dụ: useFriendStatus Hook để xác định một friend có đang online hay không:
```javascript
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

2. Sử dụng Hook

Ví dụ sử dụng useFriendStatus hook ở trên:
```javascript
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

## Common Hooks

### useCallback
```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
Trả về một memoized callback.

Truyền một hàm callback và một mảng các dependency. useCallback sẽ trả về một phiên bản memoized của hàm callback này và chỉ thay đổi khi một trong các dependency thay đổi -> tránh các render không cần thiết.

`useCallback(fn, deps)` tương ứng với `useMemo(() => fn, deps)`.

### useRef

`useRef` trả về một đối tượng ref có thuộc tính `current` được khởi tạo là tham số truyền vào. Đối tượng này được giữ trong toàn bộ vòng đời của component. Ví dụ:

```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // current trỏ đến element text input đã được mount
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

`useRef` tạo một plain Javascript object. Điều khác biệt duy nhất giữa `useRef()` và tạo một đối tượng `{current: ...}` là `useRef` sẽ trả về một đối tượng `ref` duy nhất với mỗi lần render. 

### useMemo
```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```
Trả về một giá trị memoized.

Truyền một hàm "tạo" và một mảng các dependency. useMemo chỉ tính toán lại giá trị memoized khi một trong các dependency thay đổi -> tránh tính toán nhiều lần mỗi lần render.

Nên sử dụng useMemo để tối ưu hiệu năng chứ không phải để đảm bảo tính đúng đắn.

### useReducer
```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
- Một phương án thay thế cho `useState`. 
- Nhận tham số reducer có dạng `(state, action) => newState` và trả về state hiện tại và phương thức `dispatch`
- Sử dụng khi có logic state phức tạp
- `initialArg` là giá trị khởi tạo của state
- `init` là hàm để trả về giá trị khởi tạo

### useContext
```javascript
const value = useContext(MyContext);
```
- `useContext` nhận một đối tượng context và trả về giá trị hiện tại của context đó. Giá trị context hiện tại được xác định bởi `<MyContext.Provider>` gần nhất. Khi `<MyContext.Provider>` gần nhất cập nhật, component sẽ được re-render với giá trị context mới. 

- `useContext(MyContext)` tương ứng với `static contextType = MyContext` trong class component hoặc `<MyContext.Consumer>`
# Context

Context cung cấp phương thức truyền dữ liệu qua một cây component mà không cần truyền props tại mỗi level.

**Sử dụng context khi nào**:
- Context được thiết kế để chia sẻ các dữ liệu "toàn cục" cho một cây component, vd như user hiện tại, theme, hoặc ngôn ngữ hiển thị.
- Sử dụng context khi có nhiều component tại nhiều nesting level cần truy cập tới dữ liệu, tránh lạm dụng do nó khiến component khó tái sử dụng hơn.

## Tạo Context
```javascript
const MyContext = React.createContext(defaultValue);
```
`React.createContext()` tạo một đối tượng Context. Khi React render component subscribe tới đối tượng này nó sẽ đọc giá trị context hiện tại từ Provider gần nhất. Nếu không tìm được Provider, nó sử dụng giá trị `defaultValue`.

## Context.Provider
```javascript
<MyContext.Provider value={/* some value */}>
```
- Mỗi đối tượng Context có một component Provider cho phép các component khác subscribe tới sự thay đổi của context.
- Provider component chấp nhận một prop là `value` để truyền cho các component là con cháu của Provider này.
- Khi value của Provider thay đổi các component con cháu sẽ được re-render.

## Context.Consumer
```javascript
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```
- component Consumer sẽ subscribe tới sự thay đổi của context. Sử dụng component này cho phép subscribe context trong một function component.
- yêu cầu có child là một function. Hàm này nhận `value` của context hiện tại và trả về một React node. 

## Class.contextType
Thuộc tính `contextType` của một class có thể được gán bằng một đối tượng Context. Thuộc tính này cho phép component sử dụng `value` của context đó thông qua `this.context`.

# Refs
Ref cung cấp cách thức truy cập các nút DOM hoặc React element được tạo từ phương thức render.

**Khi nào sử dụng ref**
- focus, text selection hoặc media playback.
- trigger hình ảnh động
- tích hợp với các thư viện DOM của bên thứ ba.
Tránh sử dụng ref cho những thứ có thể làm được bằng cách khai báo.

## Tạo ref
- ref được khởi tạo sử dụng `React.createRef` và gắn vào React element thông qua thuộc tính `ref`.
- ref thường được gán cho một đối tượng ở giai đoạn khởi tạo.
Ví dụ:
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

## Truy cập ref
Khi một element được truyền thuộc tính ref, một reference tới nút đó được truy cập thông qua thuộc tính `current` của ref.
```javascript
const node = this.myRef.current;
```
- Khi thuộc tính `ref` sử dụng cho HTML element, đối tượng `ref` có thuộc tính `current` là DOM element.
- Khi thuộc tính `ref` sử dụng cho class component, đối tượng `ref` có thuộc tính `current` là thực thể đã mount của component.
- Không sử dụng thuộc tính `ref` cho function component do chúng không có thực thể.

# Render props
`render props` là một kỹ thuật chia sẻ code giữa các React component sử dụng prop có giá trị là một function.
Một component có `render props` nhận một function trả về React element và dùng nó thay vì tự implement render logic.

Sử dụng render props:
```javascript
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```
Trong ví dụ trên, component Mouse có một render props là một hàm, trong Mouse gọi hàm `this.props.render` để thực thi logic hiển thị. MouseTracker gọi Mouse với render props là một hàm trả về Cat component.

Một render prop là một prop dạng hàm mà component sử dụng để biết cần render gì.

Lưu ý: tránh sử dụng render prop với PureComponent.

# High order components

High order components (HOC) là một hàm nhận một component và trả về một component mới.

Ví dụ: CommentList và BlogPost đều cần subscribe tới một DataSource, ta có thể sử dụng một HOC đặt tên là `withSubscription` để thực hiện:

```javascript
// nhận component mới từ CommentList sử dụng withSubscription
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);

// HOC withSubscription nhận một component và trả về một component mới
function withSubscription(WrappedComponent, selectData) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // render component cũ với dữ liệu mới
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

Lưu ý:
- Không nên thay đổi component nguyên mẫu, trong trường hợp muốn có các chức năng cần lifecycle method, sử dụng container component (giống ví dụ trên)
- Không sử dụng HOC trong phương thức render() do khả năng component sẽ bị re-mounting.

# Portals
Portal là phương thức để render children vào một DOM node tồn tại ngoài cây DOM của component cha.

## Sử dụng
```javascript
render() {
  // React render children vào `domNode`
  // `domNode` là một nút DOM hợp lệ, bất kể vị trí của nó trong DOM
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```
- sử dụng portal khi component cha có một overflow: `hidden` hoặc `z-index` style nhưng cần component con "thoát ra" khỏi container. Vd: dialog, hovercard, tooltips.

## Event bubbling thông qua portal
Tuy một portal có thể ở bất kì đâu trong cây DOM, nó vẫn hành xử như một React con bình thường trong mọi khía cạnh khác, vd như context.

Một event khởi sự từ nút con vẫn sẽ được catch bởi nút cha trong React dù ở DOM chúng không có liên hệ nào.

# Error boundaries

Error boundaries là các React component bắt error của Javascript từ bất cứ đâu trong cây component con của nó, log các error và hiển thị một fallback UI thay vì component bị crashed. 

Error boundaries hoạt động tương tự `catch{}` block, nhưng dành cho component. Chỉ các class component mới có thể trở thành error boundary.

Một class component trở thành error boundary nếu nó định nghĩa một hoặc cả 2 phương thức lifecycle `static getDerivedStateFromError()` - render fallback UI và `componentDidCatch()` - log error. 

Ví dụ:
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // cập nhật state để render fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

Sau đó có thể sử dụng nó như một component bình thường:
```javascript
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

# Fiber architecture

Fiber là kiến trúc core mới của React từ React 16.0. 

Từ phiên bản này có thêm một số tính năng mới: Asynchronous Rendering, Portals, Error Boundaries, Fragments. Một chức năng bổ sung quan trọng là "incremental rendering", cho phép chia nhỏ công việc render thành các chunk.

React Fiber là thuật toán reconciliation mới. Reconciliation là quá trình so sánh cây mới và cây cũ để tìm ra những thành phần thay đổi hoặc bị chỉnh sửa. Trong thuật toán reconciliation cũ, quá trình này thực thi một cách đồng bộ, do vậy các tác vụ liên quan tới UI khác như: animation, layout,.. không sử dụng được luồng chính. 

React Fiber thực thi reconciliation trong 2 giai đoạn: Render và Commit, trong đó các phương thức được gọi ở giai đoạn render sẽ được thực thi bất đồng bộ. 

- Fiber giúp ứng dụng linh động và đáng tin cậy hơn.
- Trong tương lai, Fiber có thể thực hiện các công việc song song aka Time Slicing.
- Fiber cải thiện thời gian startup trong khi render sử dụng React Suspense.
