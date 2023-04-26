# Cấu hình MySQL Replication Master - Slave

MySQL replication là một tiến trình cho phép sao chép dữ liệu của MySQL một cách tự động từ máy chủ Master sang máy chủ Slave. Nó vô cùng hữu ích cho việc backup dữ liệu hoặc sử dụng để phân tích mà không cần thiết phải truy vấn trực tiếp tới CSDL chính hay đơn giản là để mở rộng mô hình.

Bài lab này sẽ thực hiện với mô hình 2 máy chủ: 2 máy chủ master sẽ gửi dữ liệu để đồng bộ với nhau. Để thực hiện, trong ví dụ này sử dụng 2 IP:

- IP 1: 10.10.10.1
- IP 2: 10.10.10.2

## **1. Cấu hình Master 1 - Slave 2**

### Cấu hình trên máy chủ Master 1

Tạm dừng dịch vụ MySQL

> sudo systemctl stop mysqld

#### Khai báo cấu hình cho Master

Thêm các dòng sau vào file cấu hình `/etc/my.cnf`

```
[mysqld]
...
bind-address=10.10.10.1
log-bin=/var/lib/mysql/mysql-bin
server-id=101
```

- `bind-address`: Cho phép dịch vụ lắng nghe trên IP. Mặc định là 127.0.0.1 - localhost
- `log-bin`: Thư mục chứa log binary của MySQL, dữ liệu mà Slave lấy về thực thi công việc replicate.
- `server-id`: Số định danh Server

#### Khởi động dịch vụ MySQL

> systemctl start mysqld

Đăng nhập vào MySQL, tạo một user sử dụng trong quá trình replication

> mysql -u root -p
```
mysql> grant replication slave on *.* to slave_user@'10.10.10.2' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
```
Bằng câu lệnh trên, ta đã tạo 1 user có tên `slave_user` với mật khẩu là `password`, user được phép truy cập từ server có IP `là 10.10.10.2`

```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      592 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

> **Chú ý**: Ghi nhớ thông tin này để khai báo khi cấu hình trên Slave


### Cấu hình máy chủ Slave 2

Thêm các dòng sau vào file cấu hình `my.cnf` trên máy chủ Slave. Mục đích là định danh máy chủ slave và chỉ ra nơi lưu trữ bin-log.

```
[mysqld]
...
log-bin=/var/lib/mysql/mysql-bin
server-id=102
```

Khởi động lại MySQL

> systemctl restart mysqld

Sau khi xong, đăng nhập vào MySQL để cấu hình Repilcate Master Slave
> mysql -u root -p
```
mysql> change master to
    -> master_host='10.10.10.1',
    -> master_user='replica',
    -> master_password='password',
    -> master_log_file='mysql-bin.000001',
    -> master_log_pos=592;
 mysql> start slave;
 ```

**Chú ý**: Điền thông tin `log_file` và `log_pos` trùng khớp với thông số mà ta đã lấy ở bước trên

## **2. Cấu hình Master 2 - Slave 1**

### Cấu hình trên máy chủ Master 2

Tạm dừng dịch vụ MySQL

> sudo systemctl stop mysqld

#### Khai báo cấu hình cho Master

Thêm các dòng sau vào file cấu hình `/etc/my.cnf`

```
[mysqld]
...
bind-address=10.10.10.2
log-bin=/var/lib/mysql/mysql-bin
server-id=102
```

- `bind-address`: Cho phép dịch vụ lắng nghe trên IP. Mặc định là 127.0.0.1 - localhost
- `log-bin`: Thư mục chứa log binary của MySQL, dữ liệu mà Slave lấy về thực thi công việc replicate.
- `server-id`: Số định danh Server

#### Khởi động dịch vụ MySQL

> systemctl start mysqld

Đăng nhập vào MySQL, tạo một user sử dụng trong quá trình replication

> mysql -u root -p
```
mysql> grant replication slave on *.* to slave_user@'10.10.10.1' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
mysql> flush privileges;
```
Bằng câu lệnh trên, ta đã tạo 1 user có tên `slave_user` với mật khẩu là `password`, user được phép truy cập từ server có IP `là 10.10.10.1`

```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |      600 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

> **Chú ý**: Ghi nhớ thông tin này để khai báo khi cấu hình trên Slave


### Cấu hình máy chủ Slave 1

Thêm các dòng sau vào file cấu hình `my.cnf` trên máy chủ Slave. Mục đích là định danh máy chủ slave và chỉ ra nơi lưu trữ bin-log.

```
[mysqld]
...
log-bin=/var/lib/mysql/mysql-bin
server-id=101
```

Khởi động lại MySQL

> systemctl restart mysqld

Sau khi xong, đăng nhập vào MySQL để cấu hình Repilcate Master Slave
> mysql -u root -p
```
mysql> change master to
    -> master_host='10.10.10.2',
    -> master_user='replica',
    -> master_password='password',
    -> master_log_file='mysql-bin.000004',
    -> master_log_pos=600;
 mysql> start slave;
 ```

> Ref: [How To Set Up MySQL Master-Master Replication](https://www.digitalocean.com/community/tutorials/how-to-set-up-mysql-master-master-replication)