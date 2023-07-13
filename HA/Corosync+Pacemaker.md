# So sánh KeepAlived với Pacemaker

## KeepAlived

- Tập trung vào cung cấp VIP và dịch vụ cân bằng tải đơn giản, thường được sử dụng để quản lý cụm máy chủ web và cung cấp VIP để truy cập dịch vụ

- Quản lý chủ yếu các tài nguyên mạng như địa chỉ IP ảo và dịch vụ cân bằng tải. Nó tập trung vào việc duy trì kết nối và khả năng truy cập đến dịch vụ

- Có thiết kế đơn giản và dễ triển khai, nhưng không cung cấp các tính năng phức tạp để quản lý và mở rộng các tài nguyên.

## Pacemaker

- Là giải pháp quản lý tài nguyên phức tạp và chủ yếu dùng cho các ứng dụng phân tán và môi trường tài nguyên lớn, cung cấp chuyển đổi tự động các tài nguyên trên nhiều node

- Có khả năng quản lý nhiều loại tài nguyên như dịch vụ ứng dụng, ổ cứng, địa chỉ IP, tài nguyên mạng, và nhiều hơn nữa. Nó cho phép quản lý chi tiết và linh hoạt hơn về cách các tài nguyên được chuyển đổi và phân phối trên các node.

- Cung cấp một hệ thống quản lý tài nguyên mạnh mẽ và khả năng mở rộng. Nó hỗ trợ cấu hình phân tán, các kiến ​​trúc dự phòng phức tạp.

# Chuyển các dịch vụ từ KeepAlived sang Corosync + Pacemaker

## Các bước cần chuẩn bị:

cài đặt Pacemaker+ corosync:

    sudo apt install pacemaker pcs

Thiết lập mật khẩu cho ngường dùng `hacluster`:

    passwd hacluster

Xác thực các node trong cluster:

    pcs host auth <node1> <node3> <node3>,..

Khởi tạo một cluster:

    pcs cluster setup <cluster name> (<node1> <node2> <node3>...) --start

Cho phép cluster khởi động cùng hệ thống:

    pcs cluster enable --all

Kiểm tra trạng thái của corosync:

    pcs status corosync

Kết quả:

    Membership information

---

    Nodeid      Votes Name
         1          1 node1 (local)
         2          1 node2
         3          1 node3

## Phương án thực hiện

- Thiết lập policy cho cơ chế quorum (bỏ qua bước này nếu như cluster có nhiều hơn 2 node)

  `pcs property set no-quorum-policy=ignore`

- Cài đặt các resouce : ocf::heartbeat:nfsserver, ocf::heartbeat:filesystem, ocf::heartbeat:exportfs,

- Cài đặt các ràng buộc liên quan: các constraint location để chuyển resource khi fail, constraint colocation để các resource được cấu hình trên cùng 1 node

Kiểm tra các resource xem có đang chạy trên node đang chứa VIP của keepalive không? Với node1 đang chứa VIP của KeepAlived, ta cho các node còn lại về standby thì các resource sẽ tự động chạy về node1 sau đó cho các node đó chạy lại bình thường:

    pcs node standby node1 node2 && pcs node unstandby node1 node2

Kiểm tra các trường hợp các xem các resource có tự động chuyển qua node khác khi một resource bị lỗi không? Nếu không thì xem lại các constraint bên trên

- Tạo file backup cho file cấu hình KeepAlived
- Ta đứng tại node đang chứa VIP của KeepAlived rồi thêm resource ocf::heartbeat:IPaddr2 để resource đó chạy trên node chứa VIP (node1). Khi đó tại node1 trên cùng 1 interface sẽ có 3 IP là IP mặc định và 2 VIP

Trước khi xóa VIP của KeepAlived, ta tiến hàng đứng tại server NFS Client tiến hành ghi dữ liệu vào thư mục được mount (/home/good/test) với thư mục mà NFS Server chia sẻ (/data) và chạy script:

```
#!/bin/bash

folder="/home/good/test"  # Đường dẫn tới thư mục bạn muốn ghi file vào

for ((i=1; i<=100000; i++))
do
    filename="file${i}.txt"
    filepath="${folder}/${filename}"
    echo "This is file ${i}" > $filepath
    echo "Created file: $filename"
done
```

Lưu đoạn script với đuôi `.sh`(test.sh). Sau đó chạy script với lệnh: `bash test.sh`

- Gỡ bỏ VIP của KeepAlive và kiểm tra lại IP của node1. Khi đó trên interface của node1 có 1 IP mặc định và 1 VIP của pacemaker

## Dự kiến downtime dịch vụ

Sau khi cài đặt xong thì t kiểm tra lại số lượng file trong thư mục /home/good/test bằng lệnh:

    ls -1 | wc -l

Kết quả trả về `100000` khi đó việc đọc ghi được thực hiện liên tục mà không có downtime cho dịch vụ

## Phương án rollback khi setup lỗi

Xóa bỏ cluster và restore file backup của cấu hình KeepAlived

## Checklist cần thiết sau khi setup

- VIP có bị active trên > 1 server tại 1 thời điểm hay không
- VIP có auto nhảy sang server khác khi server có VIP lỗi không
- Disk có được mount tự động trên server chứ VIP hay không?
- Các server không cầm VIP có auto unmount disk hay không?
