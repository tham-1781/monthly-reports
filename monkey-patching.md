Ở pull gần đây nhất của tôi bị comment và yêu cầu dùng Monkey Patching methods, tôi có tìm hiểu qua và hôm nay muốn chia sẻ nó với các bạn.

Một trong những điểm mạnh của Ruby là khả năng mở rộng lại bất kỳ class nào và thay đổi các phương thức đó.
Monkey patching là một ví dụ cho điều này, bạn có thể mở rộng class và thay đổi phương thức cho class đấy, 
bao gồm class Ruby như: Array, Hash, String. Chính nhờ khả năng như vậy mà monkey-patching là một trong những kĩ thuật rất hữu ích.

### Ví dụ về Monkey patching
Để làm ví dụ minh họa, tôi sẽ thay đổi và thêm một vài phương thức trong class `String`.

Đầu tiên, tôi thêm một phương thức để tạo ra chuỗi Lorem...:
```ruby
class String  
  def self.lipsum  
    "Lorem ipsum dolor sit amet, consectetur adipiscing elit."  
  end  
end 
```
Ở ví dụ trên, tôi đã mở rộng lớp `String` bằng cách thêm class method mới có tên là lipsum. Sau khi thêm, bạn có thể gọi method này
như gọi class method bình thường `String.lipsum`:
```
String.lipsum  
=> "Lorem ipsum dolor sit amet, consectetur adipiscing elit."  
```
Còn thú vị hơn nữa, chúng ta có thể chỉnh sửa method hiện có trong class `String` này.

```ruby
class String  
  def upcase  
    self.downcase
  end  
end 
```
Ở ví dụ này, tôi đã hijacking một method có sẵn `upcase` và thay đổi hành vi, output của phương thức này bằng cách gọi `downcase`.
```
"THAM davies".upcase # output mong đợi: THAM DAVIES
=> "tham davies" # thực tế :))
```
Như vậy, chúng ta rất dễ dàng thay đổi hoặc thêm phương thức cho class trong Ruby với kỹ thuật này.

### Khi nào thì nên sử dụng monkey-patching?
Câu trả lời là hiếm khi. Ruby cung cấp cho chúng ta rất nhiều powerful tools để làm việc với nó. 
Nhưng công cụ mạnh mẽ chưa chắc đã là công cụ phù hợp đối với yêu cầu nhất. 

Monkey patching nhìn chung là một công cụ cực kì mạnh mẽ, tuy vậy, sẽ rất đau đầu nếu ta không dùng nó đúng cách. 
Những class đã được monkey patch thường khó để hiểu và debug hơn. 

Nếu không cẩn thận, những error message mà bạn nhận được có thể sẽ chẳng cho bạn tí manh mối gì để tìm hiểu điều gì đã xảy ra.

Khi bạn monkey patch 1 method nào đó rất có khả năng nó sẽ làm ảnh hưởng đến những methods khác phụ thuộc vào nó.

Khi bạn thêm những methods mới vào core class, có khả năng bạn đã tạo thêm 1 lỗ hổng cho những trường hợp không thể đoán trước được.
### Vậy khi nào thì ok để monkey patch?
Dù monkey patching tiềm ẩn nhiều nguy cơ nhưng có nghĩa gì khi có trong tay 1 công cụ mạnh mẽ như vậy mà lại không dùng? Vẫn có những trường hợp ta có thể vận dụng nó 1 cách hợp lí.

#### Đặt chúng vào module
Khi monkey patch 1 class, đừng cứ mở class đó ra và đặt methods của ta vào
```ruby
class DateTime
  def weekday?
    !sunday? && !saturday?
  end
end
```
Cách trên có vấn đề gì?
- Nếu trong class `DateTime` đã có method `weekday?` rồi thì nó sẽ bị overwritten và biến mất mãi mãi.
- Sẽ khó để tắt những phần đã được monkey patch, bạn sẽ phải comment hết cái patch đó, hoặc bỏ qua, không require nó nếu muốn chạy code không có nó.
- Nếu như bạn quên `require 'date'` trước khi chạy cái patch này, thì vô tình bạn đã redefine lại class `DateTime` thay vì monkey patch nó. Thay vào đó, chúng ta nên đặt nó vào 1 module, nhờ đó bạn có thể sắp xếp những patch có liên quan với nhau lại 1 nhóm, và khi có lỗi xảy ra, thì rõ ràng là nó đến từ đây.

#### Rails monkey-patching convention
Việc lạm dụng monkey patch sẽ khiến code khó debug và cũng sẽ rất đau đầu cho những new members khi phải đọc source code của bạn. Để giải quyết việc này thì khi monkey patching, chúng ta nên làm theo 1 chuẩn chung, ở đây mình sẽ sử dụng cấu trúc chuẩn của Rails. Ví dụ:
- rails/activesupport/lib/active_support/core_ext/object/blank.rb
- rails/activesupport/lib/active_support/core_ext/object/inclusion.rb

Patch chung của rails là lib/core_extensions/class_name/group.rb, vậy mình sẽ cấu trúc lại ví dụ trên như sau:
```ruby
require 'date'
module CoreExtensions
  module DateTime
    module BusinessDays
      def weekday?
        !sunday? && !saturday?
      end
    end
  end
end

DateTime.include CoreExtensions::DateTime::BusinessDays
```
Patch này sẽ được đặt ở `lib/core_extensions/date_time/business_days.rb`. Vậy là bất cứ new member nào cũng có thể vào đây và xem những thay đổi gì mà ta đã thêm vào Ruby.

### Kết luận
Monkey patching là một công cụ hữu hiệu khi ta monkey patch đúng cách. Nhưng phải cực kì cẩn thận khi sử dụng chúng, dù nó cho ta 1 giải pháp đơn giản nhưng cũng không nên lạm dụng nó.
