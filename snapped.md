![](Attachments/Pasted%20image%2020260502180555.png)

![](Attachments/Pasted%20image%2020260502181031.png)

![](Attachments/Pasted%20image%2020260502190235.png)

yV+cDV8aqZy8OyUo/mjCji3DsSwnGm0F/+6rFyaMmUo=:5eNDv6wSYAc8a7KJgiSAcA==
The encryption keys are Base64-encoded AES-256 key (32 bytes) and IV (16 bytes), formatted as key:iv.

https://github.com/advisories/GHSA-g9w5-qffc-6762

![](Attachments/Pasted%20image%2020260502191915.png)

jonathan$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq
admin$2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvmg

![](Attachments/Pasted%20image%2020260502192719.png)

![](Attachments/Pasted%20image%2020260502192739.png)

![](Attachments/Pasted%20image%2020260502192809.png)

admin:CS)!518bRX$ce

![](Attachments/Pasted%20image%2020260502194127.png)

![](Attachments/Pasted%20image%2020260502194251.png)

hmm, thử đăng nhập ssh, wtf:

![](Attachments/Pasted%20image%2020260502194616.png)

![](Attachments/Pasted%20image%2020260503173913.png)

https://cdn2.qualys.com/advisory/2026/03/17/snap-confine-systemd-tmpfiles.txt

to set up a snap's sandbox, snap-confine creates a directory named /tmp/snap-private-tmp/$SNAP/tmp (as user root, mode 01777) that is later bind-mounted onto the /tmp directory inside the snap's sandbox.
And inside this /tmp directory (inside the snap's sandbox), snap-confine creates a directory named **/tmp/.snap (as user root, mode 0755) to create "mimics"**; for example, inside the sandbox of each and every snap that is installed by default on Ubuntu Desktop, snap-confine bind-mounts the /usr/lib/x86_64-linux-gnu/webkit2gtk-4.0 directory:
but inside the snap's sandbox, /usr/lib/x86_64-linux-gnu is in a read-only filesystem (the "core22" base's squashfs);
snap-confine must first create a "mimic" of /usr/lib/x86_64-linux-gnu (a writable copy of /usr/lib/x86_64-linux-gnu), by:
1 bind-mounting the original, read-only /usr/lib/x86_64-linux-gnu onto **/tmp/.snap/usr/lib/x86_64-linux-gnu** (inside the snap's sandbox);
2 mounting a new, writable tmpfs onto /usr/lib/x86_64-linux-gnu;
3 bind-mounting every file and directory from /tmp/.snap/usr/lib/x86_64-linux-gnu back into /usr/lib/x86_64-linux-gnu;
4 creating the /usr/lib/x86_64-linux-gnu/webkit2gtk-4.0 mountpoint (which is in a writable tmpfs now);
5/ finally bind-mounting /snap/firefox/6565/gnome-platform/usr/lib/x86_64-linux-gnu/webkit2gtk-4.0 (for example) onto /usr/lib/x86_64-linux-gnu/webkit2gtk-4.0.

![](Attachments/Pasted%20image%2020260504140029.png)

![](Attachments/Pasted%20image%2020260504141606.png)

![](Attachments/Pasted%20image%2020260504141621.png)

![](Attachments/Pasted%20image%2020260504141640.png)

https://github.com/nomaisthere/CVE-2026-3888

