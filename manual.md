## **OpenSSH 설정**

- Windows

1. 설정 -> 시스템 -> 선택적 기능 -> 선택적 기능 추가 -> OpenSSH Server 다운
or 
``` powershell
$ Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
``` 
입력

2. 
``` powershell
$ Start-Service sshd
```
```powershell
 Set-Service -Name sshd -StartupType 'Automatic'
 ```

```powershell
$ New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH-Server-In-TCP' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```




