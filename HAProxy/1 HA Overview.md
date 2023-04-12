# Mục lục

# HA Proxy - High Availability Proxy

## 1. Mục đích của HA Proxy

Là phần mềm cân bằng tải TCP/HTTP và giải pháp proxy mã nguồn mở phổ biến

Nó thường dùng để cải thiện hiệu suất (performance) và sự tin cậy (reliability) của môi trường máy chủ bằng cách phân tán lưu lượng tải (workload) trên nhiều máy chủ (như web, application, database)

HA Proxy có thể linh hoạt chuyển đổi sang các chế độ `TCP` tương ứng với Layer 4 hoặc `HTTP` tương ứng với Layer 7 bằng cách đặt chế độ của nó trong cấu hình HAProxy

### Load Balancing Layer 4:

Hoạt động ở tầng Transport ở mô hình OSI cung cấp khả năng quản lý lưu lượng truy cập ở lớp giao thức mạng (TCP/UDP), cung cấp lưu lượng truy cập với thông tin mạng hạn chế bằng thuật toán cân bằng tải và bằng cách tính toán máy chủ tốt nhất dựa trên ít kết nối và thời gian phản hồi của máy chủ nhanh nhất

### Load Balancing Layer 7:

Hoạt động cao nhất trong mô hình OSI (Application). Do đó nó đưa qua quyết định định tuyến dựa trên thông tin chi tiết nội dung, dữ liệu cookie, đặc điểm header HTTP/HTTPS, loại dữ liệu,...

Các giao thức DNS, FTP, HTTP, SMTP,.. đều là truy cập ở cấp Application.

### So sánh LB layer 4 vs LB Layer 7:

Bộ cân bằng tải đưa ra quýết định định tuyến dựa trên nguồn và loại thông tin 

## 2. Cài đặt HA Proxy

- Với Centos:
```
sudo yum install -y haproxy
```

- Với Ubuntu:
```
sudo apt install -y haproxy
```

## 3. Tổng qua về cấu trúc file cấu hình HA Proxy

Sau khi cài đặt `haproxy`, ta quản lý các cấu hình cho hệ thống với quản lý nội dung trong file `/etc/haproxy/haproxy.cfg`. 

File có cấu trúc như:

```
# Simple configuration for an HTTP proxy listening on port 80 on all
    # interfaces and forwarding requests to a single backend "servers" with a
    # single server "server1" listening on 127.0.0.1:8000
    global
        daemon
        maxconn 256
    defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms
    frontend http-in
        bind *:80
        default_backend servers
    backend servers
        server server1 127.0.0.1:8000 maxconn 32
    # The same configuration defined with a single listen block. Shorter but
    # less expressive, especially in HTTP mode.
    global
        daemon
        maxconn 256
    defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms
    listen http-in
        bind *:80
        server server1 127.0.0.1:8000 maxconn 32
```

Nội dung file cấu hình HAProxy cơ bản gồm 4 phần chính:
- `global`: Là phần dùng để khai báo các cấu hình dùng chung, các tham số liên quan đến hệ thống và chỉ khai báo 1 lần để dùng cho tất cả các phần khác trong file cấu hình
- `defaults`:
    - Khai báo các cấu hình cùng các thông số mặc định cho tất cả các file cấu hình sau sự xuất hiện khai báo `defaults`.
    - Để thay đổi giá trị của các tham số trong nó ứng với các trường hợp, ta cần viết lại khai báo cấu hình đó trong 1 phần khác của file cấu hình hoặc đưa vào trong 1 phần `defaults` mới 
- `frontend`: Khai báo cấu hình và cách thức các request dẽ được chuyển hướng đến backend. Phần này gồm các thành phần sau:
    - ACLs:
    - Các quy tắc để sử dụng backends phù hợp hoặc sử dụng 1 defaults backend phụ thuộc vào điều kiện ACL.
    - Nó dùng để khai báo danh sách các sockets đang lắng nghe kết nối để cho phép client kết nối tới.
- `backend`: là phần khai báo danh sách các server mà các kết nối client sẽ được chuyển hướng tới đó với các thuật toán kèm theo: round robin, health check, least connection,....
- Ngoài ra còn có các phần khác điển hình như `listen`:
    - Phần này tổ hợp khai báo của frontend và backend. Nó kém linh hoạt hơn khi sử dụng frontend và backend riêng biệt.
    - Nó thể hiện như một cấu hình tĩnh trong cấu hình HAProxy
