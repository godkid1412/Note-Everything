https://github.com/hocchudong/thuctap032016/blob/master/ThaiPH/OpenStack/ThaiPH_baocaotimhieucacprojecttrongopenstack.md#keystone

Nova-conductor:

Nova-scheduler: là dịch vụ chọn compute node có để chạy instance trên đó

1. Keystone lấy user credential và thực hiện REST call đến Keystone để xác thực
2. Keystone xác minh user credentials, tạo và gửi auth-token (dùng để gửi request đến thành phần khác qua REST-call)
3. CLI gửi request 'launch instance' đến 'nova-api'
4. 'nova-api' nhận request và gửi gửi request xác mình auth-token, quyền truy cập đến 'keystone'
5. Keystone xác nhận token và gửi update auth headers (roles, permissions)
6. 'nova-api' tương tác với database
7. Tạo db entry (bản ghi trong db) cho instance mới
8. nova-api gửi yêu cầu rpc.call đến nova-scheduler để cập nhật host ID cho instance entry
9. nova-scheduler' lấy request từ queue
10. nova-scheduler tương tác với nova-database để tìm host phù hợp qua filtering và weighing
11. update instance entry với host ID sau quá trình filtering và weighing
12. nova-scheduler gửi request rpc.call đến nova-compute để 'launching instance' ở host trên
13. nova-compute lấy request từ queue
14. nova-compute gửi request rpc.call đến nova-conductor để lấy thông tin của instance: host ID, flavor (RAM, CPU, disk)
15. nova-conductor lấy request từ queue
16. nova-conductor lấy thông tin từ nova-database: host ID, flavor,...
17. nova-database trả về các thông tin của instance: host ID, flavor
18. nova-compute lấy thông tin của instance từ queue
19. nova-compute REST call bằng cách chuyển auth-token đến glance-api để lấy Image URI qua Image ID từ glance
20. glance-api đối chiếu auth-token với keystone
21. nova-compute lấy image metadata
22. nova-compute REST call bằng cách chuyển auth-token đến network-api để cấu hình network cho instance và nhận IP address
23. network-api đối chiếu auth-token với keystone
24. nova-compute lấy thông tin network: IP address
25. nova-compute REST call bằng cách chuyển auth-token đến cinder-api để gắn volumes vào instance
26. cinder-api đối chiều auth-token với keystone
27. nova-compute lấy thông tin block storage: volume ID
28. nova-compute tạo dữ liệu cho driver ảo hóa và thực thi request khởi tạo instance thông qua libvirt
