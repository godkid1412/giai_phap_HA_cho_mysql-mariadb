
# Giới thiệu

**Galera Cluster** là giải pháp tăng tính sẵn sàng cho cách Database bằng các phân phối các thay đổi (đọc - ghi dữ liệu) tới các máy chủ trong Cluster. Trong trường hợp một máy chủ bị lỗi thì các máy chủ khác vẫn sẵn sàng hoạt động phục vụ các yêu cầu từ phía người dùng.

Galera Cluster có 2 chế độ hoạt động là **Active - Passive** và **Active - Active**:

- **Active - Passive**: Tất cả các thao tác ghi sẽ được thực hiện ở máy chủ Active, sau đó sẽ được sao chép sang các máy chủ Passive. Các máy chủ Passive này sẽ sẵn sàng đảm nhiệm vai trò của máy chủ Active khi xảy ra sự cố. Trong một vài trường hợp, **Active - Passive** cho phép SELECT ở các máy chủ Passive.

- **Active - Active**: Thao tác đọc - ghi dữ liệu sẽ diễn ra ở mỗi node. Khi có thay đổi, dữ liệu sẽ được đồng bộ tới tất cả các node

# Chuẩn bị

## Môi trường cài đặt

```
galera@glarea:/etc/mysql$ cat /etc/*rele*
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.2 LTS"
PRETTY_NAME="Ubuntu 22.04.2 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.2 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
...
```

## Cài đặt IP cho các node

```
Node 1: 172.16.217.157/24
Node 2: 172.16.217.158/24
Node 3: 172.16.217.159/24
```

# Các bước tiến hành

## Bước 1: THêm repo cho các máy chủ

```
apt-key adv --keyserver keyserver.ubuntu.com --recv BC19DDBA
apt-add-repository 'deb https://releases.galeracluster.com/galera-VERSION/DIST RELEASE main'
apt-get update
```
Nếu không MariaDB, ta không cần thêm repo sau, chỉ cần khi dùng mysql:
```
apt-add-repository 'deb https://releases.galeracluster.com/mysql-wsrep-VERSION/DIST RELEASE main' 
```

`VERSION` là phiên bản bạn cài đặt galera hoặc wsrep

`DIST`: Thay bằng tên phiên bản Linux. Ví dụ: ubuntu, centos,...

`RELEASE` thay bằng `distribution release`. Ví dụ: focal, jammy,..

Xem thông tin DIST, RELEASE tại phần môi trường cài đặt

## Bước 2: Cài đặt MySQL/MariaDB và Galera trên các máy chủ

### Với MaariaDB:

```
apt-get install -y galera-4 galera-arbitrator-4 mariadb-server mariadb-client rsync
```
Mặc định khi cài đặt xong MariaDB thì root không có mật khẩu

### Với MySQL:

```
apt-get install -y galera-4 galera-arbitrator-4 mysql-wsrep-VERSION rsync
```

`VERSION` thay bằng số phiên bản của `wsrep`

## Bước 3: Cấu hình ở máy chủ thứ nhất

Tạo một file có tên `galera.cnf` trong thư mục `/etc/mysql/conf.d` với nội dung

```
node1@cluster: sudo vim /etc/mysql/conf.d/galera.cnf

[mysqld]
query_cache_size=0
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
query_cache_type=0
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://first_ip,second_ip,third_ip"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="this_node_ip"
```

- Điền địa chỉ của các máy chủ thay thế cho `first_ip,second_ip,third_ip`
- Điền địa chỉ của máy đang cấu hình thay thế cho `this_node_ip`
- Điền tên của node vào this_node_name (Tên tùy chọn)

## Bước 4: Cấu hình trên các node còn lại

Ở các node còn lại, chúng ta copy file galera.cnf ở node thứ nhất vào thư mục /etc/mysql/conf.d/ của 2 node còn lại. Chỉnh sửa nội dung cho phù hợp với node. Cụ thể:

```
. . .
# Galera Node Configuration
wsrep_node_address="this_node_ip"
wsrep_node_name="this_node_name"
. . .
```

## Bước 5: Cấu hình Firewall trên các máy chủ

`Galera` sử dụng 4 port để làm việc

- `3306`: Cho phép các MySQL-Client kết nối đến server
- `4567`: DÙng cho các traffic Galera CLuster Replication. Multicast replicaton dùng cả UDP và TCP trên port này
- `4568`: Dùng cho [Incremental State Transfer (IST)](https://galeracluster.com/library/documentation/state-transfer.html#state-transfer-ist) - cluster đưa dữ liệu bị thiếu đến một node
- `4444`: [State Snapshot Transfer](https://galeracluster.com/library/documentation/glossary.html#term-state-snapshot-transfer) dùng để copy dữ liệu từ một node đến một node khác vừa gia nhập vào cluster

```
ufw enable
ufw allow 22,3306,4567,4568,4444/tcp
ufw allow 4567/udp
```

Khi bật Firewall, hệ thống hỏi có giữ lại phiên SSH hiện tại. Ta nhấn `y` để tiếp tục phiên SSH hiện tại

## Bước 6: Khởi động Cluster

Dùng dịch vụ mysql trên tất cả các node:
```
systemctl stop mysql
```

### Chạy dịch vụ ở node 1

khi dùng MySQL:
```
/etc/init.d/mysql start --wsrep-new-cluster
```

Còn khi dùng MariaDB:
```
galera_new_cluster
```

Kiểm tra số node hiện tại trong cluster ta dùng:
```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```
Kết quả:
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+
```

### Chạy dịch vụ trên node 2 và node 3:

```
sudo systemctl start mysql
```
Sau khi đã chạy mysql trên node 2 và node 3, ta kiểm tra lại số lượng node trên cluster:
```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

Kết quả như sau:
```
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```
> Trong trường hợp tất cả các node bị stop, ta cần chỉ định một node để khởi động lại cluster, xem thêm [tại đây](https://galeracluster.com/library/documentation/crash-recovery.html)

> Để khởi động ta sửa `safe_to_boootstrap:0` thành `safe_to_boootstrap:1` tại `/var/lib/mysql/grastate.dat` Rồi ta khởi động lại cluster bình thường trên node đó bằng `systemctl start mysql`
