# Flow Control

Galera Cluster quản lí các tiến trình replication bằng cơ chế phản hồi (feedback), nó được gọi là Flow Control. Flow Control cho phép các node tạm dừng hoặc tiếp tục replication dựa theo node đó cần replication hay không.

## Flow Control làm việc như nào?

Galera Cluster đạt được sự đồng bộ trong replication bằng việc transactions sao chép đến tất cả các node và thực thi theo thứ tự từ cluster-wide.

Các node nhận được write-sets (là một transaction cần được replication trên các node) và đưa chúng vào global ordering. Khi đó các transaction được các node nhận từ cluster, khi đó các transaction chưa đc applied và committed mà được giữ ở hàng chờ (received queue).

Khi hàng chờ (received queue) đạt đến kích thước nhất định của node. Khi đó các node sẽ tạm dừng việc replication và bắt đầu làm việc với hàng đợi (received queue). Khi hàng chờ giảm đến mức dễ quản lý (manageable size) thì node đó tiếp tục quá trình replication.

## Understanding Node States
Galera Cluster thực hiện một số hình thức Flow COntrol, tùy thuộc vào trạng thái node. Điều này đảm bảo sự đồng bộ theo thời gian và tính nhất quán.

Có 4 kiểu chính của Flow Control:
- No Flow Control
- Write-set Caching
- Caching Up
- Cluster Sync




>Reference:
> [Flow Control](https://galeracluster.com/library/documentation/node-states.html)