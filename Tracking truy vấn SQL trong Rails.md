Để tối ưu phần mềm nói chung và Rails nói riêng, một trong những công việc quan trọng đó là hiểu và tối ưu được các SQL queries vì phần lớn tốc độ web chậm là do các logic xử  lý hoặc truy vấn DB chưa hợp lý.

Thông thường, chúng ta thường tìm ra chúng bằng cách check file log hoặc , nhưng nếu chúng ta muốn phân tích chi tiết một request hoặc nếu chúng ta muốn track nhiều requests cùng 1 lúc, sẽ rất khó để chúng ta tìm và lọc ra các thông tin liên quan trong log file.
Để trả lời cho các câu hỏi đó, Steven đã tạo ra một gem tên là sql_tracker.

Trong bài này, ta sẽ tìm hiểu cách hoạt động của gem sql_tracker và các tính năng của nó để giúp track và phân tích các SQL queries.

### Tracking

Để bắt đầu theo dõi, chỉ cần chạy rails s. sql_tracker sẽ lưu tất cả dữ liệu đã theo dõi vào một hoặc nhiều (các) file .json trong thư mục tmp khi bạn dừng chạy app.

sql_tracker cũng có thể theo dõi các truy vấn sql khi chạy test, nó sẽ lưu tất cả dữ liệu đã theo dõi sau khi tất cả kết thúc việc chạy test.

### Filtering
Ta có thể không muốn lắng nghe tất cả các câu queries. Ví dụ như chỉ muốn track những câu SELECT của các queries, hoặc ta muốn track các queries được gọi với một thư mục nhất định.
Mặc định, sql_tracker tracks và ghi lại tất cả 4 commands (SELECT, INSERT, UPDATE và DELETE), nhưng ta có thể thay đổi bằng việc thay đổi biến settings tracked_sql_command
```rb
SqlTracker::Config.tracked_sql_command = %w(SELECT)
```
Mặc định, sql_tracker tracks thư mục app và lib folders để bỏ qua các queries không liên quan (ví dụ như queries được gọi từ thư mục tests).

### Grouping
sql_tracker sẽ thử đơn giản hoá và gộp các câu queries bằng việc thay thế các giá trị đặc biệt với xxx. Ví dụ, nếu ta thực hiện một câu SQL query sau:
```sql
SELECT users.* FROM users WHERE users.id = 1
```
và sau đó là một câu query khác:
```sql
SELECT users.* FROM users WHERE users.id = 2
```
sql_tracker sẽ gộp chúng thành 1 câu query:
```sql
SELECT users.* FROM users WHERE users.id = xxx
```
sql_tracker sử dụng regular expressions để viết lại các câu SQL queries. Ví dụ như ở đây, nó đã tìm và thay thế các giá trị sau các operators so sánh như =, <>, >, < và cả BETWEEN ... AND ... Sau khi đã nhóm lại, nó sẽ trở nên dễ dàng hơn để phân tích các câu queries, và tính toán tổng số query, thời gian trung bình, …

### Storing
sql_tracker giữ tất cả các data trong bộ nhớ, và exports data trong 1 file JSON khi Rails process kết thúc. Tất cả các data chứa trong 1 Hash, format sẽ có dạng:
```rb
{
  key1: {
    sql: 'SELECT users.* FROM users ...',
    count: 1,
    duration: 0.34,
    source: ['apps/model/user.rb:57:in ...', ...]
  },
  key2: { ... },
  ... ...
}
```
Khi các keys là md5 hoặc các câu sql queries đã được đơn giản hoá, và values là full câu sql query + một số thông tin thống kê.
Mặc định, file sẽ được lưu trong thư mục tmp trong Rails roots folder, ta cũng có thể thay đổi bằng config:
```rb
SqlTracker::Config.output_path = File.join(Rails.root.to_s, 'folder_name')
```
Nếu đã sử dụng app server puma, và set nhiều hơn 1 worker, ta sẽ thấy nhiền hơn 1 JSON output file trong output folder vì mỗi một worker sẽ track và lưu data riêng biệt.

### Reporting
Cuối cùng, chúng ta có thể sinh ra report bằng cách sử dụng 1 hoặc nhiều JSON dump files:
```rb
 sql_tracker tmp/sql_tracker-*.json
```
Report sẽ có dạng:
```rb
==================================
Total Unique SQL Queries: 24
==================================
Count | Avg Time (ms)   | SQL Query                                                                                                 | Source
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
8     | 0.33            | SELECT `users`.* FROM `users` WHERE `users`.`id` = xxx LIMIT 1                                            | app/controllers/users_controller.rb:125:in `create'
      |                 |                                                                                                           | app/controllers/projects_controller.rb:9:in `block in update'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
4     | 0.27            | SELECT `projects`.* FROM `projects` WHERE `projects`.`user_id` = xxx AND `projects`.`id` = xxx LIMIT 1    | app/controllers/projects_controller.rb:4:in `update'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
2     | 0.27            | UPDATE `projects` SET `updated_at` = xxx WHERE `projects`.`id` = xxx                                      | app/controllers/projects_controller.rb:9:in `block in update'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
2     | 1.76            | SELECT projects.* FROM projects WHERE projects.priority BETWEEN xxx AND xxx ORDER BY created_at DESC      | app/controllers/projects_controller.rb:35:in `index'
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
... ...
```
Nó sẽ show cho ta tổng số lần gọi của từng câu truy vấn, thời gian trung bình của từng câu query, và dòng code nào gọi câu query đó. Từ report này, ta có thể hiểu được từng câu query đã được gọi nhưu thế nào, nơi nào gọi nó

### Kết luận
Như vậy bài viết đã tìm hiểu về tracking query trong rails với gem sql_tracker. Hy vọng bài viết giúp ích cho mọi người để có thể debug hoặc cải thiện lại chất lượng của Rails application của mình. Enjoy coding!
