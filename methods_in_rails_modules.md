## Methods in Rails modules
Module là một tập các lớp, phương thức, hằng số do đó module cũng giống như các class. Chỉ khác là module không thể tạo các đối tượng và không thể kế thừa.
Module là một trong những công cụ mạnh nhất của ruby on rails, nó được sử dụng như namespace của class trong ruby. Thường thì chúng ta sẽ gộp các lớp, phương thức và hằng số có liên quan đến nhau vào một module để tránh xung đột tên.

Trong ví dụ dưới đây, chúng ta có thuộc tính `status` trong class `Order`, nó chứa các giá trị như `pending, confirmed, cancelled,...`

```rb
class Order < ApplicationRecord

  module Status
    PENDING = 'Pending'
    CONFIRMED = 'Confirmed'
    CANCELLED = 'Cancelled'
    DECLINED = 'Declined'
    COMPLETED = 'Completed'
  end

  validates_inclusion_of :status, in: [Status::PENDING, Status::CONFIRMED, Status::CANCELLED, Status::DECLINED, Status::COMPLETED]
end
```

Mục đích của dòng này `validates_inclusion_of` là để validate tất cả giá trị status của đơn hàng phải nằm trong list đã liệt kê trong `in:`, nếu không sẽ thông báo giá trị không hợp lệ.
Nhưng mà khoan, chúng ta đang lặp đống code với `Status::`, vậy có cách nào để avoid và làm cho nó dễ nhìn hơn không? chắc chắn là có rồi =)).

Thêm 1 class method `all` trong module `Status` và sử dụng biến constants để lấy tất cả constants từ module. Cách làm đơn giản như sau:
```rb
class Order < ApplicationRecord

  module Status
    PENDING = 'Pending'
    CONFIRMED = 'Confirmed'
    CANCELLED = 'Cancelled'
    DECLINED = 'Declined'
    COMPLETED = 'Completed'

    def self.all
      constants.map {|const| const_get(const)}
    end
  end

  validates_inclusion_of :status, in: Status.all
end
```

Giờ đây các bạn nhìn lại dòng `validates_inclusion_of` gọn gàng đẹp đẽ hơn rồi đúng không! Đội ơn [const_get](https://apidock.com/ruby/Module/const_get)

![magic](https://i.pinimg.com/originals/d8/20/48/d820481ef14b1cd2a16ff4e7660deb5f.gif)

Tôi thêm một method mới tên là `active?`, nó sẽ kiểm tra trạng thái đơn hàng đang hoạt đọng hay không khi trạng thái của đơn hàng đang là `Confirmed` hoặc `Confirmed`.

```rb
def active?
  status.in?([Status::PENDING, Status::CONFIRMED])
end
```

Nguyên module sẽ như thế này:
```rb
class Order < ApplicationRecord

  module Status
    PENDING = 'Pending'
    CONFIRMED = 'Confirmed'
    CANCELLED = 'Cancelled'
    DECLINED = 'Declined'
    COMPLETED = 'Completed'

    def self.all
      constants.map {|const| const_get(const)}
    end

    def self.active
      [Status::PENDING, Status::CONFIRMED]
    end
  end

  validates_inclusion_of :status, in: Status.all

  def active?
    status.in?(Status.active)
  end
end
```

Hy vọng tips này sẽ được bạn áp dụng vào việc viết code gọn hơn. Happy coding!!!
