### Enumerate

![](Attachments/Pasted%20image%2020260505185457.png)

### Exploit Web Service

Dùng ffuf, phát hiện web có chứa đường dẫn đến trang đăng nhập admin:

![](Attachments/Pasted%20image%2020260505190353.png)

Vào trang đăng nhập, ta hoàn toàn có thể tạo mới một tài khoản và truy cập được trang quản trị, nhưng dưới chức năng Client.

![](Attachments/Pasted%20image%2020260505190613.png)

Khai thác CVE 2025-2304 trên Camaleon CMS 2.9.0: https://medium.com/@iamkumarraj/mass-assignment-vulnerability-in-camaleon-cms-2-9-0-ajax-privilege-escalation-9a09c8253b52


![](Attachments/Pasted%20image%2020260505143231.png)

Thêm trường `password[role]='admin'.` Khi request này chạm đến backend, Rails sẽ parse (biên dịch) nó thành một Nested Hash như sau:

```json
params = {
  "user_id" => "5",
  "password" => {
    "password" => "NewPass123",
    "password_confirmation" => "NewPass123",
    "role" => "admin" # <-- Payload độc hại
  }
}
```

Bởi vì Rails không phân biệt được đâu là trường dữ liệu hợp lệ (như password) và đâu là trường bị chèn trái phép (như role), hàm `@user.update()` sẽ gọi các setter tương ứng cho model. Nó sẽ thực thi việc gán `@user.role = "admin"` và tự động lưu thay đổi này vào cơ sở dữ liệu.

![](Attachments/Pasted%20image%2020260505143356.png)

Xem xét trang quản trị, ta biết có một dịch vụ lưu trữ AWS trên cổng 54321, là 1 trong 3 cổng lúc đầu ta phát hiện được.

![](Attachments/Pasted%20image%2020260505191031.png)



![](Attachments/Pasted%20image%2020260505170126.png)

![](Attachments/Pasted%20image%2020260505181410.png)

![](Attachments/Pasted%20image%2020260505181435.png)

![](Attachments/Pasted%20image%2020260505183342.png)

![](Attachments/Pasted%20image%2020260505184020.png)


