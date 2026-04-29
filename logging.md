### Enumarate

![](Attachments/Pasted%20image%2020260423060335.png)

Quét nâng cao:
`nmap -Pn -p- --min-rate 2000 -T 4 <\IP>`
-> quét nâng cao phát hiện cổng 8530 và 8531, là cổng để update windows nội bộ (?)

### Lateral Movement
Truy cập share SMB, nhận được nhiều file log, trong đó:

![](Attachments/Pasted%20image%2020260423064329.png)

"LOGGING\svc_recovery", BindPass: "Em3rg3ncyPa\$\$2025"
Thay 2025 bằng 2026 thì được mật khẩu đúng của SVC_RECOVERY
SVC_RECOVERY nằm trong nhóm Protected Users -> không đăng nhập được bằng tk/mk thông thường hoặc hash, mọi thao tác đều phải thông qua Kerberos -> xin vé TGT. Ngoài ra còn phải đồng bộ về thời gian thông qua ntpupdate và faketime.

![](Attachments/Pasted%20image%2020260429141035.png)

![](Attachments/Pasted%20image%2020260429141049.png)

![](Attachments/Pasted%20image%2020260429141101.png)

SVC_RECOVERY có quyền GenericWrite tới MSA_HEALTH$ (là 1 thành viên của nhóm Remote). Ta có thể tấn công Shadow Credentials. 
https://infosecwriteups.com/how-hackers-achieve-invisible-persistence-in-active-directory-shadow-credentials-6b53a6c85e74


![](Attachments/Pasted%20image%2020260423065742.png)


Nạp vé Kerberos của SVC_RECOVERY:

![](Attachments/Pasted%20image%2020260423080830.png)

Tấn công shadow credentials bằng công cụ certipy (linux), Certify.exe (windows):

![](Attachments/Pasted%20image%2020260423094710.png)

Thu được NTLM hash của MSA_HEALTH$ ($ chỉ tài khoản máy, khó có thể crack dượdc)
603fc24ee01a9409f83c9d1d701485c5

![](Attachments/Pasted%20image%2020260423105106.png)

![](Attachments/Pasted%20image%2020260423105125.png)

### User Flag

Trong thư mục ngay khi đăng nhập, ta có một file lấy thông tin của task, sửa lại một chút để có thông tin đầy đủ của task:

```
$TaskName = "UpdateChecker Agent"
    $service = New-Object -ComObject "Schedule.Service"
    $service.Connect()
    $task = service.GetFolder("\").GetTask(TaskName)
    Write-Host $task.Xml
```


![](Attachments/Pasted%20image%2020260423132234.png)

PT3M: cứ 3 phút task lại chạy
Đường dẫn: "C:\Program Files\UpdateMonitor\UpdateMonitor.exe"

Dịch ngược thu được mã nguồn C#:

 
```
// UpdateMonitor.Program
using System;
using System.IO;
using System.IO.Compression;
using System.Runtime.InteropServices;

private static void Main(string[] args)
{
	string path = "C:\\ProgramData\\UpdateMonitor\\Logs\\monitor.log";
	string text = "C:\\ProgramData\\UpdateMonitor\\Settings_Update.zip";
	string text2 = "C:\\Program Files\\UpdateMonitor\\bin\\";
	string text3 = "settings_update.dll";
	string text4 = Path.Combine(text2, text3);
	Directory.CreateDirectory(Path.GetDirectoryName(path));
	CleanupLogs(path, 90);
	Log(path, "Starting Sentinel Update Check...");
	Log(path, "Checking for update on core server...");
	Log(path, "Info: Core did not find file Settings_Update.zip");
	Log(path, "Last status: File not found on core");
	Log(path, "Checking for update on local server...");
	if (File.Exists(text))
	{
		try
		{
			if (File.Exists(text4))
			{
				File.Delete(text4);
			}
			ZipFile.ExtractToDirectory(text, text2);
			Log(path, "Successfully unzipped update to " + text2);
		}
		catch (IOException ex)
		{
			Log(path, "Update failed: " + ex.Message);
		}
		catch (Exception ex2)
		{
			Log(path, "Update failed: " + ex2.Message);
		}
	}
	else
	{
		Log(path, "No updates found locally: C:\\ProgramData\\UpdateMonitor\\Settings_Update.zip.");
	}
	Log(path, "Loading update applier: " + text4);
	IntPtr intPtr = LoadLibrary(text4);
	if (intPtr == IntPtr.Zero)
	{
		int lastWin32Error = Marshal.GetLastWin32Error();
		Log(path, $"Failed to load {text3}. Error code: {lastWin32Error}");
		Log(path, "Update check completed.");
		return;
	}
	try
	{
		IntPtr procAddress = GetProcAddress(intPtr, "PreUpdateCheck");
		if (procAddress != IntPtr.Zero)
		{
			Log(path, "Calling 'PreUpdateCheck' in " + text3);
			((PreUpdateCheck)Marshal.GetDelegateForFunctionPointer(procAddress, typeof(PreUpdateCheck)))();
		}
		else
		{
			Log(path, "'PreUpdateCheck' not found in " + text3 + ". Continuing...");
		}
	}
	finally
	{
		FreeLibrary(intPtr);
	}
	Log(path, "Update check completed.");
}
```
-> Ta cần tạo một dll độc hại, nén nó vào 1 file zip, sau đó cop file zip vào thư mục Update, chờ task gọi đến

![](Attachments/Pasted%20image%2020260423161139.png)

![](Attachments/Pasted%20image%2020260423161207.png)

![](Attachments/Pasted%20image%2020260423161227.png)

Lấy được shell của jaylee.clifton:

![](Attachments/Pasted%20image%2020260428155731.png)

### Root flag

Dùng Certify.exe, ta thấy có 1 cert template mà nhóm IT có quyền xin, có flag ENROLLEE_SUPPLIES_SUBJECT, nhưng Extended Key Usage lại là Server Authen -> không phải ESC1
Vấn đề là máy DC01 này được cấu hình để tự động cập nhật thông qua https://wsus.logging.htb:8051. Ta có thể dựng 1 server wsus giả mạo, và hướng DNS của domain đến server này.
Có thể tham khảo ESC17 (?): https://mustafanafizdurukan.github.io/posts/esc17-wsus-dns-abuse/


![](Attachments/Pasted%20image%2020260429142943.png)

Dùng dnstool để tiêm bản ghi dns đến địa chỉ máy tấn công (kali):

![](Attachments/Pasted%20image%2020260428192539.png)

Kiểm tra bản ghi DNS: dùng adidns
![](Attachments/Pasted%20image%2020260429000818.png)

Cả 2 công cụ trên đều trong gói krbxrelay.

Xin cert cho server giả mạo:

`Certify.exe request /ca:"DC01.logging.htb\logging-DC01-CA" /template:"UpdateSrv" /subject:"CN=wsus.logging.htb" /dns:"wsus.logging.htb"`

![](Attachments/Pasted%20image%2020260429000605.png)

Dựng wsus server giả mạo với wsuks tool:

![](Attachments/Pasted%20image%2020260429000445.png)

![](Attachments/Pasted%20image%2020260429143816.png)




