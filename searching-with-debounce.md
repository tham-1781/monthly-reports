## Tối ưu api call với Debounce
Khi các bạn sử dụng các ứng dụng (web, mobile, .v.v…), các bạn có để ý bắt gặp phải một input mà khi bạn nhập vào ô tìm kiếm, bạn phải chờ một chút thì mới có gì đó xảy ra không? Đại loại như ví dụ sau:

![debounce](https://cdn-images-1.medium.com/max/1600/1*aNqkqLafqoI9FIHcwubqaA.gif)

Thực sự thì đây không phải là ứng dụng bị chậm hay lỗi gì đâu nhé. Đây gọi là, Debounce, một kỹ thuật được áp dụng rộng rãi để tối ưu hóa performance của ứng dụng bằng cách giảm thiểu số lần ứng dụng phải xử lý những sự kiện (event) mang tính liên tục. Để tiếp nối những kỹ thuật tối ưu hóa cơ bản, thì hôm nay mình sẽ giới thiệu với các bạn kỹ thuật Debounce.
### Debounce là gì?
Debounce là một phương pháp dùng để làm trì hoãn việc xử lý event nào đó cho đến khi không có event nào được emitted (bắn) ra trong một khoản thời gian (delay) đã được định trước. Debounce thường được áp dụng cho: Autocomplete, Typeahead hoặc Filter input kèm với những sự kiện như: keyup và change

### Tại sao lại phải hoãn việc xử lý event lại?
Phần lớn, xử lý những sự kiện từ input thường liên quan đến gọi API (network) hoặc xử lý mảng (array). Nếu như những tác vụ (operation) này tốn nhiều tài nguyên và thời gian của ứng dụng, và mỗi một sự kiện từ input (keyup chẳng hạn) đều bật đèn xanh cho các tác vụ này thì các bạn nghĩ là ứng dụng mình có bị chậm đi không?

Ví dụ: các bạn muốn lọc (filter) các users có chữ: tham trong tên của họ, thì chắc chắn các bạn chỉ muốn ứng dụng mình quan tâm đến chữ tham thôi. Nếu như không có debounce, thì ứng dụng sẽ quan tâm đến cả t, th, tha rồi mới đến tham. Rốt cuộc là mình lãng phí mất 3 lần xử lý không cần thiết rồi. Và Debounce sẽ giúp bạn giải quyết vấn đề này.

Giải thích tới đây chắc các bạn cũng hình dung ra và gật gù cái đầu rồi ha. Move sang demo của mình nhé (ahihi)

### Áp dụng debounce cho chức năng tìm kiếm trong Reactjs
