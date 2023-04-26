# 1. Giới thiệu về HA 
Ngày nay, công nghệ thông tin đã ăn sâu vào nhiều lĩnh vực trong đời sống phục vụ cho sản xuất, giải trí và đặc biệt nhu cầu thông tin. Các hệ thống này luôn được đầu tư với quy mô càng ngày càng mở rộng, là hướng phát triển trọng tâm của doanh nghiệp cung cấp nội dung. Để đảm bảo các dịch vụ chạy thông suốt, phục vụ tối đa đến nhu cầu của người sử dụng và nâng cao tính bảo mật, an toàn dữ liệu; giải pháp High Availability được nghiên cứu và phát triển bởi nhiều hãng công nghệ lớn. Với Database, tính an toàn và khả dụng được đặt lên hàng đầu. 

## HA làm được gì?
- Tăng tính sẵn sàng dữ liệu mọi lúc
- Nâng cao hiệu suất làm việc của hệ thống
- Đảm bảo tính an toàn của dữ liệu
- Đảm bảo hệ thống không bị gián đoạn

# 2. Các giải pháp
Có 2 giải pháp chính cho việc HA:
- Giải pháp Native: Được MySQL/MariaDB hỗ trợ
    - Master - Slave
    - Master - Master
- Các giải pháp bên thứ ba: Cùng với mục đích của giải pháp Native là để nhất quán dữ liệu giữa các node với nhau nhưng cơ chế và mô hình khác với Native
    - Galera Cluster
    - Share Storage
    - Percona Cluster
