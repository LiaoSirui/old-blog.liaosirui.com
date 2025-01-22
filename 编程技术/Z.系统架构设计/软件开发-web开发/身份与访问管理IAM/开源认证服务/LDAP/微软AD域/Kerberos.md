在 Active Directory 域控制器 (DC) 上启用和配置支持 FAST (Flexible Authentication Secure Tunneling) 身份验证，可以按照以下步骤进行。FAST 是 Kerberos 协议的一项增强功能，通过保护认证数据的完整性和保密性，提高了 Kerberos 身份验证的安全性。

在 PowerShell 中运行以下命令，查看域和林功能级别：

```bash
Get-ADDomain | Select-Object Name, DomainMode
Get-ADForest | Select-Object ForestMode
```

