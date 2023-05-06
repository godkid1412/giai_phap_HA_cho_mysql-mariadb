# **1. Giới thiệu về HA**

Ngày nay, công nghệ thông tin đã ăn sâu vào nhiều lĩnh vực trong đời sống phục vụ cho sản xuất, giải trí và đặc biệt nhu cầu thông tin. Các hệ thống này luôn được đầu tư với quy mô càng ngày càng mở rộng, là hướng phát triển trọng tâm của doanh nghiệp cung cấp nội dung. Để đảm bảo các dịch vụ chạy thông suốt, phục vụ tối đa đến nhu cầu của người sử dụng và nâng cao tính bảo mật, an toàn dữ liệu; giải pháp High Availability được nghiên cứu và phát triển bởi nhiều hãng công nghệ lớn. Với Database, tính an toàn và khả dụng được đặt lên hàng đầu.

## **HA làm được gì?**

- Tăng tính sẵn sàng dữ liệu mọi lúc
- Nâng cao hiệu suất làm việc của hệ thống
- Đảm bảo tính an toàn của dữ liệu
- Đảm bảo hệ thống không bị gián đoạn

# **2. Các giải pháp HA cho MySQL/MariaDB**

Có 2 giải pháp chính cho việc HA:

- Giải pháp Native: Được MySQL/MariaDB hỗ trợ
  - Master - Slave
  - Master - Master
  - Group Replication
- Các giải pháp bên thứ ba: Cùng với mục đích của giải pháp Native là để nhất quán dữ liệu giữa các node với nhau nhưng cơ chế và mô hình khác với Native
  - Galera Cluster
  - Share Storage
  - Percona Cluster

## **2.1. Giải pháp Native**

Cơ chế làm việc như sau: Trên mỗi server sẽ có một user làm nhiệm vụ replication dữ liệu mục đích của việc này là giúp các server đảm bảo tính nhất quán về dữ liệu với nhau.

### **Cơ chế hoạt động**

Máy chủ Master sẽ gửi các binary-log đến máy chủ Slave. Máy chủ Slave sẽ đọc các binary-log từ Master để yêu cầu truy cập dữ liệu vào quá trình replication. Một relay-log được tạo ra trên slave, nó sử dụng định dạng giống với binary-log. Các relay-log sẽ được sử dụng để replication và được xóa bỏ khi hoàn tất quá trình replication.

Các master và slave không nhất thiết phải luôn kết nối với nhau. Nó có thể được đưa về trạng thái offline và khi được kết nối lại, quá trình replication sẽ được tiếp tục ở thời điểm nó online.

#### **Trên Master**

Master phải ghi các sự kiện replica vào một tập tin log đặc biệt là binary log. Sau đó các slave đọc dữ liệu từ tập tin này

Bất cứ khi nào slave kết nối với master, master tạo một luồng kết nối mới (Binary Log Dump Thread)

#### **Trên Slave**

Khi quá trình replica được chạy, có 2 luồng chạy trên slave:

- **Luồng nhập xuất(I/O thread)**: Kết nối với master, đọc các sự kiện trong binary log và sao chép vào tập tin relay log

- **Luồng SQL(SQL Thread)**: đọc các sự kiện từ relay log được lưu trữ cục bộ trên slave (tập tin được ghi bởi luồng IO) và thực thi các sự kiện đó nhanh nhất có thể.

#### **Binary-log là gì**

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

### **2.2.1 Group Replication**

Group replication là một cách để triển khai cơ chế fault-tolerant, linh hoạt hơn. Quá trình này liên quan đến việc thiết lập một nhóm máy chủ, mỗi máy chủ đều tham gia vào việc đảm bảo dữ liệu được sao chép chính xác. Nếu máy chủ master gặp sự cố, sẽ có cuộc bình chọn thành viên để chọn các máy chủ slave làm máy chủ master mới từ nhóm đã cài đặt. Điều này cho phép các nút còn lại tiếp tục hoạt động, ngay cả khi gặp sự cố.

Tham khảo cách cài đặt Group Replication [tại đây](https://github.com/godkid1412/giai_phap_HA_cho_mysql-mariadb/blob/main/3.%20Group%20Replication/3.%20Group%20Replication.md)

## **2.2 Các giải pháp bên thứ ba**

### **2.2.1 Galera Cluster**

Galera Cluster là một giải pháp multi master cho database.Khi sử dụng galera cluster, application có thể read/write trên bất cứ node nào. Một node có thể thêm vào cluster hay gỡ ra khỏi cluster mà không có downtime dịch vụ

Bản thân các database như mariadb, mysql, percona xtradb không được tích hợp sẵn bên tính năng multi master. Các database này sử dụng plugin là galera replication để sử dụng tính năng multi master do galera cluster cung cấp. Về bản chất, galera replication plugin sử dụng phiên bản mở rộng của mysql replication api, bản mở rộng này có tên **wsrep api**

**Cách galera cluster hoạt động**
Một writeset, chính là một transaction cần được replication trên các node. Transaction này sẽ được certificate trên từng node nhận được (qua replication) xem có conflict với bất cứ transaction nào đang có trong queue của node đó không. Nếu có thì replicated writeset này sẽ bị node discard. Nếu không thì replicated writeset này sẽ được applied. Một transaction chỉ xem là commit sau khi đã pass qua bước certificate trên tất cả các node. Điều này đảm bảo transaction đó đã được phân phối trên tất cả các node.

Xem chi tiết hơn về cách hoạt động [tại đây](https://github.com/godkid1412/giai_phap_HA_cho_mysql-mariadb/blob/main/4.%20Galera%20Cluster/Flow_Control.md)

> Reference:
>
>- [Certification-Based Replication](https://galeracluster.com/library/documentation/certification-based-replication.html)

### **Điểm mạnh**

- Giải pháp multi master hoàn chỉnh nên cho phép read/write trên bất cứ node nào
- Multi thread slave nên cho phép writeset nhanh hơn
- Không cần failover vì node nào cũng là master

### **Điểm yếu**

- Không có scale up về dung lượng do galera cluster thì tất cả node đều có dữ liệu giống hệt nhau
- Vẫn có hiện tượng stale data do bất đồng bộ khi apply writeset trên các node

### **2.2.2 Shared Storage**

#### **Share Storage là gì**

Share Storage kết hợp files và các thông tin khác ở trung tâm dữ liệu có thể truy cập hoặc chia sẻ bởi nhiều người dùng.

Giải pháp `Share Storage` bao gồm network attached storage (NAS) và storage area network (SAN).

Share storage kết nối đến server bằng _fibre channel_ để đảm bảo I/O performance

Các tín năng chính:

- Không mất dữ liệu
- Chỉ sao chép duy nhất một logical
- Tự động failover (Thường dùng cùng với pacemaker)

### **2.2.3 Percona Cluster**

Percona XtraDB Cluster là giải pháp HA cho MySQL. Nó tích hợp Percona Server cho MySQL và Percona XtraBackup sử dụng thư viện cửa `Galera` để đồng bộ multi-source replication.

Một cluster bao gồm nhiều node, mỗi node chứa nhiều dữ liệu giống nhau được đồng bộ giữa các node với nhau.

Giống với Galera, Percona có ít nhất 3 node luôn đồng bộ dữ liệu với nhau, tuy nhiên ta cũng có thể dùng hai node. Dữ liệu có thể được đọc/ghi lên bất kỳ node nào. Một máy chủ đứng ở bên trên tiếp nhận các truy vấn và phân phối lại một cách đồng đều cho các server.

#### **Ưu đểm**

- Khi thực thi một câu truy vấn (query), nó thực thi trên local của node. Tất cả dữ liệu luôn ở trên node, do vậy không cần truy cập từ xa
- Không có trung tâm quản lý (central management). Ta có thể cho bất kì node nào ra khỏi cluster tại bất kỳ thời gian nào mà cluster vẫn tiếp tục mà không bị mất dữ liệu
- Giải pháp tốt cho mở rộng read workload.

#### **Nhược điểm**

- Phải copy toàn bộ dữ liệu từ node cũ đến node mới khi thêm một node mới.
- Không hiệu quả cho giải pháp cần scaling cho việc ghi dữ liệu
- Có nhiều bản sao của dữ liệu: 3 node thì ta có 3 bản sao của dữ liệu.

> Reference:
>
>[About Percona XtraDB Cluster](https://docs.percona.com/percona-xtradb-cluster/8.0/intro.html)
