---
lab:
  title: 实验室：使用 Azure Migrate 将 Hyper-V VM 迁移到 Azure
  module: 'Module 7: Design for Migration'
ms.openlocfilehash: 9931ebc69ae739cf4996a7fb9ec233d49def3800
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907066"
---
# <a name="lab-migrating-hyper-v-vms-to-azure-by-using-azure-migrate"></a>实验室：使用 Azure Migrate 将 Hyper-V VM 迁移到 Azure

## <a name="lab-scenario"></a>实验室场景

尽管 Adatum 企业体系结构团队希望在迁移到 Azure 的过程中对其工作负载进行现代化处理，但他们意识到，由于时间安排紧张，在很多情况下，必须采用“直接迁移”方法。 为了简化此任务，该团队开始探索 Azure Migrate 的功能。 Azure Migrate 充当一个集中化中心，用于评估本地服务器、基础结构、应用程序和数据，并将它们迁移到 Azure。

Azure Migrate 提供以下功能：

- 统一迁移平台：用于启动、运行和跟踪到 Azure 的迁移的单一门户。
- 工具范围：用于评估和迁移的一系列工具。 工具包括“Azure Migrate: 服务器评估和 Azure Migrate:服务器迁移。 Azure Migrate 集成了其他 Azure 服务，以及其他工具和独立软件供应商 (ISV) 产品/服务。
- 评估和迁移：在 Azure Migrate 中心内，可以评估和迁移以下项目：
    - 服务器：评估本地服务器并将其迁移到 Azure 虚拟机。
    - 数据库：评估本地数据库，并将其迁移到 Azure SQL 数据库或 SQL 托管实例。
    - Web 应用程序：使用 Azure 应用服务迁移助手评估本地 Web 应用程序并将其迁移到 Azure 应用服务。
    - 虚拟桌面：评估本地虚拟桌面基础结构 (VDI) 并将其迁移到 Azure 中的 Windows 虚拟桌面。
    - Data：使用 Azure Data Box 产品快速且经济高效地将大量数据迁移到 Azure。

虽然在迁移计划的下一阶段才会处理数据库、Web 应用和虚拟桌面，但 Adatum 企业体系结构团队希望先评估使用 Azure Migrate 将其本地 Hyper-V 虚拟机迁移到 Azure VM 的情况。

## <a name="objectives"></a>目标
  
完成本实验室后，你将能够：

-  使用 Azure Migrate 为评估和迁移做好准备。
-  使用 Azure Migrate 评估 Hyper-V 的迁移。
-  使用 Azure Migrate 迁移 Hyper-V VM。

## <a name="estimated-time-120-minutes"></a>估计时间：120 分钟

## <a name="lab-environment"></a>实验室环境
  
虚拟机：AZ-801T00A-SEA-DC1 和 AZ-801T00A-SEA-SVR2 必须正在运行 。 其他 VM 可以运行，但它们不是此实验室不需要这些 VM。

> **注意**：AZ-801T00A-SEA-DC1 和 AZ-801T00A-SEA-SVR2 虚拟机托管 SEA-DC1 和 SEA-SVR2 的安装   

1. 选择 SEA-SVR2。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境和 Azure 订阅。 在开始实验室之前，请确保拥有 Azure 订阅以及具有该订阅中“所有者”角色的用户帐户。

## <a name="exercise-1-prepare-the-lab-environment"></a>练习 1：准备实验室环境

此练习的主要任务如下：

1. 使用 Azure 资源管理器快速入门模板部署 Azure VM。
1. 部署 Azure Bastion。
1. 在 Azure VM 中部署嵌套 VM。

#### <a name="task-1-deploy-an-azure-vm-by-using-an-azure-resource-manager-quickstart-template"></a>任务 1：使用 Azure 资源管理器快速入门模板部署 Azure VM

1. 在 SEA-SVR2 上，启动 Microsoft Edge，浏览到 [301-nested-vms-in-virtual-network Azure 快速启动模板](https://github.com/az140mp/azure-quickstart-templates/tree/master/demos/nested-vms-in-virtual-network)，然后选择“部署到 Azure”。  （“部署到 Azure”按钮位于 `README.md` 文件中模板创建的资源列表的后面。）
1. 在 Azure 门户中，出现提示时，使用你在此实验室中使用的订阅中具有所有者角色的用户帐户的凭据登录。
1. 在 Azure 门户的“具有嵌套 VM 的 Hyper-V 主机虚拟机”页面上，使用以下设置执行部署（其他设置保留默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0701-RG 的名称 |
   | 区域 | 可以预配 Azure VM 的 Azure 区域的名称 |
   | 虚拟网络名称 | **az801l07a-hv-vnet** |
   | 主机网络 Interface1Name | **az801l07a-hv-vm-nic1** |
   | 主机网络 Interface2Name | **az801l07a-hv-vm-nic2** |
   | 主机虚拟机名称 | **az801l07a-hv-vm** |
   | 主机管理员用户名 | **Student** |
   | 主机管理员密码 | **Pa55w.rd1234** |

   > 注意：请等待部署完成。 部署可能需要大约 10 分钟时间。

#### <a name="task-2-deploy-azure-bastion"></a>任务 2：部署 Azure Bastion 

> 注意：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

1. 在 SEA-SVR2 上，在显示 Azure 门户的浏览器窗口中，在 Cloud Shell 窗格中打开一个 PowerShell 会话。 
1. 在 Cloud Shell 窗格上的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的名为 az801l07a-hv-vnet 的虚拟网络中：  

   ```powershell
   $resourceGroupName = 'AZ801-L0701-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l07a-hv-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.0.7.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户中，使用以下设置创建一个 Azure Bastion 实例：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0701-RG** |
   | 名称 | **az801l07a-bastion** |
   | 区域 | 在本练习的先前任务中部署资源的同一 Azure 区域 |
   | 层 | **基本** |
   | 虚拟网络 | **az801l07a-hv-vnet** |
   | 子网 | **AzureBastionSubnet (10.0.7.0/24)** |
   | 公共 IP 地址 | **新建** |
   | 公共 IP 名称 | **az801l07a-hv-vnet-ip** |

   > **注意**：必须在与虚拟网络相同的 Azure 区域创建 Bastion。
   
   > 注意：在继续下一个任务之前，请等待部署完成。 部署可能需要大约 5 分钟时间。

#### <a name="task-3-deploy-a-nested-vm-in-the-azure-vm"></a>任务 3：在 Azure VM 中部署嵌套 VM

1. 在 Azure 门户中，浏览到 az801l07a-hv-vm 虚拟机页。
1. 在 az801l07a-hv-vm 页面中，使用以下凭据通过 Azure Bastion 连接到在虚拟机中运行的操作系统：

   | 设置 | 值 | 
   | --- | --- |
   | 用户名 |**Student** |
   | 密码 |**Pa55w.rd1234** |

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在“服务器管理器”窗口中，禁用“IE 增强的安全配置”。  
1. 使用文件资源管理器创建两个文件夹：F:\\VHDs 和 F:\\VMs。 
1. 使用 Microsoft Edge 从[Windows Server 评估](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)中下载 Windows Server 2022 VHD 文件，并将其复制到 F:\VHDs 文件夹。  
1. 使用 Hyper-V 管理器新建一个具有以下设置的虚拟机：

   | 设置 | 值 | 
   | --- | --- |
   | 名称 | **az801l07a-vm1** | 
   | 将虚拟机存储在其他位置 | 已选定 | 
   | 位置 | **F:\VMs** |
   | Generation | **1** |
   | 内存 | 静态设置为 2048 MB |
   | 连接 | **NestedSwitch** |
   | 虚拟硬盘 | 下载到 F:\VHDs 文件夹的 VHD 文件 |

1. 启动新预配的 VM，并使用虚拟机连接来连接到该 VM。 
1. 在“虚拟机连接”窗口中，初始化操作系统。 
1. 将内置管理员帐户的密码设置为 Pa55w.rd。 
1. 在“虚拟机连接”窗口中，登录到 az801l07a-vm1。  
1. 打开 Windows PowerShell 窗口，并通过运行以下命令将计算机名称设置为 az801l07a-vm1： 

   ```powershell
   Rename-Computer -NewName 'az801l07a-vm1' -Restart
   ```

## <a name="exercise-2-prepare-for-assessment-and-migration-by-using-azure-migrate"></a>练习 2：使用 Azure Migrate 为评估和迁移做好准备
  
此练习的主要任务如下：

1. 配置 Hyper-V 环境。
1. 创建 Azure Migrate 项目。
1. 实现目标 Azure 环境。

#### <a name="task-1-configure-hyper-v-environment"></a>任务 1：配置 Hyper-V 环境

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在浏览器窗口中，转到 [https://aka.ms/migrate/script/hyperv](https://aka.ms/migrate/script/hyperv)，下载 Azure Migrate 配置 PowerShell 脚本。

   >**注意**：脚本执行以下任务： 
   >- 检查你是否在受支持的 PowerShell 版本中运行该脚本
   >- 验证你对 Hyper-V 主机是否拥有管理权限
   >- 允许你创建 Azure Migrate 服务用于与 Hyper-V 主机通信的本地用户帐户（此用户帐户将添加到 Hyper-V 主机上的“远程管理用户”、“Hyper-V 管理员”和“性能监视用户”组。  ） 
   >- 检查主机是否正在运行受支持的 Hyper-V 版本，并检查 Hyper-V 角色
   >- 启用 WinRM 服务，并打开主机上的端口 5985 (HTTP) 和 5986 (HTTPS)（元数据收集需要这些端口。） 
   >- 在主机上启用 PowerShell 远程控制
   >- 检查主机管理的所有 VM 上是否已启用 Hyper-V Integration Services
   >- 根据需要在主机上启用 CredSSP

1. 在与 az801l07a-hv-vm 的远程桌面会话中，启动“Windows PowerShell ISE”。  
1. 在“管理员: Windows PowerShell ISE”窗口中，运行以下命令，将脚本复制到 C:\\Labfiles\\Lab07 文件夹并删除 Zone.Identifier 备用数据流，在这种情况下，该数据流指示已从 Internet 下载文件： 

   ```powershell
   New-Item -ItemType Directory -Path C:\Labfiles\Lab07 -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\MicrosoftAzureMigrate-Hyper-V.ps1" -Destination 'C:\Labfiles\Lab07'
   Unblock-File -Path C:\Labfiles\Lab07\MicrosoftAzureMigrate-Hyper-V.ps1
   Set-Location -Path C:\Labfiles\Lab07
   ```

1. 在“管理员: Windows PowerShell ISE”窗口中，打开 C:\\Labfiles\\Lab07 文件夹中的 MicrosoftAzureMigrate-Hyper-V.ps1 脚本，然后运行该脚本  。 出现确认提示时，输入 Y 并按 Enter，但以下提示除外，在这种情况下，请输入 N ，然后按 Enter ：

   - 是否使用 SMB 共享来存储 VHD？
   - 是否要为 Azure Migrate 和 Hyper-V 主机通信创建非管理员本地用户？ 

#### <a name="task-2-create-an-azure-migrate-project"></a>任务 2：创建 Azure Migrate 项目

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在浏览器窗口，转到 [Azure 门户](https://portal.azure.com)，然后使用你在此实验室中使用的订阅中具有所有者角色的用户帐户的凭据登录。
1. 在 Azure 门户中，浏览到 Azure Migrate 页面，并创建具有以下设置的项目（其他设置保留其默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0702-RG 的名称 |
   | 迁移项目 | **az801l07a-migrate-project** |
   | 地理位置 | 你所在国家或地区的名称 |

#### <a name="task-3-implement-the-target-azure-environment"></a>任务 3：实现目标 Azure 环境

1. 在 Azure 门户中，使用以下设置创建一个虚拟网络（其他设置保留默认值）：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0703-RG 的名称 |
   | 名称 | **az801l07a-migration-vnet** |
   | 区域 | 在本实验室前面将虚拟机部署到的 Azure 区域的名称 |
   | 子网名称 | **subnet0** |
   | 子网地址范围 | **10.7.0.0/24** |

1. 在 Azure 门户中，使用以下设置再创建一个虚拟网络（其他设置保留默认值）：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0703-RG** |
   | 名称 | **az801l07a-test-vnet** |
   | 区域 | 在本实验室前面将虚拟机部署到的 Azure 区域的名称 |
   | 设置 | 值 |
   | --- | --- |
   | 子网名称 | **subnet0** |
   | 子网地址范围 | **10.7.0.0/24** |

1. 在 Azure 门户中，使用以下设置创建一个存储帐户（其他设置保留默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0703-RG** |
   | 存储帐户名称 | 长度介于 3-24 个字符的任何全局唯一名称（由字母和数字组成） |
   | 位置 | 之前在此任务中创建虚拟网络的 Azure 区域的名称 |
   | 性能 | **标准** |
   | 冗余 | **本地冗余存储 (LRS)** |
   | 为 blob 启用软删除 | 禁用 |
   | 启用容器软删除 | 禁用 |

## <a name="exercise-3-assess-hyper-v-for-migration-by-using-azure-migrate"></a>练习 3：使用 Azure Migrate 评估 Hyper-V 的迁移
  
此练习的主要任务如下：

1. 部署并配置 Azure Migrate 设备
1. 配置、运行和查看评估

#### <a name="task-1-deploy-and-configure-the-azure-migrate-appliance"></a>任务 1：部署和配置 Azure Migrate 设备

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在浏览器窗口中浏览到新创建的 Azure Migrate 项目。 
1. 在“Azure Migrate | 入门”页上的“服务器、数据库和 Web 应用”部分，选择“发现、评估和迁移”，然后转到“发现”页面。   
1. 在“发现”页上，确保已选中“发现正在使用的设备”选项，然后将“服务器是否已虚拟化?”设置的值设为“是，使用 Hyper-V”。    
1. 在“发现”页上，将设备的值设置为 az801l07a-vma1，并生成相应的密钥。 

   >**注意**：等待密钥生成完毕并记录其值。 本练习中稍后会用到它。

1. 从“发现”页将设备 VM 的 .VHD 文件下载到 F:\VMs 文件夹。  

   >**注意**：等待下载完成。 这可能需要大约 5 分钟。

1. 下载完成后，将下载的 .ZIP 内容提取到 F:\VMs 文件夹中。 
1. 在与 az801l07a-hv-vm 的远程桌面会话中，使用 Hyper-V 管理器控制台将复制其 VM 文件的虚拟机导入 F:\VMs 文件夹。   使用“就地注册虚拟机(使用现有的唯一 ID)”选项，将“虚拟处理器数”设置为 4，然后将它连接到 NestedSwitch。   

   >**注意**：在实验室环境中，可忽略任何引用虚拟处理器数量更改的错误消息。 在生产场景中，应确保虚拟设备分配了足够数量的计算资源。

1. 将新导入的虚拟机重命名为 az801l07a-vma1。
1. 启动新导入的虚拟机，然后使用“虚拟机连接”窗口连接到该虚拟机。 
1. 在到虚拟设备的“虚拟机连接”窗口中，接受许可条款，并将内置管理员帐户的密码设置为 Pa55w.rd。   
1. 在虚拟机的“虚拟机连接”窗口中，在“操作”菜单中，选择“Ctrl + Alt + Delete”，然后在出现提示时，使用新设置的密码登录  。

   >**注意**：在虚拟设备的“虚拟机连接”窗口中，将将自动打开显示“设备配置服务器”的浏览器窗口 。

1. 在“设备配置服务器”页上，选择“我同意”按钮，并等待设置先决条件验证成功 。 
1. 在“设备配置管理器”页上的“通过将密钥粘贴到此处注册 Hyper-V 设备”文本框中，将复制的密钥粘贴到此练习前面的记事本中，选择“登录”，然后使用设备代码登录到你的订阅。   
1. 当系统提示“是否要尝试登录到 Microsoft Azure PowerShell?”时，请选择“继续”，再关闭新打开的浏览器选项卡。 
1. 在“设备配置管理器”页上，验证注册是否成功。
1. 在“设备配置管理器”页上的“管理凭据和发现源”部分，使用以下设置添加凭据： 

   | 设置 | 值 | 
   | --- | --- |
   | 友好名称 | **az801l07ahvcred** | 
   | 用户名 | **Student** |
   | 密码 | **Pa55w.rd1234** |

1. 在浏览器窗口的“设备配置服务器”页面上，在“提供 Hyper-V 主机/群集详细信息”部分，添加设置为“Hyper-V 主机/群集”的发现源，将其友好名称设置为 az801l07ahvcred，并将其 IP 地址/FQDN 设置为 10.0.2.1。      

   >**注意**：10.0.2.1 是附加到内部交换机的 Hyper-V 主机的网络接口的 IP 地址。

1. 在“设备配置管理器”页上的“提供 Hyper-V 主机/群集详细信息”部分，启动发现功能。  

   >**注意**：每个主机可能需要大约 15 分钟才能发现服务器的元数据显示在 Azure 门户。

#### <a name="task-2-configure-run-and-view-an-assessment"></a>任务 2：配置、运行和查看评估

1. 从虚拟设备的“虚拟机连接”窗口，切换到与 az801l07a-hv-vm 的远程桌面会话 。 在显示 Azure 门户的浏览器窗口中，浏览 回“Azure Migrate | 服务器、数据库和 Web 应用”页，然后选择“刷新” 。 在“Azure Migrate: 发现和评估”部分，选择”评估”，然后在下拉菜单中选择 Azure VM  。
1. 在“创建评估”页上，指定以下设置（其他设置保留默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 目标位置 | 你在此实验室中使用的 Azure 区域的名称 |
   | 存储类型 | **高级托管磁盘** |
   | 预留实例 | **没有预留实例** |
   | 调整大小标准 | **作为本地** |
   | VM 系列 | **Dsv3_series** |
   | 舒适因子 | **1** |
   | 产品/服务 | **即用即付** |
   | 货币 | 美元 ($) | 
   | 折扣 | **0** |
   | VM 运行时间 | 每月 31 天，每天 24 小时  | 
   | 评估名称 | az801l07a-assessment** |
   | 选择或创建组 | **az801l07a-assessment-group** |
   | 要添加到组的计算机列表 | **az801l07a-vm1** |

   >**注意**：考虑到实验室环境固有的有限时间，在这种情况下唯一可行的选项是“作为本地”评估。 

1. 返回到“Azure Migrate \| 服务器、数据库和 Web 应用”页上，选择“刷新” 。 在“Azure Migrate: 发现和评估”部分，验证“总评估数”行是否包含 1 个条目，然后将其选中   。
1. 在“Azure Migrate: 发现和评估 \| 评估”页上，选择新创建的评估 az801l07a-assessment 。 
1. 在 az801l07a-assessment 页上，查看指示 Azure 就绪情况以及计算和存储的每月成本估算的信息。 

   >**注意**：在真实场景中，应考虑安装 Dependency Agent，以在评估阶段提供对服务器依赖项的更多见解。

## <a name="exercise-4-migrate-hyper-v-vms-by-using-azure-migrate"></a>练习 4：使用 Azure Migrate 迁移 Hyper-V VM
  
此练习的主要任务如下：

1. 为迁移 Hyper-V VM 做好准备。
1. 配置 Hyper-V VM 的复制。
1. 执行 Hyper-V VM 的迁移。
1. 删除实验室中部署的 Azure 资源。

#### <a name="task-1-prepare-for-migration-of-hyper-v-vms"></a>任务 1：为迁移 Hyper-V VM 做好准备

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在显示 Azure 门户的浏览器窗口中，浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页面 。 
1. 在“Azure Migrate 服务器、数据库和 Web 应用”页的“Azure Migrate: 服务器迁移”部分，选择“发现”链接  。 
1. 在“发现”页上，通过指定以下设置来创建资源（其他设置保留默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 计算机是否已虚拟化？ | **是，使用 Hyper-V** |
   | 目标区域 | 你在此实验室中使用的 Azure 区域的名称 | 
   | 确认迁移的目标区域 | 已选定 |

   >**注意**：此步骤会自动触发对 Azure Site Recovery 保管库的预配操作。

1. 在“发现”页上的“步骤 1. 准备 Hyper-V 主机服务器”，选择第一个“下载”链接（而不是“下载”按钮），以便下载 Hyper-V 复制提供程序软件安装程序   。

   > **注意：** 如果收到浏览器通知，指示无法安全下载 AzureSiteRecoveryProvider.exe，请打开“下载”链接的上下文相关菜单，然后在菜单中选择“复制”链接 。 在同一浏览器窗口中打开另一个选项卡，粘贴复制的链接，然后按 Enter。

1. 使用下载的文件，通过默认设置安装 Azure Site Recovery 提供程序。
1. 在安装过程中，切换到 Azure 门户，然后在“发现计算机”页上，在准备本地 Hyper-V 主机的过程的步骤 1 中，选择“下载”按钮以下载保管库注册密钥，然后用它来注册 Azure Site Recovery 提供程序。  
1. 刷新显示“发现”页的浏览器窗口。 
1. 在“Azure Migrate 服务器、数据库和 Web 应用”页的“Azure Migrate: 服务器迁移”部分，选择“发现”链接  。 
1. 在“发现”页上完成注册。

   >**注意**：完成虚拟机的发现最长可能需要 15 分钟。

#### <a name="task-2-configure-replication-of-hyper-v-vms"></a>任务 2：配置 Hyper-V VM 的复制

1. 收到注册已完成的确认后，请浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页，接着在“Azure Migrate: 服务器迁移”部分，选择“复制”链接。   

   >**注意**：可能需要刷新显示“Azure Migrate | 服务器、数据库和 Web 应用”页面的浏览器页面。

1. 在“复制”页上的“计算机是否虚拟化?”下拉列表中，选择“是，使用 Hyper-V”。  
1. 在“复制”页的“虚拟机”选项卡上，指定以下设置（其他设置保留默认值）： 

   | 设置 | 值 | 
   | --- | --- |
   | 从 Azure Migrate 评估导入迁移设置 | **是，从 Azure Migrate 评估应用迁移设置** |
   | 选择组 | **az801l07a-assessment-group** |
   | 选择评估 | **az801l07a-assessment** |
   | 虚拟机 | **az801l07a-vm1** |

1. 在“复制”页的“目标设置”选项卡上，指定以下设置（其他设置保留默认值）： 

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0703-RG** |
   | 复制存储帐户 | 先前在此实验室中创建的存储帐户的名称 | 
   | 虚拟网络 | **az801l07a-migration-vnet** |
   | 子网 | **subnet0** |

1. 在“复制”页的“计算”选项卡上，确保“虚拟机大小”下拉列表中已选中“Standard_D2s_v3”   。 在“OS 类型”下拉列表中，选择“Windows”。 
1. 若要监视复制状态，请返回到“Azure Migrate | 服务器、数据库和 Web 应用”页面，选择“刷新”，然后在“Azure Migrate: 服务器迁移”部分，选择“复制服务器”条目   。 在“Azure Migrate: 服务器迁移 | 复制计算机”页上，检查复制计算机列表中的“状态”列 。 
1. 等待状态更改为“受保护”。 这可能额外需要 15 分钟时间。

   >**注意**：需要刷新“Azure Migrate: 服务器迁移 | 复制计算机”才能更新状态信息。 

#### <a name="task-3-perform-migration-of-hyper-v-vms"></a>任务 3：执行 Hyper-V VM 迁移

1. 在 Azure 门户中，在“Azure Migrate: 服务器迁移 | 复制计算机”页面，选择表示 az801l07a-vm1 虚拟机的条目 。
1. 在 az801l07a-vm1 页中，使用 az801l07a-test-vnet 虚拟网络作为目标来启动测试迁移。  

   >**注意**：等待测试迁移完成。 这可能需要大约 5 分钟。

1. 在 Azure 门户，转到“虚拟机”页面，并记下表示新复制的虚拟机 az801l07a-vm1-test 的条目。 

   > **注意：** 最初，虚拟机的名称由 asr- 前缀和随机生成的后缀组成，但最终将重命名为 az801l07a-vm1-test 。

1. 在 Azure 门户中，浏览回“Azure Migrate: 服务器迁移 | 复制计算机”页面，选择“刷新”，然后验证 az801l07a-vm1 虚拟机是否以“清理测试故障转移挂起”状态列出   。
1. 在“Azure Migrate: 服务器迁移 | 复制计算机”页面上，转到 az801l07a-vm1 复制计算机页，然后出发“清理测试迁移”操作，指定“测试已完成。删除测试虚拟机”。   
1. 测试故障转移清理作业完成后，刷新显示 az801l07a-vm1 复制计算机页的浏览器页面，你将发现，工具栏中的“迁移”图标会自动变为可用 。
1. 在 az801l07a-vm1 复制计算机页上，触发“迁移”操作。  
1. 在“迁移”页上，确保选中“迁移前关闭计算机以最大程度地减少数据丢失?”选项。 
1. 若要监视迁移状态，请浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页面。 在“Azure Migrate: 服务器迁移”部分，选择“复制服务器”条目，然后在“Azure Migrate: 服务器迁移 | 复制计算机”页面，检查复制计算机列表中的“状态”列   。 验证状态是否显示“计划的故障转移已完成”状态。

   >**注意**：迁移应是不可逆的操作。 如果需要查看已完成的信息，请浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页面，刷新该页面，然后验证“Azure Migrate: 服务器迁移”部分的“已迁移的服务器数”条目的值是否为 1   。

#### <a name="task-4-remove-azure-resources-deployed-in-the-lab"></a>任务 4：删除实验室中部署的 Azure 资源

1. 在 SEA-SVR2 上，在显示 Azure 门户的浏览器窗口中，在 Azure Cloud Shell 窗格中打开一个 PowerShell 会话。 
1. 在 Cloud Shell 窗格中，运行以下命令，列出你在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*'
   ```

   > 备注：验证输出是否仅包含你在此实验室中创建的资源组。 此任务中将删除该组。

1. 在 Cloud Shell 窗格中，运行以下命令，删除你在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **注意：** 该命令以异步方式执行（由 -AsJob 参数决定），因此，虽然你可以随后立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。
