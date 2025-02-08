```powershell
 Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} | Group-Object {$_.pRoperties[5].value} | Select-Object Count,Name | Sort count -Des
cending
```

![[Pasted image 20250207202742.png]]