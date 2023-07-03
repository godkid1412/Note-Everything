https://github.com/hocchudong/thuctap032016/blob/master/ThaiPH/OpenStack/ThaiPH_baocaotimhieucacprojecttrongopenstack.md#keystone

Nova-conductor:

Nova-scheduler: là dịch vụ chọn compute node có để chạy instance trên đó

1.
2. Keystone xác minh user credentials, tạo và gửi auth-token (dùng để gửi request đến thành phần khác qua REST-call)
3. CLI gửi request 'launch instance' đến 'nova-api'
4. 'nova-api' nhận request và gửi gửi request xác mình auth-token, quyền truy cập đến 'keystone'
5. Keystone xác nhận token và gửi update auth headers (roles, permissions)
6. 'nova-api' tương tác với database
7. Tạo initial db cho instance mới
8. nova-api gửi yêu cầu rpc.call đến nova-scheduler
9. nova-scheduler' lấy request từ queue
10. nova-scheduler tương tác với nova-database để tìm host phù hợp qua filtering và weighing
11. update instance entry với host ID sau quá trình filtering và weighing
12. nova-scheduler gửi request rpc.call đến nova-compute để 'launching instance' ở host trên
13. nova-compute lấy request từ queue
14. nova-compute gửi request rpc.call đến nova-conductor để lấy thông tin của instance: host ID, flavor (rRAM,CPU, disk)
15.
