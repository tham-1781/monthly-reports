### Giới thiệu
React là một thư viện javascript giúp lập trình viên phát triển giao diện người dùng, nó cho phép chúng ta kết hợp những component với nhau để tạo nên một web single page application.
Vì nó được tạo ra bởi các component nên khi app của bạn ngày một phình to, có nhiều component mà
muốn dùng chung state thì chúng ta sẽ phải quản lý những state đó như thế nào? Đây cũng là lý do React đã tạo ra một [Context API](https://reactjs.org/docs/context.html), cho phép chúng ta chia sẻ những state cho nhiều component khác nhau.
### Dùng props thay vì Context API được không?
Đúng, bạn hoàn toàn có thể dùng props để truyền state giữa các component, nhưng điều này không thuận lợi tí nào, 
vì nếu chúng ta có nhiều cấp components, những component ở giữa không dùng đến props trung gian.
Chúng chỉ làm nhiệm vụ trung chuyển state từ component cha xuống nhiều cấp component con khác nhau.
### So sánh với cách thông thường
Giả sử mình có 3 component `FooBar`, `Foo` và `Bar` với quan hệ là:  component `FooBar` chứa component `Foo` và component `Foo` chứa component `Bar`.
- Cách quản lý state thông thường:
  - Nếu muốn truyền state từ component `FooBar` xuống cho Bar, thì mình phải gán state của component `FooBar` thành props của component `Foo`, rồi từ `Foo` truyền xuống cho `Bar`.
  Kể cả trong trường hợp component `Foo` không cần sử dụng đến các thuộc tính của state `FooBar`, mà chỉ component Bar cần thôi, thì mình vẫn phải truyền chúng xuống `Foo`. Chúng ta không thể truyền trực tiếp từ `FooBar` xuống `Bar`.
  Ngược lại, khi cần cập nhật state từ `Bar` lên `FooBar`, mình cũng cần phải cập nhật từ `Bar` lên `Foo`, rồi mới cập nhật từ `Foo` lên `FooBar` được.
    
  ![Bạn thấy nó phiền phức khi truyền đi truyền lại props giữa các component như vậy chứ?](https://i.imgur.com/PrqM7He.png)
  
  Bạn thấy nó phiền phức khi truyền đi truyền lại props giữa các component như vậy chứ? :d
- Cách quản lý state với React Context API
  - Ý tưởng của React Context API là nó sẽ tập trung dữ liệu vào một nơi. Sau đó, nó cung cấp cơ chế cho phép mỗi component có thể lấy state cũng như cập nhật state trực tiếp tại đó mà không cần qua các component trung gian.
 
  ![Cách quản lý state với React Context API]( https://i.imgur.com/awu5aPc.png)
  
So với cách quản lý state thông thường, mình thấy cách này có ưu, nhược điểm là:

  - Ưu điểm: hạn chế lặp lại code, hạn chế việc truyền props không cần thiết và quản lý state dễ dàng hơn.
  - Nhược điểm: khó tái sử dụng component vì state tập trung tại một chỗ, tương tự như việc bạn sử dụng biến toàn cục vậy. Như vậy là mình đã tìm hiểu về cơ chế làm việc của React Context API xong rồi. Tiếp theo là các API có thể sử dụng với React Context.
 ### Tìm hiểu các API của React Context
 #### `React.createContext`
 ```javascript
 const MyContext = React.createContext(defaultValue);
 ```
 Đây là API đầu tiên bắt buộc phải sử dụng. API này sẽ tạo mới một object đóng vai trò là Context. Và khi một component đăng ký sử dụng Context này thì nó sẽ đọc giá trị context từ Provider gần nhất. 

Ngược lại, khi bên ngoài nó không có một Provider nào thì giá trị của context sẽ ứng với giá trị defaultValue bên trên.
#### `Context.Provider`
```javascript
<MyContext.Provider value={/* some value */}>
```
Với mỗi một đối tượng Context sẽ tồn tại một đối tượng Provider. Đối tượng này có một property là value. Giá trị của value được hiểu là giá trị của Context. 

Mỗi khi giá trị của value này thay đổi thì các thành phần bên trong Provider này sẽ bị render lại. Vì vậy, giá trị của value sẽ tương ứng là state của chương trình.
#### `Class.contextType`
```javascript
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```
Nếu như hai API trên là để khởi tạo Context và truyền giá trị cho Context thì API này dùng để sử dụng giá trị của Context.

Ở đây, thuộc tính contextType được gán giá trị là MyContext – thành phần được khởi tạo bởi `React.createContext()` bên trên.

Sau đó, mình có thể truy cập đến giá trị của Context thông qua this.context ở bất kỳ phương thức nào thuộc React Lifecycle và cả phương thức render() nữa.
#### `Context.Consumer`
```javascript
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```
Tương tự Class.contextType, API này giúp sử dụng giá trị của Context. Tuy nhiên hơi khác một chút là API này chỉ sử dụng ở phần JSX. 

Trong đó, value chính là giá trị của Context. Và dĩ nhiên, khi bên ngoài component hiện tại không có Provider nào thì value chính là defaultValue.

Trên đây là những thông tin cơ bản về các API của React Context. Tiếp theo, mình sẽ làm một ví dụ cụ thể để demo cho các API này.
### Demo
#### Kịch bản
Mình sẽ làm một ví dụ đơn giản cho phép người dùng thêm hoặc bớt số lượng sản phẩm cần mua trong giỏ hàng. Mình sẽ dùng 3 component mình đã đề cập ở trên,
 component FooBar và Bar sẽ hiển thị số lượng, Foo sẽ chứa button tăng/giảm số lượng khi người dùng click.
#### Bước làm
  - Khởi tạo Context
  ```javascript
  const AppContext = React.createContext();
  ```
  Ở đây mình không sử dụng giá trị defaultValue vì mình luôn sử dụng Provider.
  - Định nghĩa AppProvider
  ```javascript
  class AppProvider extends React.Component {
  state = {
    number: 1,
    inc: () => {
      this.setState({ number: this.state.number + 1 });
    },
    dec: () => {
      this.setState({ number: this.state.number - 1 });
    }
  };

  render() {
    return (
      <AppContext.Provider value={this.state}>
        {this.props.children}
      </AppContext.Provider>
    );
  }
}
```
Như bạn thấy trong đoạn code trên, state bao gồm:

Biến number: dùng để hiển thị ở component FooBar và Bar.

Phương thức inc(): dùng để tăng giá trị biến state number lên 1 đơn vị.

Phương thức dec(): dùng để giảm giá trị biến state number đi 1 đơn vị.

Cuối cùng, bên trong hàm render(), mình truyền giá trị this.state vào value của  AppContext.Provider.
  - Sử dụng AppProvider
  ```javascript
  const App = () => (
  <AppProvider>
    <FooBar />
  </AppProvider>
);

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```
  - Sử dụng contextType với component FooBar
  ```javascript
  class FooBar extends React.Component {
  render() {
    return (
      <div className="red">
        {this.context.number}
        <Foo />
      </div>
    );
  }
}
FooBar.contextType = AppContext;
```
Ở đây, mình sử dụng Class.contextType nên cần khai báo FooBar dạng class component. Và cách sử dụng thì khá đơn giản như mình đã đề cập ở trên.


  - Sử dụng consumer tại component Foo
  ```javascript
  const Foo = () => (
  <div className="blue">
    <AppContext.Consumer>
      {context => (
        <>
          <button onClick={context.dec}>-</button>
          <button onClick={context.inc}>+</button>
        </>
      )}
    </AppContext.Consumer>
    <Bar />
  </div>
);
```
  - Sử dụng consumer tại component Bar
  ```javascript
  class Bar extends React.Component {
  render() {
    return (
      <div className="green">
        <AppContext.Consumer>
          {context => context.number}
        </AppContext.Consumer>
      </div>
    );
  }
}
```
### Kết luận
Như vậy là mình đã giới thiệu với các bạn cách bắt đầu với Context API trong react, đây là kết quả [demo](https://codesandbox.io/embed/patient-rgb-1g9yi), cảm ơn mọi người đã theo dõi bài viết. Xin cảm ơn!!!
