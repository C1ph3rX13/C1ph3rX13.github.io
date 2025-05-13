# LDAP

[Active Directory 架构（AD 架构） - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adschema/active-directory-schema)

[类（AD 架构） - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adschema/classes)

[属性（AD 架构） - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adschema/attributes)

[Active Directory 域服务 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/ad/active-directory-domain-services)

## UserAccountControl

[UserAccountControl 属性标志 - Windows Server | Microsoft Learn](https://learn.microsoft.com/zh-cn/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties)

[UserAccountControl property flags - Windows Server | Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties)

[UserAccountControl Attribute Values - Active Directory Pro](https://activedirectorypro.com/useraccountcontrol-check-and-manage-attribute-value/)

下面是特定对象的默认 UserAccountControl 值：

- 典型用户：0x200（512）
- 域控制器：0x82000（532480）
- 工作站/服务器：0x1000 （4096）
- 信任：0x820 （2080）

## msDS-SupportedEncryptionTypes

用户、计算机或信任帐户支持的加密算法

[ms-DS-Supported-Encryption-Types 属性 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adschema/a-msds-supportedencryptiontypes)

[Supported Encryption Types Bit Flags | Microsoft Learn](https://learn.microsoft.com/zh-cn/openspecs/windows_protocols/ms-kile/6cfc7b50-11ed-4b4d-846d-6f08f0812919)

[Set MSDS-SupportedEncryptionTypes PowerShell Explained](https://powershellcommands.com/set-msds-supportedencryptiontypes-powershell)

## ObjectGUID

[Object-Guid 属性 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adschema/a-objectguid)

[读取对象的 objectGUID 和创建 GUID 的字符串表示形式 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/ad/reading-an-objectampaposs-objectguid-and-creating-a-string-representation-of-the-guid)

[RFC 4122 标准 | datatracker.ietf.org](https://datatracker.ietf.org/doc/html/rfc4122.html)

[RFC 4122 标准 | 中文版本](https://rfc2cn.com/rfc4122.html)

**objectGUID** 属性是用户的唯一标识符。 该属性是单值 128 位全球唯一标识符 (GUID)，并以 [**ADS_OCTET_STRING**](https://learn.microsoft.com/zh-cn/windows/win32/api/iads/ns-iads-ads_octet_string) 结构的形式存储。 GUID 由 Active Directory 服务器在创建用户对象时创建。

如果重命名或移动对象，则对象可分辨名称会发生变化，因此可分辨名称并非可靠的对象标识符

在 Active Directory 域服务中，对象的 **objectGUID** 属性永远不会更改，即使重命名或移动对象也是如此

可以使用[IAD 属性方法](https://learn.microsoft.com/zh-cn/windows/desktop/ADSI/iads-property-methods) 中的 **GUID** 属性方法来检索 **objectGUID** 的字符串形式

### UUID 的结构

UUID 遵循 **RFC 4122 标准** ，格式为：

| 字段                 | 位数 | 十六进制字符数 | 含义                                                         | 版本相关性                             |
| -------------------- | ---- | -------------- | ------------------------------------------------------------ | -------------------------------------- |
| time_low             | 32位 | 8              | 时间戳的低32位（自1582年10月15日以来的100纳秒数）            | 版本1专用                              |
| time_mid             | 16位 | 4              | 时间戳的中16位                                               | 版本1专用                              |
| time_hi_version      | 16位 | 4              | 时间戳的高12位，高4位表示UUID版本号（M字段，如4xxx中的4）    | 版本1专用（版本号占4位，其余版本不同） |
| clock_seq_hi_variant | 8位  | 2              | 时钟序列的高8位，最高2位表示变体标识（N字段，如yxxx中的y，值为10xx） | 版本1/4通用                            |
| clock_seq_low        | 8位  | 2              | 时钟序列的低8位                                              | 版本1/4通用                            |
| node                 | 48位 | 12             | 节点地址（如MAC地址）或随机生成的48位值                      | 版本1（MAC）、版本4（随机）            |

### objectGUID 存储格式

在 Windows Active Directory 中，`objectGUID` 以 **Windows GUID 结构** 存储，其二进制格式与标准 UUID 不同，主要区别在于 **字节序** （大小端）

### **生成原理**

1. **版本依赖**

objectGUID 的生成通常基于 **UUID版本1** （时间戳+节点ID）或 **版本4** （随机数），具体取决于AD环境配置：

+ 版本1 ：结合时间戳（自1582年10月15日以来的100纳秒数）和节点唯一标识符（如网卡MAC地址）生成，确保跨时间和空间的唯一性 

+ 版本4 ：通过加密安全的随机数生成器创建，降低可预测性，适用于对隐私敏感的场景 

2. **唯一性保障**

+ 域控制器在创建对象时生成objectGUID，并保证其全局唯一性

+ 即使跨域或跨森林迁移，objectGUID也不会重复 

+ 若无法获取MAC地址（如虚拟化环境），节点ID可能替换为其他唯一硬件标识符或随机值 

3. **变体标识**

objectGUID遵循RFC 4122定义的变体标识规则，第四组首字节的高两位固定为`10xx`（如`8xxx`、`9xxx`、`axxx`、`bxxx`），表明其符合标准UUID格式

### 变体标识的规则

| 变体标识（二进制） | 十六进制范围        | 含义                                                |
| ------------------ | ------------------- | --------------------------------------------------- |
| 0xxx               | 0xxx                | 保留（未使用）                                      |
| 10xx               | 8xxx,9xxx,axxx,bxxx | 标准变体（RFC 4122定义的UUID格式）                  |
| 110x               | cxxx,dxxx           | 保留（Microsoft的GUID使用此变体，但不符合RFC 4122） |
| 111x               | exxx - fxxx         | 保留（未定义）                                      |

+ 核心规则：

标准UUID（如版本1-5）的变体标识必须为 10xx，即第四组首字符的十六进制值必须是 8、9、a 或 b

+ 版本1的UUID ：

生成时，第四组的首字符会随机选择 8、9、a 或 b 中的一个

例如：6ba7b810-9dad-11d1-80b4-00c04fd430c8（首字符为 8）

+ 版本4的UUID ：

完全随机生成，但必须保证第四组首字符符合 8、9、a、b 的规则

例如：f47ac10b-58cc-4372-a567-0e02b2c3d479（首字符为 a）

## Windows GUID

Windows GUID与标准UUID的字节顺序不同

Windows Active Directory 中的 `objectGUID` 以 **Windows GUID结构** 存储，其二进制格式为：

```go
type GUID struct {
    Data1 uint32  // 小端序 (4字节)
    Data2 uint16  // 小端序 (2字节)
    Data3 uint16  // 小端序 (2字节)
    Data4 [8]byte // 原始顺序 (8字节)
}
```

### 变体标识的位置

对于微软的GUID `be959a0e-e78e-4d33-96be-7664d7fce138`，其变体标识（Variant）的解析如下：

- **UUID格式** ：`xxxxxxxx-xxxx-4xxx-**yxxx**-xxxxxxxxxxxx`
  其中 `yxxx` 是第四组，首字符为 `9`（即 `96be` 中的 `9`）。
- **二进制表示** ：
  十六进制的 `9` 对应二进制 `1001`，**高两位为 `10`** ，符合RFC 4122定义的标准变体标识（`10xx`）

### **具体分析**

- **版本号** ：第三组首字符为 `4`（`4d33` 中的 `4`），表示该GUID是 **版本4（随机生成）** 
- **变体标识** ：第四组首字符为 `9`（`96be` 中的 `9`），对应变体 `10xx`，符合RFC 4122标准
- **结论** ：这是一个标准的RFC 4122 GUID，由微软生成但遵循通用规则，**不属于微软私有的非标准变体** 

### 变体标识的含义

| 字段     | 值   | 含义                                                         |
| -------- | ---- | ------------------------------------------------------------ |
| 变体标识 | 10xx | 表示该GUID遵循 RFC 4122标准（即通用唯一标识符的标准格式）    |
| 具体值   | 9    | 第四组首字符为 9，对应二进制 1001，高两位 10 符合标准变体规则 |

### 微软GUID的变体规则

微软的GUID生成规则与RFC 4122基本一致，但存在以下特点：

+ 标准变体（10xx）

大多数现代Windows系统生成的GUID（如版本1、4）均使用标准变体，即第四组首字符为 8、9、a 或 b。

+ 非标准变体（110x）

早期Windows系统（如Windows NT 4.0）可能生成变体标识为 110x（十六进制以 c 或 d 开头）的GUID，但这类格式已逐渐淘汰

## ObjectSID

[对象 SID （WinNT 提供程序） - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adsi/object-sid)

[Attribute objectSid | Microsoft Learn](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adls/789fbfb5-2ddb-4121-8126-e594a9a9ecaf)

[将 objectSid 转换为可绑定字符串的示例代码 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/ad/example-code-for-converting-an-objectsid-into-a-bindable-string-)

**objectSid** 属性是用户的安全标识符 (SID)

+ 在与 Windows 安全系统交互时，系统会使用 SID 来识别用户及其组成员身份

+ 该属性为单值， SID 是一个唯一的二进制值，用于将用户识别为安全主体

+ SID 由系统在创建用户时设置，每个用户都有一个由 Windows 域签发的唯一 SID，并存储在目录中用户对象的 **objectSid** 属性中

+ 用户每次登录时，系统都会从目录中检索用户的 SID，并将其放入用户的访问令牌中

+ 用户的 SID 还用于检索用户所属组的 SID，并将其放入用户的访问令牌中。当一个 SID 被用作一个用户或组的唯一标识符时，就不能再用来标识其他的用户或组

### SID的二进制结构

[安全标识符 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-server/identity/ad-ds/manage/understand-security-identifiers)

[SID (winnt.h) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/winnt/ns-winnt-sid)

SID（安全标识符）的二进制格式遵循固定结构，具体如下：

| 字段                | 长度（字节） | 说明                                             |
| ------------------- | ------------ | ------------------------------------------------ |
| Revision            | 1            | 版本号（通常为1）                                |
| SubAuthorityCount   | 1            | 子权威数量（表示后续有多少个子权威值）           |
| IdentifierAuthority | 6            | 标识符权威（高32位的权威值，通常以大端序存储）   |
| SubAuthorities      | 4 × n        | 子权威列表（每个子权威为32位整数，以小端序存储） |

使用标准表示法将 SID 从二进制转换为字符串格式时，SID 的组件更容易可视化：

`S-R-X-Y1-Y2-Yn-1-Yn`

在这种表示法中，下表描述了 SID 的组成部分：

| 组件 | 描述                                  |
| :--- | :------------------------------------ |
| S    | 指示字符串是 SID                      |
| R    | 指示修订级别                          |
| X    | 指示标识符颁发机构值                  |
| Y    | 表示一系列子授权值，其中 n 是值的数目 |

SID 最重要的信息包含在一系列子颁发机构值中，系列的第一部分 (-Y1-Y2-Yn-1) 是域标识符。

子颁发机构值系列中的最后一项 (-Yn) 是 RID，它将一个帐户或组与域中的所有其他帐户和组区分开，在任何域中，两个帐户或组不共享相同的 RID

例如，内置管理员组的 SID 以标准化 SID 表示法表示为以下字符串：

S-1-5-32-544

此 SID 包含四个组成部分：

- 修订级别 (1)
- 标识符颁发机构值（5, NT Authority）
- 域标识符 (32, Builtin)
- RID（544，管理员）

内置帐户和组的 SID 始终具有相同的域标识符值，即 32

### 通用公认 SID

下表列出了通用的已知 SID：

| 通用公认 SID | 名称                        | 标识                                                         |
| :----------- | :-------------------------- | :----------------------------------------------------------- |
| S-1-0-0      | 空 SID                      | 没有成员的组。 当 SID 值未知时，通常使用此值。               |
| S-1-1-0      | 世界                        | 包括所有用户的组。                                           |
| S-1-2-0      | 本地                        | 登录到本地（物理）连接到系统的终端的用户。                   |
| S-1-2-1      | 控制台登录                  | 包括登录到物理控制台的用户的组。                             |
| S-1-3-0      | 创建者所有者 ID             | 要替换为创建新对象的用户的 SID 的 SID。 此 SID 用于可继承的访问控制条目 (ACE)。 |
| S-1-3-1      | 创建者组 ID                 | 要替换为创建新对象的用户的主组 SID 的 SID。 在可继承 ACE 中使用此 Sid. |
| S-1-3-2      | 所有者服务器                | 可继承的 ACE 中的占位符。 继承 ACE 后，系统会将此 SID 替换为对象的所有者服务器的 SID，并存储给定对象或文件的创建者相关信息。 |
| S-1-3-3      | 组服务器                    | 可继承的 ACE 中的占位符。 继承 ACE 后，系统将此 SID 替换为对象的组服务器的 SID。 系统还会存储有关允许使用对象的组的信息。 |
| S-1-3-4      | 所有者权限                  | 表示对象的当前所有者的组。 当承载此 SID 的 ACE 应用于对象时，系统将忽略对象所有者对隐式 READ_CONTROL 和 WRITE_DAC 的标准访问权限。 |
| S-1-4        | 非唯一颁发机构              | 表示标识符颁发机构的 SID。                                   |
| S-1-5        | NT Authority (NT AUTHORITY) | 表示标识符颁发机构的 SID。                                   |
| S-1-5-80-0   | 所有服务                    | 包含系统上配置的所有服务进程的组。 操作系统控制此组的成员身份。 |

更详细内容查阅[安全标识符 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-server/identity/ad-ds/manage/understand-security-identifiers)

## GeneralizedTime

[String（Generalized-Time） 语法 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/adschema/s-string-generalized-time)

[ASN.1 - GeneralizedTime](https://obj-sys.com/asn1tutorial/node14.html)

ASN.1 标准定义的时间字符串格式，使用此语法以通用时间格式存储时间值

Go 中的时间转换：

```go
func GeneralizedTime(entry *ldap.Entry, attribute string) (string, error) {
	b := entry.GetAttributeValue(attribute)

	t, err := time.Parse("20060102150405.0Z", b)
	if err != nil {
		return "", err
	}

	return t.Local().Format(time.DateTime), nil
}
```

## FileTimeToTime

[FileTimeToLocalFileTime 函数 (fileapi.h) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/fileapi/nf-fileapi-filetimetolocalfiletime)

[FileTimeToSystemTime 函数 (timezoneapi.h) - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/timezoneapi/nf-timezoneapi-filetimetosystemtime)

Go 中的时间转换：类似 FileTimeToSystemTime 函数，转换后得到 Go 的 time.Time

```go
// Windows FileTime 时间起点（1601-01-01 00:00:00 UTC）
const (
    // FileTimeToUnixEpochDiff 1601-01-01 到 1970-01-01 的 100 纳秒间隔数
    FileTimeToUnixEpochDiff = 116444736000000000
    // NanoSecondsPerHundredNanoSeconds 100 纳秒单位转换为纳秒的比例因子
    NanoSecondsPerHundredNanoSeconds = 100
)

// FileTimeToTime 将 LDAP 条目中的 Windows FileTime 属性转换为格式化时间字符串
// 支持的属性包括: lastLogon, pwdLastSet, accountExpires 等
// 返回格式: "2006-01-02 15:04:05" (UTC 时间)
func FileTimeToTime(entry *ldap.Entry, attribute string) (string, error) {
    // 参数校验
    if entry == nil {
        return "", fmt.Errorf("ldap entry is nil")
    }
    if attribute == "" {
        return "", fmt.Errorf("attribute name is empty")
    }

    // 获取属性值
    rawValue := entry.GetAttributeValue(attribute)
    if rawValue == "" {
        return "", fmt.Errorf("attribute '%s' not found or is empty", attribute)
    }

    // 转换为整数（Windows FileTime 是 18 位数字字符串）
    fileTime, err := strconv.ParseInt(rawValue, 10, 64)
    if err != nil {
        return "", fmt.Errorf("failed to parse '%s' as int64: %w", attribute, err)
    }

    // 处理特殊值：0 表示从未发生（如未登录）
    if fileTime == 0 {
        return "", fmt.Errorf("zero value for '%s' (never occurred)", attribute)
    }

    // 时间转换逻辑
    var unixNano int64
    if fileTime >= FileTimeToUnixEpochDiff {
        // 正常情况：从 1601-01-01 开始的时间值
        unixNano = (fileTime - FileTimeToUnixEpochDiff) * NanoSecondsPerHundredNanoSeconds
    } else {
        // 异常情况：小于 epochDiff 的值（例如未来时间或无效数据）
        return "", fmt.Errorf("invalid filetime value '%d' for '%s'", fileTime, attribute)
    }

    // 构造 time.Time 对象并格式化输出
    timestamp := time.Unix(0, unixNano).UTC()
    return timestamp.Format("2006-01-02 15:04:05"), nil
}
```
