### Một vài mẹo giúp Rails test của bạn chạy nhanh hơn

Với số lượng test case lớn bạn có thể cảm nhận được là việc chạy test case đó nhanh hay chậm. Tốc độ của Rspec test có thể ảnh hưởng bởi những vấn đề như sau:
#### Việc sử dụng before(:each) và before(:all)
`before(:each)` được gọi trong mọi test case. Nếu code trong before(:each) chậm sẽ làm cho mỗi test case sẽ chậm. Những cái nào dùng chung và chỉ cần tạo một lần thì nên đưa vàobefore(:all)
```ruby
before(:each) do
  @article = create(:article)
  @author = create(:author)
end
```
đưa vào `before(:all)`

```ruby
before(:all) do
  @article = create(:article)
  @author = create(:author)
end
```
#### Sử dụng stubs và mock
Trường hợp ko cần thiết phải query vào db nên sử dụng stubs và mock thay vì tạo data thật. Việc này sẽ giúp giảm bớt đi thời gian tạo data.

```ruby
class Student < ActiveRecord::Base
  def name
    first_name +" "+ last_name
  end
end

describe Student do
  let (:student) {create(:student, first_name: 'Red', last_name: 'Panther')}
  it 'should return name' do
    student.name.should == 'Red Panther'
  end
end
```

thay dòng lệnh tạo data bằng `build_stubbed`

```ruby
let(:student) {build_stubbed(:student, first_name: 'Red', last_name: 'Panther')
```
#### Shared variables and tests
Viết những test giống nhau vào trong một file và gọi để sử dụng nhiều chỗ.

vd:

```ruby
# shared_examples.rb
RSpec.shared_examples 'presence validations' do
  fields.each do |field|
    it { is_expected.to validate_inclusion_of(field) }
  end
end

require 'shared_examples.rb'

describe Model do
  let(fields) { %w[id name description <etc...>] }  
  include_examples 'presence validations'
end
```

#### Các quan hệ không cần thiết trong trong factories
Nếu bạn có sử dụng FactoryGirl và trong factory có gọi đến các quan hệ vd:

```ruby
FactoryGirl.define do
  factory :user do
    contact
    location  
  end
end
```
Khi gọi `FactoryGirl.create :user`, contact và location sẽ tự động tạo. Có lúc mình cần sử dụng quan hệ cũng có lúc không cần phải sử dụng nên việc gọi đến quan hệ trong factory có thể ảnh hưởng đến tốc độ test. Để xử lý vấn đề này mình có thể dùng trait

```ruby
FactoryGirl.define do
  factory :user do
    first_name { "John" }
    last_name { "Doe" }

    trait :with_location do
      location
    end
  end
end

FactoryGirl.create :user, :with_location
```

Trên đây là một vài tips cải thiện tốc độ rspec của bạn giúp bạn code tự tin hơn. Enjoy coding <3
