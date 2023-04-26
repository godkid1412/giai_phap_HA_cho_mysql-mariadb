# **1. Giới thiệu về HA**
Ngày nay, công nghệ thông tin đã ăn sâu vào nhiều lĩnh vực trong đời sống phục vụ cho sản xuất, giải trí và đặc biệt nhu cầu thông tin. Các hệ thống này luôn được đầu tư với quy mô càng ngày càng mở rộng, là hướng phát triển trọng tâm của doanh nghiệp cung cấp nội dung. Để đảm bảo các dịch vụ chạy thông suốt, phục vụ tối đa đến nhu cầu của người sử dụng và nâng cao tính bảo mật, an toàn dữ liệu; giải pháp High Availability được nghiên cứu và phát triển bởi nhiều hãng công nghệ lớn. Với Database, tính an toàn và khả dụng được đặt lên hàng đầu. 

## **HA làm được gì?**
- Tăng tính sẵn sàng dữ liệu mọi lúc
- Nâng cao hiệu suất làm việc của hệ thống
- Đảm bảo tính an toàn của dữ liệu
- Đảm bảo hệ thống không bị gián đoạn

# **2. Các giải pháp**
Có 2 giải pháp chính cho việc HA:
- Giải pháp Native: Được MySQL/MariaDB hỗ trợ
    - Master - Slave
    - Master - Master
- Các giải pháp bên thứ ba: Cùng với mục đích của giải pháp Native là để nhất quán dữ liệu giữa các node với nhau nhưng cơ chế và mô hình khác với Native
    - Galera Cluster
    - Share Storage
    - Percona Cluster

## **2.1. Giải pháp Native**

Cơ chế làm việc như sau: Trên mỗi server sẽ có một user làm nhiệm vụ replication dữ liệu mục đích của việc này là giúp các server đảm bảo tính nhất quán về dữ liệu với nhau.

**Cơ chế hoạt động**

Máy chủ Master sẽ gửi các binary-log đến máy chủ Slave. Máy chủ Slave sẽ đọc các binary-log từ Master để yêu cầu truy cập dữ liệu vào quá trình replication. Một relay-log được tạo ra trên slave, nó sử dụng định dạng giống với binary-log. Các relay-log sẽ được sử dụng để replication và được xóa bỏ khi hoàn tất quá trình replication.

Các master và slave không nhất thiết phải luôn kết nối với nhau. Nó có thể được đưa về trạng thái offline và khi được kết nối lại, quá trình replication sẽ được tiếp tục ở thời điểm nó online.

**Binary-log là gì**

Binary-log chứa những bản ghi ghi lại những thay đổi của các database. Nó chứa dữ liệu và cấu trúc của DB (có bao nhiêu bảng, bảng có bao nhiêu trường,...), các câu lệnh được thực hiện trong bao lâu,... Nó bao gồm các file nhị phân và các index.

Binary-log được lưu trữ ở dạng nhị phân không phải là dạng văn bản plain-text.

### **2.1.1 Master - Slave**

**Master - Slave**: là một kiểu của giải pháp HA cho database, mục đích đồng bộ dữ liệu của DB chính (master) sang một máy chủ DB khác (slave) một cách tự động

![image](https://user-images.githubusercontent.com/54473576/234454971-f5048aec-6e03-4e7a-af89-0f66a7461e00.png)

Tham khảo cách cấu hình [tại đây](https://github.com/godkid1412/giai_phap_HA_cho_mysql-mariadb/blob/main/1.%20Master%20Slave/1.%20Master%20Slave.md)

### **2.1.2: Master - Master**

là một kiểu của giải pháp HA cho database, mục đích đồng bộ dữ liệu giữa 2 DB master với nhau


![image](https://user-images.githubusercontent.com/54473576/234460714-deafa432-c77a-4f69-9b26-4478f5acc93b.png)

Tham khảo cách cấu hình [tại đây](https://github.com/godkid1412/giai_phap_HA_cho_mysql-mariadb/blob/main/2.%20Master%20%20Master/Master%20Master.md)