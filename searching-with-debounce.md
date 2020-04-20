## Tối ưu api call với Debounce
Khi các bạn sử dụng các ứng dụng (web, mobile, .v.v…), các bạn có để ý bắt gặp phải một input mà khi bạn nhập vào ô tìm kiếm, bạn phải chờ một chút thì mới có gì đó xảy ra không? Đại loại như ví dụ sau:

![debounce](https://cdn-images-1.medium.com/max/1600/1*aNqkqLafqoI9FIHcwubqaA.gif)

Thực sự thì đây không phải là ứng dụng bị chậm hay lỗi gì đâu nhé. Đây gọi là, Debounce, một kỹ thuật được áp dụng rộng rãi để tối ưu hóa performance của ứng dụng bằng cách giảm thiểu số lần ứng dụng phải xử lý những sự kiện (event) mang tính liên tục. Để tiếp nối những kỹ thuật tối ưu hóa cơ bản, thì hôm nay mình sẽ giới thiệu với các bạn kỹ thuật Debounce.
### Debounce là gì?
Debounce là một phương pháp dùng để làm trì hoãn việc xử lý event nào đó cho đến khi không có event nào được emitted (bắn) ra trong một khoản thời gian (delay) đã được định trước. Debounce thường được áp dụng cho: Autocomplete, Typeahead hoặc Filter input kèm với những sự kiện như: keyup và change

### Tại sao lại phải hoãn việc xử lý event lại?
Phần lớn, xử lý những sự kiện từ input thường liên quan đến gọi API (network) hoặc xử lý mảng (array). Nếu như những tác vụ (operation) này tốn nhiều tài nguyên và thời gian của ứng dụng, và mỗi một sự kiện từ input (keyup chẳng hạn) đều bật đèn xanh cho các tác vụ này thì các bạn nghĩ là ứng dụng mình có bị chậm đi không?

Ví dụ: các bạn muốn lọc (filter) các users có chữ: tham trong tên của họ, thì chắc chắn các bạn chỉ muốn ứng dụng mình quan tâm đến chữ tham thôi. Nếu như không có debounce, thì ứng dụng sẽ quan tâm đến cả t, th, tha rồi mới đến tham. Rốt cuộc là mình lãng phí mất 3 lần xử lý không cần thiết rồi. Và Debounce sẽ giúp bạn giải quyết vấn đề này.

Giải thích tới đây chắc các bạn cũng hình dung ra vấn đề và gật gù cái đầu rồi ha. Move sang demo của mình nhé (ahihi)

### Áp dụng debounce cho chức năng tìm kiếm trong Reactjs
Mình với các bạn sẽ build app với chức năng search có sử dụng kỹ thuật debounce như thế này: có 1 input để user nhập tìm kiếm bài viết, sau khi người dùng ngừng gõ thì mới thực hiện tìm kiếm.

![search-with-debounce](https://i.imgur.com/Y0oM2W6.gif)

Vì bài demo của mình không phức tạp nên mình gom mọi thứ trong 1 file App.js
- thêm input để user nhập từ khóa:
```javascript
  <input
    className="search-term"
    placeholder="Enter post title..."
    onChange={searchTermChange}
    value={searchTerm}
    type="text"
  />
 ```
 - hàm xử lý khi user nhập value vào input
 ```javascript
 const searchTermChange = (event) => {
    setIsTyping(true);
    const { value } = event.target;
    setSearchTerm(value);

    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }

    typingTimeoutRef.current = setTimeout(() => {
      setIsTyping(false);
      filter(value.trim()); // Nếu không sử dụng debounce thì cứ mỗi lần nhập đều gọi tới hàm filter này, nếu hàm filter là api filter từ server thì nó tốn rất nhiều lần query -> server chậm.
    }, 300);
  };
 ```
 Ý tưởng của hàm này là dùng ref để giữ nguyên giá trị sau mỗi lần render. Trong khi mỗi lần user nhập ký tự, mình sẽ setTimeout cho `typingTimeoutRef` và bắt nó delay 300ms (thay đổi tùy ý nhé, ở đây đa số mọi người đều gõ khá nhanh nên mình để 300ms :v), nếu user gõ liên tiếp 2 ký tự < 300ms thì clear timeout trước đó và setTimeout mới với config tương tự, hết 300ms thì mình sẽ thực hiện tìm kiếm.
 
 #### Dưới đây là toàn bộ code
 ```javascript
import React, { useState, useRef } from 'react';
import './App.css';
import { posts } from './data';

function App() {
  const [searchTerm, setSearchTerm] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const [filteredPosts, setFilteredPosts] = useState(posts);
  const typingTimeoutRef = useRef();

  const filter = (value) => {
    if (value === '') {
      return setFilteredPosts(posts);
    }
    setFilteredPosts(posts.filter((p) => p.title.includes(value)));
  };

  const searchTermChange = (event) => {
    setIsTyping(true);
    const { value } = event.target;
    setSearchTerm(value);

    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    }

    typingTimeoutRef.current = setTimeout(() => {
      setIsTyping(false);
      filter(value.trim());
    }, 300);
  };

  return (
    <div className="App">
      <input
        className="search-term"
        placeholder="Enter post title..."
        onChange={searchTermChange}
        value={searchTerm}
        type="text"
      />
      <ul className="post-list">
        {isTyping ? <span>Typing...</span> : null}
        {filteredPosts.length === 0 ? (
          <div>No result for '{searchTerm}'</div>
        ) : (
          filteredPosts.map((post, index) => (
            <li key={post.id}>{post.title}</li>
          ))
        )}
      </ul>
    </div>
  );
}

export default App;
 ```

Bài viết của mình tới đây thôi, với kỹ thuật debounce thì chúng ta tối ưu được lượt query tìm kiếm của api hơn rất nhiều phải không ạ :v. Enjoy coding.
