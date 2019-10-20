#### Giới thiệu
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
Trong bài này mình sẽ giới thiệu cho các bạn 
