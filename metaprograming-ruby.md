Trong bài viết này mình sẽ giới thiệu cho các bạn từ khóa `self` và các phương thức hay dùng trong Ruby Metaprograming.

### Metaprogramming là gì?
Metaprogramming hiểu đơn giản là "code sinh ra code", đó một kỹ thuật mà bạn có thể  viết một chương trình và 
chương trình này sẽ sinh ra, điều khiển các chương trình khác hoặc làm 1 phần công việc ở thời điểm chương trình đang chạy (runtime).
Chính vì vậy, metaprogramming giúp source của chúng ta trở nên ngắn gọn hơn, giảm thiểu vấn đề trùng lặp,
như các phương thức có chức năng tương tự nhau, dễ dàng thay đổi cũng như chỉnh sửa, giúp hệ thống trở nên gọn nhẹ và linh hoạt hơn.
### Hiểu về `self`
Trước khi đi tìm  hiểu sâu một vấn để, trước tiên chúng ta cần biết những điều cơ bản trước. Nhìn vào ví dụ sau chắc các bạn biết
đoạn code sau đang làm gì:
```ruby
class Developer
  def self.backend
    "I am backend developer"
  end
  
  def frontend
    "I am frontend developer"
  end
end
```
Mình tạo ra 2 phương thức (method) lần lượt là class method và instance method.
Đây là điều mà chúng ta làm hàng ngày nhưng nếu những ai chưa rõ điều xảy ra đằng sau đoạn code này thì chúng ta cần hiểu trước khi tiếp tục nhé.
Trong Ruby, mọi thứ là đối tượng (object), bao gồm class. Vì `Developer` là một instance của lớp `Class`. Và đây là hình ảnh mô hình đối tượng trong ruby:

![alt text](https://uploads.toptal.io/blog/image/129273/toptal-blog-image-1551788860688-4693584ff597cddbe0a9ccdb51d4a2d8.png)

```ruby
p Developer.class # Class
p Class.superclass # Module
p Module.superclass # Object
p Object.superclass # BasicObject
```
Để rõ hơn, chúng ta cần hiểu được từ khóa `self` nó làm gì?
```ruby
1. p Developer.backend # I am backend developer
2. p Developer.frontend # undefined method `frontend' for Developer:Class
3. p Developer.new.frontend # I am frontend developer
4. p Developer.new.backend # undefined method `backend' for #<Developer:0x0000000001cd41e0>
```
Qua ví dụ trên, chúng ta biết rằng 2 method đều nằm trong class Developer, nhưng tại sao dòng 2 và 4 lại bị lỗi? 

Lý do `self` luôn luôn trỏ tới một object, nhưng object đó thay đổi phụ thuộc vào đoạn code đang thực thi.
`self` nằm trong class và class method sẽ trỏ tới class đó, `self` nằm trong instance method sẽ trỏ tới instance của class đó (`ClassName.new` để tạo instance).
Cùng xem thêm ví dụ sau nhé:
```ruby
class Developer
  p self 
end
# Developer
```
`self` ở đoạn code trên trỏ tới class Developer
```ruby
class Developer
  def frontend
    self
  end
end
 
p Developer.new.frontend
# #<Developer:0x2c8a148>
```
`self` trỏ tới instance của class Developer
```ruby
class Developer
  def self.backend
    self
  end
end

p Developer.backend
# Developer
```
`self` ở đoạn code trên trỏ tới class Developer

Tới đây chắc bạn đã hiểu cách hoạt động của `self`. Tiếp theo mình sẽ giới thiệu với các bạn những phương thức thường dùng trong Ruby Metaprograming.

### 

