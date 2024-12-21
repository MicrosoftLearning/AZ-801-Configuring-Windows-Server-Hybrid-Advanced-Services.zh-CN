---
lab:
  title: 实验室：实现故障转移群集
  type: Answer Key
  module: 'Module 3: High availability in Windows Server'
---

# 实验室解答：实现故障转移群集

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Implementing%20Failover%20Clustering)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 练习 1：配置 iSCSI 存储

#### 任务 1：安装故障转移群集

1. 连接到 **SEA-SVR2**，然后根据需要使用讲师提供的凭据登录。
1. 在 SEA-SVR2 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。
1. 若要安装故障转移群集服务器功能（包括 SEA-SVR1 和 SEA-SVR2 上的管理工具），请在 Windows PowerShell 命令提示符下，输入以下命令，并在输入每个命令后按 Enter ：

   ```powershell
   Install-WindowsFeature –Name Failover-Clustering –IncludeManagementTools
   Install-WindowsFeature -ComputerName 'SEA-SVR1.contoso.com' –Name Failover-Clustering –IncludeManagementTools
   ```

   > 注意：等待安装过程完成。 安装过程大约需要 1 分钟。

1. 若要在 SEA-DC1 上安装 iSCSI 目标服务器角色服务，请在 Windows PowerShell 命令提示符下，输入以下命令并按 Enter：

   ```powershell
   Install-WindowsFeature -ComputerName 'SEA-DC1.contoso.com' –Name FS-iSCSITarget-Server –IncludeManagementTools
   ```

   > 注意：等待安装过程完成。 安装过程大约需要 1 分钟。

#### 任务 2：配置 iSCSI 虚拟磁盘

> 重要提示：实验室使用 SEA-DC1 作为 Active Directory 域服务 (AD DS) 域控制器来托管基于 Windows Server 的群集的共享 iSCI 存储 。 这不是为了以任何方式表示建议的配置，而是为了简化实验室配置并尽量减少实验室虚拟机的数量。 在任何生产环境中，域控制器都不应用于托管故障转移群集的共享存储。 相反，此类存储应托管在高可用性基础结构上。 

1. 在 SEA-SVR2 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。
1. 要建立与 SEA-DC1 的 PowerShell 远程会话，请在新打开的 Windows PowerShell 窗口中，输入以下命令并按 Enter ：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. 若要在 SEA-DC1 上创建 iSCSI 虚拟磁盘，请在 SEA-SVR2 上，在与 SEA-DC1 的 PowerShell 远程会话中，输入以下命令，并在输入每个命令后按 Enter  ：

   ```powershell
   New-Item -ItemType Directory C:\Storage -Force
   New-IscsiVirtualDisk C:\Storage\disk1.VHDX –size 10GB
   New-IscsiVirtualDisk C:\Storage\disk2.VHDX –size 10GB
   New-IscsiVirtualDisk C:\Storage\disk3.VHDX –size 10GB
   ```

1. 在 SEA-SVR2 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。
1. 要建立与 SEA-SVR1 的 PowerShell 远程会话，请在新打开的 Windows PowerShell 窗口中输入以下命令，然后按 Enter ：

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```

   > 注意：此时，应有三个 Windows PowerShell 窗口处于打开状态 。 你将使用第一个窗口在 SEA-SVR2 上本地运行命令，同时使用另外两个窗口与 SEA-DC1 和 SEA-SVR1 进行交互  。 可通过辨认 PowerShell 提示符轻松识别每个窗口（第二个和第三个窗口的提示符将分别包含 [SEA-DC1.contoso.com] 和 [SEA-SVR1.contoso.com] 前缀） 。

1. 若要在 SEA-SVR2 上启动 Microsoft iSCSI 发起程序服务，请在提供本地会话访问权限的 Windows PowerShell 提示符下，输入以下命令，并在输入每个命令后按 Enter ：

   ```powershell
   Start-Service -ServiceName MSiSCSI
   Set-Service -ServiceName MSiSCSI -StartupType Automatic
   ```

1. 若要在 SEA-SVR1 上启动 Microsoft iSCSI 发起程序服务，请切换到托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口，输入以下命令，并在输入每个命令后按 Enter  ：

   ```powershell
   Start-Service -ServiceName MSiSCSI
   Set-Service -ServiceName MSiSCSI -StartupType Automatic
   ```

1. 若要在 SEA-DC1 上创建 Microsoft iSCSI 目标，请切换到托管与 SEA-DC1 的 PowerShell 远程会话的 Windows PowerShell 窗口，输入以下命令，然后按 Enter  ：

   ```powershell
   New-IscsiServerTarget -TargetName “iSCSI-L03” –InitiatorIds “IQN:iqn.1991-05.com.microsoft:sea-svr1.contoso.com","IQN:iqn.1991-05.com.microsoft:sea-svr2.contoso.com"
   ```

## 练习 2：配置故障转移群集

#### 任务 1：将客户端连接到 iSCSI 目标

1. 若要在 SEA-DC1 上挂载 iSCSI 磁盘，请通过 SEA-SVR2，在托管与 SEA-DC1 的 PowerShell 远程会话的 Windows PowerShell 窗口中，输入以下命令，并在输入每个命令后按 Enter   ：

   ```powershell
   Add-IscsiVirtualDiskTargetMapping -TargetName “iSCSI-L03” -DevicePath “C:\Storage\Disk1.VHDX”
   Add-IscsiVirtualDiskTargetMapping -TargetName “iSCSI-L03” -DevicePath “C:\Storage\Disk2.VHDX”
   Add-IscsiVirtualDiskTargetMapping -TargetName “iSCSI-L03” -DevicePath “C:\Storage\Disk3.VHDX”
   ```

1. 若要从 SEA-SVR2 连接到 SEA-DC1 上托管的 iSCSI 目标，请切换到提供本地会话访问权限的 Windows PowerShell 提示符下，输入以下命令，并在输入每个命令后按 Enter  ：

   ```powershell
   New-iSCSITargetPortal –TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget –NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > 注意：在运行最后一个命令后，验证 IsConnected 变量的值为 True。

1. 若要从 SEA-SVR1 连接到 SEA-DC1 上托管的 iSCSI 目标，请切换到托管与 SEA-SVR1 的 PowerShell 远程会话的 Windows PowerShell 窗口，输入以下命令，并在输入每个命令后按 Enter  ：

   ```powershell
   New-iSCSITargetPortal –TargetPortalAddress SEA-DC1.contoso.com 
   Connect-iSCSITarget –NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > 注意：在运行最后一个命令后，验证 IsConnected 变量的值为 True。

#### 任务 2：初始化磁盘

1. 若要列出 SEA-SVR2 上的磁盘，请切换到提供本地会话访问权限的 Windows PowerShell 提示符下，输入以下命令，然后按 Enter：

   ```powershell
   Get-Disk
   ```

   > 注意：确保列出的三个 iSCSI 磁盘都处于“脱机”操作状态 。 这些磁盘的编号应该是 2、3 和 4。

1. 若要初始化磁盘，请在提供本地会话访问权限的 Windows PowerShell 提示符下，输入以下命令，并在输入每个命令后按 Enter：

   ```powershell
   Get-Disk | Where OperationalStatus -eq 'Offline' | Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 2 -Size 5gb -AssignDriveLetter
   New-Partition -DiskNumber 3 -Size 5gb -AssignDriveLetter
   New-Partition -DiskNumber 4 -Size 5gb -AssignDriveLetter
   Format-Volume -DriveLetter E -FileSystem NTFS
   Format-Volume -DriveLetter F -FileSystem NTFS
   Format-Volume -DriveLetter G -FileSystem NTFS
   ```

   > **注意：** 在运行命令之前，验证磁盘编号是否与上一个命令的输出内容匹配。 验证每个命令是否成功完成。

#### 任务 3：创建故障转移群集

1. 若要创建故障转移群集，请在 SEA-SVR2 上，在提供本地会话访问权限的 Windows PowerShell 提示符下，输入以下命令，然后按 Enter：

   ```powershell
   New-Cluster -Name SEA-CL03 -Node SEA-SVR2.contoso.com -StaticAddress 172.16.10.125
   ```

   > 注意：该命令应返回新创建的群集的名称 (SEA-CL03) 。 

1. 若要将 SEA-SVR1 作为另一个节点添加到新创建的群集，请在 SEA-SVR2 上，在提供本地会话访问权限的 Windows PowerShell 提示符下，输入以下命令，然后按 Enter ：

   ```powershell
   Add-ClusterNode -Cluster SEA-CL03 -Name SEA-SVR1.contoso.com
   ```

   > 注意：验证命令是否成功完成。

## 练习 3：部署和配置高可用文件服务器

#### 任务 1：将文件服务器应用程序添加到故障转移群集

1. 在 SEA-SVR2 上，选择“开始”，在“开始”菜单中选择“服务器管理器”，然后在服务器管理器中，选择“工具”菜单中的“故障转移群集管理器”      。

   > 注意：“故障转移群集管理器”控制台将自动连接到 SEA-CL03，因为 SEA-SVR2 是群集节点之一   。

1. 展开“SEA-CL03.contoso.com”节点，选择“角色”，然后验证群集此时是否未托管任何角色 。
1. 选择“节点”节点，并验证“SEA-SVR1”和“SEA-SVR2”节点是否显示“启动”状态   。
1. 展开“存储”节点，然后选择“磁盘” 。 请注意，三个群集磁盘显示为“联机”状态。
1. 在“故障转移群集管理器”页中，右键单击或访问“角色”的上下文菜单，然后选择“配置角色”  。 随即将启动“高可用性向导”。
1. 在高可用性向导的“准备工作”页中，单击“下一步”  。
1. 在高可用性向导的“选择角色”页中，选择“文件服务器”，然后选择“下一步”   。
1. 在高可用性向导的“文件服务器类型”页中，确保选择了“常规用途文件服务器”选项，然后选择“下一步”   。
1. 在高可用性向导“客户端访问点”页的“名称”框中，输入FSCluster   。
1. 在“地址”框中，输入 172.16.10.130，然后选择“下一步”  。
1. 在高可用性向导的“选择存储”页中，选择“群集磁盘 1”和“群集磁盘 2”，然后选择“下一步”    。
1. 在高可用性向导的“确认”页中，选择“下一步”  。
1. 在高可用性向导的“摘要”页中，选择“完成”  。

   > 注意：在“存储”节点中，选择“磁盘”节点，验证三个群集磁盘是否处于联机状态  。 群集磁盘 1 和群集磁盘 2 应分配给 FSCluster  。

#### 任务 2：将共享文件夹添加到高可用文件服务器

1. 在 SEA-SVR2 上的“故障转移群集管理器”中，依次选择“角色”和“FSCluster”，然后在“操作”窗格中，选择“添加文件共享”    。 

   > 注意：这将启动“新建共享向导” 。

1. 在“选择配置文件”页中，确保选择了“SMB 共享 - 快速”配置文件，然后选择“下一步”  。
1. 在“共享位置”页中，选择“下一步” 。
1. 在“共享名”页中，输入“Docs”作为共享名，然后选择“下一步”  。
1. 在“其他设置”页中，选择“下一步” 。
1. 在“权限”页中，选择“下一步” 。
1. 在“确认”页面，选择“创建” 。
1. 在“查看结果”页中，选择“关闭” 。

#### 任务 3：配置故障转移和故障回复设置

1. 在 SEA-SVR2 的“故障转移群集管理器”控制台中，在“角色”节点中选择“FSCluster”，然后在“操作”窗格中选择“属性”    。
1. 选择“故障转移”选项卡，然后选择“允许故障回复”选项 。
1. 选择“故障回复时间”选项，然后输入以下值：

   - 第一个文本框中，输入 4
   - 第二个文本框中，输入 5。

1. 选择“常规”选项卡****。
1. 在“首选所有者”部分中，确保“SEA-SVR1”列为第一个条目，然后选择“确定”  。


## 练习 4：验证高可用文件服务器的部署

#### 任务 1：验证高可用文件服务器部署

1. 在 SEA-SVR2 上，打开“文件资源管理器”并浏览到 \\\\FSCluster\\Docs 文件夹 。
1. 在“Docs”文件夹中，右键单击或访问该文件夹空白区域中的上下文菜单，选择“新建”，然后选择“文本文档”  。
1. 若要接受将 New Text Document.txt 用作文档的默认名称，请按 Enter。
1. 在 SEA-SVR2 上，切换到“故障转移群集管理器”控制台，右键单击或访问“FSCluster”的上下文菜单，依次选择“移动”、“选择节点”、“SEA-SVR1”和“确定”      。
1. 在 SEA-SVR2 上，切换回“文件资源管理器”并验证是否仍可访问 \\\\FSCluster\\Docs 文件夹的内容 。

#### 任务 2：验证文件服务器角色的故障转移和仲裁配置

1. 在 SEA-SVR2 上，切换到“故障转移群集管理器”控制台，确定 FSCluster 角色的当前所有者  。
1. 选择“节点”，然后右键单击或访问在上一步中确定的节点的上下文菜单。
1. 在该上下文菜单中，选择“更多操作”，然后选择“停止群集服务” 。
1. 切换到“文件资源管理器”并验证是否仍可访问 \\\\FSCluster\\Docs 文件夹的内容。
1. 切换到“故障转移群集管理器”控制台，然后右键单击或访问状态为“关闭”的节点的上下文菜单 。
1. 在该上下文菜单中，选择“更多操作”，然后选择“启动群集服务” 。
1. 在“故障转移群集管理器”控制台中，右键单击或访问 SEA-CL03.Contoso.com 群集的上下文菜单，选择“更多操作”，然后选择“配置群集仲裁设置”   。 随即将启动“配置群集仲裁向导”。
1. 在“开始之前”**** 页面上，选择“下一步”****。
1. 在“选择仲裁配置选项”页中，确保选中“使用默认仲裁配置”选项，然后选择“下一步”  。
1. 在“确认”页中，注意默认情况下选择“群集磁盘 3”作为“磁盘见证”，然后选择“下一步”   。 
1. 在“摘要”页中，选择“完成” 。
1. 在“故障转移群集管理器”控制台中，浏览到“磁盘”节点，选择配置为磁盘见证的“群集磁盘 3”，然后在“操作”窗格中选择“脱机”   。
1. 出现确认提示时，选择“是”****。
1. 切换到“文件资源管理器”并验证是否仍可访问 \\\\FSCluster\\Docs 文件夹的内容。
1. 切换到“故障转移群集管理器”控制台，然后在“磁盘”节点内的磁盘列表中，选择配置为磁盘见证的“群集磁盘 3”，然后在“操作”窗格中选择“联机”   。
