---
lab:
  title: 实验室：在 Windows Server 中升级和迁移
  type: Answer Key
  module: Module 6 - Upgrade and migrate in Windows Server
---

# 实验室答案：在 Windows Server 中升级和迁移

**注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Upgrading%20and%20migrating%20in%20Windows%20Server)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## 练习 1：在 Azure 中部署 AD DS 域控制器

#### 任务 1：使用 Azure 资源管理器 (ARM) 模板部署域控制器

1. 连接到 **SEA-SVR2**，然后根据需要使用讲师提供的凭据登录。
1. 在 SEA-SVR2 上，启动 Microsoft Edge，并在[创建新的 Windows VM 并创建新的 AD 林、域和 DC](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain) 处访问自定义版本的快速入门模板 。 
1. 在“创建新的 Windows VM 并创建新的 AD 林、域和 DC”页上，选择“部署到 Azure” 。 这会自动将浏览器重定向到 Azure 门户中的“使用新 AD 林创建 Azure VM”页。
1. 在“使用新 AD 林创建 Azure VM”页上，选择“编辑模板” 。
1. 在“编辑模板”页上，浏览到“storageProfile”部分（从第 195 行开始），并验证“sku”（第 199 行）设置为“2022-Datacenter”，“版本”（第 200 行）设置为“最新”且“dataDisks 缓存”（第 213 行）设置为“无”             。

   > 注意：托管 AD DS 数据库和日志文件的磁盘上的缓存应设置为“无” 。

1. 在“编辑模板”页上，浏览到 extension 部分（从第 233 行开始），并注意模板使用 PowerShell Desired State Configuration 在部署的 Azure 虚拟机 (VM) 中运行 CreateADPDC.ps1 脚本   。

   > 注意：若要查看脚本，可使用以下步骤：

   1. 在 SEA-SVR2 上，在 Microsoft Edge 窗口中打开另一个选项卡，然后在[创建新的 Windows VM 并创建新的 AD 林、域和 DC](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain) 处再次访问自定义版本的快速入门模板 。
   1. 在“创建新的 Windows VM 并创建新的 AD 林、域和 DC”页上的存储库内容列表中，选择“DSC”文件夹，然后选择“CreateADPDC.ps1”文件  。
   1. 在“azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain/DSC/CreateADPDC.ps1”页上，查看脚本的内容，并注意它安装了多个服务器角色，包括 Active Directory 域服务和 DNS，将 NTDS 数据库和日志以及 SYSOVL 共享放在驱动器 F 上 。 
   1. 关闭“Microsoft Edge”选项卡，然后切换回 Azure 门户中显示“编辑模板”页的选项卡。

1. 在“编辑模板”页上，浏览到预配可用性集的部分（从第 110 行开始），并注意该模板创建了一个可用性集，并将 VM 部署到其中（如第 181 行的 dependsOn 元素所示）   。

   > 注意：稍后在本练习中，在同一可用性集中部署另一个 Azure VM，并将其配置为同一域中的附加域控制器。 使用可用性集可提供额外的复原能力。

1. 浏览到预配 Azure VM 的网络接口的部分（从第 110 行开始），并注意专用 IP 地址分配方法设置为“静态”（第 164 行）  。

   > 注意：在部署域控制器时，使用静态分配很常见，但对于托管 DNS 服务器角色的服务器，则必须使用静态分配。

1. 浏览到部署嵌套模板的部分（从第 266 行开始），并注意该模板更新虚拟网络中的 DNS 服务器地址，该虚拟网络托管作为安装了 DNS 服务器角色的域控制器运行的Azure VM。

   > 注意：若配置指向运行具有 DNS 服务器角色的域控制器的 Azure VM 的自定义 DNS 服务器虚拟网络设置，可确保随后部署到同一虚拟网络中的任何 Azure VM 将自动使用该 DNS 服务器进行名称解析，从而有效地提供域加入功能。

1. 在“编辑模板”页上，选择“放弃” 。
1. 回到“使用新 AD 林创建 Azure VM”页上，选择“编辑参数” 。
1. 在“编辑参数”页上，选择“加载文件”，在“文件上传”对话框中，浏览到 C:\\Labfiles\\Lab06 文件夹，选择“L06-rg_template.parameters.json”文件，然后选择“打开”     。
1. 在“编辑模板”页上选择“保存” 。
1. 回到“使用新 AD 林创建 Azure VM”页，在“资源组”下拉列表下，选择“新建”，在“名称”文本框中，输入“AZ801-L0601-RG”，然后选择“确定”     。
1. 在“使用新 AD 林创建 Azure VM”页上，根据需要调整部署设置，使其具有以下值（将其他值保留为默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0601-RG 的名称 |
   | 区域 | 可以预配 Azure VM 的 Azure 区域的名称 |
   | 管理员用户名 | **学生** |
   | 管理员密码 | **Pa55w.rd1234** |
   | 域名 | **contoso.com** |
   | VM 大小 | **Standard_DS2_v2** |
   | _artifacts 位置 | **`https://raw.githubusercontent.com/az140mp/azure-quickstart-templates/master/application-workloads/active-directory/active-directory-new-domain/`** |
   | 虚拟机名称 | az801l06a-dc1 |
   | 虚拟网络名称 | az801l06a-vnet |
   | 虚拟网络地址范围 | 10.6.0.0/16 |
   | 网络接口名称 | az801l06a-dc1-nic1 |
   | 专用 IP 地址 | 10.6.0.4 |
   | 子网名称 | adSubnet |
   | 子网范围 | 10.6.0.0/24 |
   | 可用性集名称 | adAvailabilitySet |

1. 在“使用新 AD 林创建 Azure VM”页上，选择“查看 + 创建”，然后选择“创建”  。

   > 注意：在继续下一个任务之前，请等待部署完成。 这可能需要大约 15 分钟。 

#### 任务 2：部署 Azure Bastion 

> 注意：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Azure 门户中的 Cloud Shell 按钮打开“Azure Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。

   > 注意：如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格中的 PowerShell 会话中，运行以下命令以将名为 AzureBastionSubnet 的子网添加到你在本练习前面创建的虚拟网络 az801l06a-vnet 中 ：

   ```powershell
   $resourceGroupName = 'AZ801-L0601-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l06a-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.6.255.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 关闭 Cloud Shell 窗格。
1. 在 Azure 门户工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择 Bastions，然后在 Bastions 页面上，选择“+ 创建”   。
1. 在“创建 Bastion”页的“基本”标签上，指定以下设置，然后选择“查看 + 创建”  ：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0602-RG 的名称 |
   | 名称 | az801l06a-bastion |
   | 区域 | 在本练习的先前任务中部署资源的同一 Azure 区域 |
   | 层 | **基本** |
   | 虚拟网络 | az801l06a-vnet |
   | 子网 | AzureBastionSubnet (10.6.255.0/24) |
   | 公共 IP 地址 | **新建** |
   | 公共 IP 名称 | az801l06a-vnet-ip |

1. 在“创建 Bastion”页面的“查看 + 创建”标签上，选择“创建”  ：

   > 注意：请不要等待部署完成，而是继续执行下一个任务。 部署可能需要大约 5 分钟时间。

#### 任务 3：使用 Azure 门户部署 Azure VM

> 注意：在本练习的第一个任务中预配的第一个域控制器所在的同一域中，可采用另一域控制器的形式完全自动执行第二个 Azure VM 的部署及其设置。 但是，在这种情况下，图形界面的使用应提供有关在本地场景和基于 Azure 的场景中预配域控制器之间的差异的其他指导。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，于工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟机”  。 
1. 在“虚拟机”页上，选择“+ 创建”，然后在下拉菜单中选择“Azure 虚拟机”  。
1. 在“创建虚拟机”边栏选项卡的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值） ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 选择现有资源组 AZ801-L0601-RG |
   | 虚拟机名称 | az801l06a-dc2 |
   | 区域 | 选择之前在此练习中部署第一个虚拟机的同一 Azure 区域 |
   | 可用性选项 | **可用性集** |
   | 可用性集 | adAvailabilitySet |
   | 映像 | Windows Server 2022 Datacenter：Azure Edition - Gen2 |
   | 使用 Azure Spot 折扣运行 | **是** |
   | 大小 | Standard D2s v3 |
   | 用户名 | **学生** |
   | 密码 | **Pa55w.rd1234** |
   | 公共入站端口 | **无** |
   | 是否要使用现有的 Windows Server 许可证？ | **是** |

1. 选择“下一步: 磁盘 >”，然后在“创建虚拟机”边栏选项卡的“磁盘”选项卡上，指定以下设置（将其他设置保留为默认值）  ：

   | 设置 | Value |
   | --- | --- |
   | OS 磁盘类型 | **标准 SSD** |

1. 在“创建虚拟机”边栏选项卡的“磁盘”选项卡上，在“数据磁盘”部分中，选择“创建并附加新磁盘”   。
1. 在“创建新磁盘”页上，指定以下设置（将其他设置保留为默认值），然后选择“确定” ：

   | 设置 | 值 |
   | --- | --- |
   | 名称 | az801l06a-dc2_DataDisk_0 |
   | 源类型 | **无(空磁盘)** |
   | 大小 | 32 GiB 高级 SSD |

1. 回到“创建虚拟机”边栏选项卡的“磁盘”选项卡，选择“下一步: 网络 >”，然后在“创建虚拟机”边栏选项卡的“网络”选项卡上，指定以下设置（将其他设置保留为默认值）    ：

   | 设置 | 值 |
   | --- | --- |
   | 虚拟网络 | az801l06a-vnet |
   | 子网 | adSubnet (10.6.0.0/24) |
   | 公共 IP | **无** |
   | NIC 网络安全组 | **无** |
   | 启用加速网络 | disabled |
   | 是否将此虚拟机置于现有负载均衡解决方案之后？ | disabled |

1. 选择“下一步: 管理 >”，然后在“创建虚拟机”边栏选项卡的“管理”选项卡上，指定以下设置（将其他设置保留为默认值）  ：

   | 设置 | 值 |
   | --- | --- |
   | 补丁业务流程选项 | **手动更新** |

1. 选择“下一步: 监视 >”，然后在“创建虚拟机”边栏选项卡的“监视”选项卡上，指定以下设置（其他设置保留默认值）：

   | 设置 | 值 |
   | --- | --- |
   | 启动诊断 | 使用托管存储帐户启用（推荐） |

1. 选择“下一步: 高级 >”，在“创建虚拟机”页的“高级”选项卡上，查看可用的设置，而不修改任何设置，然后选择“查看 + 创建”   。
1. 在“查看 + 创建”边栏选项卡上，选择“创建” 。

   > 注意：请等待部署完成。 部署可能需要大约 3 分钟。

#### 任务 4：在 Azure VM 中手动升级域控制器

1. 在“SEA-SVR2”上，在显示 Azure 门户的 Microsoft Edge 窗口中，在部署页面上选择“转到资源” 。
1. 在“az801l06a-dc2”页左侧垂直菜单的“设置”部分中，选择“网络”  。
1. 在“az801l06a-dc2”\|“网络”页上，选择“az801l06a-dc2”虚拟机的网络接口的链接 。
1. 在网络接口页左侧垂直菜单的“设置”部分中，选择“IP 配置” 。
1. 在“IP 配置”页上，选择“ipconfig1”条目 。
1. 在“ipconfig1”页上的“专用 IP 地址设置”部分中，选择“静态”，然后选择“保存”   。

   > 注意：在部署域控制器时，使用静态分配很常见，但对于托管 DNS 服务器角色的服务器，则必须使用静态分配。

   > 注意：向 Azure VM 的网络接口分配静态 IP 地址将触发其操作系统的重启。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览回“az801l06a-dc2”页 。
1. 在“az801l06a-dc2”页上，选择“连接”，从下拉菜单中，选择“Bastion”  。 
1. 在 Bastion 上，提供以下凭据，然后选择“连接”：

   | 设置 | 值 | 
   | --- | --- |
   | 用户名 |**学生** |
   | 密码 |**Pa55w.rd1234** |

> 注意：默认情况下，Microsoft Edge 阻止弹出窗口。 若要允许 Bastion 弹出窗口，请转到 Microsoft Edge 中的“设置”，在左侧选择“Cookie 和网站权限”，在“所有权限”下选择“弹出窗口和重定向”，最后关闭“阻止(推荐)”      。

1. 在与 az801l06a-dc2 的远程桌面会话中，选择“启动”，然后选择“Windows PowerShell”  。
1. 若要安装 AD DS 和 DNS 服务器角色，请在 Windows PowerShell 命令提示符处输入以下命令，然后按 Enter：
    
   ```powershell
   Install-WindowsFeature -Name AD-Domain-Services,DNS -IncludeManagementTools
   ```

   > 注意：请等待安装完成。 这可能需要大约 3 分钟。

1. 若要配置数据磁盘，请在 Windows PowerShell 提示符处输入以下命令，然后在输入每个命令后，按 Enter：

   ```powershell
   Get-Disk | Where PartitionStyle -eq 'RAW' |  Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter F -FileSystem NTFS
   ```

1. 在与 az801l06a-dc2 的远程桌面会话中，切换到“服务器管理器”窗口 。
1. 在“服务器管理器”中，选择“通知”标志符号，然后在“部署后配置”部分中，选择“将此服务器提升为域控制器”链接   。 这将启动“Active Directory 域服务配置向导”。
1. 在“Active Directory 域服务配置向导”的“部署配置”页上，在“选择部署操作”下，确认已选中“向现有域添加域控制器”   。
1. 在“域”文本框中，输入“contoso.com”域 。
1. 在“提供执行此操作的凭据”部分中，选择“更改” 。
1. 在“部署操作的凭据”对话框的“用户名”框中，输入“Student@contoso.com”，在“密码”框中，输入“Pa55w.rd1234”，然后选择“确定”     。 
1. 回到“Active Directory 域服务配置向导”的“部署配置”页，选择“下一步”  。
1. 在“域控制器选项”页上，确保已选中“域名系统(DNS)服务器”和“全局编录(GC)”复选框  。 确保清除“只读域控制器(RODC)”复选框。
1. 在“键入目录服务还原模式(DSRM)密码”部分中，输入并确认密码“Pa55w.rd1234”，然后选择“下一步”  。
1. 在“Active Directory 域服务配置向导”的“DNS 选项”页上，选择“下一步”  。
1. 在“其他选项”页上，选择“下一步” 。
1. 在“路径”页上，将 Database 文件夹、Log files 文件夹和 SYSVOL 文件夹的路径设置的驱动器从“C:”更改为“F:”，然后选择“下一步”      。
1. 在“查看选项”页上，选择“下一步” 。
1. 在“先决条件检查”页上，注意有关网络适配器没有静态 IP 地址的警告，然后选择“安装” 。

   > 注意：由于静态 IP 地址是在平台级别而不是在操作系统中分配的，因此预期会出现警告。

   > 注意：操作系统将自动重启以完成提升过程。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口的“az801l06a-dc2”页上，选择“连接”，从下拉菜单中，选择“Bastion”   。  
1. 在 Bastion 上，提供以下凭据，然后选择“连接”：

   | 设置 | 值 | 
   | --- | --- |
   | 用户名 |**学生** |
   | 密码 |**Pa55w.rd1234** |

1. 在与 az801l06a-dc2 的远程桌面会话中，请等待“服务器管理器”窗口打开，并确认本地安装的角色列表包括“AD DS”和“DNS”   。

#### 任务 5：删除练习中部署的 Azure 资源

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Azure 门户中的 Cloud Shell 按钮，在 Azure Cloud Shell 窗格中打开一个 PowerShell 会话。
1. 在 Cloud Shell 窗格中，运行以下命令，列出你在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*'
   ```

   > 备注：验证输出是否仅包含你在此实验室中创建的资源组。 此任务中将删除该组。

1. 在 Cloud Shell 窗格中，运行以下命令，删除你在此实验室中创建的资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > 注意：该命令以异步方式执行（由 -AsJob 参数决定），因此，虽然你可以立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。

## 练习 2：使用存储迁移服务迁移文件服务器

#### 任务 1：安装 Windows Admin Center

1. 在 SEA-SVR2 上，选择“开始”，然后选择“Windows PowerShell”  。
1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 备注：请等待安装完成。 这大约需要 2 分钟。

#### 任务 2：设置文件服务

1. 在 SEA-SVR2 的任务栏上，选择“文件资源管理器” 。
1. 在文件资源管理器中，浏览到 C:\\Labfiles\\Lab06 文件夹。
1. 在文件资源管理器的“详细信息”窗格中，选择“L06_SetupFS.ps1”文件，显示其上下文菜单，然后在菜单中选择“编辑” 。

   >注意：这会自动在 Windows PowerShell ISE 的脚本窗格中打开 L06_SetupFS.ps1 文件 。

1. 在 Windows PowerShell ISE 脚本窗格中，查看脚本，然后通过选择工具栏中的“运行脚本”图标或按 F5 来执行该脚本。 

   >注意：等待脚本完成。 这大约需要 1 分钟。

   >注意：该脚本会在 SEA-SVR1 和 SEA-SVR2 上初始化一个额外的数据磁盘，在每个磁盘上创建一个 NTFS 卷，向每个卷分配“S:”驱动器号，使用 SEA-SVR1 上的 S:\Data 文件夹创建一个名为 Data 的共享，并向其中添加总大小约为 1 GB 的示例文件      。 

#### 任务 3：使用存储迁移服务执行迁移

1. 在 SEA-SVR2 上，启动 Microsoft Edge，然后转到 https://SEA-SVR2.contoso.com 。 
   
   >注意：如果链接不起作用，请在 SEA-SVR2 上打开文件资源管理器，选择“下载”文件夹，在“下载”文件夹中选择 WindowsAdminCenter.msi 文件并手动安装。 安装完成后，刷新 Microsoft Edge。

   >注意：如果收到“NET::ERR_CERT_DATE_INVALID”错误，请在 Microsoft Edge 浏览器页上选择“高级”，在页面底部选择“继续访问 sea-svr2-contoso.com (不安全)”   。

1. 出现提示时，请在“**Windows 安全**”对话框中输入讲师提供的凭据，然后选择“**确定**”。

1. 查看“此版本中的新增功能”弹出窗口，然后在其右上角选择“关闭” 。
1. 在 Windows Admin Center 的“所有连接”窗格的右上角，选择“设置”图标（齿轮） 。
1. 在左窗格中，选择“扩展”****。 查看可用的扩展。
1. 在“详细信息”窗格中，选择“已安装扩展”并确认列表包含“存储迁移服务”扩展 。

   >注意：如果有可用的更新，请选择“存储迁移服务”扩展条目，然后选择“更新”  。

1. 在顶部菜单的“设置”旁边，选择下拉箭头，然后选择“服务器管理器” 。
1. 在“所有连接”窗格中，选择“sea-svr2.contoso.com”链接 。
1.  在“sea-svr2.contoso.com”页上的“工具”菜单中，选择“存储迁移服务”条目  。
1. 在“存储迁移服务”窗格中，选择“安装” 。

   >注意：这将自动安装存储迁移服务及其所需的组件。

1. 在“三步迁移存储”窗格中，选择“关闭” 。
1. 在“存储迁移服务”窗格中，向下滚动到页面底部，然后选择“+ 新建作业” 。
1. 在“新建作业”窗格的“作业名称”文本框中，输入“SVR1toSVR2”，确保选中“Windows 服务器和群集源设备”选项，然后选择“确定”     。
1. 在“存储迁移服务 > SVR1toSVR2”窗格的“清单服务器”选项卡上，查看“检查先决条件”窗格并选择“下一步”   。
1. 在“清单服务器”选项卡上的“输入凭据”窗格中，如有必要，输入 CONTOSO\\Administrator 用户帐户的凭据，清除“从故障转移群集迁移”复选框，然后选择“下一步”    。
1. 在“清单服务器”选项卡上的“安装所需功能”窗格中，选择“下一步”  。
1. 在“清单服务器”选项卡上的“添加和扫描设备”窗格中，选择“+ 添加设备”  。
1. 在“添加源设备”上，确保选中“设备名称”选项，在“名称”文本框中，输入“SEA-SVR1.contoso.com”，然后选择“添加”    。
1. 在“**指定凭据**”窗格中，选择“**对此连接使用另一个帐户**”选项，输入讲师提供的凭据，选择“**对所有连接使用这些凭据**”，然后选择“**继续**”。

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派<!--Marcin can this be 'a Kerberos constrained delegation'?-->。

1. 在设备列表上，选择新添加的 SEA-SVR1.contoso.com 条目，在“添加和扫描设备”窗格的工具栏中，选择 省略号 (...) 符号，然后在下拉菜单中选择“开始扫描”   。

   >注意：请等待扫描成功完成。 这大约需要 1 分钟。

1. 在“清单服务器”选项卡上的“添加和扫描设备”窗格中，选择“下一步”  。 

   >注意：这会过渡到可通过“存储迁移服务 > SVR1toSVR2”窗格中的“传输数据”选项卡访问的迁移作业的第二阶段  。

1. 在“传输数据”选项卡上的“输入目标设备的凭据”窗格中，确认正在使用 CONTOSO\\Administrator 用户帐户，并选择“下一步”   。
1. 在“指定此项的目标: sea-svr1.contoso.com”窗格中，确保“目标”选项设置为“使用现有服务器或 VM”，在“目标设备”文本框中，输入“SEA-SVR2.contoso.com”，然后选择“扫描”     。

   >注意：请等待扫描成功完成。 这大约需要 1 分钟。

   >注意：在混合方案中，还可以选择自动创建充当迁移作业目标的 Azure VM。

1. 扫描完成后，在“指定此项的目标: sea-svr1.contoso.com”窗格中，查看“将每个源卷映射到目标卷”部分，并确保“S:”源卷映射到“S:”目标卷   。
1. 在“指定此项的目标: sea-svr1.contoso.com”窗格中，查看“选择要传输的共享”部分，确保传输中包含 Data 源共享，然后选择“下一步”   。
1. 在“传输数据”选项卡上的“调整传输设置”窗格中，指定以下设置（将其他设置保留为默认值），然后选择“下一步”  ：

   | 设置 | 值 | 
   | --- | --- |
   | 备份将被覆盖的文件夹（不备份启用了 Azure 文件同步的共享） | enabled |
   | 验证方法 | CRC 64 |
   | 最长持续时间（分钟） | **60** |
   | 迁移用户和组 | “重复使用同名帐户” |
   | 最多重试次数 | **3** |
   | 重试之间的延迟（秒） | **60** |

1. 在“传输数据”选项卡上的“安装所需的功能”窗格中，等待 SEA-SVR2.contoso.com 上的 SMS-Proxy 安装完成，然后选择“下一步”    。
1. 在“传输数据”选项卡上的“验证源和目标设备”窗格中，选择“验证”，验证成功完成后，选择“下一步”   。
1. 在“传输数据”选项卡上的“启动传输”窗格中，选择“启动传输”，等待它完成，然后选择“下一步”   。

   >注意：请等待传输成功完成。 此过程应该会在 1 分钟内完成。

   >注意：这会过渡到可通过“存储迁移服务 > SVR1toSVR2”窗格中的“切换到新服务器”选项卡访问的迁移作业的第三阶段  。

1. 在“切换到新服务器”选项卡上的“输入源设备的凭据”和“输入目标设备的凭据”部分中，接受 CONTOSO\\Administrator 用户帐户的存储凭据，然后选择“下一步”    。
1. 在“切换到新服务器”选项卡上，在“配置从 sea-svr1.contoso.com 到 sea-svr2.contoso.com 的切换”窗格的“源网络适配器”部分中，指定以下设置  ：

   | 设置 | 值 | 
   | --- | --- |
   | 使用 DHCP | disabled |
   | IP 地址 | 172.16.10.111 |
   | 子网 | **255.255.0.0** |
   | 网关 | 172.16.10.1 |

1. 在“切换到新服务器”选项卡上，在“配置从 sea-svr1.contoso.com 到 sea-svr2.contoso.com 的切换”窗格的“目标网络适配器”下拉列表中，选择“以太网”   。
1. 在“切换到新服务器”选项卡上，在“配置从 sea-svr1.contoso.com 到 sea-svr2.contoso.com 的切换”窗格的“切换后重命名源设备”部分中，选择“选择新名称”选项，在“新的源计算机名称”中    ，<!--text box?-->输入“SEA-SVR1-OLD”，然后选择“下一步” 。
1. 在“切换到新服务器”选项卡上，在“调整切换设置”窗格的“切换超时(分钟)”文本框中，输入“30”，在“输入 AD 凭据”部分，将“存储凭据”选项保留为启用，然后选择“下一步”      。
1. 在“切换到新服务器”选项卡上的“验证源和目标设备”窗格中，选择“验证”，验证成功完成后，选择“下一步”   。
1. 在“切换到新服务器”选项卡上的“切换到新服务器”窗格中，选择“开始切换”  。

   >注意：切换将触发 SEA-SVR1 和 SEA-SVR2 的两次连续重启  。

#### 任务 4：验证迁移结果

1. 在 **SEA-SVR2** 上，使用讲师提供的凭据登录。
1. 在 SEA-SVR2 上，选择“开始”，然后选择“Windows PowerShell”  。
1. 若要确定分配给 SEA-SVR2 的网络接口的 IPv4 地址，请在 Windows PowerShell 控制台中输入以下命令，然后按 Enter ：
    
   ```powershell
   Get-NetIPAddress | Where-Object AddressFamily -eq 'IPv4' | Select-Object IPAddress
   ```

   >注意：确认输出同时包含 172.16.10.11 和 172.16.10.12  。

1. 若要确定分配给 SEA-SVR2 的 NetBIOS 名称，请在 Windows PowerShell 控制台中输入以下命令，然后按 Enter ：
    
   ```powershell
   nbtstat -n
   ```

   >注意：确认输出同时包含 SEA-SVR1 和 SEA-SVR2  。

1. 若要确定 SEA-SVR2 上的本地共享，请在 Windows PowerShell 控制台中输入以下命令，然后按 Enter ：
    
   ```powershell
   Get-SMBShare
   ```

   >注意：确认输出包含 S:\Data 文件夹中托管的 Data 共享  。

1. 若要确定 SEA-SVR2 上 Data 共享的内容，请在 Windows PowerShell 控制台中输入以下命令，然后按 Enter  ：
    
   ```powershell
   Get-ChildItem -Path 'S:\Data'
   ```

