---
lab:
  title: 实验室：实现故障转移群集
  module: 'Module 3: High availability in Windows Server'
ms.openlocfilehash: dce41bd9411fc821c35e903255156bc6f8b558b3
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907067"
---
# <a name="lab-implementing-failover-clustering"></a>实验室：实现故障转移群集

## <a name="scenario"></a>场景

随着 Contoso， Ltd. 业务的增长，其网络上许多应用程序和服务始终可用变得越来越重要。 Contoso 有许多服务和应用程序，这些服务和应用程序必须可供在世界各地的不同时区工作的内部和外部用户使用。 许多此类应用程序无法通过使用 NLB 负载均衡和 NLB (高可用性) 。 因此，应使用不同的技术使这些应用程序高度可用。

作为 Contoso 的高级网络管理员之一，你负责在运行 Windows Server 的服务器上实现故障转移群集，为网络服务和应用程序提供高可用性。 你还负责规划故障转移群集配置，以及部署故障转移群集上的应用程序和服务。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 配置故障转移集群
- 在故障转移群集上部署和配置高度可用的文件服务器。
- 验证高可用性文件服务器的部署。

## <a name="estimated-time-60-minutes"></a>估计时间： **60 分钟**

## <a name="lab-setup"></a>实验室设置

虚拟机： **必须运行 AZ-801T00A-SEA-DC1**、 **AZ-801T00A-SEA-SVR1** 和 **AZ-801T00A-SEA-SVR2** 。 其他 VM 可以运行，但本实验室不需要这些 VM。

> **注意**： **AZ-801T00A-SEA-DC1**、 **AZ-801T00A-SEA-SVR1** 和 **AZ-801T00A-SEA-SVR2** 虚拟机托管 **SEA-DC1**、 **SEA-SVR1** 和 **SEA-SVR2 的安装**

1. 选择 **"SEA-SVR2"**。
1. 使用下列凭据登录：

   - 用户名： **管理员**
   - 密码：Pa55w.rd
   - /Domain Contoso 

对于本实验室，你将使用可用的 VM 环境。

## <a name="exercise-1-configuring-iscsi-storage"></a>练习 1：配置 iSCSI 存储

### <a name="scenario"></a>场景

Contoso 有重要的应用程序和服务，它希望这些应用程序和服务高度可用。 其中一些服务不能使用 NLB，因此你已决定实现故障转移群集。 你决定使用 iSCSI (iSCSI) Internet 小型计算机系统接口进行故障转移群集。 首先，将 iSCSI 存储配置为支持故障转移群集。

本练习的主要任务是：

- Install Failover Clustering
- 配置 iSCSI 虚拟磁盘。

#### <a name="task-1-install-failover-clustering"></a>任务 1：安装故障转移群集

1. 在 **SEA-SVR2** 上，Windows PowerShell管理员角色。
1. 在 **SEA-SVR2** 上，Windows PowerShell安装故障转移 **群集** 功能，包括 **SEA-SVR1** 和 **SEA-SVR2 上的管理工具**。
1. 在 **SEA-SVR2** 上，Windows PowerShell在 **SEA-DC1** 上安装 **FS-iSCSITarget-Server** 功能。
 
#### <a name="task-2-configure-iscsi-virtual-disks"></a>任务 2：配置 iSCSI 虚拟磁盘

> **重要：** 实验室使用 **SEA-DC1**，它充当 Active Directory 域 Services (AD DS) 域控制器，为基于服务器的群集托管共享 iSCI Windows。 这不会以任何方式表示建议的配置，但这样做是为了简化实验室配置并尽量减少实验室虚拟机的数量。 在任何生产环境中，不应使用域控制器来托管故障转移群集的共享存储。 相反，此类存储应托管在高度可用的基础结构上。 

1. 在 **SEA-SVR2** 上，以管理员Windows PowerShell启动两个窗口。
1. 使用第二个和第三个远程处理窗口分别与 **SEA-DC1** 和 **SEA-SVR1** 建立 PowerShell 远程Windows PowerShell会话。
1. 使用 **SEA-DC1** 的 PowerShell 远程处理会话在 **SEA-DC1** 上创建三个 iSCSI 虚拟磁盘，以 **创建 C：\\存储** 目录，并运行具有以下参数的 **New-IscsiVirtualDisk** cmdlet：

   - Disk1
      - 存储位置：**C：\\存储**
      - 磁盘名称： **Disk1**
      - 大小： **10 GB**
   - Disk2
      - 存储位置：**C：\\存储**
      - 磁盘名称： **Disk2**
      - 大小： **10 GB**
   - Disk3
      - 存储位置：**C：\\存储**
      - 磁盘名称： **Disk3**
      - 大小： **10 GB**

1. 在 **SEA-SVR2** 和 **SEA-SVR1** 上启动 iSCSI 发起程序 (MSiSCSI) 服务，然后使用 **SEA-SVR2** 的本地 Windows PowerShell 会话和到 **SEA-SVR1** 的 PowerShell 远程处理会话来运行 **Start-Service** 和 **Set-Service** cmdlet，将其配置为自动启动。
1. 使用 **SEA-DC1** 的 PowerShell 远程处理会话在 **SEA-DC1** 上创建 iSCSI 目标，以运行具有以下参数的 **New-IscsiServerTarget** cmdlet：

   - 目标名称： **ISCSI-L03**
   - InitiatorsIds： 
      - **"IQN：iqn.1991-05.com.microsoft：sea-svr1.contoso.com"**
      - **"IQN：iqn.1991-05.com.microsoft：sea-svr2.contoso.com"**

### <a name="results"></a>结果

完成本练习后，应已成功在 **SEA-SVR1** 和 **SEA-SVR2** 上安装故障转移群集功能，以及 **SEA-DC1**、 **SEA-SVR1** 和 **SEA-SVR2** 上初始化的 iSCSI 组件。

## <a name="exercise-2-configuring-a-failover-cluster"></a>练习 2：配置故障转移群集

### <a name="scenario"></a>场景

在此练习中，你将为群集安装做好准备，然后创建群集。

本练习的主要任务是：

1. 连接客户端到 iSCSI 目标。
1. 初始化磁盘。
1. 验证并创建故障转移群集。

#### <a name="task-1-connect-clients-to-the-iscsi-targets"></a>任务 1：连接客户端到 iSCSI 目标

1. 在 **SEA-SVR2** 上，使用 **托管** PowerShell 远程处理会话的 Windows PowerShell 窗口将 iSCSI 磁盘装载到 **SEA-DC1**，以运行 **Add-IscsiVirtualDiskTargetMapping** cmdlet。
1. 在 **SEA-SVR2** 上，使用本地 Windows PowerShell 会话从 **SEA-SVR2** 连接到 **SEA-DC1** 上托管的 iSCSI 目标，以运行以下命令：

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **注意：** 验证在运行最后一个命令后 **，IsConnected** 变量的值为 True。

1. 在 **SEA-SVR2** 上，使用 **SEA-SVR1** 的 PowerShell 远程处理会话连接到 **SEA-SVR1** 上托管在 **SEA-DC1** 上的 iSCSI 目标，以运行以下命令：

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **注意：** 验证在运行最后一个命令后 **，IsConnected** 变量的值为 True。


#### <a name="task-2-initialize-the-disks"></a>任务 2：初始化磁盘

1. 在 **SEA-SVR2** 上，使用本地 Windows PowerShell 会话列出本地磁盘，以运行 **Get-Disk** cmdlet。
1. 在 **SEA-SVR2** 上，使用本地存储会话通过以下设置初始化三个 iSCSI Windows PowerShell磁盘：

   - PartitionStyle： **MBR**
   - New-Partition大小： **5GB**
   - 文件系统： **NTFS**
   - 分别分配 **驱动器号 E**、 **F** **和 G**。

#### <a name="task-3-create-a-failover-cluster"></a>任务 3：创建故障转移群集

1. 在 **SEA-SVR2** 上，使用本地 Windows PowerShell 会话创建名为 **SEA-CL03** 且 IP 地址设置为 **172.16.10.125** 且 **SEA-SVR2.contoso.com** 作为第一个节点的故障转移群集。
1. 在 **SEA-SVR2** 上，使用本地群集会话将 **SEA-SVR1** 作为第二个节点添加到Windows PowerShell群集。

### <a name="results"></a>结果

完成本练习后，应已配置磁盘并创建了故障转移群集。

## <a name="exercise-3-deploying-and-configuring-a-highly-available-file-server"></a>练习 3：部署和配置高度可用的文件服务器

### <a name="scenario"></a>场景

在 Contoso，文件服务是必须高度可用的重要服务。 创建群集基础结构后，你决定配置高度可用的文件服务器，然后实现故障转移和故障回复的设置。

本练习的主要任务是：

1. 将文件服务器应用程序添加到故障转移群集。
1. 将共享文件夹添加到高度可用的文件服务器。
1. 配置故障转移和故障回复设置。

#### <a name="task-1-add-the-file-server-application-to-the-failover-cluster"></a>任务 1：将文件服务器应用程序添加到故障转移群集

1. 在 **SVR2** 上，打开 **故障转移群集管理器** 控制台，并验证是否已自动连接到 **SEA-CL03.contoso.com** 群集。
1. 在 " **节点** " 节点中，验证 **SVR2** 和 **SVR1** 节点是否都在运行。
1. 在 **存储**"节点中，选择"**磁盘**"，然后验证三个群集磁盘是否处于联机状态。
1. 添加 **文件服务器** 作为群集角色，并使用 " **文件服务器通用** " 选项和以下设置：

   - 客户端访问名称： **FSCluster**
   - 地址： **172.16.0.130**
   - 存储：**群集磁盘1、群集磁盘 2**

#### <a name="task-2-add-a-shared-folder-to-a-highly-available-file-server"></a>任务2：将共享文件夹添加到高可用文件服务器

1. 在 **SVR2** 控制台中，在 **故障转移群集管理器** 控制台中，使用以下设置将文件共享添加到 **FSCluster** 角色中 (使用默认值) 保留所有其他值：

   - 文件共享配置文件： **SMB 共享-快速**
   - 名称： **文档**

#### <a name="task-3-configure-the-failover-and-failback-settings"></a>任务3：配置故障转移和故障回复设置

1. 在 **SVR2** 上，在 **故障转移群集管理器** 控制台中，使用 **FSCluster** 群集角色的 "**属性**" 对话框来配置以下设置：

   - 故障回复： **4 到5小时**
   - 首选所有者：在列表顶部有 **SVR2** 和 **SVR1** **SVR1** 的海平面

### <a name="results"></a>结果

完成本练习后，你应该已配置高度可用的文件服务器。

## <a name="exercise-4-validating-the-deployment-of-the-highly-available-file-server"></a>练习4：验证高可用文件服务器的部署

### <a name="scenario"></a>场景

在实现故障转移群集时，你需要执行故障转移和故障回复测试。 此外，还需要更改仲裁中的见证磁盘。

本练习的主要任务是：

1. 验证高可用文件服务器部署。
1. 验证文件服务器角色的故障转移和仲裁配置。

#### <a name="task-1-validate-the-highly-available-file-server-deployment"></a>任务1：验证高可用文件服务器部署

1. 在 **SVR2** 上，打开文件资源管理器并验证是否可以访问 **\\ \\ FSCluster \\ 文档** 共享。
1. 在共享中创建测试文本文档。
1. 在 **SVR2** 上，使用 **故障转移群集管理器** 控制台将 **FSCluster** 角色转移到另一个节点。
1. 在 **SVR2** 的 "文件资源管理器" 中，验证是否仍可访问 **\\ \\ FSCluster \\ 文档** 共享。

#### <a name="task-2-validate-the-failover-and-quorum-configuration-for-the-file-server-role"></a>任务2：验证文件服务器角色的故障转移和仲裁配置

1. 在 **SVR2** 的 " **故障转移群集管理器** " 控制台中，确定 **FSCluster** 角色的当前所有者。
1. 停止节点上的群集服务，这是 **FSCluster** 角色的当前所有者。
1. 在 **SVR2** 上，使用文件资源管理器验证 **\\ \\ FSCluster \\ 文档** 共享是否仍然可用。
1. 在 **SVR2** 上，使用 **故障转移群集管理器** 控制台在步骤2中将其停止的节点上启动群集服务。
1. 在 **SVR2** 的 " **故障转移群集管理器** 控制台中，使用默认设置配置 **FSCluster** 的群集仲裁。
1. 在 **故障转移群集管理器** 控制台中，浏览到 " **磁盘** " 节点，并 **在 "仲裁** 脱机" 中将被标记为见证磁盘的磁盘放入磁盘。
1. 在 **SVR2** 上，使用文件资源管理器验证 **\\ \\ FSCluster \\ 文档** 共享是否仍然可用。
1. 使见证服务器联机。

### <a name="results"></a>结果

完成本练习后，你应该具有故障转移群集的高可用性。

