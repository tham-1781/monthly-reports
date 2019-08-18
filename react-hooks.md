
### React Hooks là gì?
React Hooks là tính năng bổ sung của React ở phiên bản 16.8, nó là những function cho phép bạn sử dụng state và các tính năng khác của React (lifecycle methods, refs, context …) mà không cần sử dụng class component.

### Tại sao React Hooks?
Chắc hẳn không ít bạn đã gặp những trường hợp này trong quá trình làm dự án:
- Cần quản lý state hoặc load data trong quá trình khởi tạo => chuyển function component thành class component hoặc bọc function component bằng một class component.
- Quên bind this => `this.setState()` is not a function
- Component bọc component (wrapper hell)… do trường hợp 1 => khó debug, dễ phát sinh lỗi trong quá trình quản lí state và props.

Nếu bạn đã gặp những trường hợp trên thì React Hooks có thể là câu trả lời cho bạn, cụ thể là useState() và useEffect().
Trong bài này mình chỉ giới thiệu về useState() và useEffect(). Các hooks khác thì các bạn có thể tham khảo thêm [tại đây](https://reactjs.org/docs/hooks-intro.html).
### `useState()`
`useState()` cho phép bạn khai báo state trong function component.
`useState()` sẽ trả về cho chúng ta một array. Tại `array[0]` sẽ là giá trị của biến state và `array[1]` sẽ là hàm tương đương với hàm `this.setState({ tên_biến_state: giá_trị_của_state })`. Params truyền vào `useState()` là giá trị mặc định của state đó.

```javascript
var fruitStateVariable = useState('banana'); // Returns a pair
var fruit = fruitStateVariable[0]; // First item in a pair
var setFruit = fruitStateVariable[1]; // Second item in a pair

// sử dụng array destructuring, ta có
const = [fruit, setFruit] = useState('banana');

// ví dụ khác
const [isLoggedIn, setIsLoggedIn] = useState(false);
const [name, setName] = useState('Tham Davies');
```
Bạn hoàn toàn có thể sử dụng nhiều `useState()` trong một component.  Lưu ý là khi sử dụng hàm set, các bạn vẫn phải đảm bảo state là immutable. `useState()` không tự hỗ trợ các bạn đâu.
### `useEffect()`
`useEffect()` cho phép bạn tạo ra side effect.

Side effect ở đây có thể là các thay đổi về DOM khi chạy (sửa title của trang web, hiển thị thông báo khi resize windows, …), fetch data, sub/unsubscribe … và nhiều thứ khác nữa.

Có thể xem `useEffect()` như một sự kết hợp của `componentDidMount()`, `componentDidUpdate()` và `componentWillUnmount()`.

Cách chạy của `useEffect()` sẽ phụ thuộc vào cách bạn khai báo nó, cụ thể như sau:
```javascript
// xử lý và tạo ra sideEffect, ví dụ như fetch data từ API 
function addSideEffect() {...}
// xóa sideEffect khi component unmount (optional)
function removeSideEffect() {...}
// danh sách các biến "cờ" (flag) để quyết định useEffect có chạy lại hay không (optional)
const watchedVariables = [var1, var2, var3];

// cú pháp đầy đủ của useEffect() hook
useEffect(function() {
  applySideEffect();
  return removeSideEffect;
}, watchedVariables);
```

Như các bạn thấy, `useEffect()` có rất nhiều biến thể. Tùy vào mỗi trường hợp thì `useEffect()` sẽ chạy khác nhau.

Các bạn đặc biệt lưu ý `watchedVariables` vì nó sẽ ảnh hưởng tới cách `useEffect()` chạy. Sau không biết bao nhiêu phốt khi vọc `useEffect()` thì mình xin túm lại như sau:

- Không truyền: `useEffect()` chạy ở `componentDidMount`, `componentDidUpdate` và `componentWillUnmount`. Lưu ý: nếu có `setState()` trong `useEffect()` thì đây sẽ không khác gì vòng lặp vĩnh viễn.
- Truyền array rỗng ( [ ] ): `useEffect()` chỉ chạy ở `componentDidMount`, `componentWillUnmount`. Đa số fetch data dạng list (/users, /posts, …) thì mình dùng cái này
- Truyền vào một hoặc nhiều biến `( [var1, var2] )`: `useEffect()` chạy ở `componentDidMount` và `componentWillUnmount` như bình thường. Tuy nhiên, ở `componentDidUpdate` thì `useEffect()` sẽ kiểm tra các biến được khai báo có thay đổi hay không? Nếu có thì sẽ chạy lại `addSideEffect()`. Fetch data theo id thì mình dùng kiểu này


Bên trên chỉ là một số trường hợp mình gặp phải, rút ra cho các bạn tránh. Tuy nhiên, mình vẫn khuyến khích các bạn đọc docs để hiểu rõ hơn cơ chế hoạt động của `useEffect()`.
### Đề mô nho nhỏ
Mình sẽ làm 1 example fetch user từ api, và show ra chi tiết của user đó. Bắt đầu nào!!!
- Component `App`
```javascript
import React, { useState, useEffect } from "react";
import * as api from "./api";
import UserDetail from "./UserDetail";

const App = props => {
  const [userId, setUserId] = useState(""); // Khai báo biến state là userId và hàm setUserId để gán giá trị vào biết userId
  const [users, setUsers] = useState([]); // Khai báo biến state là users và hàm setUsers để gán danh sách user từ api trả về

  useEffect(() => {
    api
      .getAllUsers()
      .then(json => setUsers(json.data))
      .catch(error => console.log(error));
  }, []);

  return (
    <main className="container">
      <div className="user-list">
        {users.map(user => (
          <div
            key={user.id}
            className={`user ${user.id === userId ? "user--selected" : ""}`}
            onClick={() => setUserId(user.id)}
          >
            <img src={user.avatar} />
            <p>
              {user.first_name} {user.last_name}
            </p>
          </div>
        ))}
      </div>
      <div className="user-detail">
        <h3>Selected User</h3>
        <UserDetail userId={userId} />
      </div>
    </main>
  );
};

export default App;
```
- Ở component `UserDetails`
```javascript
import React, { useState, useEffect } from "react";
import * as api from "./api";

const UserDetail = props => {
  // Hai dòng bên dưới bạn thấy quen chứ :)) 
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    const { userId } = props;
    if (userId) {
      setIsLoading(true);
      api
        .getUserById(userId)
        .then(json => {
          setIsLoading(false);
          setUser(json.data);
        })
        .catch(error => console.log(error));
    } else {
      // do nothing
    }
  }, [props.userId]);

  if (isLoading) return <div>Fetching user...</div>;
  if (!user) return <div>No user selected</div>;

  // has user
  return (
    <div className="user user--selected">
      <img src={user.avatar} />
      <p>
        {user.first_name} {user.last_name}
      </p>
    </div>
  );
};

export default UserDetail;
```
