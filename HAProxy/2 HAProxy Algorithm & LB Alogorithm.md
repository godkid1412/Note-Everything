# Static LB

## Round Robin

![sensors-20-07342-g001](https://user-images.githubusercontent.com/54473576/227494219-505cfc33-a5de-4e6e-a7b2-a8fa4b9de187.png)

## Weight Round Robin

- Là thuật toán mở rộng của RR. Đối với RR, server phải xử lý khối lượng request như nhau. Nếu 1 server có nhiều CPU, RAM hơn, thuật toán RR không thể phân phối nhiều request hơn cho server mạnh hơn đc. Khi đó server có khản năng xử lý thấp có thể bị overload.
- Thuật toán weight RR yêu cầu quản lý cho chỉ định trọng lượng cho mỗi server trên năng lực xử lý
![sensors-20-07342-g002](https://user-images.githubusercontent.com/54473576/227494799-fc67fb0c-5d68-4941-bdd9-ae219219eedb.png)

## Source Hash

- Là thuật toán sử dụng srcIP và desIP của client và server để tạo 1 hash key. Key này để chỉ định máy chủ cho client
- Mỗi key đc sinh ra, nếu session bị hỏng, Source Hash method đảm bảo client đc chỉ đích đến server trước đó
- Điều này rất hữu ích nếu điều quan trọng client kết nối với một phiên vẫn hoạt động sau khi ngắt kết nối và kết nối lại.


# Dynamic LB

## Least Connections

- Kết nối tới máy chủ có ít kết nối đang hoạt động nhất.
- Được sử dụng nhiều trong các phiên dài hạn: MariaDB, SQL,...
- Ít được sử dụng trong các phiên ngắn như: HTTP

![image](https://user-images.githubusercontent.com/54473576/231328173-27bc8a00-742d-4b17-bb57-b9a68f6fd726.png)

Kết nối xám: Kết nối ko hoạt động
Kết nối vàng: Kết nối vẫn còn hoạt động

### Least Response Time

- Thuật toán dựa trên thời gian đáp ứng của mỗi server. LB sẽ chọn ra server ít kết nối đang hoạt động nhất và thời gian đáp ứng nhanh nhất.
- Thời gian đáp ứng tính bằng khoảng thời gian giữa 1 gói tin đến server và thời điểm nó nhận đc gói tin trả lời.Việc gửi và nhận do LB đảm nhiệm
- Thường được sử dụng khi các server ở các vị trí địa lý khác nhau. Người dùng gần server nào thì thời gian ở server đó sẽ nhanh nhất

![image](https://user-images.githubusercontent.com/54473576/231331346-3a160fe1-97b4-4b00-afd6-77f3514d1063.png)

Kết nối 1, 2, 3, 4, 5, 6: các kết nối ban đầu
Kết nối 7, 8: đến server C do C có ít kết nối đang hoạt động nhất
Kết nối 9: đến server C vì giữa B và C có cùng số kết nối đang hoạt động và C có TTFB bé nhất

### Least Bandwidth Algorithm

- LB cấu hình để sử dụng phương thức băng thông ít nhật. CHọn dịch vụ hiện đang phục vụ ít lưu lượng truy cập nhất. Đo bằng Mbps

![image](https://user-images.githubusercontent.com/54473576/231325619-45cef4bf-fb9e-44a4-87de-99791b5c1ae2.png)

### Least packet method

- Cấu hình LB để chọn dịch vụ nhận ít packets nhât trong 14s

![Screenshot from 2023-04-03 13-31-02](https://user-images.githubusercontent.com/54473576/229429128-6a784f5f-aeb0-4a54-b9e6-d219bbc161a7.png)
