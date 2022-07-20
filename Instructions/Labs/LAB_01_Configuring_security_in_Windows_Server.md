---
lab:
  title: 实验室：配置 Windows Server 中的安全性
  module: 'Module 1: Windows Server security'
ms.openlocfilehash: e3ff4a435b249f1688749a3992cb3375136f046c
ms.sourcegitcommit: d2e9d886e710729f554d2ba62d1abe3c3f65fcb6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2022
ms.locfileid: "147046997"
---
# <a name="lab-configuring-security-in-windows-server"></a>实验室：配置 Windows Server 中的安全性

## <a name="scenario"></a>场景

Contoso Pharmaceuticals 是一家在世界各地约有 5000 名员工的医疗研究公司。 它们具有确保医疗记录和数据保密的特定需求。 公司有一个总部和多个全球分部。 Contoso 最近部署了一个 Windows Server 和 Windows 客户端基础结构。 你的任务是在服务器安全配置中实现改进。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 配置 Windows Defender Credential Guard。
- 找到有问题的用户帐户。
- 实现并验证 LAPS（本地管理员密码解决方案）。

## <a name="estimated-time-40-minutes"></a>估计时间：40 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 必须运行  。 其他 VM 可以运行，但本实验室不需要这些 VM。

> 注意：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 虚拟机托管 SEA-DC1、SEA-SVR1 和 SEA-SVR2 的安装      。

1. 选择“SEA-SVR2”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境。

## <a name="exercise-1-configuring-windows-defender-credential-guard"></a>练习 1：配置 Windows Defender Credential Guard

### <a name="scenario"></a>场景

你决定在服务器和管理工作站上实现 Windows Defender Credential Guard，以防范通过哈希传递攻击和票证传递攻击盗窃凭据。 你将使用组策略在现有服务器上启用 Credential Guard。 对于所有新服务器，你将使用 虚拟机监控程序 保护的代码完整性 (HVCI) 和 Windows Defender Credential Guard 硬件准备工具在新服务器加入域之前启用 Credential Guard。

在本实验室中，你将设置组策略并在现有服务器上运行 HVCI 和 Windows Defender Credential Guard 硬件准备工具。

> 注意：在实验室环境中，Credential Guard 在 VM 上不可用，因为它们不符合先决条件。 但这并不妨碍你使用组策略逐步执行实现过程，并使用相应的工具评估其准备情况。

本练习的主要任务是：

1. 使用组策略启用 Windows Defender Credential Guard。
1. 使用 HVCI 和 Windows Defender Credential Guard 硬件准备工具启用 Windows Defender Credential Guard。

#### <a name="task-1-enable-windows-defender-credential-guard-using-group-policy"></a>任务 1：使用组策略启用 Windows Defender Credential Guard

1. 在 SEA-SVR2 上，打开“组策略管理”控制台 。
1. 在“组策略管理”控制台中，浏览“林: contoso.com”、“域”、“contoso.com”，然后创建一个名为 CredentialGuard_GPO 且链接到“IT”组织单位 (OU) 的组策略对象 (GPO)     。
1. 在组策略管理编辑器中打开“CredentialGuard_GPO”，浏览到 Computer Configuration\\Policies\\Administrative Templates\\\\Device Guard 节点 。
1. 启用“打开基于虚拟化的安全”选项并采用以下设置：

   - 选择平台安全级别：安全启动和 DMA 保护。
   - Credential Guard 配置：使用 UEFI 锁启用。
   - 安全启动配置：已启用

1. 关闭“组策略管理编辑器”窗口。
1. 关闭“组策略管理”控制台窗口。

#### <a name="task-2-enable-windows-defender-credential-guard-using-the-hvci-and-windows-defender-credential-guard-hardware-readiness-tool"></a>任务 2：使用 HVCI 和 Windows Defender Credential Guard 硬件准备工具启用 Windows Defender Credential Guard

1. 在 SEA-SVR2 上，以管理员身份启动 Windows PowerShell。
1. 若要运行 HVCI 和 Windows Defender Credential Guard 硬件准备工具，请在 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   Set-Location -Path C:\Labfiles\Lab01\
   .\DG_Readiness_Tool.ps1 -Enable -AutoReboot
   ```

1. 等到工具完成运行，并按照提示重新启动操作系统。
1. 重启完成后，使用密码 Pa55w.rd 以 CONTOSO\\Administrator 身份重新登录 SEA-SVR2  。

### <a name="results"></a>结果

完成本练习后，你将完成以下操作：

1. 使用组策略在组织中的所有计算机上实现 Windows Defender Credential Guard。
1. 在本地计算机上立即启用 Windows Defender Credential Guard。

## <a name="exercise-2-locating-problematic-accounts"></a>练习 2：查找有问题的帐户

### <a name="scenario"></a>场景

你需要检查组织的用户帐户的密码是否配置为不过期，并修正此设置。 还需要检查 90天或更长时间未登录的帐户，并将其禁用。

本练习的主要任务是：

1. 查找并重新配置具有不过期密码的域帐户。
1. 找到并禁用在至少 90 天内未用于登录的域帐户。

#### <a name="task-1-locate-and-reconfigure-domain-accounts-with-non-expiring-passwords"></a>任务 1：查找并重新配置具有不过期密码的域帐户

1. 在 SEA-SVR2 上，以管理员身份启动 Windows Powershell。
1. 若要列出 Active Directory 启用的具有不过期密码的用户帐户，请在 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true}
   ```

1. 查看返回的用户帐户列表。
1. 若要在结果集中为所有用户帐户启用密码过期，请在 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true} | Set-ADUser -PasswordNeverExpires $false
   ```

1. 若要验证结果，请重新运行步骤 2 中的命令，注意未返回任何结果。

#### <a name="task-2-locate-and-disable-domain-accounts-that-have-not-been-used-to-sign-in-for-at-least-90-days"></a>任务 2：找到并禁用在至少 90 天内未用于登录的域帐户

1. 若要找到在至少 90 天内未用于登录的 Active Directory 用户帐户，请在 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   $days = (Get-Date).AddDays(-90)
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp
   ```

   > 注意：在实验室环境中，不会返回任何结果。

1. 若要禁用在至少 90 天内未用于登录的 Active Directory 用户帐户，请运行以下命令：

   ```powershell
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp | Disable-ADAccount
   ```

   > 注意：在实验室环境中，不会返回任何结果。

## <a name="exercise-3-implementing-laps"></a>练习 3：实现 LAPS

### <a name="scenario"></a>场景

目前，在 Contoso 的所有服务器和工作站上使用相同的本地管理员帐户密码。 若要解决此问题，你将配置和部署 LAP。

本练习的主要任务是：

1. 准备用于实现 LAPS（本地管理员密码解决方案）的计算机帐户。
1. 为 LAPS 准备 Active Directory (AD DS)。
1. 部署 LAPS 客户端扩展。
1. 验证 LAPS。

#### <a name="task-1-prepare-computer-accounts-for-implementing-laps-local-administrator-password-solution"></a>任务 1：准备计算机帐户来实现 LAPS（本地管理员密码解决方案）

1. 若要创建指定的 OU，并将 SEA-SVR1 计算机帐户移到其上，请在 SEA-SVR2 上的 Windows PowerShell 命令提示符处运行以下命令 ：

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle_Servers"
   Get-ADComputer SEA-SVR1 | Move-ADObject –TargetPath "OU=Seattle_Servers,DC=Contoso,DC=com"
   ```

1. 若要安装 LAPS，请在 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   Msiexec /i C:\Labfiles\Lab01\LAPS.x64.msi
   ```

1. 出现提示时，请使用默认设置逐步执行“欢迎使用本地管理员密码解决方案”安装向导，直到进入“自定义安装”页 。
1. 在“本地管理员密码解决方案”安装向导的“自定义安装”页上，在“管理工具”旁边的下拉菜单中，选择“整个功能将安装在本地硬盘上”，并通过接受默认设置并启动安装程序来完成该向导   。 
1. 等待安装完成。 
1. 若要启用 Windows Defender 防火墙并使用高级安全规则允许来自其他加入域的服务器的传入服务器消息块 (SMB) 连接，请在 Windows PowerShell 命令提示符处输入以下命令，并在每个命令之后按 Enter 键：

   ```
   $rule = Get-NetFirewallRule | Where-Object DisplayName -eq 'File and Printer Sharing (SMB-In)' 
   $rule | Set-NetFirewallRule -Profile Domain
   $rule | Enable-NetFirewallRule
   ```

   > 注意：本实验室稍后需要从 SEA-SVR1 连接到 SEA-SVR2  。

#### <a name="task-2-prepare-ad-ds-for-laps"></a>任务 2：为 LAPS 准备 AD DS

1. 若要为 LAPS 准备域，请在 SEA-SVR2 上的 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   Import-Module admpwd.ps
   Update-AdmPwdADSchema
   Set-AdmPwdComputerSelfPermission -Identity "Seattle_Servers"
   ```

1. 在 SEA-SVR2* 上，打开“组策略管理”控制台。
1. 在“组策略管理”控制台中，浏览“林: contoso.com”、“域”、“contoso.com”，然后创建一个名为 LAPS_GPO 且链接到 Seattle_Servers OU 的组策略对象 (GPO)     。
1. 在组策略管理编辑器中打开 LAPS_GPO，并浏览到 Computer Configuration\\Policies\\Administrative Templates\\LAPS 节点 。
1. 启用“启用本地管理员密码管理”选项。
1. 使用以下设置配置“密码设置”选项：

   - 选择平台安全级别：安全启动和 DMA 保护。
   - Credential Guard 配置：使用 UEFI 锁启用。
   - 安全启动配置：已启用

1. 关闭“组策略管理编辑器窗口”。

   - 密码长度：20
   - 密码期限（天）：30

1. 关闭“组策略管理编辑器”。
1. 关闭“组策略管理”控制台窗口。

#### <a name="task-3-deploy-laps-client-side-extension"></a>任务 3：部署 LAPS 客户端扩展

1. 将控制台会话切换到 SEA-SVR1，然后根据需要使用密码 Pa55w.rd 以 CONTOSO\\Administrator 身份登录  。

   > 注意：系统会提示你更改密码，因为上一练习中运行了启用密码过期的脚本。 选择任意密码，并在整个实验室的其余部分中使用它。

1. 登录后，若要访问 Windows PowerShell 命令提示符，请退出“SConfig”菜单。
1. 若要使用默认设置以无提示方式安装 LAPS，请在 Windows PowerShell 命令提示符处，运行以下命令：

   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/i \\SEA-SVR2.contoso.com\c$\Labfiles\Lab01\LAPS.x64.msi /quiet'
   ```

1. 若要触发将在本地应用 LAPS 设置的组策略处理，请在 Windows PowerShell 命令提示符处运行以下命令：

   ```powershell
   gpupdate /force
   ```

#### <a name="task-4-verify-laps"></a>任务 4：验证 LAPS

1. 将控制台会话切换 SEA-SVR2。
1. 在“开始”菜单中，启动 LAPS UI 。
1. 在“LAPS UI”对话框的“ComputerName”文本框中，输入“SEA-SVR1”，然后选择“搜索”   。
1. 查看“密码”和“密码过期”值，然后选择“退出”  。
1. 切换到 Windows PowerShell 控制台，然后在 Windows PowerShell 命令提示符处运行以下命令以验证密码的值：

   ```powershell
   Get-ADComputer -Identity SEA-SVR1 -Properties ms-Mcs-AdmPwd
   ```

1. 查看分配给 SEA-SVR1 的密码，并注意它与 LAPS UI 工具中显示的密码匹配。

   > 注意：在本例中，密码的值括在一对大括号中。

### <a name="results"></a>结果

完成本练习后，你将完成以下操作：

- 为 LAPS 准备 OU 和计算机帐户。
- 为 LAPS 准备 AD DS。
- 部署 LAPS 客户端扩展。
- 验证你已成功实现 LAPS。
