Chắc hẳn 3 methods này `call()`, `apply()` và `bind()` không còn xa lạ gì đối với những bạn chuyên về javascript (nodejs, mongodb). Và quan trọng là nhiều tài liệu cũng nói về 3 methods này, nhưng trường hợp nào sử dụng và sử dụng chúng có tác dụng gì thì chưa thấy tài liệu nào nói rõ. 

Và nếu như các bạn developerjs (devjs) đang đi tìm câu trả lời này thì hy vọng bài này sẽ giúp bạn một phần nào đó hiểu sâu hơn về `call()`, `apply()` và `bind()`. 

![js](https://res.cloudinary.com/dcpvrespg/image/upload/c_scale,w_500/v1577327630/blog/es6/call-apply-bind-in-javascript.jpg)
### Nói về `this` keyword
Vì sao chúng ta nói về `this` keyword. Vì nó có liên quan đến `call()`, `apply()` và `bind()`. Và hầu như thì `this` thì dev nào cũng rõ hơn ai hết rồi. Chỉ nói sơ qua cho những bạn đang mông lung chút thôi. 

Mở console của chrome và Xem xét đoạn code sau:
```js
function anonystick(){
  console.log(this);// xem nó ra cái gì????
}
Output:
Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
```
đó là `this` đó. Khi chúng ta goị function anonystick() thì chúng ta đang get một window Object. `this` chính là refers level cao nhất của window object. Xem hình ảnh console Chrome nè. Nhắc lại, học thao tác trên này luôn nha. 

Bài học này tôi sẽ làm trên Chrome luôn. 

![](https://res.cloudinary.com/dcpvrespg/image/upload/v1565148510/apply%28%29-bind%28%29-call%28%29-in-javascript.png)

Trên đó là nhắc nhẹ về `this` thôi nha. OK nè, đi tiếp vấn đề chính nè. Call, bind & apply methods Vì sao javascript lại sản sinh ra 3 methods này. Chính là vì những trường hợp này? Một trong những trường hợp huyền thoại của các cháu mới ra trường :D 

Ví dụ: Khi click vào button thì hiển thị thông tin Student lên thôi, ez game.
```js
var user = {
 students: {
    name: "Anonystick blog's javascript",
    website: "https://anonystick.com"
 },
  showInfo: function (event) {
     // Hiển thị thông tin student lên
     console.log (this.students.name + " " + this.students.website);
   }
}
 
// Gán hàm thực thi lên sự kiện click chuột vào button
$ ("button").click (user.showInfo);
```
Rồi khi chạy nó sẽ ok không??? Dừng đây suy nghĩ chút đi??? Đúng rồi, đương nhiên bị lỗi rồi. Lúc đó trình duyệt sẽ báo
```js
(index):39 Uncaught TypeError: Cannot read property 'name' of undefined
    at HTMLButtonElement.showInfo ((index):39)
    at HTMLButtonElement.dispatch (jquery.js:5183)
    at HTMLButtonElement.elemData.handle (jquery.js:4991)
```
 Vì name đâu ra?? Khi `this` bây giờ là của button chứ không phải là User nữa. Giờ là hiểu vì sao bị lỗi hoài đúng chưa? Chính vì lý do đó, cho nên 3 methods này ra đời thôi. Nhiệm vụ của nó chính là thay đổi ngữ cảnh của 'this' mà thôi, chứ chẳng có gì cao siêu cả. 

Ở trong Javascript, những hàm này được sử dụng để thiết đặt hoặc thay đổi giá trị của con trỏ `this` theo ý muốn của anh em mình thôi. Sử dung thì nó cũng giống nhau thôi. 

Giờ ta thử sử dụng `call()` cho ví dụ trên nè: 

Thay vì
```js
// Gán hàm thực thi lên sự kiện click chuột vào button
$ ("button").click (user.showInfo);
```
thì sử dụng `call()`
```js
// Gán hàm thực thi lên sự kiện click chuột vào button
$ ("button").click (user.showInfo.call(user));
Output:
Anonystick blog's javascript https://anonystick.com
```
Ngon chưa!!! Sử dụng `bind()`
```js
// Gán hàm thực thi lên sự kiện click chuột vào button
$ ("button").click (user.showInfo.bind(user));
```
`call()`, `apply()` và `bind()` giống nhau về cách sử dụng nhưng khác nhau về cách gọi một chút thôi. 
### Syntax `call()`, `apply()` và `bind()`
Giờ ta lấy ví dụ cụ thể và chạy trên Console của Chrome thử luôn
```ja
var students = {
    name: "Anonystick blog's javascript",
    website: "https://anonystick.com"
}

function Register(course, fees){
 var studentName = this.name;
 console.log("Registered student: " + studentName);
 console.log("Course Name: " + course);
 console.log("Course Fees: " + fees);
}
Ok vậy nha. Nếu gọi console.log(Register('javascript', 99)); 
```

Thử nha 

![](https://res.cloudinary.com/dcpvrespg/image/upload/v1565150035/apply%28%29-bind%28%29-call%28%29-in-javascript-1.png)

Không có gì hết 

- Syntax `call()`

![](https://res.cloudinary.com/dcpvrespg/image/upload/v1565150400/Screen_Shot_2019-08-07_at_10.56.47_AM.png)

- Syntax `apply()` 

Thì cũng giống như `call()` thôi, nhưng khác là truyền các tham số vào là arrays thôi. 

![](https://res.cloudinary.com/dcpvrespg/image/upload/v1565150400/Screen_Shot_2019-08-07_at_10.57.13_AM.png)

- Syntax `bind()` 

Thì hơi khác một chút, với `bind()` thì returns a new function mới sử dụng được. 

![](https://res.cloudinary.com/dcpvrespg/image/upload/v1565150401/Screen_Shot_2019-08-07_at_10.58.00_AM.png)

Đó các bạn có thể thấy tôi sử dụng các gọi khác nhau và thao tác trên console của chrome đó. 

### Kết Luận
Bây giờ các bạn sau khi đọc xong về `call()`, `apply()` và `bind()` thì các bạn có thấy gì lạ không? Và bây giờ các bạn có thể hiểu hơn nhiều rồi chứ, ahihi
