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
         1          1 lb01 (local)
         2          1 lb02
         3          1 lb03

## Phương án thực hiện

- Thiết lập policy cho cơ chế quorum (bỏ qua bước này nếu như cluster có nhiều hơn 2 node)

  `pcs property set no-quorum-policy=ignore`

- Cài đặt các resouce : ocf::heartbeat:nfsserver, ocf::heartbeat:filesystem, ocf::heartbeat:exportfs

- Cài đặt các ràng buộc liên quan: các constraint location để chuyển resource khi fail, constraint colocation để các resource được cấu hình trên cùng 1 node

Kiểm tra các trường hợp các xem các resource có tự động chuyển qua node khác khi một resource bị lỗi không? Nếu không thì xem lại các constraint bên trên

- Tạo file backup cho file cấu hình KeepAlive

- Gỡ bỏ VIP ở KeepAlive và thêm resource ocf::heartbeat:IPaddr2 vào trong cluster

## Dự kiến downtime dịch vụ

- Downtime trong thời gian chuyển đổi VIP từ KeepAlive sang Pacemaker

## Phương án rollback khi setup lỗi

Xóa bỏ cluster và restore file backup của cấu hình KeepAlived

## Checklist cần thiết sau khi setup

- VIP có bị active trên > 1 server tại 1 thời điểm hay không
- VIP có auto nhảy sang server khác khi server có VIP lỗi không
- Disk có được mount tự động trên server chứ VIP hay không?
- Các server không cầm VIP có auto unmount disk hay không?
