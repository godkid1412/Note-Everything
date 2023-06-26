# Cài đặt openstack trên ubuntu20

## Các bước cài đặt:

**1. Cài đăt ban đầu trên cả 2 node controller và compute**

1.1. update các gói phần mềm và package cơ bản:

      # apt update -y

1.2. setup network, đặt ip tĩnh # vi /etc/netplan/00-cloud-init.yaml

    	network:

ethernets:
eth0:
addresses: - 10.20.1.52/24
eth1:
addresses: - 10.3.22.40/24
gateway4: 172.16.6.1
nameservers:
addresses: [ 172.16.6.0, 8.8.8.8 ]
version: 2
1.3, Khai báo hosts

        # vim /etc/hosts

         10.20.1.52 controller

         10.20.1.177 compute

_khởi động lại và test ping_

**2. Cài đặt openstack**

2.1. Cài đặt package openstack trên tất cả các node

```
apt-get install -y software-properties-common
add-apt-repository cloud-archive:yoga
apt -y upgrade
apt install python3-openstackclient
```

2.2. Cài đặt NTP server ( đồng bộ thời gian ) ta sử dụng chrony

    2.2.1. Cài đặt chrony server

    apt -y install chrony

**sau đó ta set timezone về HoChiMinh**

    timedatectl set-timezone Asia/Ho_Chi_Minh

sau đó ta sửa file cấu hình trên controller node

    server 3.vn.pool.ntp.org iburst

    server 3.asia.pool.ntp.org iburs
    server 0.asia.pool.ntp.org iburst

sau đó ta add các dải của compute và storage vào để đồng bộ

    allow 10.3.22.0/24

khởi động lại chrony

    service chrony restart;
    service chrony status

iểm tra xem thời gian đã được đồng bộ chưa

    chronyc souces

    	210 Number of sources = 1
    MS Name/IP address         Stratum Poll Reach LastRx Last sample
    ===============================================================================
    ^* time.cloudflare.com           3   8   377    81   -325us[ -323us] +/-   78ms

2.2.2. Cài đặt chrony trên compute node

- Các bước cài đặt giống với trên controller node, chỉ khác trong file cấu hình

Sửa file cáu hình /etc/chrony.conf

    server controller iburst

Sau đó ta khởi đôngj lại chrony và kiểm tra

**2.3. SQL database for Ubuntu**

Hầu hết các dịch vụ của OpenStack sử dụng cơ sở dữ liệu SQL để lưu thông tin. DB thường sẽ chạy trên node controller. Các dịch vụ OpenStack cũng hỗ trợ các cơ sở dữ liệu SQL khác bao gồm PostgreSQL.

**Cài đặt**

    apt install -y mariadb-server python3-pymysql mariadb-client

**Tạo và chỉnh sửa file /etc/mysql/mariadb.conf.d/99-openstack.cnf:**

    [mysqld]
    bind-address = 10.20.1.51
    default-storage-engine = innodb
    innodb_file_per_table = on
    max_connections = 4096
    collation-server = utf8_general_ci
    character-set-server = utf8

Sau đó ta khởi động lại mysql:

    service mysql restart

**2.4. Message queue for Ubuntu**

OpenStack sử dụng hàng đợi tin nhắn để phối hợp các hoạt động và thông tin trạng thái giữa các dịch vụ. Dịch vụ hàng đợi tin nhắn thường chạy trên nút điều khiển. OpenStack hỗ trợ một số dịch vụ hàng đợi tin nhắn bao gồm RabbitMQ, Qpid và ZeroMQ. Tuy nhiên, hầu hết các bản phân phối đóng gói OpenStack đều hỗ trợ một dịch vụ hàng đợi tin nhắn cụ thể:

Cài đặt và cấu hình các thành phần

Cài đặt gói:

    apt install -y rabbitmq-server

khởi động dịch vụ:

    service rabbitmq-server start

Khai báo plugin cho rabbitmq:

    rabbitmq-plugins enable rabbitmq_management

Cấu hình trang quản lý rabbitmq trên UI :

    curl -O http://localhost:15672/cli/rabbitmqadmin
    chmod a+x rabbitmqadmin
    mv rabbitmqadmin /usr/sbin/

Thêm openstackngười dùng:
rabbitmqctl add_user openstack 1234

> Creating user "openstack" ...

Cho phép truy cập cấu hình, ghi và đọc cho openstackngười dùng:

    rabbitmqctl set_permissions {username} ".*" ".*" ".*"

> Setting permissions for user "{username}" in vhost "/" ...

    rabbitmqctl set_user_tags openstack administrator

Hiển thị danh sách user rabbitmq :

    rabbitmqadmin list users

> Truy cap Rabbitmq: http://10.3.22.40:15672

**2.5. Memcached**

Cơ chế xác thực dịch vụ Danh tính cho các dịch vụ sử dụng Memcached để lưu trữ mã thông báo. Dịch vụ memcached thường chạy trên nút điều khiển.

Cài đặt và cấu hình các thành phần

Cài đặt các gói:

    apt install memcached python3-memcache

Chỉnh sửa file /etc/memcached.conf

    -l 10.20.1.52

Sau đó, khởi động lại và kiểm tra service này
service memcached restart;
service memcaches status

**2.6. Etcd for Ubuntu**

Các dịch vụ OpenStack có thể sử dụng Etcd, một kho lưu trữ khóa-giá trị đáng tin cậy được phân phối để khóa khóa phân tán, lưu trữ cấu hình, theo dõi hoạt động của dịch vụ và các tình huống khác.

_Dịch vụ Etcd chạy trên controller node_

Cài đặt và cấu hình các thành phần

    apt install -y etcd

Ghi chú

    Kể từ Ubuntu 18.04, etcdgói này không còn có sẵn từ kho lưu trữ mặc định. Để cài đặt thành công, kích hoạt Universekho lưu trữ trên Ubuntu.

Chỉnh sửa /etc/default/etcd tệp

    ETCD_NAME="controller"
    ETCD_DATA_DIR="/var/lib/etcd"
    ETCD_INITIAL_CLUSTER_STATE="new"
    ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
    ETCD_INITIAL_CLUSTER="controller=http://10.20.1.52:2380"
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.20.1.52:2380"
    ETCD_ADVERTISE_CLIENT_URLS="http://10.20.1.52:2379"
    ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
    ETCD_LISTEN_CLIENT_URLS="http://10.20.1.52:2379"

**Lưu ý: ip ở đây là ip của controller.**

Hoàn tất cài đặt
Kích hoạt và khởi động lại dịch vụ etcd:

    systemctl enable etcd
    systemctl restart etcd

**3. Cài đặt keystone**

Trước khi cài đặt và định cấu hình dịch vụ Danh tính, bạn phải tạo cơ sở dữ liệu:

    mysql -u root -p1234

Tạo keystonecơ sở dữ liệu:

    CREATE DATABASE keystone;

Cấp quyền truy cập thích hợp vào keystonecơ sở dữ liệu:

    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '1234';

    GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '1234';

    FLUSH PRIVILEGES;
    exit;

Sau đó, ta cài đặt keystone:

    apt install -y keystone

chỉnh sửa /etc/keystone/keystone.conf tệp và hoàn tất các thao tác sau:

    [database]
    connection = mysql+pymysql://keystone:1234@controller/keystone
    [token]
    provider = fernet

Sau đó, ta phân lại quyền cho keystone:

    chown root:keystone /etc/keystone/keystone.conf

sau đó, Đồng bộ để sync database cho Keystone

    su -s /bin/sh -c "keystone-manage db_sync" keystone

Sinh các file cho fernet :

    keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

Sau khi chạy 2 lệnh trên, thư mục /etc/keystone/fernet-keys sẽ được sinh ra và chứa các file key của fernet

Thiết lập bootstrap cho Keystone :

    keystone-manage bootstrap --bootstrap-password 1234 \
    --bootstrap-admin-url http://controller:5000/v3/ \
    --bootstrap-internal-url http://controller:5000/v3/ \
    --bootstrap-public-url http://controller:5000/v3/ \
    --bootstrap-region-id RegionOne

Keystone sẽ sử dụng httpd để chạy service, các request vào keystone sẽ thông qua apache2. Do vậy cần cấu hình apache2 để keystone sử dụng. Sửa dòng 95 trong file cấu hình /etc/apache2/apache2.conf của dịch vụ apache2 :

    vim /etc/apache2/apache2.conf

Thêm dòng: ServerName controller

**Sau đó, ta khởi động lại dịch vụ apache2**

sau đó, Tạo file biến môi trường cho Keystone :

    cat <<EOT >> admin-openrc
    export OS_PROJECT_DOMAIN_NAME=Default
    export OS_USER_DOMAIN_NAME=Default
    export OS_PROJECT_NAME=admin
    export OS_USERNAME=admin
    export OS_PASSWORD=1234
    export OS_AUTH_URL=http://controller:5000/v3
    export OS_IDENTITY_API_VERSION=3
    export OS_IMAGE_API_VERSION=2
    EOT

sau đó Thực thi biến môi trường để sử dụng được CLI của OpenStack:

    source admin-openrc

Kiểm tra lại hoạt động của Keystone:

    openstack token issue

Khai báo user demo, project demo :

    openstack project create --domain default --description "Service Project" service;
    openstack project create --domain default --description "Demo Project" demo;
    openstack user create demo --domain default --password 1234;
    openstack role create user;
    openstack role add --project demo --user demo user;

**4. Cài đặt và cấu hình Glance trên node controller**

Tạo Database, user và phân quyền cho glance :

# mysql

    CREATE DATABASE glance;
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '1234';
    FLUSH PRIVILEGES;
    exit;

B2 : Thực thi biến môi trường :

    source admin-openrc

Tạo user, project cho glance :

    openstack user create glance --domain default --password 1234;
    openstack role add --project service --user glance admin;
    openstack service create --name glance --description "OpenStack Image" image;
    openstack endpoint create --region RegionOne image public http://controller:9292;
    openstack endpoint create --region RegionOne image internal http://controller:9292;
    openstack endpoint create --region RegionOne image admin http://controller:9292;

Cài đặt glance và các package cần thiết :

    apt install -y glance

**Cấu hình Glance :** Ta vim /etc/glance/glance-api.conf

    [database]
    connection = mysql+pymysql://glance:1234@controller/glance
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = glance
    password = 1234
    [paste_deploy]
    flavor = keystone
    [glance_store]
    stores = file,http
    default_store = file
    filesystem_store_datadir = /var/lib/glance/images/

Đồng bộ để sync database cho Glance :

    su -s /bin/sh -c "glance-manage db_sync" glance

Khởi động dịch vụ glance :

    service glance-api restart
    service glance-api status

Tải image và import vào glance :

    wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
    openstack image create "ubuntu" --fit --file bionic-server-cloudimg-amd64.img --disk-format qcow2 --container-format bare --public

    Đây là một test image của OpenStack. Download các image khác tại đây


    https://docs.openstack.org/image-guide/obtain-images.html

Kiểm tra lại xem image đã được up hay chưa :

    openstack image list

**5. Cài đặt và cấu hình Placement trên node controller**

Tạo Database, user và phân quyền cho placement :

    mysql -u root -p1234


    CREATE DATABASE placement;
    GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY '1234';
    FLUSH PRIVILEGES;
    exit;

Thực thi biến môi trường để sử dụng được CLI của OpenStack:

    source admin-openrc

Tạo service, gán quyền, endpoint cho placement :

    openstack user create placement --domain default --password 1234;
    openstack role add --project service --user placement admin;
    openstack service create --name placement --description "Placement API" placement;
    openstack endpoint create --region RegionOne placement public http://controller:8778;
    openstack endpoint create --region RegionOne placement internal http://controller:8778;
    openstack endpoint create --region RegionOne placement admin http://controller:8778;

Cài đặt placement :

    apt install -y placement-api

Sao lưu file cấu hình của placement :

    cp /etc/placement/placement.conf /etc/placement/placement.conf.bak

Cấu hình placement :

Ta sửa file: vim /etc/placement/placement.conf

    [placement_database]
    connection = mysql+pymysql://placement:1234@controller/placement
    [api]
    auth_strategy = keystone
    [keystone_authtoken]
    auth_url = http://controller:5000/v3
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = placement
    password = 1234

Tạo các bảng, đồng bộ dữ liệu cho placement :

    su -s /bin/sh -c "placement-manage db sync" placement

Khởi động lại httpd :

    service apache2 restart

**2.10.Cài đặt Nova**

2.10.1 Cài đặt Nova trên node controller

Tạo các database, user, mật khẩu cho service nova :

mysql -u root -pWelcome123

    CREATE DATABASE nova_api;
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '1234';
    CREATE DATABASE nova;
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '1234';
    CREATE DATABASE nova_cell0;
    GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '1234';
    FLUSH PRIVILEGES;
    exit

Thực thi biến môi trường để sử dụng được CLI của OpenStack :

    source admin-openrc

Tạo endpoint cho nova :

    openstack user create nova --domain default --password 1234;
    openstack role add --project service --user nova admin;
    openstack service create --name nova --description "OpenStack Compute" compute;
    openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1;
    openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1;
    openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1;

Cài đặt nova và các package đi kèm :

    apt install -y nova-api nova-conductor nova-novncproxy nova-scheduler

Sao lưu file cấu hình của nova :

    cp /etc/nova/nova.conf /etc/nova/nova.conf.bak

Cấu hình Nova :

    [DEFAULT]
    my_ip = 10.20.1.52
    use_neutron = true
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    enabled_apis = osapi_compute,metadata
    transport_url = rabbit://openstack:1234@controller:5672/
    [api_database]
    connection = mysql+pymysql://nova:1234@controller/nova_api
    [database]
    connection = mysql+pymysql://nova:1234@controller/nova
    [api]
    auth_strategy = keystone
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000/
    auth_url = http://controller:5000/
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = 1234
    [vnc]
    enabled = true
    server_listen = $my_ip
    server_proxyclient_address = $my_ip
    [glance]
    api_servers = http://controller:9292
    [oslo_concurrency]
    lock_path = /var/lib/nova/tmp
    [placement]
    region_name = RegionOne
    project_domain_name = default
    project_name = service
    auth_type = password
    user_domain_name = default
    auth_url = http://controller:5000/v3
    username = placement
    password = 1234
    [scheduler]
    discover_hosts_in_cells_interval = 300
    [neutron]
    url = http://controller:9696
    auth_url = http://controller:5000
    region_name = RegionOne
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = 1234
    service_metadata_proxy = True
    metadata_proxy_shared_secret = 1234

Thực hiện lệnh để sinh bảng cho Nova :

    su -s /bin/sh -c "nova-manage api_db sync" nova
    su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
    su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
    su -s /bin/sh -c "nova-manage db sync" nova

Kiểm tra lại xem cell0 đã được đăng ký chưa :

    su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova

Kích hoạt và khởi động các dịch vụ của Nova :

    service nova-api restart;
    service nova-scheduler restart;
    service nova-conductor restart;
    service nova-novncproxy restart;

Kiểm tra lại xem dịch vụ của nova đã hoạt động chưa :

    openstack compute service list

**10.2.Cài đặt Nova trên các node compute**

Cài đặt các gói của nova :

    apt install -y nova-compute

Sao lưu file cấu hình của Nova :

    cp /etc/nova/nova.conf /etc/nova/nova.conf.bak

Cấu hình nova :

    [DEFAULT]
    enabled_apis = osapi_compute,metadata
    transport_url = rabbit://openstack:1234@controller
    my_ip = 10.3.6.140
    use_neutron = true
    firewall_driver = nova.virt.firewall.NoopFirewallDriver
    [api_database]
    connection = mysql+pymysql://nova:1234@controller/nova_api
    [database]
    connection = mysql+pymysql://nova:1234@controller/nova
    [api]
    auth_strategy = keystone
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000/
    auth_url = http://controller:5000/
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = 1234
    [vnc]
    enabled = true
    server_listen = 0.0.0.0
    server_proxyclient_address = $my_ip
    novncproxy_base_url = http://controller:6080/vnc_auto.html
    [glance]
    api_servers = http://controller:9292
    [oslo_concurrency]
    lock_path = /var/lib/nova/tmp
    [placement]
    region_name = RegionOne
    project_domain_name = default
    project_name = service
    auth_type = password
    user_domain_name = default
    auth_url = http://controller:5000/v3
    username = placement
    password = 1234

Sau đó, ta chỉnh sửa file /etc/nova/nova-compute.conf:

    vim /etc/nova/nova-compute.conf

    [libvirt]
    virt_type = qemu
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000/
    auth_url = http://controller:5000/
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = nova
    password = 1234

Khởi động Nova :
service nova-compute start
service nova-compute status

**2.10.3. Thêm các node compute vào hệ thống (trên node controller)**

Kiểm tra các node compute đã up hay chưa :

    source admin-openrc
    openstack compute service list --service nova-compute

Add các note compute vào cell :

    su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

\*\*2.11. Cài đặt neutron

2.11.1. Cài đặt neutron trên controller\*\*

Tạo DATABASE cho NEUTRON

mysql -u root -p1234

    CREATE DATABASE neutron;
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '1234';
    FLUSH PRIVILEGES;
    exit;

Tạo project, user, endpoint cho Neutron :

    source admin-openrc
    openstack user create neutron --domain default --password 1234;
    openstack role add --project service --user neutron admin;
    openstack service create --name neutron --description "OpenStack Networking" network;
    openstack endpoint create --region RegionOne network public http://controller:9696;
    openstack endpoint create --region RegionOne network internal http://controller:9696;
    openstack endpoint create --region RegionOne network admin http://controller:9696;

Cài đặt Neutron :

    apt install neutron-server neutron-plugin-ml2 \
    neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
    neutron-metadata-agent

Sao lưu các file cấu hình của Neutron :

    cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
    cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak
    cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
    cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak

B5 : Cấu hình file /etc/neutron/neutron.conf :

    [DEFAULT]
    core_plugin = ml2
    service_plugins = router
    transport_url = rabbit://openstack:1234@controller
    auth_strategy = keystone
    notify_nova_on_port_status_changes = True
    notify_nova_on_port_data_changes = True
    [database]
    connection = mysql+pymysql://neutron:1234@controller/neutron
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = 1234
    [nova]
    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    region_name = RegionOne
    project_name = service
    username = nova
    password = 1234
    [oslo_concurrency]
    lock_path = /var/lib/neutron/tmp

Sửa file cấu hình /etc/neutron/plugins/ml2/ml2_conf.ini :

    [ml2]
    type_drivers = flat,vlan,vxlan
    tenant_network_types = vxlan
    mechanism_drivers = linuxbridge
    extension_drivers = port_security
    [ml2_type_flat]
    flat_networks = provider
    vni_ranges = 1:1000
    [securitygroup]
    enable_ipset = True

Sửa file cấu hình /etc/neutron/plugins/ml2/linuxbridge_agent.ini :

    [linux_bridge]
    physical_interface_mappings = provider:eth1
    [vxlan]
    enable_vxlan = True
    local_ip = 10.20.1.52
    [securitygroup]
    enable_security_group = True
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDrive

Khai báo sysctl :

    echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
    echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
    modprobe br_netfilter
    /sbin/sysctl -p

**Sửa file cấu hình /etc/neutron/l3_agent.ini :**

    [DEFAULT]
    interface_driver = linuxbridge

**Sửa file cấu hình /etc/neutron/dhcp_agent.ini :**

    [DEFAULT]
    interface_driver = linuxbridge
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    enable_isolated_metadata = true

**Sửa file cấu hình vi /etc/neutron/metadata_agent.ini :**

    [DEFAULT]
    nova_metadata_host = controller
    metadata_proxy_shared_secret = 1234

Thiết lập database :

    # su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
    --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

Khởi động dịch vụ neutron :

    service nova-api restart
    service neutron-server restart
    service neutron-linuxbridge-agent restart
    service neutron-dhcp-agent restart
    service neutron-metadata-agent restart
    service neutron-l3-agent restart

Kiểm tra lại trạng thái dịch vụ :

    openstack network agent list

**2.11.2 Cài đặt Neutron trên các node compute**
Cai dat goi:

    apt install -y neutron-server neutron-plugin-ml2 \

neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent \
 neutron-metadata-agent

Khai báo bổ sung cho Nova : vi /etc/nova/nova.conf

    [neutron]
    url = http://controller:9696
    auth_url = http://controller:5000
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = 1234

Cài đặt Neutron :

    apt install -y neutron-linuxbridge-agent

Sao lưu các file cấu hình của Neutron :

    cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
    cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
    cp /etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/linuxbridge_agent.ini.bak
    cp /etc/neutron/dhcp_agent.ini /etc/neutron/dhcp_agent.ini.bak
    cp /etc/neutron/metadata_agent.ini /etc/neutron/metadata_agent.ini.bak

Sửa file cấu hình /etc/neutron/neutron.conf :

    # [DEFAULT]
    auth_strategy = keystone
    core_plugin = ml2
    transport_url = rabbit://openstack:1234@controller
    notify_nova_on_port_status_changes = true
    notify_nova_on_port_data_changes = true
    # [keystone_authtoken]
    www_authenticate_uri = http://controller:5000
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = default
    user_domain_name = default
    project_name = service
    username = neutron
    password = 1234
    # [oslo_concurrency]
    lock_path = /var/lib/neutron/tmp

Khai báo sysctl :

    echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.conf
    echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.conf
    modprobe br_netfilter
    /sbin/sysctl -p

Sửa file cấu hình /etc/neutron/plugins/ml2/linuxbridge_agent.ini :

    #[linux_bridge]
    physical_interface_mappings = provider:eth1
    #[vxlan]
    enable_vxlan = True
    local_ip = $(ip addr show dev eth0 scope global | grep "inet " | sed -e 's#.*inet ##g' -e 's#/.*##g')
    l2_population = True
    #[securitygroup]
    enable_security_group = True
    firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

Khai báo trong file /etc/neutron/metadata_agent.ini

    [DEFAULT]
    nova_metadata_host = 10.20.1.52
    metadata_proxy_shared_secret = 1234

Khai báo cho file /etc/neutron/dhcp_agent.ini :

    # [DEFAULT]
    interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
    enable_isolated_metadata = True
    dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
    force_metadata = True

Khởi động Neutron :

    service neutron-linuxbridge-agent restart

Khởi động lại dịch vụ openstack-nova-compute :

    ervice nova-compute restart

**Tham khảo thêm cài đặt neutron tại https://docs.openstack.org/neutron/queens/install/controller-install-ubuntu.html **

**2.12. Cài đặt và cấu hình Horizon trên node controller**

Cài đặt openstack-dashboard :

    apt install -y openstack-dashboard

Sao lưu file /etc/openstack-dashboard/local_settings :

    cp /etc/openstack-dashboard/local_settings /etc/openstack-dashboard/local_settings.bak

Chỉnh sửa file /etc/openstack-dashboard/local_settings :

Chỉnh sửa 1 số dòng sau :

    		ALLOWED_HOSTS = ['*']

    		CACHES = {
    		    'default': {
    			'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
    			'LOCATION': 'controller:11211',
    		    }
    		}

    		SESSION_ENGINE = 'django.contrib.sessions.backends.file'

    		OPENSTACK_HOST = "controller"

    		OPENSTACK_NEUTRON_NETWORK = {
    		    ...
    		    'enable_distributed_router': False,
    		    'enable_fip_topology_check': False,
    		    'enable_ha_router': False,
    		    'enable_quotas': False,
    		    ...
    		}

    		TIME_ZONE = "Asia/Ho_Chi_Minh"

    Thêm các dòng sau vào cuối file :

    		OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
    		OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "default"
    		OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
    		OPENSTACK_API_VERSIONS = {
    		    "identity": 3,
    		    "image": 2,
    		    "volume": 3,
    		}
     Khởi động lại dịch vụ :

    		# systemctl reload apache2.service

     Truy cập đường dẫn sau trên trình duyệt để vào dashboard. Đăng nhập bằng tài khoản admin/ 1234
     vừa tạo ở trên: http://IP_CONTROLLER/horizon

# Tạo network và VM
