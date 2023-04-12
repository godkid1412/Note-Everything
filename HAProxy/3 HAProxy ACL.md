# ACL

Có thể kiểm tra các điều kiện khác nhau thông qua ACL và thực hiện một hành động nhất định dựa trên các kiểm tra đó.
- Tìm các chuỗi hoặc mẫu trong các yêu cầu hoặc phản hồi
- Kiểm tra IP
- Kiểm tra trạng thái phản hồi của máy chủ,
- ...

Các hành động có thể thực hiện: quyết định định tuyến, chuyển hướng yêu cầu, trả về static responses,...

# Formatting an ACL

**Name ACL**

    acl is_static path -i -m beg /static/

Bắt đầu với từ khóa `acl`, theo sau là tên của ACL đó và điều kiện

Ở ví dụ trên, tên của ACL là `is_static`

ACL name có thể sử dụng với `if` hoặc `unless` : `use_backend be_static if is_static`

    acl is_static path -i -m beg /static/
    use_backend be_static if is_static

Điều kiện `path -i -m beg /static/`, kiểm tra nếu URL bắt đầu bằng **/static/**

<br>

**anonymous or in-line ACL**

    use_backend be_static if { path -i -m beg /static/ }

Đối với in-line ACL: điều kiện được đặt trong ngoặc nhọn `{}`

Trong trường hợp có nhiều điều kiện. Các điều kiện được viết liền với nhau thì đc hiểu là nối với nhau bằng `AND`. Điều kiện đúng nếu tất cả cùng đúng

    http-request deny if { path -i -m beg /api/ } { src 10.0.0.0/16 }
Chặn các máy  khách có IP trong dải 10.0.0.0/16, bắt đầu với /api/ trong khi vẫn có thể truy cập các path khác

<br>

    http-request deny if { path -i -m beg /api/ } !{ src 10.0.0.0/16 }

Cho phép các máy có IP trong dải 10.0.0.0/16 được phép truy cập path /api/ trong khi các đường dẫn khác bị cấm

<br>

Xác định ACL trong đó có 1 trong 2 điều kiện `true` bằng cách dùng `||`

    http-request deny if { path -i -m beg /evil/ } || { path -i -m end /evil }

Yêu cầu bị từ chối nết đường dẫn bắt đầu bằng **/evil/** (/evil/boo) hoặc kết thúc bằng **/evil/** (/boo/evil/)

# Fetch
Để  kiểm tra điều kiện thì cần có nguồn của thông tin, 

Nguồn thông tin trong HAProxy được gọi là **Fetch**. Điều này cho phép ACL nhận được một phần thông tin để làm việc.

Các fetch mà HAProxy thường được sử dụng:

|Fetch|Description|
|--|--|
|`src`|Trả về IP address của máy client yêu cầu|
|`path`|trả về path mà client yêu cầu|
|`url_param(foo)`|trả về giá trị tham số URL|
|`req.hdr(foo)`|Trả về giá trị của HTTP header |
|`ssl_fc`|Kết quả là true false, đúng nếu kết nối qua SSL|

# Converters

Khi có thông tin từ fetch, ta có thể convert nó. Các bộ convert phân cách bằng dấu `,` thừ fetch hoặc các bộ convert khác nếu có nhiều bộ convert

Một số bộ convert thường dùng:

|Convert|Description|
|--|--|
|`lower`|Thay mẫu thành chữ thường|
|`upper`|Thay mẫu thành chữ hoa|
|`base64`|Mã hóa chuỗi chỉ định|
|`map`|So sánh mẫu với 1 tập tin được chỉ định và xuất giá trị kết quả|

# Flag

Có thể dùng nhiều flags trong 1 ACL

    path -i -m beg -f /etc/hapee/paths_secret.acl

`-i` không phân biệt chữ hoa và thường

`-f` Khớp trên 1 file ACL. File ACL có thể danh sách IP, chuỗi, regEx

`-m` Chỉ định loại đối sánh

# Matching Methods

|Matching Methods|Description|
|--|--|
|`str`|Thực hiện so sánh chính xác|
|`beg`|Kiểm tra phần đầu của chuỗi bằng mẫu,có 1 mẫu: `abcdef` thì khớp với `abc` nhưng không khớp với `def`|
|`end`|Kiểm tra phần cuối của chuỗi bằng mẫu,có 1 mẫu: `abcdef` thì khớp với `def` nhưng không khớp với `abc`|
|`sub`|Kiểm tra khớp với chuỗi con của nó, có chuỗi `abcdef` thì khớp với chuỗi `abc`,`bcd`,`cde`,...|
|`reg`|Chuỗi được so sánh với 1 regular expression|
|`found`||
|`len`|Trả về độ dài của mẫu (mẫu là  `foo` khớp với `-m len 3`)|

