## Sử dụng cron với sidekiq trong rails

### Cron là gì?
Cron là một lệnh Linux dùng để lên kế hoạch cho một nhiệm vụ sẽ được thực thi trong tương lai. Thông thường, lệnh sẽ được sử dụng để lên lịch định kỳ cho một tác vụ
Ví dụ, bạn có 1 trang web về chia sẻ cuộc sống hàng ngày, có tính năng gửi mail cho bạn đọc các bài viết mới nhất theo khung giờ,
thì một trong số cách để làm điều này là sử dụng [cron](https://en.wikipedia.org/wiki/Cron)

Với phần lớn các cron, có ba thành phần như sau:

1. Script (kịch bản lệnh) được gọi hoặc được thực hiện.

2. Command (Câu lệnh) thực thi script trên cơ sở reoccurring. Có thể các bạn sẽ thấy ở đâu đó gọi là cron expression

![alt text](https://i.imgur.com/Or2yKVF.png)

3. Các hoạt động hoặc đầu ra/output của script, phụ thuộc vào những gì script được gọi ra và thực thi. Thông thường, các script được gọi là cron job sẽ sửa đổi các tệp hoặc các cơ sở dữ liệu; tuy nhiên chúng cũng có thể thực hiện các tác vụ khác không bao gồm sửa đổi dữ liệu trên máy chủ, như gửi thông báo qua email chẳng hạn.

Hầu hết các script yêu cầu sử dụng cron job sẽ cung cấp các hướng dẫn cụ thể để bạn biết cần phải thiết lập những gì với các ví dụ thường xuyên được đưa ra.

### Sidekiq
[Sidekiq](https://github.com/mperham/sidekiq) là gem giúp chúng ta xử lý tác vụ ngầm (background job)
### Sidekiq-Cron
Sidekiq-Cron nó là tiện ích mở rộng cho sidekiq, là gem giúp chạy các tác vụ linh động và dễ dàng hơn vào các thời điểm đã được chỉ định,
đặc biệt là sử dụng các cú pháp của cron

##### Cách hoạt động
Khi chạy sidekiq, instance của `Sidekiq::Poller` sẽ được gọi để thêm các công việc vào hàng đợi, retryes... 

Cứ sau 10 giây, kiểm tra các công việc mới để lên lịch và không lên lịch cho cùng một công việc nhiều lần khi có nhiều hơn một Sidekiq worker đang chạy.
Job chỉ được thêm vào lịch chạy khi có ít nhất một tiến trình sidekiq đang chạy.

Sidekiq-Cron tự thêm vào quy trình đang chạy và bắt đầu một luồng khác với `Sidekiq::Cron::Poler` kiểm tra tất cả các cron job sidekiq được kích hoạt trong vòng 10 giây, nếu chúng cần được thêm vào hàng đợi (cronline khớp với thời gian kiểm tra).

### Example

1. Tạo app mới `$ rails new cron-sidekiq`

2. Thêm gem vào `Gemfile`:
```gem 'redis'
gem 'sidekiq'
gem 'sidekiq-cron'
```

3. Cài đặt gem `bundle install`

4. Tạo 1 worker cho sidekiq `rails g sidekiq:worker Greeting`

5. Sửa nội dung trong file `app/workers/hard_worker.rb`, ở method `perform()`, ở đây để đơn giản mình sửa thành:
```ruby
class GreetingWorker
  include Sidekiq::Worker

  def perform(*args)
    p 'Hello with love from Cron Job!'
  end
end

```

7. Tạo file `schedule.yml`
```ruby
greeting_schedule:
  cron: "*/1 * * * *" # Cron expression như mình đã để cập ở trên, GreetingWorker sẽ chạy sau mỗi 1 phút
  class: "GreetingWorker" # Worker hoặc Job bạn muốn chạy
  queue: default # Chạy ở queue default
# Nếu bạn có nhiều job cần chạy thì config tương tự
...
```
8. Cập nhật file `sidekiq.rb`

```ruby
Sidekiq.configure_server do |config|
  config.redis = { url: 'redis://localhost:6379/0' }
  # Sidekiq sẽ chạy tác vụ giống với config trong file schedule.yml
  schedule_file = "config/schedule.yml"
  if File.exists?(schedule_file)
    Sidekiq::Cron::Job.load_from_hash YAML.load_file(schedule_file)
  end
end

Sidekiq.configure_client do |config|
  config.redis = { url: 'redis://localhost:6379/0'  }
end

```

9. Chạy sidekiq để thấy kết quả `bundle exec sidekiq`:

![Result](https://i.imgur.com/maQKalF.png)

10. Vài lời về cron expression
### Nguồn tham khảo
- https://github.com/ondrejbartas/sidekiq-cron
- https://en.wikipedia.org/wiki/Cron
- https://www.driftingruby.com/episodes/periodic-tasks-with-sidekiq-cron
### Kết luận

Như vậy là mình đã hướng dẫn xong việc sử dụng cron trong rails với sidekiq. Enjoy coding :)
