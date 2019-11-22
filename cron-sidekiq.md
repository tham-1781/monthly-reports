## Sử dụng cron với sidekiq trong rails

### Cron là gì?
Cron là một lệnh Linux dùng để lên kế hoạch cho một nhiệm vụ sẽ được thực thi trong tương lai. Thông thường, lệnh sẽ được sử dụng để lên lịch định kỳ cho một tác vụ
Ví dụ, bạn có 1 trang web về chia sẻ cuộc sống hàng ngày, có tính năng gửi mail cho bạn đọc các bài viết mới nhất theo khung giờ,
thì một trong số cách để làm điều này là sử dụng [cron](https://en.wikipedia.org/wiki/Cron)

Với phần lớn các cron, có ba thành phần như sau:

1. Script (kịch bản lệnh) được gọi hoặc được thực hiện.

2. Command (Câu lệnh) thực thi script trên cơ sở reoccurring. Thao tác này thường được thiết lập trong cPanel.

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
