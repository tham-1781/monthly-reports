Dạo gần đây mình đọc blogs rất thường xuyên nghe kể đến Docker mà không biết nó là cái chi. Một hồi google thì biết nó cũng có số má trong giới lập trình và được nhiều developer sử dụng cho các dự án lớn nhỏ. Docker nổi lên cùng với xu hướng Microservice đang thịnh hành, nó là gì và nó lợi hại ra sao, hãy theo dõi bài viết của mình:

<!-- TOC -->

  - [Tổng Quan Docker](#tổng-quan-về-docker)
    - [Docker là gì](#docker-là-gì)
    - [Thực trạng](#thực-trạng)
    - [Sự khác biệt giữa Docker và Máy ảo(Virtual Machine)](#sự-khác-biệt-giữa-docker-và-máy-ảo(virtual-machine))
    - [Docker dùng để làm gì và khi nào cần đến nó](#docker-dùng-để-làm-gì-và-khi-nào-cần-đến-nó)
    - [Các lệnh docker cần nhớ](#các-lệnh-docker-cần-nhớ)

<!-- /TOC -->

### Tổng quan về Docker
#### Docker là gì
Docker là một công cụ tạo môi trường được đóng gói (container) trên máy tính, chạy độc lập, không làm ảnh hưởng tới môi trường hiện tại của máy tính.

![docker](https://miro.medium.com/max/750/0*0KmPAP1TNK_C9aNF)

Slogan của docker là "Docker — Build, Ship, and Run Any App, Anywhere" nhấn mạnh tính di động của nền tảng này. Người dùng (lập trình viên, DevOps,…) có thể deploy hoặc chuyển ứng dụng từ máy này sang máy khác, chạy ngay lập tức chỉ qua vài câu lệnh với kết quả y hệt như ở môi trường máy cũ.

#### Thực trạng:
Anh `A` và `B` cùng làm việc trên một dự án có sử dụng rails, postgres, redis, nginx và nodejs (vì dùng Reactjs :)), và mỗi người đều có vai trò riêng. Sau khi cả 2 thảo luận về requirement các thứ, anh `A` xung phong tạo project và cài tất cả yêu cầu trên vào chiếc Macbook mới mua và xịn xò của anh ta. Mọi thứ có vẻ chạy ổn trên máy của anh ta và push project lên github. Anh `B` khi được thông báo init project xong thì lên trên repo của anh `A` và đọc qua `README.md` thì có cả đống thứ cần phải setup, mỗi thứ lại phải kèm theo version bao nhiêu, một tá thư viện, vân vân và mây mây.

![docker](https://miro.medium.com/max/320/0*84HnAy4JvMeUGkcE)

Anh `B` lại đi google để tìm, cài cắm, setup giống như anh `A`. Cài chưa xong đã conflic tùm lum, cái nọ xung đột cái kia chẳng hạn, lại còn ảnh hưởng tới những chương trình cũ đã cài đặt trong máy nữa chứ, thôi cài lại luôn cả hệ điều hành cho máy.

![docker](https://miro.medium.com/max/775/0*YyVOtsJzxcXQShQk)

Đó chính là lúc bạn nên nghĩ tới Docker, mọi thứ sẽ đơn giản hơn nhiều.
Với Docker bạn không cần phải lo lắng về sự tương thích giữa phần cứng, phần mềm nữa.

### Sự khác biệt giữa Docker và Máy ảo(Virtual Machine)

| Docker        | Máy ảo        |
| ------------- |:-------------:|
| Ít tốn tài nguyên      | Tốn tài nguyên |
| Linh hoạt      | Kém linh hoạt      |
| Khởi động nhanh | Tùy cấu hình, lâu hơn docker      |
| Chạy script khởi tạo | Không chạy script khởi tạo     |
| Đồng bộ môi trường | Không đồng bộ môi trường      |

![docker](https://miro.medium.com/max/646/1*lMdWI7swnZbbcurffv8nKg.png)

### Khi nào cần dùng đến Docker?
- Đồng nhất môi trường.
- Làm việc nhóm (lập trình).
- Deploy lên server.
- Kiểm thử phần mềm trên nhiều phiên bản thư viện, framework.
