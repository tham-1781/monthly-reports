Dạo gần đây mình đọc blogs rất thường xuyên nghe kể đến Docker mà không biết nó là cái chi. Một hồi google thì biết nó cũng có số má trong giới lập trình và được nhiều developer sử dụng cho các dự án lớn nhỏ. Docker nổi lên cùng với xu hướng Microservice đang thịnh hành, nó là gì và nó lợi hại ra sao, hãy theo dõi bài viết của mình:

<!-- TOC -->

  - [Tổng Quan Docker](#tổng-quan-về-docker)
    - [Docker là gì](#docker-là-gì)
    - [Thực trạng](#thực-trạng)
    - [Sự khác biệt giữa Docker và Máy ảo(Virtual Machine)](#sự-khác-biệt-giữa-docker-và-máy-ảovirtual-machine)
    - [Docker dùng để làm gì và khi nào cần đến nó](#khi-nào-cần-dùng-đến-docker)
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

#### Sự khác biệt giữa Docker và Máy ảo(Virtual Machine)

| Docker        | Máy ảo        |
| ------------- |:-------------:|
| Ít tốn tài nguyên      | Tốn tài nguyên |
| Linh hoạt      | Kém linh hoạt      |
| Khởi động nhanh | Tùy cấu hình, lâu hơn docker      |
| Chạy script khởi tạo | Không chạy script khởi tạo     |
| Đồng bộ môi trường | Không đồng bộ môi trường      |

![docker](https://miro.medium.com/max/646/1*lMdWI7swnZbbcurffv8nKg.png)

#### Khi nào cần dùng đến Docker?
- Đồng nhất môi trường.
- Làm việc nhóm (lập trình).
- Deploy lên server.
- Kiểm thử phần mềm trên nhiều phiên bản thư viện, framework.


#### Các lệnh Docker cần nhớ
Pull images từ Docker hub: `sudo docker pull image_name`

Tạo mới container bằng cách chạy image, kèm theo các tùy chọn: `docker run -v <thư mục trên máy tính>:<thư mục trong container> -it <image name> /bin/bash`

Tham số `-v` để link một thư mục trong docker container ra thư mục ở máy host: `docker run -v /abc:/abc -p 8080:80 -it ubuntu`

Tham số `-p` để link 1 port trong docker container ra thư mục ngoài máy host (ex:` 8080:80` link port 8080 trong docker ra port 80 máy vật lý.

```ruby
docker images # Liệt kê các docker image.
docker ps || docker ps -a # Liệt kê các docker container đang chạy, thêm tham số -a để hiển thị tất cả các docker container
docker start <ID || NAME> # Chạy một docker container đã tắt
docker rm -f <ID hoặc NAME> # Xoá một docker container bằng tên hoặc id.
docker rmi -f <ID || NAME> # Xoá một docker image bằng tên hoặc id.
docker build -t <Tên Images> # Build một docker image từ các định nghĩa trong dockerfile ( phải cd vào đúng nơi lưu docker file)
```

### Các thành phần và khái niệm chính
- **Docker Engine**: là thành phần chính của Docker, điều khiển và tạo DockerFile, Docker Images..

- **Docker Hub**: tương tự như Github nhưng dành cho chia sẻ DockerFile, Docker Images. Ở đây có những DockerFile, Docker Images của người dùng cũng như những bản chính thức từ các nhà phát triển lớn như Google, Oracle, Microsoft…

- **Dockerfile**: Là một tập hợp các lệnh để yêu cầu và định nghĩa, cho Docker Engine biết phải tạo Docker Image như thế nào.

- **Docker images**: Được tạo nên từ các định nghĩa ở dockerfile, không thể sửa chữa image này, chỉ có thể update dockerfile và tạo ra docker image khác (hiểu đơn giản, dockerfile giống như file `.exe` để các bạn cài đặt phần mềm, dockerfile như source code)

- **Docker container**: hoạt động như một thư mục có chứa tất cả những thứ cần thiết để một ứng dụng có thể chạy được. Mỗi một docker container được tạo ra từ một docker image.
- **Docker registries**: Là kho chứa images. Người dùng có thể tạo ra các images của mình và tải lên đây hoặc tải về các images được chia sẻ

![docker](https://miro.medium.com/max/1000/0*d_ncfI0BrkN3M6ef.png)

### Cài đặt

- Docker hỗ trợ nhiều nền tảng hệ điều hành khác nhau bao gồm Linux, Windows và cả Mac. Ngoài ra, Docker còn hỗ trợ nhiều dịch vụ điện toán đám mây nổi tiếng như Microsoft Azure hay Amazon Web Services.

- Lưu ý là ban đầu nó được xây dựng trên nền tảng Linux. Vì Docker cần can thiệp vào phần lõi, nhân Kernel trong khí đó Linux là mã nguồn mở, gọi là cần gì có nấy.

- Đến khi thấy Docker hay quá, bác Windows ngỏ lời, thế là công ty Docker và công ty Microsoft hợp tác với nhau nhưng nghe chừng chưa khả quan lắm bởi vì nhân Windows có nhưng thứ không public được ( bản quyền mà ).

- Cho tới hiện tại khi cài Docker trên Windows hay Mac thì Docker sẽ cài một máy ảo Linux trên máy thật và Docker hoạt động dựa trên máy ảo Linux đó. Còn tương lai về sau thì ai biết được.

- Docker có 2 phiên bản, CE( dành cho nhà phát triển, nhóm nhỏ, coder như chúng ta) và EE (dành cho doanh nghiệp).

Dưới đây mình sẽ giới thiệu cài đặt đối với bản CE trên Ubuntu thông qua repository như sau:
```ruby
I. Chuẩn bị:

# Update the apt package index:
$ sudo apt-get update

# Install packages to allow apt to use a repository over HTTPS:
$ sudo apt-get install \
   apt-transport-https \
   ca-certificates \
   curl \
   software-properties-common

# Add Docker’s official GPG key:
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -


# Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint.
$ sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
     Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

# Use the following command to set up the stable repository
$ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

II. Cài đặt docker CE:

## Update the apt package index:
$ sudo apt-get update

## Install the latest version of Docker CE, or go to the next step to install a specific version. Any existing installation of Docker is replaced.
$ sudo apt-get install docker-ce

## Verify that Docker CE is installed correctly by running the hello-world image.
$ sudo docker run hello-world
```
![docker](https://github.com/tham-1781/monthly-reports/blob/master/Screenshot%20from%202020-02-21%2011-39-15.png?raw=true)

Nếu màn hình hiện như hình là bạn đã cài docker thành công rồi đấy (dinhcuadinh)

