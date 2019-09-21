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
    "Tôi là lập trình viên backend"
  end
  
  def frontend
    "Tôi là lập trình viên frontend"
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
1. p Developer.backend # Tôi là lập trình viên backend
2. p Developer.frontend # undefined method `frontend' for Developer:Class
3. p Developer.new.frontend # Tôi là lập trình viên frontend
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

### Một số method hữu dụng trong Metaprogramming
#### 1. Inspect một object
Trong Ruby ta có thể xem thông tin của một Class hay Object trong khi chương trình đang chạy, chúng ta có thể sử dụng một số method như `class(), instance_methods(), instance_variables()` để làm các việc như vậy:
```ruby
class Developer
  def name
    @dev_name = 'Tham Davies'
    'Tham Davies đẹp trai'
  end
end
obj = Developer.new
puts obj.class # => Developer
puts obj.class.instance_methods(false) # => name
obj.name # => Tham Davies is đẹp trai
puts obj.instance_variables # => @dev_name
```
#### 2. `send()`
`send()` là một instance method của class `Object`. Tham số đầu tiền truyền vào hàm là tên của 1 method. Có thể truyền vào cả `String` hoặc `Symbol`. Tất cả các tham số còn lại đều lần lượt là các tham số truyền vào method đó. Điều đặc biệt là `send()` có thể gọi được private methods:
```ruby
class Greeting
 private
 def welcome(*args)
   "Xin chào " + args.join(' ')
 end
end
obj = Greeting.new
puts(obj.send(:welcome, "Tham", "Davies"))   # => Xin chào Tham Davies
```
#### 3. `define_method`
`Module#define_method()` là một private instance method của class `Module`. Bạn có thể define một instance method trong `receiver` với `define_method()`. Bạn chỉ cần truyền vào tên method và một block, block này sẽ trở thành body cho method đó.
```ruby
class Person
  define_method :name do |arg|
    arg
  end
end
obj = Person.new
puts(obj.name('Thâm')) # => Thâm
```
#### 4. `method_missing`
Khi Ruby ko tìm ra 1 method nhất định thì nó gọi đến method `method_missing()` trên `receiver` gốc. 
```ruby
def method_missing(method_name, *args, &block); end
```
Như đoạn code trên, Method `method_missing()` nhận 3 tham số là tên method cần gọi, tham số và block.
#### 5. `remove_method` và `undef_method`
Để loại bỏ các methods nhất định, bạn có thể sử dụng `remove_method`, nếu có tồn tại một method có cùng tên được khai báo ở các class cha thì các method ở các class cha đều không bị loại bỏ. Còn về  `undef_method` thì ngược lại, nó chặn việc gọi method đó cho dù method có có trùng tên với một method ở các class cha.
```ruby
class Person
  def name
    puts "person"
  end
end

class Student < Person
  def name
    puts "student"
  end
end

obj = Student.new
obj.name # => student

class Student
  remove_method :name  # loại bỏ khỏi Student, nhưng vẫn có trong Person
end
obj.name # => person

class Student
  undef_method :name   # chặn việc gọi method 'name'
end
obj.name # => NoMethodError
```
