---
lab:
  title: 实验室：实现 Hyper-V 副本和 Windows Server 备份
  module: 'Module 4: Disaster Recovery in Windows Server'
---

# 实验室：实现 Hyper-V 副本和 Windows Server 备份

## 场景

你在 Contoso, Ltd. 担任管理员。Contoso 希望评估和配置新的灾难恢复与备份的功能和技术。 作为系统管理员，你的任务是执行该评估和实现。 你决定评估 Hyper-V 副本和 Windows Server 备份。

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Implementing%20Hyper-V%20Replica%20and%20Windows%20Server%20Backup)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 目标

完成本实验室后，你将能够：

- 配置和实现 **Hyper-V 副本**。
- 使用 Windows Server 备份配置和实现备份。

## 估计时间：45 分钟

## 实验室设置

虚拟机：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 必须运行  。 其他 VM 可以运行，但此实验室不需要这些 VM。

> 备注：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 虚拟机托管 SEA-DC1、SEA-SVR1 和 SEA-SVR2 的安装

1. 选择 SEA-SVR2。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境。

## 练习 1：实现 Hyper-V 副本

### 场景

在开始群集部署之前，你已决定评估 Hyper-V 中用于在主机之间复制 VM 的新技术。 如果活动副本或主机出现故障，你希望能够在另一台主机上手动装载 VM 的副本。

本练习的主要任务是：

1. 安装和配置 Hyper-v 副本。
1. 配置 Hyper-V 复制。
1. 验证故障转移。

#### 任务 1：安装和配置 Hyper-V 副本

1. 在 SEA-SVR2 上，以管理员身份启动 Windows PowerShell。
1. 要识别 SEA-SVR2 上具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙的状态，请在 Windows PowerShell 提示符处运行以下命令 ：

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. 要在 SEA-SVR2 上启用具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙，请在 Windows PowerShell 提示符处运行以下命令 ：

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. 若要将 SEA-SVR2 配置为 Hyper-V 副本的副本服务器，请运行以下命令 ：

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -KerberosAuthenticationPort 8080 -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

1. 要验证 SEA-SVR2 是否配置为 Hyper-V 副本的副本服务器，请运行以下命令 ：

   ```powershell
   Get-VMReplicationServer
   ```
  
   > 注意：验证命令的输出是否包括以下设置：

   - **RepEnabled：True**
   - **AuthType：Kerb**
   - **KerAuthPort：8080**
   - **CertAuthPort：443**
   - **AllowAnyServer：True**

1. 若要确定 SEA-SVR2 上存在的虚拟机，请在 Windows PowerShell 提示符处运行以下命令：

   ```powershell
   Get-VM
   ```

   > 注意：验证命令的输出是否包含 SEA-CORE1 。 

   > 注意：请将“Administrator: Windows PowerShell”窗口保持打开状态 。

1. 在 SEA-SVR2 上，以管理员身份打开另一个 Windows PowerShell 窗口。
1. 要建立与 SEA-SVR1 的 PowerShell 远程会话，请在新打开的 Windows PowerShell 窗口中输入以下命令，然后按 Enter 键：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```
 
   > 注意：可以根据包含 [SEA-SVR1.contoso.com] 前缀的 PowerShell 提示来识别 PowerShell 远程会话 。

1. 要识别 SEA-SVR1 上具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙的状态，请在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令  ：

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

   > 注意：请查看输出并验证 Enabled 属性是否设置为 False  。 要使用 Hyper-V 副本，需要启用此防火墙规则。

1. 要在 SEA-SVR1 上启用高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙，请在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令  ：

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. 要将 SEA-SVR1 配置为 Hyper-V 副本的副本服务器，请在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令  ：

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

   > 注意：请将第二个“Administrator: Windows PowerShell”窗口保持打开状态 。

#### 任务 2：配置 Hyper-V 复制

1. 在 SEA-SVR2 上，切换到显示本地 PowerShell 会话的“Administrator: Windows PowerShell”窗口 。
1. 要将虚拟机 SEA-CORE1 从 SEA-SVR2 复制到 SEA-SVR1，请在 SEA-SVR2 上，在本地会话的 Windows PowerShell 提示符处运行以下命令   ：

   ```powershell
   Enable-VMReplication SEA-CORE1 -ReplicaServerName SEA-SVR1.contoso.com -ReplicaServerPort 80 -AuthenticationType Kerberos -ComputerName SEA-SVR2.contoso.com
   ```

1. 要开始将虚拟机 SEA-CORE1 从 SEA-SVR2 复制到 SEA-SVR1，请在 SEA-SVR2 上运行以下命令   ：

   ```powershell
   Start-VMInitialReplication SEA-CORE1
   ```

1. 要识别虚拟机 SEA-CORE1 从 SEA-SVR2 到 SEA-SVR1 的复制是否已成功开始，请运行以下命令  ：

   ```powershell
   Get-VMReplication
   ```

   > 注意：在命令的输出中，识别 State 值并验证它是否列为 InitialReplicationInProgress  。 等待大约 5 分钟，重新运行相同的命令，并验证 State 值是否更改为 Replicating 。 等待此操作完成，然后再继续执行后续步骤。 此外，确保主服务器列为 SEA-SVR2，副本服务器列为 SEA-SVR1   。

1. 在 SEA-SVR2 上，切换到显示 SEA-SVR1 的 PowerShell 远程会话的“Administrator: Windows PowerShell”窗口  。
1. 要验证 SEA-CORE1 的副本是否存在于 SEA-SVR1 上，请在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令  ：

   ```powershell
   Get-VM
   ```

   > 注意：请确保命令的输出列出 SEA-CORE1 。

   > 注意：请让两个 Windows PowerShell 会话保持打开状态。

##### 任务 3：验证故障转移

1. 在 SEA-SVR2 上，切换到显示本地 PowerShell 会话的“Administrator: Windows PowerShell”窗口 。
1. 要准备将 SEA-CORE1 虚拟机故障转移到 SEA-SVR1，请在 SEA-SVR2 上，在托管本地会话的 Windows PowerShell 窗口中，运行以下命令  ：

   ```powershell
   Start-VMFailover -Prepare -VMName SEA-CORE1 -ComputerName SEA-SVR2.contoso.com
   ```

   > 注意：出现提示时，输入 Y，然后按 Enter 。 此命令通过触发任何挂起更改的复制来为 SEA-CORE1 的计划内故障转移做准备。

1. 在 SEA-SVR2 上，切换到显示 SEA-SVR1 的 PowerShell 远程会话的“Administrator: Windows PowerShell”窗口  。
1. 要启动 SEA-CORE1 虚拟机到 SEA-SVR1 的故障转移，请在 SEA-SVR2 上，在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令   ：

   ```powershell
   Start-VMFailover -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. 要将副本 VM 配置为主 VM，请在 SEA-SVR2 上，在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令 ：

   ```powershell
   Set-VMReplication -Reverse -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. 要在 SEA-SVR1 上启动新指定的主 VM，请在 SEA-SVR2 上，在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令  ：

   ```powershell
   Start-VM -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. 要验证 VM 是否已成功启动，请在 SEA-SVR2 上，在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令 ：

   ```powershell
   Get-VM
   ```

   > 注意：在结果表中，验证 State 是否列为 Running  。

1. 要识别虚拟机 SEA-CORE1 从 SEA-SVR1 到 SEA-SVR2 的复制状态，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令    ：

   ```powershell
   Get-VMReplication
   ```

   > 注意：在命令的输出中，识别 State 值并验证它是否列为 Replicating  。 此外，确保主服务器列为 SEA-SVR1，副本服务器列为 SEA-SVR2   。

1. 要在主服务器上停止复制 VM，请在 SEA-SVR2 上，在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令 ：

   ```powershell
   Stop-VM -VMName SEA-CORE1
   ```

1. 将两个 Windows PowerShell 窗口保持打开状态。

   > 注意：如果要使用图形工具验证本练习的结果，可以在 SEA-SVR2 上使用 Hyper-V 管理器，然后将 SEA-SVR1 和 SEA-SVR2 服务器添加到 Hyper-V 控制台    。 然后，你可以验证 SEA-CORE1 VM 是否存在于 SEA-SVR1 和 SEA-SVR2 上，并且正在运行从 SEA-SVR2 到 SEA-SVR1 的复制    。

## 练习 2：使用 Windows Server Backup 实现备份和还原

### 场景

你需要评估成员服务器的 Windows Server 备份。 你已决定配置 SEA-SVR2 服务器的 Windows Server 备份，并尝试备份到 SEA-SVR2 上的网络共享 。

本练习的主要任务是：

1. 配置 Windows Server 备份设置。
1. 备份到网络共享。

#### 任务 1：配置 Windows Server 备份设置

1. 在 SEA-SVR2 上，使用文件资源管理器在 SEA-SVR2 上创建 C:\\BackupShare 文件夹  。 共享文件夹，使经过身份验证的用户具有读/写权限。
1. 在 SEA-SVR2 上，切换到显示 SEA-SVR1 的 PowerShell 远程会话的“Administrator: Windows PowerShell”窗口  。
1. 在 SEA-SVR1 的 PowerShell 远程会话中，使用 Install-WindowsFeature cmdlet 在 SEA-SVR1 上安装 Windows Server 备份功能   。
1. 在 SEA-SVR1 的 PowerShell 远程会话中，使用 wbadmin /? 和 Get-Command 命令查看 wbadmin 实用工具的功能和 WindowsServerBackup 模块的 cmdlet  。

#### 任务 2：备份到网络共享

1. 在 SEA-SVR2 中，使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令，创建要在 SEA-SVR1 上备份的文件夹和文件  ：

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Files' -Force
   fsutil file createnew C:\Files\report1.docx 25432108
   fsutil file createnew C:\Files\report2.docx 25432108
   fsutil file createnew C:\Files\report3.docx 25432108
   fsutil file createnew C:\Files\report4.docx 25432108
   ```

1. 要使用 Windows Server 备份定义备份策略的变量和要备份的文件路径，请在 SEA-SVR2 上，在托管 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，运行以下命令 ：

   ```powershell
   $policy = New-WBPolicy
   $fileSpec = New-WBFileSpec -FileSpec 'C:\Files'
   ```

1. 要定义一个引用你在上一步中定义的变量的 Windows Server 备份策略，请在 SEA-SVR2 上使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令 ：

   ```powershell
   Add-WBFileSpec -Policy $policy -FileSpec $fileSpec
   ```

1. 要在 SEA-SVR2 上使用你在上一个任务中创建的网络共享配置一个备份位置，请在 SEA-SVR2 上使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令（当提示登录时，输入用户名 CONTOSO\\Administrator 和密码 Pa55w.rd）    ：

   ```powershell
   $cred = Get-Credential
   $networkBackupLocation = New-WBBackupTarget -NetworkPath "\\SEA-SVR2.contoso.com\BackupShare" -Credential $cred
   ```

1. 要将备份位置添加到备份策略，请使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令：

   ```powershell
   Add-WBBackupTarget -Policy $policy -Target $networkBackupLocation
   ```

1. 要启用卷影复制服务，请使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令：

   ```powershell
   Set-WBVssBackupOptions -Policy $policy -VssCopyBackup
   ```

1. 要启动备份作业，请使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令：

   ```powershell
   Start-WBBackup -Policy $policy
   ```

   > 注意：请等待备份完成。 这大约需要 1 分钟。

1. 在 SEA-SVR2 上，切换到文件资源管理器，浏览到 C:\\BackupShare，然后验证该文件夹是否包含 WindowsImageBackup 子文件夹中新创建的备份  。
