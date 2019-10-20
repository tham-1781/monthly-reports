### Giới thiệu
Trong lúc tiếp xúc và làm việc với Reactjs, chắc hẳn chúng ta ai cũng dùng hàng tá thư viện, ở đây mình sẽ đề cập tới thư viện
`react-redux`. Trong quá trình làm chắc bạn cũng từng viết đoạn code như sau:
```javascript
...
import { connect } from 'react-redux';
...
export default connect(mapStateToProps, mapDispatchToProps)(ComponentA); 
// Kết nối Component với Store của Redux bằng thư viện react-redux
```
Và `ComponentA` sẽ nhận được `props` là state được connect từ redux. Có bao giờ bạn tự hỏi `connect` kia là gì và tại sao lại viết như vậy không?
Trong bài này mình sẽ giới thiệu cho các bạn một patterns khá hay ho trong react, đó là Higher-Order Components (HOCs).
### HOCs là gì?
Về bản chất, HOCs không phải là một phần của React API, nó là một pattern xuất hiện từ những thành phần đặc tính của React. Thường được implement như một function, mà về cơ bản, là một class factory.

JavaScript là một ngôn ngữ phù hợp để lập trình chức năng vì nó có thể chấp nhận các hàm bậc cao hơn* (higher-order function). Higher-order function là những function có thể truyền function như là tham số và trả về function là kết quả. Kỳ diệu quá =))
> Higher-Order Components (HOCs) là một function nhận vào một component và trả về một component mới.

`EnhancedComponent = higherOrderComponent(WrappedComponent);`

Khá quen thuộc với những ai học javascript phải không nào =))
HOCs cho phép chúng ta trừu tượng hóa các hành động, không chỉ các giá trị. Nó được sử dụng phổ biến trong các thư viện như mình đã đề cập ở phần giới thiệu như Redux, React Router...

Mục đích chính của HOCs trong React là chia sẻ chức năng chung giữa các components mà không lặp lại code. Vậy lợi ích mà nó mang lại là gì? Lướt xuống dưới nữa nhé =))

### Lợi ích của HOCS
- Tái sử dụng code, logic và tự động trừu tượng hóa (bootstrap abstraction)
- Chiếm quyền render (Render Highjacking)
- Trừu tượng hóa (abstraction) và điều khiển (manipulation) State
- Điều khiển Props
### Các loại HOCs
Có 2 cách implement HoCs thường thấy trong React: Props Proxy (PP) và Inheritance Inversion (II). Cả 2 cách cho phép các cách khác nhau để thao tác với WrappedComponent.
Trước khi bắt đầu chúng ta cần một project:
```javascript
create-react-app learn-hoc
cd learn-hoc/src/
mkdir hocs
cd hocs
touch propsProxyHOC.js
npm start
```

#### Props Proxy (PP)
Props Proxy (PP) được implement thông thường theo cách sau:
```javascript
function propsProxyHOC(WrappedComponent) {
  return class PP extends React.Component {
    render() {
      return <WrappedComponent {...this.props}/>
    }
  }
}
```
Phần quan trọng nhất ở đây là method render của HoC trả về một React Element của kiểu WrappedComponent. Chúng ta cũng truyền props mà HoC nhận được, vì thế phương pháp này mới có tên Props Proxy. Vậy chúng ta có thể làm gì với Props Proxy?
- Điều khiển props
- Truy cập instance thông qua Refs
- Trừu tượng hóa (Abstracting) State
- Bao WrappedComponent với elements khác
##### Điều khiển props
Chúng ta có thể đọc, thêm, sửa đổi và xóa props được truyền cho WrappedComponent.

Nhưng cẩn thận với việc xóa hay sửa đổi các prop quan trọng, chúng ta nên đặt namespace cho HoC props để nó không phá vỡ WrappedComponent.

Ví dụ: Thêm mới props.
```javascript
// propsProxyHOC.js
import React from 'react';

export default function propsProxyHOC(WrappedComponent, text) {
  return class PropsProxyHOC extends React.Component {
    render() {
      const newProps = {
        ...this.props,
        text,
      }

      return <WrappedComponent {...newProps}/>
    }
  }
}
```
Qua file App.js sửa lại chút.

```javascript
import React from 'react';
import logo from './logo.svg';
import './App.css';
import propsProxyHOC from './hocs/propsProxyHOC'

function App(props) {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        {console.log(props)}
      </header>
    </div>
  );
}

export default propsProxyHOC(App, "This is new props from App Component");
```
`propsProxyHOC` có hai tham số truyền vào là App component và chuối message là props mình muốn điều kiển trong `propsProxyHOC` component.

Và ở console chúng ta có kết quả:

![](https://i.imgur.com/wJmJ7gP.png)

##### Truy cập instance thông qua Refs
Chúng ta có thể truy cập this (instance của WrappedComponent) với ref, nhưng chúng ta sẽ cần một quá trình render đầy đủ của WrappedComponent để ref có thể được tính toán. Điều này có nghĩa là chúng ta cần trả về WrappedComponent element từ method render của HoC, để React có thể làm quá trình đối chiếu (reconciliation process) và chúng ta sẽ có ref đến WrappedComponent instance.

Ví dụ: Chúng ta sẽ tìm hiểu làm thế nào để truy cập instance methods và instance của chính WrappedComponent thông qua refs

```javascript
import React from 'react';

export default function refsPropsProxy(WrappedComponent) {
  return class RefsPropsProxy extends React.Component {
    proc(wrappedComponentInstance) {
      console.group('refs Proc');
      console.log(wrappedComponentInstance);
      wrappedComponentInstance.test();
      console.groupEnd();
    }

    render() {
      const props = Object.assign({}, this.props, {ref: this.proc.bind(this)})
      return <WrappedComponent {...props}/>
    }
  }
}
```

Ở file `App.js` sửa lại một chút như sau:

```javascript
import React from 'react';
import logo from './logo.svg';
import './App.css';
import refsPropsProxy from './hocs/refsPropsProxy';

class App extends React.Component {
  test = () =>  {
    console.log('call Test');
  }

  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            Edit <code>src/App.js</code> and save to reload.
          </p>
          <a
            className="App-link"
            href="https://reactjs.org"
            target="_blank"
            rel="noopener noreferrer"
          >
            Learn React
          </a>
        </header>
      </div>
    );
  }
}

export default refsPropsProxy(App);
```
Và ở console ta có kết quả như sau:

![](https://i.imgur.com/ItReJ0F.png)

Khi WrappedComponent được render xong thì ref callback sẽ được thực thi, và chúng ta sẽ có ref đến WrappedComponent instance. Điều này có thể được sử dụng để đọc/thêm các props và gọi các instance method.

##### Trừu tượng hóa (Abstracting) State
Chúng ta có thể trừu tượng hóa state bằng cách cung cấp props và callbacks cho WrappedComponent. Ví dụ: Chúng ta sẽ thực hiện trừu tượng hóa state để kiểm soát input.

```javascript
import React from 'react';

export default function statePropsProxy(WrappedComponent) {
  return class StatePropsProxy extends React.Component {
    constructor(props) {
      super(props);
      this.state = { fields: {} };
    }

    getField(fieldName) {
      if (!this.state.fields[fieldName]) {
        this.state.fields[fieldName] = {
          value: "",
          onChange: event => {
            this.state.fields[fieldName].value = event.target.value;
            this.forceUpdate();
          }
        };
      }

      return {
        value: this.state.fields[fieldName].value,
        onChange: this.state.fields[fieldName].onChange
      };
    }

    render() {
      const props = Object.assign({}, this.props, {
        fields: this.getField.bind(this)
      });

      return <WrappedComponent {...props} />;
    }
  };
}

// App.js

import React from 'react';
import './App.css';
import statePropsProxy from './hocs/statePropsProxy';

class App extends React.Component {
  test = () =>  {
    console.log('call Test');
  }

  render() {
    console.group('App');
    console.log('render');
    console.log('name', this.props.fields('name'));
    console.log('email', this.props.fields('email'));
    console.groupEnd();
    return (
      <div className="App">
        <form>
          <label>Automatically controlled input!</label>
          <input type="text" placeholder="Name" {...this.props.fields('name')}/>
          <input type="email" placeholder="Email" {...this.props.fields('email')}/>
        </form>
      </div>
    );
  }
}

export default statePropsProxy(App);

```
Và chúng ta có kết quả:

![](https://i.imgur.com/sNcW2I3.png)

Việc trừu tượng hóa state có nhiều ứng dụng, và được sử dụng khá nhiều trong việc giải quyết các vấn đề mà Stateless component gặp phải như không có ref chẳng hạn.

##### Bao WrappedComponent với elements khác
Chúng ta có thể bao WrappedComponent với component hoặc element khác để styling, layout hoặc mục đích khác. Cách sử dụng cơ bản có thể hoàn thành bởi Parent Components nhưng chúng ta có nhiều sự linh hoạt hơn với HoCs như đã mô tả ở trên.
```javascript
// elmWrapPP.js
function elmWrapPP(WrappedComponent) {
  return class ElmWrapPP extends React.Component {
    render() {
      return (
        <div style={{display: 'block'}}>
          <WrappedComponent {...this.props}/>
        </div>
      )
    }
  }
}
```
#### Inheritance Inversion
Inheritance Inversion (II) thường được implement như sau:

```javascript
function iiHOC(WrappedComponent) {
  return class Enhancer extends WrappedComponent {
    render() {
      return super.render()
    }
  }
}
```

Như các bạn thấy, HOCs trả về class (Enhancer) kế thừa (extends) WrappedComponent. Phương pháp này gọi là Inheritance Inversion là do thay vì WrappedComponent mở rộng (kế thừa) Enhancer class nào đó, nó lại được mở rộng (kế thừa) bởi Enhancer. Theo cách này, mối quan hệ giữa chúng dường như bị đảo ngược.

II cho phép HoC truy cập vào WrappedComponent instance thông qua this, điều này có nghĩa là HoC có quyền truy cập state, props, component lifecycle hooks và cả phương thức render.

Chúng ta sẽ không đi sau vào chi tiết chúng ta có thể làm gì với component lifecycle hooks, đó không phải là những gì cụ thể HoC làm, nó là React. Nhưng lưu ý rằng chúng ta hoàn toàn có thể tạo ra lifecycle hooks mới cho WrappedComponent. Và nhớ răng luôn gọi `super.[lifecycleHook]` để không phá vỡ WrappedComponent.

Inversion Inheritance HOCs thường được sử dụng trong các tình huống sau:
- Chiếm quyền render (Render Highjacking)
- Điều khiển state (Manipulating state)

##### Render Highjacking
Phương pháp này gọi là Render Highjacking bởi vì HOCs kiểm soát render output của WrappedComponent và chúng ta có thể làm bất kì điều gì với nó.

Trong Render Highjacking chúng ta có thể:
- Đọc, thêm, sửa, xóa props trong bất kì React Elements nào xuất ra bởi render.
- Đọc và sửa đổi React elements tree xuất ra bởi render.
- Hiển thị elements tree theo điều kiện.
- Bao element tree cho mục đích styling (giống như đã nói ở PP)
##### Manipulating state
HOCs có thể đọc, chỉnh sửa và xóa state của WrappedComponent instance, và chúng ta cũng có thể thêm state nếu cần. Hãy nhớ rằng chúng ta đang làm rối state của WrappedComponent, điều có thể dẫn chúng ta đến việc hủy hoại mọi thứ. Hầu hết các HOC nên được giới hạn để đọc hoặc thêm state, và sau đó được đặt tên (namespace) để không làm rối state của WrappedComponent.

Ví dụ: Debugging bằng cách truy cập props và state của WrappedComponent

```javascript
function IIHOCDEBUGGER(WrappedComponent) {
  return class II extends WrappedComponent {
    render() {
      return (
        <div>
          <h2>HOC Debugger Component</h2>
          <p>Props</p> <pre>{JSON.stringify(this.props, null, 2)}</pre>
          <p>State</p><pre>{JSON.stringify(this.state, null, 2)}</pre>
          {super.render()}
        </div>
      )
    }
  }
}
```
HOCs này sẽ bao WrappedComponent với element khác đồng thời hiện các props và state của WrappedComponent.

### Kết luận
Hi vọng là sau khi đọc bài này, mọi người sẽ hiểu hơn một chút về React HOCs. Kỹ thuật này được các thư viện sử dụng rất phổ biến như React-Router, Redux,...

Enjoy coding <3.
