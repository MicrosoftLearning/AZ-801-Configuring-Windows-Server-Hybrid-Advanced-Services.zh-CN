---
lab:
  title: 实验室：配置 Windows Server 中的安全性
  type: Answer Key
  module: 'Module 1: Windows Server security'
---

# <a name="lab-answer-key-configuring-security-in-windows-server"></a>实验室解答：配置 Windows Server 中的安全性

## <a name="exercise-1-configuring-windows-defender-credential-guard"></a>练习 1：配置 Windows Defender Credential Guard

> 备注：在实验室环境中，Credential Guard 在 VM 上不可用，因为它们不符合先决条件。 但这并不妨碍你使用组策略逐步执行实现过程，并使用相应的工具评估其准备情况。

#### <a name="task-1-enable-windows-defender-credential-guard-using-group-policy"></a>任务 1：使用组策略启用 Windows Defender Credential Guard

1. 连接到 SEA-SVR2，并根据需要，以 CONTOSO\\Administrator 的身份使用密码 Pa55w.rd 登录  。
1. 在“启动”按钮旁的“在此键入进行搜索”文本框中，输入“组策略管理”  。
1. 在结果列表中，选择“组策略管理”。
1. 在“组策略管理”控制台中，依次展开“林: contoso.com”、“域”和“contoso.com”，右键单击或访问“IT”组织单位 (OU) 的上下文菜单，然后选择“在这个域中创建 GPO 并在此处链接”      。
1. 在“新建 GPO”对话框中，在“名称”文本框中输入“CredentialGuard_GPO”，然后选择“确定”   。
1. 在“组策略管理”窗口中，在“IT”下，右键单击或访问“CredentialGuard_GPO”的“上下文”菜单，然后选择“编辑”    。
1. 在“组策略管理编辑器”中，浏览到 Computer Configuration\\Policies\\Administrative Templates\\System\\Device Guard。 
1. 选择“打开基于虚拟化的安全”，然后选择“策略设置”链接 。
1. 在“打开基于虚拟化的安全”窗口中，选择“已启用”选项 。
1. 在“选择平台安全级别”下拉列表中，确保已选中“安全启动和 DMA 保护”条目 。
1. 在“Credential Guard 配置”下拉列表中，选择“已使用 UEFI 锁启用”条目 。
1. 在“安全启动配置”下拉列表中，选择“已启用”条目，然后选择“确定”  。
1. 关闭“组策略管理编辑器”窗口。
1. 关闭“组策略管理”控制台窗口。

#### <a name="task-2-enable-windows-defender-credential-guard-using-the-hypervisor-protected-code-integrity-hvci-and-windows-defender-credential-guard-hardware-readiness-tool"></a>任务 2：使用虚拟机监控程序保护的代码完整性 (HVCI) 和 Windows Defender Credential Guard 硬件准备工具启用 Windows Defender Credential Guard

1. 在 SEA-SVR2 上，选择“开始”，右键单击或访问“Windows PowerShell”的“上下文”菜单，然后选择“以管理员身份运行”    。
1. 若要运行 HVCI 和 Windows Defender Credential Guard 硬件准备工具，请在 Windows PowerShell 命令提示符处输入以下命令，在第一个提示符处选择“[R] 运行一次”，然后按 Enter 以获取其余提示：

   ```powershell
   Set-Location -Path C:\Labfiles\Lab01\
   .\DG_Readiness_Tool.ps1 -Enable -AutoReboot
   ```

1. 等待该工具完成运行，当出现提示时，在“你即将注销”对话框中，选择“确定” 。

   > 备注：操作系统将重启。 

1. 重启完成后，使用密码 Pa55w.rd 以 CONTOSO\\Administrator 的身份重新登录 SEA-SVR2  。

## <a name="exercise-2-locating-problematic-accounts"></a>练习 2：查找有问题的帐户

#### <a name="task-1-locate-and-reconfigure-domain-accounts-with-non-expiring-passwords"></a>任务 1：查找并重新配置具有不过期密码的域帐户

1. 在 SEA-SVR2 上，选择“开始”，右键单击或访问“Windows PowerShell”的“上下文”菜单，然后选择“以管理员身份运行”    。
1. 若要列出 Active Directory 启用的具有不过期密码的用户帐户，请在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键：

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true}
   ```

1. 查看返回的用户帐户列表。
1. 若要在结果集中为所有用户帐户启用密码过期，请在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键：

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true} | Set-ADUser -PasswordNeverExpires $false
   ```

1. 若要验证结果，请重新运行步骤 2 中的命令，注意未返回任何结果。

#### <a name="task-2-locate-and-disable-domain-accounts-that-have-not-been-used-to-sign-in-for-at-least-90-days"></a>任务 2：找到并禁用在至少 90 天内未用于登录的域帐户

1. 若要识别在至少 90 天内未用于登录的 Active Directory 用户帐户，请在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键：

   ```powershell
   $days = (Get-Date).AddDays(-90)
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp
   ```

   > 备注：在实验室环境中，不会返回任何结果。

1. 若要禁用在至少 90 天内未用于登录的 Active Directory 用户帐户，请输入以下命令并按 Enter 键：

   ```powershell
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp | Disable-ADAccount
   ```

   > 备注：在实验室环境中，不会返回任何结果。

## <a name="exercise-3-implementing-laps"></a>练习 3：实现 LAPS

#### <a name="task-1-prepare-computer-accounts-for-implementing-laps-local-administrator-password-solution"></a>任务 1：准备计算机帐户来实现 LAPS（本地管理员密码解决方案）

1. 若要创建指定的 OU 并将 SEA-SVR1 计算机帐户移到其上，请在 SEA-SVR2 上的 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键 ： 

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle_Servers"
   Get-ADComputer SEA-SVR1 | Move-ADObject –TargetPath "OU=Seattle_Servers,DC=Contoso,DC=com"
   ```

1. 若要安装 LAPS，请在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键：

   ```powershell
   Msiexec /i C:\Labfiles\Lab01\LAPS.x64.msi
   ```

1. 在“本地管理员密码解决方案安装”向导的“欢迎使用本地管理员密码解决方案安装向导”页上，选择“下一步”  。
1. 在“本地管理员密码解决方案安装”向导的“最终用户许可协议”页上，选择“我接受许可协议中的条款”，然后选择“下一步”   。
1. 在“本地管理员密码解决方案安装”向导的“自定义安装”页上，在“管理工具”旁边的下拉菜单中，选择“整个功能将安装在本地硬盘上”，然后选择“下一步”    。
1. 在“本地管理员密码解决方案安装”向导的“准备安装本地管理员密码解决方案”页上，选择“安装”  。 
1. 安装完成后，在“本地管理员密码解决方案安装”向导的最后一页上，选择“完成” 。
1. 若要启用 Windows Defender 防火墙并使用高级安全规则允许来自其他加入域的服务器的传入服务器消息块 (SMB) 连接，请在 Windows PowerShell 命令提示符处输入以下命令，并在输入每条命令后按 Enter 键：

   ```
   $rule = Get-NetFirewallRule | Where-Object DisplayName -eq 'File and Printer Sharing (SMB-In)' 
   $rule | Set-NetFirewallRule -Profile Domain
   $rule | Enable-NetFirewallRule
   ```

   > 注意：本实验室稍后需要从 SEA-SVR1 连接到 SEA-SVR2  。

#### <a name="task-2-prepare-ad-ds-for-laps"></a>任务 2：为 LAPS 准备 AD DS

1. 若要为 LAPS 准备域，请在 SEA-SVR2 上的 Windows PowerShell 命令提示符处输入以下命令，并在输入每条命令后按 Enter 键：

   ```powershell
   Import-Module admpwd.ps
   Update-AdmPwdADSchema
   Set-AdmPwdComputerSelfPermission -Identity "Seattle_Servers"
   ```

1. 在 SEA-SVR2 上，在“启动”按钮旁的“在此键入进行搜索”文本框中，输入“组策略管理”   。
1. 在结果列表中，选择“组策略管理”。
1. 在“组策略管理”控制台中，依次展开“林: contoso.com”、“域”和“contoso.com”，右键单击或访问“Seattle_Servers”OU 的上下文菜单，然后选择“在这个域中创建 GPO 并在此处链接”      。
1. 在“新建 GPO”对话框中，在“名称”文本框中输入“LAPS_GPO”，然后选择“确定”   。
1. 在“组策略管理”窗口中，在“Seattle_Servers”下，右键单击或访问“LAPS_GPO”的“上下文”菜单，然后选择“编辑”    。
1. 在“组策略管理编辑器”窗口中，在“Computer Configuration”下，依次展开“策略”节点和“管理模板”节点，然后选择“LAPS”    。
1. 选择“启用本地管理员密码管理”策略，然后选择“策略设置”链接 。
1. 在“启用本地管理员密码管理”窗口中，选择“已启用”，然后选择“确定”  。
1. 选择“密码设置”策略，然后选择“策略设置”链接 。
1. 在“密码设置”策略对话框中，选择“已启用”，然后将“密码长度”配置为“20”   。
1. 验证“密码期限(天)”已配置为“30”，然后选择“确定”  。
1. 关闭“组策略管理编辑器”。

#### <a name="task-3-deploy-laps-client-side-extension"></a>任务 3：部署 LAPS 客户端扩展

1. 将控制台会话切换到 SEA-SVR1，然后根据需要使用密码 Pa55w.rd 以 CONTOSO\\Administrator 的身份登录  。

   > 备注：系统会提示你更改密码，因为上一练习中运行了启用密码过期的脚本。 选择任意密码，并在整个实验室的其余部分中使用它。

1. 登录后，若要访问 Windows PowerShell 命令提示符，请在“SConfig”菜单提示符处，输入“15”并按 Enter 键 。
1. 若要使用默认设置以无提示方式安装 LAPS，请在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键：

   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/i \\SEA-SVR2.contoso.com\c$\Labfiles\Lab01\LAPS.x64.msi /quiet'
   ```

1. 若要触发将在本地应用 LAPS 设置的组策略处理，请在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键：

   ```powershell
   gpupdate /force
   ```

#### <a name="task-4-verify-laps"></a>任务 4：验证 LAPS

1. 将控制台会话切换到 SEA-SVR2。
1. 选择“开始”。 在“开始”菜单中，选择“LAPS”，然后选择“LAPS UI”  。
1. 在“LAPS UI”对话框的“计算机名”文本框中，输入“SEA-SVR1”，然后选择“搜索”   。
1. 查看“密码”和“密码过期”值，然后选择“退出”  。
1. 切换到 Windows PowerShell 控制台，然后在 Windows PowerShell 命令提示符处输入以下命令并按 Enter 键以验证密码的值：

   ```powershell
   Get-ADComputer -Identity SEA-SVR1 -Properties ms-Mcs-AdmPwd
   ```

1. 查看分配给 SEA-SVR1 的密码，并注意它与 LAPS UI 工具中显示的密码匹配。

   > 备注：在本例中，密码的值括在一对大括号中。
