---
lab:
  title: 实验室：使用 Azure Migrate 将 Hyper-V VM 迁移到 Azure
  type: Answer Key
  module: 'Module 7: Design for Migration'
---

# <a name="lab-answer-key-migrating-hyper-v-vms-to-azure-by-using-azure-migrate"></a>实验室答案：使用 Azure Migrate 将 Hyper-V VM 迁移到 Azure

                **注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Migrating%20Hyper-V%20VMs%20to%20Azure%20by%20using%20Azure%20Migrate)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## <a name="exercise-1-prepare-the-lab-environment"></a>练习 1 - 准备实验室环境

#### <a name="task-1-deploy-an-azure-vm-by-using-an-azure-resource-manager-quickstart-template"></a>任务 1：使用 Azure 资源管理器快速入门模板部署 Azure VM

1. 连接到 SEA-SVR2，然后根据需要，以 CONTOSO\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-SVR2 上，启动 Microsoft Edge，转至 [301-nested-vms-in-virtual-network Azure 快速启动模板](https://github.com/az140mp/azure-quickstart-templates/tree/master/demos/nested-vms-in-virtual-network)，然后选择“部署到 Azure”  。 （你会在模板创建的资源列表之后的 `README.md` 文件中找到“部署到 Azure”按钮。）这会自动将浏览器重定向到 Azure 门户中的“具有嵌套 VM 的 Hyper-V 主机虚拟机”页面 。
1. 在 Azure 门户中，出现提示时，使用你在此实验室中使用的订阅中具有所有者角色的用户帐户的凭据登录。
1. 在 Azure 门户的“具有嵌套 VM 的 Hyper-V 主机虚拟机”页面上，指定以下设置（保留其他设置为默认值。）：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0701-RG 的名称 |
   | 区域 | 可以预配 Azure VM 的 Azure 区域的名称 |
   | 虚拟网络名称 | **az801l07a-hv-vnet** |
   | 主机网络 Interface1Name | **az801l07a-hv-vm-nic1** |
   | 主机网络 Interface2Name | **az801l07a-hv-vm-nic2** |
   | 主机虚拟机名称 | **az801l07a-hv-vm** |
   | 主机管理员用户名 | **学生** |
   | 主机管理员密码 | **Pa55w.rd1234** |

1. 在“具有嵌套 VM 的Hyper-V 主机虚拟机”页面上，选择“查看 + 创建”，然后选择“创建”  。

   > **注意**：等待部署完成。 部署可能需要大约 10 分钟时间。

#### <a name="task-2-deploy-azure-bastion"></a>任务 2：部署 Azure Bastion 

> **注意**：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

1. 在 SEA-SVR2 上，在显示 Azure 门户的浏览器窗口中，通过选择 Azure 门户中的 Cloud Shell 按钮打开“Azure Cloud Shell”窗格  。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。

   > **注意：** 如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格上的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的虚拟网络 az801l07a-hv-vnet 中  ：

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
1. 在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择 Bastions，然后在 Bastions 页面上，选择“+ 创建”   。
1. 在“创建 Bastion”页面的基本标签上，指定以下设置并选择“查看 + 创建”  ：

   | 设置 | Value | 
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

1. 在“创建 Bastion”页面的“查看 + 创建”标签上，选择“创建”  ：

   > **注意**：在继续下一个任务之前，请等待部署完成。 部署可能需要大约 5 分钟时间。

#### <a name="task-3-deploy-a-nested-vm-in-the-azure-vm"></a>任务 3：在 Azure VM 中部署嵌套 VM

1. 在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟机”，然后在“虚拟机”页面上，选择 az801l07a-hv-vm   。
1. 在 az801l07a-hv-vm 页面上，选择“连接”，然后在下拉菜单中选择“Bastion”  。 在“az801l07a-hv-vm \| 连接”页面的的“Bastion”选项卡上，选择“使用 Bastion”  。
1. 出现提示时，提供以下凭据并选择“连接”：

   | 设置 | 值 | 
   | --- | --- |
   | 用户名 |**学生** |
   | 密码 |**Pa55w.rd1234** |

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在“服务器管理器”窗口中，选择“本地服务器”，然后选择“IE 增强的安全配置”标签旁边的“开启”链接    。 在“IE 增强的安全配置”对话框中，选择两个“关闭”选项，然后选择“确定”  。
1. 在远程桌面会话中，打开文件资源管理器并浏览到“F:”盘。 创建两个文件夹“F:\\VHD”和“F:\\VM” 。 
1. 在与 az801l07a-hv-vm 的远程桌面会话中，启动 Microsoft Edge，完成初始设置，转到 [Windows Server 评估](https://techcommunity.microsoft.com/t5/windows-11/accessing-trials-and-kits-for-windows-eval-center-workaround/m-p/3361125)，提供请求的信息，下载 Windows Server 2022 VHD 文件，然后将其复制到“F:\VHD”文件夹  。 
1. 在与 az801l07a-hv-vm 的远程桌面会话中，选择“开始”，再选择“Windows 管理工具”，然后选择“Hyper-V 管理器”   。 
1. 在“Hyper-V 管理器”控制台中 ，选择 az801l07a-hv-vm 节点 。 
1. 选择“新建”，然后在级联菜单中选择“虚拟机” 。 这将启动“新建虚拟机向导”。 
1. 在“新建虚拟机”向导的“准备工作”页面上，单击“下一步 >”  。
1. 在“新建虚拟机”向导的“指定名称和位置”页面上，指定以下设置，然后选择“下一步 >”  ：

   | 设置 | 值 | 
   | --- | --- |
   | 名称 | **az801l07a-vm1** | 
   | 将虚拟机存储在其他位置 | 已选定 | 
   | 位置 | **F:\VM** |

1. 在“新建虚拟机”向导的“指定代系”页上，确保选中“第 1 代”选项，然后选择“下一步 >”   。
1. 在“新建虚拟机”向导的“分配内存”页上，将“启动内存”设置为 2048，然后选择“下一步 >”    。
1. 在“新建虚拟机”向导的“配置网络”页上的“连接”下拉列表中，选择 NestedSwitch，然后选择“下一步 >”    。
1. 在“新建连接”向导的“虚拟硬盘”页上，选择“使用现有虚拟硬盘”选项，将位置设置为下载到“F:\VHD”文件夹的 VHD 文件，然后选择“下一步 >”     。
1. 在“新建虚拟机向导”的“摘要”页上，选择“完成”  。
1. 在“Hyper-V 管理器”控制台中，选择新创建的虚拟机，然后选择“启动” 。 
1. 在“Hyper-V 管理器”控制台中，验证虚拟机是否正在运行，然后选择“连接” 。 
1. 在 az801l07a-vm1 的“虚拟机连接”窗口中，在“Hi there”页面上，选择“下一步”   。 
1. 在 az801l07a-vm1 的“虚拟机连接”窗口中，在“许可条款”页面上，选择“接受”   。 
1. 在 az801l07a-vm1 的“虚拟机连接”窗口中，在“自定义设置”页上，将内置管理员帐户的密码设置为 Pa55w.rd，然后选择“完成”    。 
1. 在 az801l07a-vm1 的“虚拟机连接”窗口中，在“操作”菜单中，选择“Ctrl + Alt + Delete”，然后在出现提示时，使用新设置的密码登录   。
1. 在 az801l07a-vm1 的“虚拟机连接”窗口中，选择“启动”  。 在“开始”菜单中，选择“Windows PowerShell”，然后在“管理员：Windows PowerShell”窗口中，运行以下命令以设置计算机名称  。 

   ```powershell
   Rename-Computer -NewName 'az801l07a-vm1' -Restart
   ```

## <a name="exercise-2-prepare-for-assessment-and-migration-by-using-azure-migrate"></a>练习 2：使用 Azure Migrate 为评估和迁移做好准备
  
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

1. 在与 az801l07a-hv-vm 的远程桌面会话中，选择“启动”，然后选择“Windows PowerShell ISE”  。 
1. 在“管理员: Windows PowerShell ISE”窗口的控制台窗格中，运行以下命令，将脚本复制到 C:\\Labfiles\\Lab07 文件夹并删除 Zone.Identifier 备用数据流，在这种情况下，该数据流指示已从 Internet 下载文件 ：

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
1. 在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择 Azure Migrate，然后在“Azure Migrate \| 入门”页上的“服务器、数据库和 Web 应用”部分，选择“发现、评估和迁移”    。
1. 在“Azure Migrate \| 服务器、数据库和 Web 应用”页上，选择“创建项目” 。
1. 在“创建项目”页上，指定以下设置（将其他设置保留为默认值），然后选择“创建” ：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0702-RG 的名称 |
   | 迁移项目 | **az801l07a-migrate-project** |
   | 地理位置 | 你所在国家或地区的名称 |

#### <a name="task-3-implement-the-target-azure-environment"></a>任务 3：实现目标 Azure 环境

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟网络”  。 在“虚拟网络”页上 ，选择命令栏上的“+ 创建” 。 
1. 在“创建虚拟网络”页的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值）并选择“下一步: IP 地址”  ：

   | 设置 | Value |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0703-RG 的名称 |
   | 名称 | **az801l07a-migration-vnet** |
   | 区域 | 在本实验室前面将虚拟机部署到的 Azure 区域的名称 |

1. 在“创建虚拟网络”页的“IP 地址”选项卡上，选择“回收站”图标  。 在“IPv4 地址空间”文本框中，键入 10.7.0.0/16 并选择“+ 添加子网”  。
1. 在“添加子网”页上，指定以下设置（将其他设置保留为默认值），然后选择“添加” ：

   | 设置 | 值 |
   | --- | --- |
   | 子网名称 | **subnet0** |
   | 子网地址范围 | **10.7.0.0/24** |

1. 返回到“创建虚拟网络”页的“IP 地址”选项卡上，选择“查看 + 创建”  。
1. 在“创建虚拟网络”页的“查看 + 创建”选项卡上，选择“创建”  。
1. 在 Azure 门户中，浏览回“虚拟网络”页并选择工具栏上的“+ 创建” 。
1. 在“创建虚拟网络”页的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值）并选择“下一步: IP 地址”  ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0703-RG** |
   | 名称 | **az801l07a-test-vnet** |
   | 区域 | 在本实验室前面将虚拟机部署到的 Azure 区域的名称 |

1. 在“创建虚拟网络”页的“IP 地址”选项卡上，选择“回收站”图标  。 在“IPv4 地址空间”文本框中，键入 10.7.0.0/16 并选择“+ 添加子网”  。
1. 在“添加子网”页上，指定以下设置（将其他设置保留为默认值），然后选择“添加” ：

   | 设置 | 值 |
   | --- | --- |
   | 子网名称 | subnet0 |
   | 子网地址范围 | **10.7.0.0/24** |

1. 返回到“创建虚拟网络”页的“IP 地址”选项卡上，选择“查看 + 创建”  。
1. 在“创建虚拟网络”页的“查看 + 创建”选项卡上，选择“创建”  。
1. 在 Azure 门户中，搜索并选择“存储帐户”。 然后，在“存储帐户”页上，选择命令栏上的“+ 创建” 。
1. 在“创建存储帐户”页的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值） ：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0703-RG** |
   | 存储帐户名称 | 长度介于 3-24 个字符的任何全局唯一名称（由字母和数字组成） | 
   | 位置 | 之前在此任务中创建虚拟网络的 Azure 区域的名称 |
   | 性能 | **标准** |
   | 冗余 | **本地冗余存储 (LRS)** |

1. 在“创建存储帐户”页的“基本信息”选项卡上，选择“数据保护”选项卡  。
1. 在“创建存储帐户”页的“数据保护”选项卡上，清除“为 Blob 启用软删除”和“为容器启用软删除”复选框，然后选择“查看 + 创建”    。
1. 在“查看 + 创建”选项卡上，选择“创建”。 

## <a name="exercise-3-assess-hyper-v-for-migration-by-using-azure-migrate"></a>练习 3：使用 Azure Migrate 评估 Hyper-V 的迁移
  
#### <a name="task-1-deploy-and-configure-the-azure-migrate-appliance"></a>任务 1：部署和配置 Azure Migrate 设备

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在 Azure 门户的浏览器窗口，搜索并选择 Azure Migrate 。
1. 在“Azure Migrate | 入门”页上的“服务器、数据库和 Web 应用”部分，选择“发现、评估和迁移”  。
1. 在“Azure Migrate \| 服务器、数据库和 Web 应用”页的“Azure Migrate: 发现和评估”部分，选择“发现”  。
1. 在 “发现”页上，确保已选中“使用设备发现”选项，然后在“是否已虚拟化服务器”下拉列表中，选择“是，使用 Hyper-V”   。 
1. 在“发现”页上的“为设备命名”文本框中，键入 az801l07a-vma1，然后选择“生成密钥”按钮   。

   >**注意**：等待密钥生成完成并记录其值。 本练习中稍后会用到它。

1. 在“发现”页上的“下载 Azure Migrate 设备”文本框中，选择“.VHD 文件”选项，选择“下载”，然后在出现提示时，将下载位置设置为 F:\VM 文件夹    。

   >**注意**：等待下载完成。 这可能需要大约 5 分钟。

1. 下载完成后，将下载的 .ZIP 内容提取到 F:\VM 文件夹中。

   >**注意**：由于 Microsoft Edge 默认不提示，你可能需要手动将 .VHD 文件复制到 F:\VMs 文件夹。

1. 在与 az801l07a-hv-vm 的远程桌面会话中，切换到“Hyper-V 管理器”控制台，选择 AZ801L07A-VM1 节点，然后选择“导入虚拟机”   。 这将启动“导入虚拟机”向导。
1. 在“导入虚拟机”向导的“准备工作”页面上，单击“下一步 >”  。
1. 在“导入虚拟机”向导的“查找文件夹”页上，指定提取的“虚拟机”文件夹的位置，然后选择“下一步 >”。   
1. 在“导入虚拟机”向导的“选择虚拟机”页上，选择“下一步 >”。  
1. 在“导入虚拟机”向导的“选择导入类型”页上，选择“就地注册虚拟机(使用现有的唯一 ID)”，然后选择“下一步 >”   。
1. 在“导入虚拟机”向导的“配置处理器”页上，将“虚拟处理器数目”设置为4，然后选择“下一步 >”    。

   >**注意**：在实验室环境中，可忽略任何引用虚拟处理器数量更改的错误消息。 在生产场景中，应确保虚拟设备分配了足够数量的计算资源。

1. 在“导入虚拟机”向导的“配置网络”页上的“连接”下拉列表中，选择 NestedSwitch，然后选择“下一步 >”    。
1. 在“导入虚拟机”向导的“摘要”页上，选择“完成”  。

   >**注意**：等待导入完成。

1. 在“Hyper-V 管理器”控制台中，选择新导入的虚拟机，选择“重命名”，然后将其名称设置为 az801l07a-vma1  。
1. 将虚拟机的内存大小增加到 4096 MB。
1. 在“Hyper-V 管理器”控制台中，选择新导入的虚拟机，然后选择“启动” 。 
1. 在“Hyper-V 管理器”控制台中，验证虚拟机是否正在运行，然后选择“连接” 。 
1. 在虚拟设备的“虚拟机连接”窗口中，在“许可条款”页面上，选择“接受”  。 
1. 在虚拟机的“虚拟机连接”窗口中，在“自定义设置”页上，将内置管理员帐户的密码设置为 Pa55w.rd，然后选择“完成”   。 
1. 在虚拟机的“虚拟机连接”窗口中，在“操作”菜单中，选择“Ctrl + Alt + Delete”，然后在出现提示时，使用新设置的密码登录  。

   >**注意**：在虚拟设备的“虚拟机连接”窗口中，将将自动打开显示“设备配置服务器”的浏览器窗口 。

1. 在“设备配置服务器”页上，选择“我同意”按钮，并等待设置先决条件验证成功 。 
1. 在“设备配置管理器”页上的“注册 Azure Migrate”部分，在“通过将密钥粘贴到此处注册 Hyper-V 设备”文本框中，将复制的密钥粘贴到此练习前面的记事本中，选择“登录”，然后选择“复制代码和登录名”    。 这将自动打开新的浏览器选项卡，提示你输入复制的代码。
1. 在新打开的浏览器选项卡的“输入代码”窗格中，粘贴复制到剪贴板上的代码，然后选择“下一步” 。 出现提示时，使用你在此实验室中使用的订阅中具有所有者角色的用户帐户的凭据登录。
1. 当系统提示“是否要尝试登录到 Microsoft Azure PowerShell?”时，请选择“继续”，然后关闭新打开的浏览器选项卡 。
1. 在“设备配置管理器”页上的浏览器窗口中，验证注册是否成功。
1. 在“设备配置管理器”页上的“管理凭据和发现源”部分，选择“添加凭据”  。 在“添加凭据”窗格上，指定以下设置，然后选择“保存” ：

   | 设置 | 值 | 
   | --- | --- |
   | 友好名称 | **az801l07ahvcred** |   
   | 用户名 | **学生** |
   | 密码 | **Pa55w.rd1234** |

1. 在“设备配置管理器”页上的浏览器窗口中，在“提供 Hyper-V 主机/群集详细信息”部分，选择“添加发现源”  。 在“添加发现源”窗格上，选择“添加单个项”选项 。 确保“发现源”下拉列表设置为“Hyper-V 主机/群集” 。 在“ 映射凭据”下拉列表中，选择 az801l07ahvcred 条目，在“IP 地址/FQDN”文本框中，键入 10.0.2.1，然后选择“保存”    。

   >**注意**：10.0.2.1 是附加到内部交换机的 Hyper-V 主机的网络接口的 IP 地址。

1. 在“设备配置管理器”页上的“提供 Hyper-V 主机/群集详细信息”部分中，启用“如果不想执行这些功能，请禁用滑块”切换按钮，然后选择“开始发现”。   

   >**注意**：每个主机可能需要大约 15 分钟才能发现服务器的元数据显示在 Azure 门户。

#### <a name="task-2-configure-run-and-view-an-assessment"></a>任务 2：配置、运行和查看评估

1. 从虚拟设备的“虚拟机连接”窗口，切换到与 az801l07a-hv-vm 的远程桌面会话 。 在显示 Azure 门户的浏览器窗口中，浏览 回“Azure Migrate | 服务器、数据库和 Web 应用”页，然后选择“刷新” 。 在“Azure Migrate: 发现和评估”部分，选择”评估”，然后在下拉菜单中选择 Azure VM  。
1. 在“创建评估”页的“基本信息”选项卡上，在“评估属性”标签旁边选择“编辑”   。   
1. 在“评估属性”页上，指定以下设置（将其他设置保留为默认值），然后选择“保存” ：

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

   >**注意**：考虑到实验室环境固有的有限时间，在这种情况下唯一可行的选项是“作为本地”评估。 

1. 返回到“创建评估”页的“基本信息”选项卡，选择“下一步”以显示“选择要评估的服务器”选项卡   。
1. 在“选择要评估的服务器”选项卡上，将 评估名称设置为 az801l07a-assessment  。
1. 确保选中“选择或创建组”设置的“新建”选项，将组名称设置为 az801l07a-assessment-group，然后在要添加到组的计算机列表中选择 az801l07a-vm1   。
1. 选择“下一步”，然后选择“创建评估” 。 
1. 返回到“Azure Migrate \| 服务器、数据库和 Web 应用”页上，选择“刷新” 。 在“Azure Migrate: 发现和评估”部分，验证“总评估数”行是否包含 1 个条目，并将其选中   。
1. 在“Azure Migrate: 发现和评估 \| 评估”页上，选择新创建的评估 az801l07a-assessment 。 
1. 在 az801l07a-assessment 页上，查看指示 Azure 就绪情况以及计算和存储的每月成本估算的信息。 

   >**注意**：在真实场景中，应考虑安装 Dependency Agent，以在评估阶段提供对服务器依赖项的更多见解。

## <a name="exercise-4-migrate-hyper-v-vms-by-using-azure-migrate"></a>练习 4：使用 Azure Migrate 迁移 Hyper-V VM

#### <a name="task-1-prepare-for-migration-of-hyper-v-vms"></a>任务 1：为迁移 Hyper-V VM 做好准备

1. 在与 az801l07a-hv-vm 的远程桌面会话中，在显示 Azure 门户的浏览器窗口中，浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页面 。 
1. 在“Azure Migrate 服务器、数据库和 Web 应用”页的“Azure Migrate: 服务器迁移”部分，选择“发现”链接  。 
1. 在“发现”页上，指定以下设置（将其他设置保留为默认值），然后选择“创建资源” ：

   | 设置 | 值 | 
   | --- | --- |
   | 计算机是否已虚拟化？ | **是，使用 Hyper-V** |
   | 目标区域 | 你在此实验室中使用的 Azure 区域的名称 | 
   | 确认迁移的目标区域 | 已选定 | 

   >**注意**：此步骤会自动触发对 Azure Site Recovery 保管库的预配操作。

1. 在“发现”页上的“步骤 1. 准备 Hyper-V 主机服务器”，选择第一个“下载”链接（而不是“下载”按钮），以便下载 Hyper-V 复制提供程序软件安装程序   。

   > **注意：** 如果收到浏览器通知，指示无法安全下载 AzureSiteRecoveryProvider.exe，请打开“下载”链接的上下文相关菜单，然后在菜单中选择“复制”链接 。 在同一浏览器窗口中打开另一个选项卡，粘贴复制的链接，然后按 Enter。

1. 下载完成后，在浏览器“下载”部分选择“打开文件”链接 。 这将启动 Azure Site Recovery Provider Setup（Hyper-V 服务器）向导。
1. 在 Azure Site Recovery Provider Setup（Hyper-V 服务器）向导的“Microsoft 更新”页面上，选择“关闭”，然后选择“下一步”   。
1. 在 Azure Site Recovery Provider Setup（Hyper-V 服务器）向导的“提供程序安装”页面上，选择“安装”  。
1. 切换到 Azure 门户，然后在“发现计算机”页上，在准备本地 Hyper-V 主机的过程的步骤 1 中，选择“下载”按钮以下载保管库注册密钥 。
1. 切换到 Azure Site Recovery Provider Setup（Hyper-V 服务器）向导的“提供程序安装”页面，选择“注册”  。 这将启动 Microsoft Azure Site Recovery 注册向导。
1. 在 Microsoft Azure Site Recovery 注册向导的“保管库设置”页面，选择“浏览”，浏览到“Downloads”文件夹，选择保管库凭据文件，然后选择“打开”    。
1. 返回到 Microsoft Azure Site Recovery 注册向导的“保管库设置”页上，选择“下一步”  。
1. 在 Microsoft Azure Site Recovery 注册向导的“代理设置”页上，接受默认设置，然后选择“下一步”  。
1. 在 Microsoft Azure Site Recovery 注册向导的“注册”页上，选择“完成”  。
1. 刷新显示“发现”页的浏览器窗口。 这会将你重定向到“Azure Migrate | 服务器、数据库和 Web 应用”页面。
1. 在“Azure Migrate 服务器、数据库和 Web 应用”页的“Azure Migrate: 服务器迁移”部分，选择“发现”链接  。 
1. 在“发现”页上，选择“完成注册” 。

   >**注意**：完成虚拟机的发现最长可能需要 15 分钟。

#### <a name="task-2-configure-replication-of-hyper-v-vms"></a>任务 2：配置 Hyper-V VM 的复制

1. 收到注册已完成的确认后，请浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页，然后在“Azure Migrate: 服务器迁移”部分，选择“复制 ”链接  。 

   >**注意**：可能需要刷新显示“Azure Migrate | 服务器、数据库和 Web 应用”页面的浏览器页面。

1. 在“复制”页的“源设置”选项卡上，在“是否虚拟化计算机?”下拉列表中，选择“是，使用 Hyper-V”，然后选择“下一步”    。
1. 在“复制”页的“虚拟机”选项卡上，指定以下设置（将其他设置保留为默认值），然后选择“下一步”  ：

   | 设置 | 值 | 
   | --- | --- |
   | 从 Azure Migrate 评估导入迁移设置 | **是，从 Azure Migrate 评估应用迁移设置** |
   | 选择组 | **az801l07a-assessment-group** |
   | 选择评估 | **az801l07a-assessment** |
   | 虚拟机 | **az801l07a-vm1** |

1. 在“复制”页的“目标设置”选项卡上，指定以下设置（将其他设置保留为默认值），然后选择“下一步”  ：

   | 设置 | Value | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0703-RG** |
   | 复制存储帐户 | 先前在此实验室中创建的存储帐户的名称 | 
   | 虚拟网络 | **az801l07a-migration-vnet** |
   | 子网 | **subnet0** |

1. 在“复制”页的“计算”选项卡上，确保“虚拟机大小”下拉列表中已选中“Standard_D2s_v3”   。 在“OS 类型”下拉列表中，选择 Windows，然后选择“下一步”  。  
1. 在“复制”页的“磁盘”选项卡上，接受默认设置，然后选择“下一步”  。
1. 在“复制”页的“标记”选项卡上，接受默认设置，然后选择“下一步”  。
1. 在“复制”页的“查看 + 启动复制”选项卡上，选择“复制”  。  
1. 若要监视复制状态，请返回到“Azure Migrate | 服务器、数据库和 Web 应用”页面，选择“刷新”，然后在“Azure Migrate: 服务器迁移”部分，选择“复制服务器”条目   。 在“Azure Migrate: 服务器迁移 | 复制计算机”页上，检查复制计算机列表中的“状态”列 。
1. 等待状态更改为“受保护”。 这可能需要 15 分钟。

   >**注意**：需要刷新“Azure Migrate: 服务器迁移 | 复制计算机”才能更新状态信息 。

#### <a name="task-3-perform-migration-of-hyper-v-vms"></a>任务 3：执行 Hyper-V VM 迁移

1. 在 Azure 门户中，在“Azure Migrate: 服务器迁移 | 复制计算机”页面，选择表示 az801l07a-vm1 虚拟机的条目 。
1. 在 az801l07a-vm1 页上，选择“测试迁移” 。
1. 在“测试迁移”页上的“虚拟网络”下拉列表中，选择 az801l07a-test-vnet，然后选择“测试迁移”   。

   >**注意**：等待测试迁移完成。 这可能需要大约 5 分钟。

1. 在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟机”，然后在“虚拟机”页上，记下表示新复制的虚拟机 az801l07a-vm1-test 的条目   。

   > **注意：** 最初，虚拟机的名称由 asr- 前缀和随机生成的后缀组成，但最终将重命名为 az801l07a-vm1-test 。

1. 在 Azure 门户中，浏览回“Azure Migrate: 服务器迁移 | 复制计算机”页面，选择“刷新”，然后验证 az801l07a-vm1 虚拟机是否以“清理测试故障转移挂起”状态列出   。
1. 在“Azure Migrate: 服务器迁移 | 复制计算机”页面上，选择表示 az801l07a-vm1 虚拟机的条目 。
1. 在 az801l07a-vm1 复制计算机页上，选择“清理测试迁移” 。
1. 在“测试迁移清理”页上，选中“测试已完成，请删除测试虚拟机”复选框，然后选择“清理测试”  。
1. 测试故障转移清理作业完成后，刷新显示 az801l07a-vm1 复制计算机页的浏览器页面，你将发现，工具栏中的“迁移”图标会自动变为可用 。
1. 在 az801l07a-vm1 复制计算机页上，选择“迁移”链接 。 
1. 在“迁移”页上，确保在“是否在迁移前关闭计算机以最大程度地减少数据丢失?”下拉列表中选中“是”，再选中 VM 名称 az801l07a-vm1 旁边的复选框，然后选择“迁移”    。
1. 若要监视迁移状态，请浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页面。 在“Azure Migrate: 服务器迁移”部分，选择“复制服务器”条目，然后在“Azure Migrate: 服务器迁移 | 复制计算机”页面，检查复制计算机列表中的“状态”列   。 验证状态是否显示“计划的故障转移已完成”状态。

   >**注意**：迁移应是不可逆的操作。 如果需要查看已完成的信息，请浏览回“Azure Migrate | 服务器、数据库和 Web 应用”页面，刷新该页面，然后验证“Azure Migrate: 服务器迁移”部分的“已迁移的服务器数”条目的值是否为 1   。

#### <a name="task-4-remove-azure-resources-deployed-in-the-lab"></a>任务 4：删除实验室中部署的 Azure 资源

1. 在 SEA-SVR2 上，在显示 Azure 门户的浏览器窗口中，通过选择 Azure 门户中的 Cloud Shell 按钮打开“Azure Cloud Shell”窗格  。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。 

   > **注意：** 如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

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
