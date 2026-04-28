![](Attachments/Pasted%20image%2020260423060335.png)

![](Attachments/Pasted%20image%2020260423064329.png)

"LOGGING\svc_recovery", BindPass: "Em3rg3ncyPa\$\$2025"

![](Attachments/Pasted%20image%2020260423065742.png)


![](Attachments/Pasted%20image%2020260423065733.png)
![](Attachments/Pasted%20image%2020260423065733.png)

![](Attachments/Pasted%20image%2020260423082227.png)


![](Attachments/Pasted%20image%2020260423080643.png)

![](Attachments/Pasted%20image%2020260423080719.png)

![](Attachments/Pasted%20image%2020260423080830.png)


![](Attachments/Pasted%20image%2020260423094710.png)

603fc24ee01a9409f83c9d1d701485c5

![](Attachments/Pasted%20image%2020260423105106.png)

![](Attachments/Pasted%20image%2020260423105125.png)

`$TaskName = "UpdateChecker Agent"`
    `$service = New-Object -ComObject "Schedule.Service"`
    `$service.Connect()`
    `$task = $service.GetFolder("\").GetTask($TaskName)`
    `Write-Host $task.Xml`
    

![](Attachments/Pasted%20image%2020260423132234.png)

![](Attachments/Pasted%20image%2020260423132506.png)

`// UpdateMonitor.Program`
`using System;`
`using System.IO;`
`using System.IO.Compression;`
`using System.Runtime.InteropServices;`

`private static void Main(string[] args)`
`{`
	`string path = "C:\\ProgramData\\UpdateMonitor\\Logs\\monitor.log";`
	`string text = "C:\\ProgramData\\UpdateMonitor\\Settings_Update.zip";`
	`string text2 = "C:\\Program Files\\UpdateMonitor\\bin\\";`
	`string text3 = "settings_update.dll";`
	`string text4 = Path.Combine(text2, text3);`
	`Directory.CreateDirectory(Path.GetDirectoryName(path));`
	`CleanupLogs(path, 90);`
	`Log(path, "Starting Sentinel Update Check...");`
	`Log(path, "Checking for update on core server...");`
	`Log(path, "Info: Core did not find file Settings_Update.zip");`
	`Log(path, "Last status: File not found on core");`
	`Log(path, "Checking for update on local server...");`
	`if (File.Exists(text))`
	`{`
		`try`
		`{`
			`if (File.Exists(text4))`
			`{`
				`File.Delete(text4);`
			`}`
			`ZipFile.ExtractToDirectory(text, text2);`
			`Log(path, "Successfully unzipped update to " + text2);`
		`}`
		`catch (IOException ex)`
		`{`
			`Log(path, "Update failed: " + ex.Message);`
		`}`
		`catch (Exception ex2)`
		`{`
			`Log(path, "Update failed: " + ex2.Message);`
		`}`
	`}`
	`else`
	`{`
		`Log(path, "No updates found locally: C:\\ProgramData\\UpdateMonitor\\Settings_Update.zip.");`
	`}`
	`Log(path, "Loading update applier: " + text4);`
	`IntPtr intPtr = LoadLibrary(text4);`
	`if (intPtr == IntPtr.Zero)`
	`{`
		`int lastWin32Error = Marshal.GetLastWin32Error();`
		`Log(path, $"Failed to load {text3}. Error code: {lastWin32Error}");`
		`Log(path, "Update check completed.");`
		`return;`
	`}`
	`try`
	`{`
		`IntPtr procAddress = GetProcAddress(intPtr, "PreUpdateCheck");`
		`if (procAddress != IntPtr.Zero)`
		`{`
			`Log(path, "Calling 'PreUpdateCheck' in " + text3);`
			`((PreUpdateCheck)Marshal.GetDelegateForFunctionPointer(procAddress, typeof(PreUpdateCheck)))();`
		`}`
		`else`
		`{`
			`Log(path, "'PreUpdateCheck' not found in " + text3 + ". Continuing...");`
		`}`
	`}`
	`finally`
	`{`
		`FreeLibrary(intPtr);`
	`}`
	`Log(path, "Update check completed.");`
`}`

![](Attachments/Pasted%20image%2020260423161139.png)

![](Attachments/Pasted%20image%2020260423161207.png)

![](Attachments/Pasted%20image%2020260423161227.png)

![](Attachments/Pasted%20image%2020260428155731.png)


