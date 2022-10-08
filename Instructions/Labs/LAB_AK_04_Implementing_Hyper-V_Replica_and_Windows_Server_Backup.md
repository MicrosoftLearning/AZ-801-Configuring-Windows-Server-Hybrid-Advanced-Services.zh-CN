---
lab:
  title: 实验室：实现 Hyper-V 副本和 Windows Server 备份
  type: Answer Key
  module: 'Module 4: Disaster Recovery in Windows Server'
---

# <a name="lab-answer-key-implementing-hyper-v-replica-and-windows-server-backup"></a>实验室答案密钥：实现 Hyper-V 副本和 Windows Server 备份

## <a name="exercise1-implementing-hyper-v-replica"></a>练习 1：实现 Hyper-V 副本

#### <a name="task-1-install-and-configure-hyper-v-replica"></a>任务 1：安装和配置 Hyper-v 副本

1. 连接到 SEA-SVR2，然后，根据需要，以 Contoso\\Administrator 身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-SVR2 上，选择“开始”，然后选择 Windows PowerShell (Admin)  。
1. 要识别 SEA-SVR2 上具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙的状态，请在 Windows PowerShell 提示符下，输入以下命令并按 Enter ：

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. 要在 SEA-SVR2 上启用具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙，请输入以下命令并按 Enter ：

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. 要将 SEA-SVR2 配置为 Hyper-V 副本的副本服务器，请输入以下命令，并在输入每个命令后按 Enter ：

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -KerberosAuthenticationPort 8080 -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

1. 要验证 SEA-SVR2 是否配置为 Hyper-V 副本的副本服务器，请输入以下命令并按 Enter ：

   ```powershell
   Get-VMReplicationServer
   ```
  
   > 备注：请验证命令的输出是否包含以下设置：

   - **RepEnabled：True**
   - **AuthType：Kerb**
   - **KerAuthPort：8080**
   - **CertAuthPort：443**
   - AllowAnyServer：True

1. 要识别 SEA-SVR2 上存在的虚拟机，请输入以下命令并按 Enter：

   ```powershell
   Get-VM
   ```

   > 备注：请验证命令的输出是否包含 SEA-CORE1 。 

   > 备注：请让“Administrator: Windows PowerShell”窗口保持打开状态 。

1. 要打开另一个“Administrator: Windows PowerShell”窗口，请在 SEA-SVR2 上，选择“开始”并选择 Windows PowerShell (Admin)   。
1. 要建立与 SEA-SVR1 的 PowerShell 远程会话，请在新打开的 Windows PowerShell 窗口中输入以下命令，然后按 Enter：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```
 
   > 备注：可以根据包含 [SEA-SVR1.contoso.com] 前缀的 PowerShell 提示来识别 PowerShell 远程会话 。

1. 要识别 SEA-SVR1 上具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙的状态，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令，然后按 Enter  ：

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

   > 备注：请查看输出并验证 Enabled 属性是否设置为 False  。 要使用 Hyper-V 副本，需要启用此防火墙规则。

1. 要在 SEA-SVR1 上启用具有高级安全性 Hyper-V 副本 HTTP 侦听器 (TCP-In) 规则的 Windows Defender 防火墙，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter  ：

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. 要将 SEA-SVR1 配置为 Hyper-V 副本的副本服务器，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令，并在输入每个命令后按 Enter  ：

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

   > 备注：请让第二个“Administrator: Windows PowerShell”窗口保持打开状态 。

#### <a name="task-2-configure-hyper-v-replication"></a>任务 2：配置 Hyper-V 复制

1. 在 SEA-SVR2 上，切换到显示本地 PowerShell 会话的“Administrator: Windows PowerShell”窗口 。
1. 要将虚拟机 SEA-CORE1 从 SEA-SVR2 复制到 SEA-SVR1，请在 SEA-SVR2 上，在本地会话的 Windows PowerShell 提示符下，输入以下命令并按 Enter   ：

   ```powershell
   Enable-VMReplication SEA-CORE1 -ReplicaServerName SEA-SVR1.contoso.com -ReplicaServerPort 80 -AuthenticationType Kerberos -ComputerName SEA-SVR2.contoso.com
   ```

1. 要开始将虚拟机 SEA-CORE1 从 SEA-SVR2 复制到 SEA-SVR1，请在 SEA-SVR2 上，输入以下命令并按 Enter   ：

   ```powershell
   Start-VMInitialReplication SEA-CORE1
   ```

1. 要识别虚拟机 SEA-CORE1 从 SEA-SVR2 到 SEA-SVR1 的复制是否已成功开始，请在 SEA-SVR2 上，输入以下命令并按 Enter   ：

   ```powershell
   Get-VMReplication
   ```

   > 备注：在命令的输出中，识别 State 值并验证它是否列为 InitialReplicationInProgress  。 等待大约 5 分钟，重新运行相同的命令，并验证 State 值是否更改为 Replicating 。 等待此操作完成，然后再继续执行后续步骤。 此外，确保主服务器列为 SEA-SVR2，副本服务器列为 SEA-SVR1   。

1. 在 SEA-SVR2 上，切换到显示与 SEA-SVR1 的 PowerShell 远程会话的“Administrator: Windows PowerShell”窗口  。
1. 要验证 SEA-CORE1 的副本是否存在于 SEA-SVR1 上，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter  ：

   ```powershell
   Get-VM
   ```

   > **注意**：确保命令的输出中列出了 **SEA-CORE1** 和复制已完成。 复制时间可能需要 5-10 分钟。

   > 注意：请让两个 Windows PowerShell 会话保持打开状态。

#### <a name="task-3-validate-a-failover"></a>任务 3：验证故障转移

1. 在 SEA-SVR2 上，切换到显示本地 PowerShell 会话的“Administrator: Windows PowerShell”窗口 。
1. 要准备将 SEA-CORE1 虚拟机故障转移到 SEA-SVR1，请在 SEA-SVR2 上，在托管本地会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter  ：

   ```powershell
   Start-VMFailover -Prepare -VMName SEA-CORE1 -ComputerName SEA-SVR2.contoso.com
   ```

   > 备注：出现提示时，输入 Y，然后按 Enter 。 此命令通过触发任何挂起更改的复制来为 SEA-CORE1 的计划内故障转移做准备。

1. 在 SEA-SVR2 上，切换到显示与 SEA-SVR1 的 PowerShell 远程会话的“Administrator: Windows PowerShell”窗口  。
1. 要启动 SEA-CORE1 虚拟机到 SEA-SVR1 的故障转移，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter   ：

   ```powershell
   Start-VMFailover -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

   > 备注：出现提示时，输入 Y，然后按 Enter 。

1. 要将副本 VM 配置为主 VM，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter ：

   ```powershell
   Set-VMReplication -Reverse -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. 要在 SEA-SVR1 上启动新指定的主 VM，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter  ：

   ```powershell
   Start-VM -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. 要验证 VM 是否已成功启动，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter ：

   ```powershell
   Get-VM
   ```

   > 注意：在结果表中，验证 State 是否列为 Running  。

1. 要识别虚拟机 SEA-CORE1 从 SEA-SVR1 到 SEA-SVR2 的复制状态，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter    ：

   ```powershell
   Get-VMReplication
   ```

   > 备注：在命令的输出中，识别 State 值并验证它是否列为 Replicating  。 此外，确保主服务器列为 SEA-SVR1，副本服务器列为 SEA-SVR2   。

1. 要在主服务器上停止复制 VM，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter ：

   ```powershell
   Stop-VM -VMName SEA-CORE1
   ```

1. 让两个 Windows PowerShell 窗口保持打开状态。

   > 注意：如果要使用图形工具验证本练习的结果，可以在 SEA-SVR2 上使用 Hyper-V 管理器，然后将 SEA-SVR1 和 SEA-SVR2 服务器添加到 Hyper-V 控制台    。 然后，你可以验证 SEA-CORE1 VM 是否存在于 SEA-SVR1 和 SEA-SVR2 上，并且正在运行从 SEA-SVR2 到 SEA-SVR1 的复制    。

## <a name="exercise-2-implementing-backup-and-restore-with-windows-server-backup"></a>练习 2：使用 Windows Server Backup 实现备份和还原

### <a name="task1-configure-windows-server-backup-settings"></a>任务 1：配置 Windows Server 备份设置

1. 在 SEA-SVR2 上，选择任务栏上的文件资源管理器图标，打开文件资源管理器 。
1. 在文件资源管理器中，在导航窗格中选择“本地磁盘(C:)” 。
1. 右键单击或访问详细信息窗格中空白区域的上下文菜单，选择“新建”，然后选择“文件夹” 。 
1. 将文件夹命名为 BackupShare。 右键单击或访问 BackupShare 文件夹的上下文菜单，选择“授予访问权限”，然后选择“特定人员”  。
1. 在“网络访问”窗口中，输入“已通过身份验证的用户”，然后选择“添加”  。 在“权限级别”列中，将“已通过身份验证的用户”的值设置为“读/写”，选择“共享”，然后选择“完成”    。
1. 在 SEA-SVR2 上，切换到托管与 SEA-SVR1 的 PowerShell 远程会话的“Administrator: Windows PowerShell”窗口  。 
1. 要在 SEA-SVR1 上安装 Windows Server 备份角色，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter   ：

   ```powershell
   Install-WindowsFeature Windows-Server-Backup
   ```

   > 备注：请等待安装完成。

1. 要查看 wbadmin 命令行实用程序的功能，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter  ：

   ```powershell
   wbadmin /?
   ```

1. 要查看 WindowsServerBackup 模块中包含的 Windows PowerShell cmdlet 的功能，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter  ：

   ```powershell
   Get-Command -Module WindowsServerBackup -CommandType Cmdlet
   ```

#### <a name="task-2-perform-a-backup-to-a-network-share"></a>任务 2：对网络共享执行备份

1. 要在 SEA-SVR1 上创建要备份的文件夹和文件，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令，并在输入每个命令后按 Enter  ：

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Files' -Force
   fsutil file createnew C:\Files\report1.docx 25432108
   fsutil file createnew C:\Files\report2.docx 25432108
   fsutil file createnew C:\Files\report3.docx 25432108
   fsutil file createnew C:\Files\report4.docx 25432108
   ```

1. 要使用 Windows Server 备份定义备份策略的变量和要备份的文件路径，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令，并在输入每个命令后按 Enter ：

   ```powershell
   $policy = New-WBPolicy
   $fileSpec = New-WBFileSpec -FileSpec 'C:\Files'
   ```

1. 要定义引用你在上一步中定义的变量的 Windows Server 备份策略，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter ：

   ```powershell
   Add-WBFileSpec -Policy $policy -FileSpec $fileSpec
   ```

1. 要使用你在上一个任务中创建的网络共享在 SEA-SVR2 上配置备份位置，请在 SEA-SVR2 上，在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令，并在输入每个命令后按 Enter（当提示登录时，输入 \\Administrator 用户名和 Pa55w.rd 密码）    ：

   ```powershell
   $cred = Get-Credential
   $networkBackupLocation = New-WBBackupTarget -NetworkPath "\\SEA-SVR2.contoso.com\BackupShare" -Credential $cred
   ```

1. 要将备份位置添加到备份策略，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter：

   ```powershell
   Add-WBBackupTarget -Policy $policy -Target $networkBackupLocation
   ```

1. 要启用卷影复制服务，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter：

   ```powershell
   Set-WBVssBackupOptions -Policy $policy -VssCopyBackup
   ```

1. 要启动备份作用，请在托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令并按 Enter：

   ```powershell
   Start-WBBackup -Policy $policy
   ```

   > 备注：请等待备份完成。 这大约需要 1 分钟。

1. 在 SEA-SVR2 上，切换到文件资源管理器，浏览到 C:\\BackupShare，并验证该文件夹是否包含 WindowsImageBackup 子文件夹中新创建的备份  。
