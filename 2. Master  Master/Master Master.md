## Bước 2: cài đặt group replication

Bước này dùng khi 1 trong 2 database server được thay đổi thì nó đồng bộ sang server còn lại

### Tạo UUID để tìm nhóm ở Mysql\

Tại máy chủ WP1 dùng:
```
wp@good:~$ uuidgen
90a22fa4-80ed-4e0e-8218-988721c98443
```
Lưu mã đó ra chỗ khác rồi dùng sau

### Cài đặt Group Replication trong mysql configuration file

```
sudo vim /etc/mysql/my.cnf
```

Thêm đoạn sau để cài đặt trên 2 server:
```
. . .
[mysqld]
# General replication settings
disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"
gtid_mode = ON
enforce_gtid_consistency = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
binlog_checksum = NONE
log_slave_updates = ON
log_bin = binlog
binlog_format = ROW
transaction_write_set_extraction = XXHASH64
loose-group_replication_bootstrap_group = OFF
loose-group_replication_start_on_boot = OFF
loose-group_replication_ssl_mode = REQUIRED
loose-group_replication_recovery_use_ssl = 1
# Shared replication group configuration
loose-group_replication_group_name = ""
loose-group_replication_ip_whitelist = ""
loose-group_replication_group_seeds = ""
# Single or Multi-primary mode? Uncomment these two lines
# for multi-primary mode, where any host can accept writes
#loose-group_replication_single_primary_mode = OFF
#loose-group_replication_enforce_update_everywhere_checks = ON
# Host specific replication configuration
server_id = 
bind-address = ""
report_host = ""
loose-group_replication_local_address = ""
```

#### Chia sẻ cài đặt group replication trên 2 máy:

loose-group_replication_group_name điền mã vừa sinh ra ở câu lênh `uuidgen`
```
# Shared replication group configuration
 loose-group_replication_group_name = "90a22fa4-80ed-4e0e-8218-988721c98443"
 loose-group_replication_ip_whitelist = "10.0.0.10,10.0.0.20"
 loose-group_replication_group_seeds = "10.0.0.10:33061,10.0.0.20:33061"
```

#### Chọn  cài đặt 1 nhóm chính hay nhóm nhiều chính

```
. . .
# Single or Multi-primary mode? Uncomment these two lines
# for multi-primary mode, where any host can accept writes
#loose-group_replication_single_primary_mode = OFF
#loose-group_replication_enforce_update_everywhere_checks = ON
. . .
```

Nếu cài đặt nhóm 1 chính thì để nguyên, còn nhóm đa chính thì bỏ comment 2 dòng cuối.

Cài đặt phải giống nhau ở từng server


#### Cài đặt cấu hình cho từng server

```
# Host specific replication configuration
server_id = 
bind-address = ""
report_host = ""
loose-group_replication_local_address = ""
```

Ở server 1:
```
# Host specific replication configuration
 server_id = 1
 bind-address = "10.0.0.10"
 report_host = "10.0.0.10"
 loose-group_replication_local_address = "10.0.0.10:33061"
```

Ở server 2
```
# # # Host specific replication configuration
 server_id = 2
 bind-address = "10.0.0.20"
 report_host = "10.0.0.20"
 loose-group_replication_local_address = "10.0.0.20:33061"
```

#### Khởi động lại mysql

```
sudo systemctl restart mysql
```

#### Cài đặt Replication User và bật Group Replication Plugin

Vào mysql trên 2 server dùng:
```
sudo mysql -u root -p
```
Khi bật group replication thì sẽ truyền cho các máy khác qua `bin log` tạo nên xung đột sau khi tạo xong nên cần tắt khi cài đặt.

```
mysql> SET SQL_LOG_BIN=0;
mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'password' REQUIRE SSL;
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
mysql> FLUSH PRIVILEGES;
mysql> SET SQL_LOG_BIN=1;
```

Tiếp theo, đặt `group_replication_recovery` kênh để sử dụng người dùng sao chép mới của bạn và mật khẩu được liên kết của họ. Sau đó, mỗi máy chủ sẽ sử dụng các thông tin đăng nhập này để xác thực với nhóm:
```
CHANGE REPLICATION SOURCE TO SOURCE_USER='repl', SOURCE_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';
```

Nếu sử dụng bản mysql cũ hơn 8.0.23, ta cần sử dụng cú pháp kế thừa của Mysql:
```
CHANGE MASTER TO MASTER_USER='repl', MASTER_PASSWORD='password' FOR CHANNEL 'group_replication_recovery';
```

Cài đặt plugin `group_replication` để khởi tạo nhóm
```
INSTALL PLUGIN group_replication SONAME 'group_replication.so';
```
`group_replication` có thể đổi tên khác theo ý muốn.

```
mysql> SHOW PLUGINS;
+----------------------------+----------+--------------------+----------------------+---------+
| Name                       | Status   | Type               | Library              | License |
+----------------------------+----------+--------------------+----------------------+---------+
|                            |          |                    |                      |         |
| . . .                      | . . .    | . . .              | . . .                | . . .   |
|                            |          |                    |                      |         |
| group_replication          | ACTIVE   | GROUP REPLICATION  | group_replication.so | GPL     |
+----------------------------+----------+--------------------+----------------------+---------+
45 rows in set (0.00 sec)
```

#### Start group_replication

Nếu được đặt biến `group_replication_bootstrap_group` sẽ cho một thành viên biết rằng họ không nên mong đợi nhận thông tin từ các máy chủ và thay vào đó nên thành lập một nhóm mới và tự bầu chọn thành viên chính. Bạn có thể bật biến này bằng lệnh sau:
```
SET GLOBAL group_replication_bootstrap_group=ON;
```

Dùng máy chủ WP1 bắt đầu group_replication
```
mysql> SET GLOBAL group_replication_bootstrap_group=ON;
```

Sau đó, bạn có thể đặt `group_replication_bootstrap_group` trở lại OFF, vì trường hợp duy nhất thích hợp là khi không có thành viên nhóm hiện tại:
```
mysql> SET GLOBAL group_replication_bootstrap_group=OFF;
```

Xem các thành viên trong nhóm:
```
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST   | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 13324ab7-1b01-11e7-9dd1-22b78adaa992 | 10.0.0.10     |        3306 | ONLINE       | PRIMARY     | 8.0.28         | XCom                       |
+---------------------------+--------------------------------------+---------------+-------------+--------------+-------------+----------------+----------------------------+
1 row in set (0.00 sec)
```

Khởi động group_replication trên máy chủ WP2:
```
mysql> START GROUP_REPLICATION;
```

Kiểm tra các thành viên trong nhóm:
```
mysql> SELECT * FROM performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION | MEMBER_COMMUNICATION_STACK |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
| group_replication_applier | 066f21f0-ced0-11ed-8e31-000c29c07655 | 10.0.0.10   |        3306 | ONLINE       | PRIMARY     | 8.0.32         | XCom                       |
| group_replication_applier | ceff2b7e-d200-11ed-b800-000c2965d9cd | 10.0.0.20   |        3306 | ONLINE       | PRIMARY     | 8.0.32         | XCom                       |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+----------------------------+
```

#### Tự động tham gia nhóm khi MySQL khởi động

```
sudo vim /etc/mysql/my.cnf
```

Đặt `loose-group_replication_start_on_boot` thành `ON`
```
[mysqld]
. . .
loose-group_replication_start_on_boot = ON
. . .
```