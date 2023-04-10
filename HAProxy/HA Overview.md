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

## 4. Các thuật toán sử dụng trong HAProxy

### Round Robin

![sensors-20-07342-g001](https://user-images.githubusercontent.com/54473576/227494219-505cfc33-a5de-4e6e-a7b2-a8fa4b9de187.png)

### Weight Round Robin

- Là thuật toán mở rộng của RR. Đối với RR, server phải xử lý khối lượng request như nhau. Nếu 1 server có nhiều CPU, RAM hơn, thuật toán RR không thể phân phối nhiều request hơn cho server mạnh hơn đc. Khi đó server có khản năng xử lý thấp có thể bị overload.
- Thuật toán weight RR yêu cầu quản lý cho chỉ định trọng lượng cho mỗi server trên năng lực xử lý
![sensors-20-07342-g002](https://user-images.githubusercontent.com/54473576/227494799-fc67fb0c-5d68-4941-bdd9-ae219219eedb.png)

### Source Hash

- Là thuật toán sử dụng srcIP và desIP của client và server để tạo 1 hash key. Key này để chỉ định máy chủ cho client
- Mỗi key đc sinh ra, nếu session bị hỏng, Source Hash method đảm bảo client đc chỉ đích đến server trước đó
- Điều này rất hữu ích nếu điều quan trọng client kết nối với một phiên vẫn hoạt động sau khi ngắt kết nối và kết nối lại.

### Least Connections

- Kết nối tới máy chủ có ít kết nối nhất.
- Được sử dụng nhiều trong các phiên dài hạn: MariaDB, SQL,...
- Ít được sử dụng trong các phiên ngắn như: HTTP

![sensors-20-07342-g004-550](https://user-images.githubusercontent.com/54473576/227495178-8740881f-90cb-4867-8272-7be9a6937b38.jpg)

### Least Response Time

- Thuật toán dựa trên thời gian đáp ứng của mỗi server. LB sẽ chọn ra server có thời gian đáp ứng nhanh nhất.
- Thời gian đáp ứng tính bằng khoảng thời gian giữa 1 gói tin đến server và thời điểm nó nhận đc gói tin trả lời.Việc gửi và nhận do LB đảm nhiệm
- Thường được sử dụng khi các server ở các vị trí địa lý khác nhau. Người dùng gần server nào thì thời gian ở server đó sẽ nhanh nhất

### Least Bandwidth Algorithm

- LB cấu hình để sử dụng phương thức băng thông ít nhật. CHọn dịch vụ hiện đang phục vụ ít lưu lượng truy cập nhất. Đo bằng Mbps

### Least packet method

- Cấu hình LB để chọn dịch vụ nhận ít packets nhât trong 14s

![Screenshot from 2023-04-03 13-31-02](https://user-images.githubusercontent.com/54473576/229429128-6a784f5f-aeb0-4a54-b9e6-d219bbc161a7.png)
