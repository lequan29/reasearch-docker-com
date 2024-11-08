# Reasearch Docker

## 1. Docker Image và Container
- Cách tạo Dockerfile để build image, đặt tên file là `Dockerfie`:
  ```ruby
  # Sử dụng image Ruby phiên bản 3.2.1 làm image gốc cho container
  FROM ruby:3.2.1
  
  # Cập nhật danh sách gói và cài đặt các gói cần thiết
  RUN apt-get update && apt-get install -y \
    build-essential \      # Công cụ build cơ bản, cần thiết cho việc biên dịch và cài đặt gems
    nodejs \               # Cài đặt Node.js để xử lý các yêu cầu liên quan đến frontend (nếu có)
    libpq-dev \            # Thư viện PostgreSQL development, cần để kết nối đến PostgreSQL từ Rails
    postgresql-client \    # PostgreSQL client để có thể thao tác với database PostgreSQL từ container
    vim                    # Trình soạn thảo văn bản vim (tiện cho việc chỉnh sửa file trực tiếp trong container, nếu cần)
  
  # Tạo thư mục /app trong container để chứa mã nguồn của ứng dụng
  RUN mkdir -p /app
  # Đặt /app làm thư mục làm việc mặc định trong container
  WORKDIR /app
  
  # Copy Gemfile và Gemfile.lock từ máy chủ vào container
  # Thao tác này giúp cài đặt các gems trước khi copy toàn bộ mã nguồn
  COPY Gemfile Gemfile.lock ./
  
  # Copy toàn bộ mã nguồn từ thư mục hiện tại của máy chủ vào thư mục /app của container
  COPY . ./
  
  # Cài đặt gem sassc với phiên bản 2.4.0 (nếu được sử dụng trong dự án)
  RUN gem install sassc:2.4.0
  # Cài đặt bundler (nếu chưa có) và cài đặt các gems trong Gemfile
  RUN gem install bundler && bundle install --jobs 20 --retry 5
  
  # Mở cổng 3000 để ứng dụng có thể được truy cập từ bên ngoài container
  EXPOSE 3000
  
  # Chạy lại bundle install để đảm bảo rằng tất cả gems đều được cài đặt đầy đủ
  RUN bundle install

- Cách sử dụng lệnh docker build, docker run, docker stop, và docker rm để quản lý containers:
  - docker build:
    - Lệnh docker build dùng để tạo Docker image từ Dockerfile
    ```ruby
    docker build -t <image_name> <path_to_dockerfile>
  - docker run:
    - Lệnh docker run dùng để tạo và khởi động một container từ Docker image
      ```ruby
      docker run [options] <image_name>
  - docker stop:
    - Lệnh docker stop dừng container đang chạy
      ```ruby
      docker stop <container_name_or_id>
  - docker rm:
    - Lệnh docker rm dùng để xóa container đã dừng
      ```ruby
      docker rm <container_name_or_id>
## 2. Quản lý Image và Container
  - Quản lý Image với Docker Hub: Docker Hub là một kho lưu trữ trực tuyến cho Docker images, nơi bạn có thể tìm, lưu trữ, tải về (pull), và tải lên (push) các Docker images. Docker Hub có các images sẵn có do cộng đồng đóng góp, cũng như các images chính thức từ các nhà cung cấp phần mềm lớn.
    - Tải Image: `docker pull <image_name>:<tag>`
    - Đẩy Image: `docker push <username>/<image_name>:<tag>` ví dụ: `docker push myusername/myapp:latest`
    - Hiển thị danh sách Images: `docker images`
    - Hiển thị Containers đang chạy: `docker ps`
    - Kiểm tra thông tin chi tiết của Image/Container: `docker inspect <container_id_or_name> / <image_id_or_name>`
## 3. Docker Compose
 - Docker Compose là một công cụ cho phép bạn định nghĩa và quản lý các ứng dụng Docker sử dụng nhiều container. Thay vì phải khởi chạy từng container bằng các lệnh riêng lẻ, Docker Compose sử dụng một file cấu hình (docker-compose.yml) để thiết lập và điều phối tất cả các container, mạng, và volumes cần thiết cho ứng dụng.
 - Trước tiên bạn phải cài đặt ``Docker Compose`
 - Tạo file và cấu trúc của File `docker-compose.yml`
 - File sẽ có cấu trúc tương tự như này:
    ```ruby
    version: '3'
    services:
      web:
        image: nginx:latest
        ports:
          - "8080:80"
      app:
        build:
          context: .
          dockerfile: Dockerfile
        volumes:
          - .:/app
        ports:
          - "3000:3000"
        environment:
          - DATABASE_URL=(name_sql)://user:password@db:3306/app_db
      db:
        image: mysql:5.7
        environment:
          MYSQL_ROOT_PASSWORD: rootpassword
          MYSQL_DATABASE: app_db
          MYSQL_USER: user
          MYSQL_PASSWORD: password
        volumes:
          - db_data:/var/lib/mysql
    
    volumes:
      db_data:
  - `version`: Phiên bản của file cấu hình Docker Compose.
  - `services`: Định nghĩa các container (dịch vụ) sẽ được khởi chạy.
    - `web`: Một container chạy `nginx`, mở cổng 8080 trên máy chủ và map với cổng 80 của container.
    - `app`: Chứa mã ứng dụng và sử dụng một Dockerfile để xây dựng image.
    - `db`: Chạy container MySQL, với các biến môi trường để cấu hình cơ sở dữ liệu.
  - `volumes`: Định nghĩa volume `db_data` để lưu trữ dữ liệu SQL lâu dài.
 - Các Lệnh Cơ Bản trong Docker Compose: 
   - Khởi chạy dịch vụ: `docker-compose up` (dùng -d để chạy ở chế độ nền).
   - Dừng dịch vụ: `docker-compose down`
   - Xem trạng thái container: `docker-compose ps`
   - Xem log container: `docker-compose logs`
   - Khởi động lại dịch vụ: `docker-compose restart <service_name>`
## 4. Network và Volume trong Docker
- Docker Network: Kết nối và chia sẻ dữ liệu giữa các container hoặc với bên ngoài
- Các loại Network chính trong Docker
  - `Bridge`: Đây là loại network mặc định mà Docker sử dụng khi bạn không chỉ định. Bridge network được sử dụng chủ yếu cho các containers trên cùng một máy host để giao tiếp với nhau.
      Ví dụ: 
      ```
      docker network create my_bridge_network
      docker run -d --name container1 --network my_bridge_network nginx
      docker run -d --name container2 --network my_bridge_network redis
      ```
    - Lúc này, container1 và container2 có thể giao tiếp với nhau qua my_bridge_network
  - `Host`: Loại network này gán trực tiếp container vào network của máy chủ, không sử dụng lớp ảo hóa của Docker. Thường dùng cho các ứng dụng cần hiệu suất cao hoặc cần kết nối trực tiếp với máy chủ.
      - Ví dụ:
      ```
      docker run --network host nginx
      ```
    - Với host network, container sẽ dùng IP của máy chủ và có thể giao tiếp trực tiếp qua cổng của máy chủ
  - `None`: Không có network nào được gán, container sẽ không thể kết nối với bất kỳ container hay mạng nào, thích hợp cho các môi trường an toàn
      - Ví dụ:
      ```
      docker run --network none busybox
      ```
  - `Overlay`: Được sử dụng cho Docker Swarm và Kubernetes, cho phép containers trên các máy chủ khác nhau kết nối với nhau

 - Các Lệnh Quản Lý Network
   - Tạo Network: `docker network create <network_name>`
   - Xóa Network: `docker network rm <network_name>`
   - Liên kết các Network: `docker network ls`
   - Kiểm tra thông tin của Network: `docker network inspect <network_name>`
- Docker Volume: là phương pháp lưu trữ dữ liệu cho containers, giúp duy trì dữ liệu lâu dài và cho phép chia sẻ dữ liệu giữa các containers hoặc với máy chủ
  - Các Loại Lưu Trữ (Storage) trong Docker:
     - `Volume`
     - `Bind Mounts`
     - `Tmpfs Mounts`
  - Sử dụng Docker Volume:
     - Tạo Volume: `docker volume create <volume_name>`
     - Gán Volume vào Container: Khi chạy container, bạn có thể dùng -v để gán volume vào container `docker run -d -v <volume_name>:/path/in/container nginx`
     - Xóa Volume: `docker volume rm <volume_name>`
     - Liệt kê các Volume: `docker volume ls`
     - Kiểm tra thông tin Volume: `docker volume inspect <volume_name>`



  
