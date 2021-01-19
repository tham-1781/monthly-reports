### I. Thế nào Load Balancing
Load Balancing hay còn gọi là Cân bằng tải, một kỹ thuật thường được sử dụng để tối ưu hóa việc sử dụng tài nguyên, băng thông, giảm độ trễ, và tăng cường khả năng chịu lỗi.

Khi chúng ta có nhiều hơn một web server, cùng với đó là sự gia tăng lưu lượng truy cập thì việc bổ sung thêm một máy chủ để phân phối lưu lượng này một cách hợp lý là cần thiết. Máy chủ được bổ sung này được gọi là Load balancer.

![](https://www.accuwebhosting.com/blog/wp-content/uploads/2019/06/benefits-of-Load-Balancing.jpg)

### II. Sử dụng NGINX làm load balancer
Trong bài viết này mình sẽ sử dụng 3 server đều được cài đặt nginx, với vai trò như sau:

- Master: Đóng vai trò là Load balancer
- Backend1: Webserver 1
- Backend2: Webserver 2
Cả 2 webserver 1 & 2 đều có chứa source code của một ứng dụng rails và kết nối tới chung một db server.

> Công việc của chúng ta là cấu hình sao cho khi người dùng truy cập vào vhost thì Master server sẽ điều hướng tới một trong hai Backend server, đồng thời không để mất session người dùng.

#### 1. Cấu hình Upstream trên Master
Để bắt đầu sử dụng NGINX với một nhóm các máy chủ web, đầu tiên, bạn cần khai báo upstream group. Directive này được đặt trong http block.
```nginx
http {
    upstream backend {
        server backend1;
        server backend2;
    }
}
```

Ở đây backend1 và backend2 chính là server name của 2 máy chủ web, ta có thể thay bằng địa chỉ IP tương ứng.

Để truyền các request từ người dùng vào một group các server, tên của group được truyền vào với directive `proxy_pass` (hoặc fastcgi_pass, memcached_pass, uwsgi_pass, scgi_pass tùy thuộc vào giao thức).
Trong bài viết này, virtual server chạy trên NGINX sẽ truyền tất cả các request tới backend upstream server:
```nginx
server {
    location / {
        proxy_pass http://backend;
    }
}
```
#### 2. Lựa chọn thuật toán cân bằng tải
Có rất nhiều thuật toán cho mọi người lựa chọn như round-robin, least_conn, least_time, ip_hash, ...

Trong bài viết này mình lựa chọn round-robin: Các request lần lượt được đẩy về 2 server backend1 và backend2 theo tỉ lệ dựa trên server weights, ở đây là 1:1:
```nginx
upstream backend {
    server backend1;
    server backend2;
}
```
#### 3. Bảo toàn session người dùng
Hãy thử tưởng tượng bạn có một ứng dụng yêu cầu đăng nhập, nếu khi đăng nhập, session lưu trên Backend 1, sau một hồi request lại được chuyển tới Backend 2, trạng thái đăng nhập bị mất, hẳn là người dùng sẽ vô cùng nản.
Giải pháp nhanh gọn để giải quyết vấn đề này là mua NGINX bản thương mại, có cung cấp sticky directive, giúp NGINX tracks user sessions và đưa họ tới đúng upstream server.

[https://www.nginx.com/products/nginx/load-balancing/#session-persistence](https://www.nginx.com/products/nginx/load-balancing/#session-persistence)

Mình là người không có tiền nên đành phải đi tham khảo các bài viết trên mạng, tìm được hướng giải quyết như sau:
```nginx
upstream backend {
    server backend1;
    server backend2;
}
map $cookie_backend $sticky_backend {
    backend1 backend1;
    backend2 backend2;
    default backend;
}
server {
    listen 80;
    server_name localhost;
    location / {
        set $target http://$sticky_backend;
        proxy_pass $target;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- Step 1: Khi user lần đầu tiên truy cập vào Master server, lúc đó sẽ ko có backend cookie nào được đưa ra, và dĩ nhiên $sticky_backend NGINX variable sẽ được chuyển hướng tới upstream group. Trong trường hợp này , request sẽ được chuyển tới Backend 1 hoặc Backend 2 theo phương thức round robin.

- Step 2: Trên các webserver Backend 1 và Backend 2, ta cấu hình ghi các cookie tương ứng với mỗi request đến:
```nginx
server {
    listen 80 default_server;
    ...
    location / {
        add_header Set-Cookie "backend=backend1;Max-Age=3600";
        ...
    }
}

server {
    listen 80 default_server;
    ...
    location / {
        add_header Set-Cookie "backend=backend2;Max-Age=3600";
        ...
    }
}
```
> Dễ thấy nếu request được pass vào backend nào, thì trên client của user sẽ ghi một cookie có name=backend & value=backend1 or backend2 tương ứng.

- Step 3: Mỗi khi user request lại tới Master, NGINX sẽ thực hiện map $cookie_backend với $sticky_backend tương ứng và chuyển hướng người dùng vào server đó qua proxy_pass.
Không biết cách này có tốt không, nhưng ở mức độ demo thì vẫn tạm ổn. Nếu chẳng may server tương ứng với cookie lăn ra chết thì vẫn chưa có hướng giải quyết :v 

Hiện tại mình đang set cookie valid trong khoảng thời gian 1h. Nếu qua 1h thì cookie sẽ hết hạn và người dùng có thể bị chuyển qua server khác
#### 4. Health checks
Health checks (Giống kiểm tra sức khỏe vậy) là thực liên tục việc kiểm tra các server trên upstream được khai báo trong config của bạn để tránh việc điều hướng các request của người dùng vào các server không hoạt động. Tóm lại là việc này nhằm đảm bảo người dùng không nhìn thấy các page thông báo lỗi khi chúng ta tắt đi 1 server nào đó, hoặc server nào đó đột xuất bị lỗi không hoạt động.

- `max_fails`: Số lần kết nối không thành công trong một khoảng thời gian nhất định tới backend server. Giá trị mặc định là 0 (disabled heath checks)
- `fail_timeout`: Khoảng thời gian xảy ra số lượng max_fails kết nối không thành công. Giá trị mặc định là 10.
Khi có 1 server backend bị fail, nginx master sẽ điều hướng toàn bộ các traffic sang lần lượt các backend còn lại.
```nginx
upstream backend {
    server backend1 max_fails=3 fail_timeout=10s;
    server backend2 max_fails=3 fail_timeout=10s;
}
```

Quay lại mục [3. Bảo toàn session người dùng] việc điều hướng tới server nào đang nắm giữ session dựa vào $sticky_backend, tuy nhiên nếu $sticky_backend=backend1 mà server backend 1 ra đi thì sao ? lúc này ta buộc phải chuyển hướng các user ở backend 1 sang các server backend còn lại. Ở đây mình trigger event set lại $sticky_backend sang server khác khi có lỗi Gateway Time-out xảy ra trên proxy server.
```nginx
upstream backend {
    server backend1;
    server backend2;
}

server {
    listen 80;
    server_name localhost;
    location / {
        set $target http://$sticky_backend;
        proxy_pass $target;
        ...
        # 504 Gateway Time-out
        error_page 504 = @backend_down;
    }

    location @backend_down {
      proxy_pass http://backend;
    }
}
```
### III. Lời kết
Như vậy mình đã nói suông với các bạn trong phạm vi hiểu biết hạn hẹp của cá nhân :)) Bài tiếp theo sẽ bớt chán nản hơn bài này, mình sẽ thực hành với rails app.
