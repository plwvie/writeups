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

Cấu hình aws:

![](Attachments/Pasted%20image%2020260505192607.png)

`aws s3 ls s3://facts.htb/` -> Lệnh này sẽ hướng lên server của Amazon, thêm option --endpoint-url để hướng về smáy có port 54321.
`aws s3 ls --endpoint-url http://facts.htb:54321`
`aws s3 cp s3://ten-bucket/file.txt ./ --endpoint-url http://facts.htb:54321`

![](Attachments/Pasted%20image%2020260505192916.png)

Trong thư mục internal, ta sẽ phát hiện được 2 file ssh: `authorized_keys` và `id_ed25519` (khoá bí mật ssh).

![](Attachments/Pasted%20image%2020260505193119.png)

Tạo một thư mục để lưu trữ:
`mkdir ssh_keys`
Kéo dữ liệu về
`aws s3 sync s3://internal/.ssh/ ./ssh_keys/ --endpoint-url http://facts.htb:54321`

Vấn đề với ssh: (1) Khoá bí mật bị mã hoá bằng 1 passphrase, (2) chưa biết tên tài khoản ssh.

Với (1), ta có thể dùng john để bẻ:

Bạn không thể ném trực tiếp file SSH vào `john`. Bạn cần dùng công cụ `ssh2john` để dịch file này sang định dạng hash mà `john` có thể hiểu được.

`ssh2john id_ed25519 > id_ed25519.hash`

![](Attachments/Pasted%20image%2020260505193530.png)

Bẻ khoá bằng john:

![](Attachments/Pasted%20image%2020260505170126.png)

Phát hiện được passphrase: `dragonballz`

Với (2), bởi vì file `id_ed25519` định dạng mới của OpenSSH lưu trữ cả thông tin của Public Key bên trong, bạn có thể dùng lệnh `ssh-keygen` để ép nó "nhả" ra Public Key.

![](Attachments/Pasted%20image%2020260505181410.png)

`trivia`

![](Attachments/Pasted%20image%2020260505181435.png)

### Privilege Escalation

![](Attachments/Pasted%20image%2020260505183342.png)

![](Attachments/Pasted%20image%2020260505184020.png)


