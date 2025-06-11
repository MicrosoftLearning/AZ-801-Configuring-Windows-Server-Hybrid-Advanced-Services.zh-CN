---
lab:
  title: 实验室：实现故障转移集群
  module: 'Module 3: High availability in Windows Server'
---

# 实验室：实现故障转移集群

## 场景

随着 Contoso, Ltd. 业务的增长，其网络上许多应用程序和服务始终可用变得越来越重要。 Contoso 有许多服务和应用程序，这些服务和应用程序必须可供在世界各地的不同时区工作的内部和外部用户使用。 许多此类应用程序无法通过使用 NLB 负载均衡 (NLB) 获得高可用性。 因此，应使用其他技术使这些应用程序高度可用。

作为 Contoso 的高级网络管理员之一，你负责在运行 Windows Server 的服务器上实现故障转移群集，为网络服务和应用程序提供高可用性。 你还负责规划故障转移群集配置，以及部署故障转移群集上的应用程序和服务。

## 目标

完成本实验室后，你将能够：

- 配置故障转移集群。
- 在故障转移群集上部署和配置高度可用的文件服务器。
- 验证高可用性文件服务器的部署。

## 估计时间：60 分钟

## 实验室设置

虚拟机：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 必须运行  。 其他 VM 可以运行，但此实验室不需要这些 VM。

> 备注：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 虚拟机托管 SEA-DC1、SEA-SVR1 和 SEA-SVR2 的安装

1. 选择 SEA-SVR2。
1. 使用讲师提供的凭据进行登录。

对于本实验室，你将使用可用的 VM 环境。

## 练习 1：配置 iSCSI 存储

### 场景

Contoso 有重要的应用程序和服务，它希望这些应用程序和服务高度可用。 其中一些服务不能使用 NLB，因此你已决定实现故障转移群集。 你决定使用 Internet 小型计算机系统接口 (iSCSI) 进行故障转移群集。 首先，将 iSCSI 存储配置为支持故障转移群集。

本练习的主要任务是：

- 安装故障转移群集。
- 配置 iSCSI 虚拟磁盘。

#### 任务 1：安装故障转移群集

1. 在 SEA-SVR2 上，以管理员身份启动 Windows PowerShell。
1. 在 SEA-SVR2 上，使用 Windows PowerShell 安装故障转移群集功能，包括 SEA-SVR1 和 SEA-SVR2 上的管理工具   。
1. 在 SEA-SVR2 上，使用 Windows PowerShell 在 SEA-DC1 上安装 FS-iSCSITarget-Server 功能  。
 
#### 任务 2：配置 iSCSI 虚拟磁盘

> 重要提示：实验室使用 SEA-DC1 作为 Active Directory 域服务 (AD DS) 域控制器来托管基于 Windows Server 的群集的共享 iSCI 存储 。 这不是为了以任何方式表示建议的配置，而是为了简化实验室配置并尽量减少实验室虚拟机的数量。 在任何生产环境中，域控制器都不应用于承载故障转移群集的共享存储。 相反，此类存储应托管在高可用性基础结构上。 

1. 在 SEA-SVR2 上，以管理员身份再启动两个 Windows PowerShell 窗口。
1. 分别使用第二个和第三个 Windows PowerShell 窗口与 SEA-DC1 和 SEA-SVR1 建立 PowerShell 远程会话 。
1. 使用 SEA-DC1 的 PowerShell 远程会话创建 C:\\Storage 目录并使用以下参数运行 New-IscsiVirtualDisk cmdlet，以便在 SEA-DC1 上创建三个 iSCSI 虚拟磁盘   ：

   - Disk1：
      - 存储位置：C:\\Storage
      - 磁盘名称：Disk1
      - 大小：10 GB
   - Disk2：
      - 存储位置：C:\\Storage
      - 磁盘名称：Disk2
      - 大小：10 GB
   - Disk3：
      - 存储位置：C:\\Storage
      - 磁盘名称：Disk3
      - 大小：10 GB

1. 在 SEA-SVR2 和 SEA-SVR1 上启动 iSCSI 发起程序 (MSiSCSI) 服务，然后使用 SEA-SVR2 的本地 Windows PowerShell 会话和 SEA-SVR1 的 PowerShell 远程会话来运行 Start-Service 和 Set-Service cmdlet，将其配置为自动启动     。
1. 使用 SEA-DC1 的 PowerShell 远程会话和以下参数运行 New-IscsiServerTarget cmdlet，以便在 SEA-DC1 上创建 iSCSI 目标  ：

   - 目标名称：ISCSI-L03
   - InitiatorsIds： 
      - **"IQN:iqn.1991-05.com.microsoft:sea-svr1.contoso.com"**
      - **"IQN:iqn.1991-05.com.microsoft:sea-svr2.contoso.com"**

### 结果

完成本练习后，应成功在 SEA-SVR1 和 SEA-SVR2 上安装故障转移群集功能，在 SEA-DC1、 SEA-SVR1 和 SEA-SVR2 上初始化 iSCSI 组件    。

## 练习 2：配置故障转移群集

### 场景

在此练习中，你将为群集安装做好准备，然后创建群集。

本练习的主要任务是：

1. 将客户端连接到 iSCSI 目标。
1. 初始化磁盘。
1. 验证并创建故障转移群集。

#### 任务 1：将客户端连接到 iSCSI 目标

1. 在 **SEA-SVR2** 上，使用**托管** PowerShell 远程处理会话的 Windows PowerShell 窗口将 iSCSI 磁盘装载到 **SEA-DC1**，以运行 **Add-IscsiVirtualDiskTargetMapping** cmdlet。****
1. 在 SEA-SVR2 上，使用本地 Windows PowerShell 会话运行以下命令，以便从 SEA-SVR2 连接到 SEA-DC1 上托管的 iSCSI 目标  ：

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > 注意：在运行最后一个命令后，验证 IsConnected 变量的值为 True 。

1. 在 SEA-SVR2 上，使用 SEA-SVR1 的 PowerShell 远程会话运行以下命令，以便从 SEA-SVR1 连接到 SEA-DC1 上托管的 iSCSI 目标   ：

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > 注意：在运行最后一个命令后，验证 IsConnected 变量的值为 True 。


#### 任务 2：初始化磁盘

1. 在 SEA-SVR2 上，使用本地 Windows PowerShell 会话运行 Get-Disk cmdlet 以列出本地磁盘 。
1. 在 SEA-SVR2 上，使用本地 Windows PowerShell 会话通过以下设置初始化这三个 iSCSI 磁盘：

   - 分区类型：MBR
   - 新建分区大小：5GB
   - 文件系统：NTFS
   - 分配驱动器号：分别为 E、F 和 G  。

#### 任务 3：创建故障转移群集

1. 在 SEA-SVR2 上，使用本地 Windows PowerShell 会话创建名为 SEA-CL03 的故障转移群集，并将 IP 地址设置为 172.16.10.125，将 SEA-SVR2.contoso.com 作为第一个节点   。
1. 在 SEA-SVR2 上，使用本地 Windows PowerShell 会话将 SEA-SVR1 作为第二个节点添加到新创建的群集 。

### 结果

完成本练习后，应配置磁盘并创建了故障转移群集。

## 练习 3：部署和配置高可用性文件服务器

### 场景

在 Contoso 中，文件服务是必须高度可用的重要服务。 创建群集基础结构后，你决定配置高度可用的文件服务器，然后实现故障转移和故障回复的设置。

本练习的主要任务是：

1. 将文件服务器应用程序添加到故障转移群集。
1. 将共享文件夹添加到高度可用的文件服务器。
1. 配置故障转移和故障回复设置。

#### 任务 1：将文件服务器应用程序添加到故障转移群集

1. 在 SEA-SVR2 上，打开“故障转移群集管理器”控制台，并验证是否已自动连接到 SEA-CL03.contoso.com 群集  。
1. 在“节点”节点中，验证 SEA-SVR2 和 SEA-SVR1 节点是否都在运行  。
1. 在“存储”节点中，选择“磁盘”，然后验证三个群集磁盘是否处于联机状态 。
1. 添加“文件服务器”作为群集角色，并使用“通用文件服务器”选项和以下设置 ：

   - 客户端访问名称：FSCluster
   - 地址：172.16.0.130
   - 存储：群集磁盘 1、群集磁盘 2

#### 任务 2：将共享文件夹添加到高度可用的文件服务器

1. 在 SEA-SVR2 上，在“故障转移群集管理器”控制台中，使用以下设置将文件共享添加到 FSCluster 角色中（其他设置保留默认值）  ：

   - 文件共享配置文件：SMB 共享 - 快速
   - 名称：Docs

#### 任务 3：配置故障转移和故障回复设置

1. 在 SEA-SVR2 上，在“故障转移群集管理器”控制台中，使用 FSCluster 群集角色的“属性”对话框来配置以下设置   ：

   - 故障回复：4 到 5 小时
   - 首选所有者：SEA-SVR2 和 SEA-SVR1，并且 SEA-SVR1 位于列表顶部

### 结果

完成本练习后，你应该配置了高度可用的文件服务器。

## 练习 4：验证高可用性文件服务器的部署

### 场景

在实现故障转移群集时，你需要执行故障转移和故障回复测试。 此外，还需要更改仲裁中的见证磁盘。

本练习的主要任务是：

1. 验证高可用文件服务器部署。
1. 验证文件服务器角色的故障转移和仲裁配置。

#### 任务 1：验证高可用文件服务器部署

1. 在 SEA-SVR2 上，打开文件资源管理器并验证是否可以访问 \\\\FSCluster\\Docs 共享 。
1. 在共享中创建测试文本文档。
1. 在 SEA-SVR2 上，使用“故障转移群集管理器”控制台将 FSCluster 角色转移到另一个节点  。
1. 在 SEA-SVR2 上，在文件资源管理器中验证是否仍可以访问 \\\\FSCluster\\Docs 共享 。

#### 任务 2：验证文件服务器角色的故障转移和仲裁配置

1. 在 SEA-SVR2 上，在“故障转移群集管理器”控制台中，确定 FSCluster 角色的当前所有者  。
1. 停止节点上的群集服务，这是 FSCluster 角色的当前所有者。
1. 在 SEA-SVR2 上，使用文件资源管理器验证 \\\\FSCluster\\Docs 共享是否仍然可用 。
1. 在 SEA-SVR2 上，使用“故障转移群集管理器”控制台在步骤 2 中停止群集服务的节点上启动该服务 。
1. 在 SEA-SVR2 上，在“故障转移群集管理器”控制台中，使用默认设置配置 FSCluster 的群集仲裁  。
1. 在“故障转移群集管理器”控制台中，浏览到“磁盘”节点，并将标记为“仲裁中的见证磁盘“脱机  。
1. 在 SEA-SVR2 上，使用文件资源管理器验证 \\\\FSCluster\\Docs 共享是否仍然可用 。
1. 将见证磁盘联机。

### 结果

完成本练习后，你应使用故障转移群集验证了高可用性。

