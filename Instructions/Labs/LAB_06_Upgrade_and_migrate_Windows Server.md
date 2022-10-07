---
lab:
  title: 实验室：在 Windows Server 中升级和迁移
  module: 'Module 6: Upgrade and migrate in Windows Server'
---

# <a name="lab-upgrade-and-migrate-in-windows-server"></a>实验室：在 Windows Server 中升级和迁移

## <a name="lab-scenario"></a>实验室方案

Contoso 正在探索适用于其基础结构服务的混合模型，该模型有助于将其本地 Windows 服务器迁移到 Azure 虚拟机 (VM)。 为协助完成此计划，你需要评估在 Azure VM 中部署 Active Directory 域服务 (AD DS) 域控制器的过程。 你的目的是确定当前用于本地部署的手动过程与 Azure 中所提供的部署方法之间的差异。 此外，你还需要测试并记录存储迁移服务功能，以验证其在本地文件服务器迁移中的使用情况。 

## <a name="objectives"></a>目标

在此实验中，将执行以下操作：

- 在 Azure 中部署 AD DS 域控制器。
- 使用存储迁移服务迁移文件服务器。

## <a name="estimated-time-60-minutes"></a>估计时间：60 分钟

## <a name="lab-environment"></a>实验室环境
  
虚拟机：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 必须运行  。 其他 VM 可以运行，但本实验室不需要这些 VM。

> 备注：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 虚拟机分别托管 SEA-DC1、SEA-SVR1 和 SEA-SVR2 的安装      。

1. 选择 SEA-SVR2。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境和 Azure 订阅。 在开始实验室之前，请确保拥有 Azure 订阅以及具有该订阅中“所有者”角色的用户帐户。

## <a name="exercise-1-deploying-ad-ds-domain-controllers-in-azure"></a>练习 1：在 Azure 中部署 AD DS 域控制器

> 注意：混合方案通常涉及通过将其他域控制器从现有本地域部署到 Azure VM 来将本地 AD DS 环境扩展到 Azure。 在实验室中执行此类任务需要设置与 Azure 虚拟网络的站点到站点 VPN 连接，或者在 Azure 中预配整个实验室环境，其中一部分模拟本地站点。 为简单起见，此练习涉及到将 Azure VM 中的域控制器部署到新的林和域中。 重点在于如何发现使用 Azure VM 时域控制器配置和预配过程的独特方面。

本练习的主要任务是：

1. 使用 Azure 资源管理器 (ARM) 模板部署域控制器。
1. 部署 Azure Bastion。 
1. 使用 Azure 门户部署 Azure VM。
1. 在 Azure VM 中手动升级域控制器。
1. 删除练习中部署的 Azure 资源。

#### <a name="task-1-deploy-a-domain-controller-by-using-an-azure-resource-manager-arm-template"></a>任务 1：使用 Azure 资源管理器 (ARM) 模板部署域控制器

1. 在 SEA-SVR2 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有所有者角色的用户帐户的凭据登录要在此实验室中使用的订阅 。
1. 在 SEA-SVR2 上，启动 Microsoft Edge，并在[创建新的 Windows VM 并创建新的 AD 林、域和 DC](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain) 处转到自定义版本的快速入门模板 。 
1. 在“创建新的 Windows VM 并创建新的 AD 林、域和 DC”页上，启动到 Azure 的部署。 
1. 在“使用新 AD 林创建 Azure VM”页上，选择“编辑模板” 。
1. 在“编辑模板”页上，浏览到“storageProfile”部分（从第 195 行开始），并验证“sku”（第 199 行上）设置为“2022-Datacenter”且“dataDisks 缓存”（第 213 行上）设置为“无”          。

   > 注意：托管 AD DS 数据库和日志文件的磁盘上的缓存应设置为“无” 。

1. 在“编辑模板”页上，浏览到 extension 部分（从第 233 行开始），并注意模板使用 PowerShell Desired State Configuration 在部署的 Azure 虚拟机 (VM) 中运行 CreateADPDC.ps1 脚本   。

   > 注意：若要查看脚本，可使用以下步骤：

   1. 在 SEA-SVR2 上，在 Microsoft Edge 窗口中打开另一个选项卡，然后在[创建新的 Windows VM 并创建新的 AD 林、域和 DC](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain) 处转到自定义版本的快速入门模板 。
   1. 在“创建新的 Windows VM 并创建新的 AD 林、域和 DC”页上的存储库内容列表中，选择“DSC”文件夹，然后选择“CreateADPDC.ps1”文件  。
   1. 在“azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain/DSC/CreateADPDC.ps1”页上，查看脚本的内容，并注意它安装了多个服务器角色，包括 Active Directory 域服务和 DNS，将 NTDS 数据库和日志以及 SYSOVL 共享放在驱动器 F 上 。 
   1. 关闭“Microsoft Edge”选项卡，然后切换回 Azure 门户中显示“编辑模板”页的选项卡。

1. 在“编辑模板”页上，浏览到预配可用性集的部分（从第 110 行开始），并注意该模板创建了一个可用性集，并将 VM 部署到其中（如第 181 行的 dependsOn 元素所示）   。

   > 注意：稍后在本练习中，在同一可用性集中部署另一个 Azure VM，并将其配置为同一域中的附加域控制器。 使用可用性集可提供额外的复原能力。

1. 浏览到预配 Azure VM 的网络接口的部分（从第 110 行开始），并注意专用 IP 地址分配方法设置为“静态”（第 164 行）  。

   > 注意：在部署域控制器时，使用静态分配很常见，但对于托管 DNS 服务器角色的服务器，则必须使用静态分配。

1. 浏览到部署嵌套模板的部分（从第 266 行开始），并注意该模板更新虚拟网络中的 DNS 服务器地址，该虚拟网络托管作为安装了 DNS 服务器角色的域控制器运行的Azure VM。

   > 注意：若配置指向运行具有 DNS 服务器角色的域控制器的 Azure VM 的自定义 DNS 服务器虚拟网络设置，可确保随后部署到同一虚拟网络中的任何 Azure VM 将自动使用该 DNS 服务器进行名称解析，从而有效地提供域加入功能。

1. 关闭“编辑模板”页，不对模板应用任何更改。
1. 回到“使用新 AD 林创建 Azure VM”页上，选择“编辑参数” 。
1. 在“编辑参数”页上，通过上传 C:\\Labfiles\\Lab06\\L06-rg_template.parameters.json 文件替换默认参数 。
1. 使用以下设置启动部署（将其他设置保留为默认值）：

   | 设置 | 值 | 
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0601-RG 的名称 |
   | 区域 | 可以预配 Azure VM 的 Azure 区域的名称 |
   | 管理员用户名 | **学生** |
   | 管理员密码 | **Pa55w.rd1234** |
   | 域名 | **contoso.com** |
   | VM 大小 | **Standard_DS2_v2** |
   | 虚拟机名称 | az801l06a-dc1 |
   | 虚拟网络名称 | az801l06a-vnet |
   | 虚拟网络地址范围 | 10.6.0.0/16 |
   | 网络接口名称 | az801l06a-dc1-nic1 |
   | 专用 IP 地址 | 10.6.0.4 |
   | 子网名称 | adSubnet |
   | 子网范围 | 10.6.0.0/24 |
   | 可用性集名称 | adAvailabilitySet |

   > 注意：在继续下一个任务之前，请等待部署完成。 这可能需要大约 15 分钟。 

#### <a name="task-2-deploy-azure-bastion"></a>任务 2：部署 Azure Bastion 

> 注意：Azure Bastion 允许在没有公共终结点（你在本练习的上一个任务中部署的终结点）的情况下连接到 Azure VM，同时提供针对暴力攻击（利用操作系统级别凭据）的保护。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在 Cloud Shell 中打开一个 PowerShell 会话。
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

1. 在 Azure 门户中，使用以下设置部署 Azure Bastion：

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

   > 注意：请不要等待部署完成，而是继续执行下一个任务。 部署可能需要大约 5 分钟时间。

#### <a name="task-3-deploy-an-azure-vm-by-using-the-azure-portal"></a>任务 3：使用 Azure 门户部署 Azure VM

> 注意：在本练习的第一个任务中预配的第一个域控制器所在的同一域中，可采用另一域控制器的形式完全自动执行第二个 Azure VM 的部署及其设置。 但是，在这种情况下，图形界面的使用应提供有关在本地场景和基于 Azure 的场景中预配域控制器之间的差异的其他指导。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，使用以下设置创建虚拟机（将其他设置保留为其默认值）：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0601-RG 的名称 |
   | 虚拟机名称 | az801l06a-dc2 |
   | 区域 | 选择之前在此练习中部署第一个虚拟机的同一 Azure 区域 |
   | 可用性选项 | **可用性集** |
   | 可用性集 | adAvailabilitySet |
   | 映像 | Windows Server 2022 Datacenter：Azure Edition - Gen2 |
   | Azure Spot 实例 | **是** |
   | 大小 | Standard D2s v3 |
   | 用户名 | **学生** |
   | 密码 | **Pa55w.rd1234** |
   | 公共入站端口 | **无** |
   | 是否要使用现有的 Windows Server 许可证? | **是** |
   | OS 磁盘类型 | **标准 SSD** |
   | 数据磁盘名称 | az801l06a-dc2_DataDisk_0 |
   | 数据磁盘源类型 | **无(空磁盘)** |
   | 数据磁盘大小 | 32 GiB 高级 SSD  |
   | 虚拟网络 | az801l06a-vnet |
   | 子网 | adSubnet (10.6.0.0/24) |
   | 公共 IP | **无** |
   | NIC 网络安全组 | **无** |
   | 加速网络 | enabled |
   | 是否将此虚拟机置于现有负载均衡解决方案之后？ | disabled |
   | 启动诊断 | 使用托管存储帐户启用（推荐） |
   | 补丁业务流程选项 | **手动更新** |  

   > 注意：请等待部署完成。 部署可能需要大约 3 分钟。

#### <a name="task-4-manually-promote-a-domain-controller-in-an-azure-vm"></a>任务 4：在 Azure VM 中手动升级域控制器

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，配置分配给 az801l06a-dc2 虚拟机网络接口的专用 IP 地址，以便使用静态分配 。 

   > 注意：在部署域控制器时，使用静态分配很常见，但对于托管 DNS 服务器角色的服务器，则必须使用静态分配。

   > 注意：向 Azure VM 的网络接口分配静态 IP 地址将触发其操作系统的重启。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“az801l06a-dc2”页 。
1. 在 az801l06a-dc2 页上，通过 Bastion 服务建立与 az801l06a-dc2 的 RDP 会话，并使用以下凭据进行身份验证 ：

   | 设置 | 值 | 
   | --- | --- |
   | 用户名 |**学生** |
   | 密码 |**Pa55w.rd1234** |

1. 在与 az801l06a-dc2 的远程桌面会话中，启动 Windows PowerShell 会话。
1. 若要安装 AD DS 和 DNS 服务器角色，请在 Windows PowerShell 提示符下运行以下命令：
    
   ```powershell
   Install-WindowsFeature -Name AD-Domain-Services,DNS -IncludeManagementTools
   ```

   > 注意：请等待安装完成。 这可能需要大约 3 分钟。

1. 若要配置数据磁盘，请在 Windows PowerShell 提示符下运行以下命令：

   ```powershell
   Get-Disk | Where PartitionStyle -eq 'RAW' |  Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter F -FileSystem NTFS
   ```

1. 在与 az801l06a-dc2 的远程桌面会话中，切换到“服务器管理器”窗口 。
1. 在服务器管理器中，启动 Active Directory 域服务配置向导来执行域控制器提升 。
1. 在 Active Directory 域服务配置向导中，选择“将域控制器添加到现有域”选项并指定“contoso.com”作为目标域  。
1. 使用用户名 Student@contoso.com 和密码 Pa55w.rd1234 作为执行提升的凭据 。
1. 指定将新域控制器指定为可写的选项，并包含域名系统 (DNS) 服务器和全局编录 (GC) 组件 。
1. 将目录服务还原模式 (DSRM) 的密码设置为 Pa55w.rd1234 。
1. 将托管 AD DS 数据库、日志文件和 SYSVOL 的文件夹从驱动器 C 更改为驱动器 F 。
1. 在“先决条件检查”页上，注意有关网络适配器没有静态 IP 地址的警告，并启动提升。 

   > 注意：由于静态 IP 地址是在平台级别而不是在操作系统中分配的，因此预期会出现警告。

   > 注意：操作系统将自动重启以完成提升过程。

1. 在 SEA-SVR2 上，通过 Bastion 服务重新连接到 az801l06a-dc2 。
1. 在与 az801l06a-dc2 的远程桌面会话中，使用服务器管理器验证本地安装的角色是否包括 AD DS 和 DNS   。

#### <a name="task-5-remove-azure-resources-deployed-in-the-exercise"></a>任务 5：删除练习中部署的 Azure 资源

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动 PowerShell 会话

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格。
1. 在 Cloud Shell 窗格中，运行以下命令，列出此练习中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*'
   ```

   > 注意：验证输出中是否仅包含你在此练习中创建的资源组。 此任务中将删除这些组。

1. 运行以下命令，删除在此练习中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > 注意：该命令以异步方式执行（由 -AsJob 参数决定），因此，虽然你可以立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。

## <a name="exercise-2-migrating-file-servers-by-using-storage-migration-service"></a>练习 2：使用存储迁移服务迁移文件服务器

本练习的主要任务是：

1. 安装 Windows Admin Center。
1. 设置文件服务。
1. 使用存储迁移服务执行迁移。
1. 验证迁移结果。

#### <a name="task-1-install-windows-admin-center"></a>任务 1：安装 Windows Admin Center

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。

   >注意：如果尚未在 SEA-ADM1 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，运行以下命令，下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 运行以下命令以安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 注意：请等待安装完成。 这大约需要 2 分钟。

#### <a name="task-2-set-up-file-services"></a>任务 2：设置文件服务

1. 在 SEA-SVR2 上，在 Windows PowerShell ISE 中打开文件 C:\\Labfiles\\Lab06\\L06_SetupFS.ps1 。
1. 在“Windows PowerShell ISE 脚本”窗格中，查看该脚本，然后执行该脚本。 

   >注意：请等待脚本完成。 这大约需要 1 分钟。

   >注意：该脚本会在 SEA-SVR1 和 SEA-SVR2 上初始化一个额外的数据磁盘，在每个磁盘上创建一个 NTFS 卷，向每个卷分配“S:”驱动器号，使用 SEA-SVR1 上的 S:\Data 文件夹创建一个名为 Data 的共享，并向其中添加总大小约为 1 GB 的示例文件      。 

#### <a name="task-3-perform-migration-by-using-storage-migration-service"></a>任务 3：使用存储迁移服务执行迁移

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接到 Windows Admin Center 的本地实例 (https://SEA-ADM1.contoso.com ) 。 
1. 如果系统提示，请使用以下凭据进行身份验证：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在 SEA-ADM1 上，在 Windows Admin Center 中，查看已安装的扩展并验证列表中是否包含“存储迁移服务”扩展 。

   >注意：如果有可用的更新，请选择“存储迁移服务”扩展条目，然后选择“更新”  。

1. 从“所有连接”窗格连接到“sea-svr2.contoso.com” 。
1. 从“sea-svr2.contoso.com”页上的“工具”菜单中，启动存储迁移服务并调用“安装”操作   。

   >注意：这将自动安装存储迁移服务及其所需的组件。

1. 关闭“三步迁移存储”窗格中。
1. 在“存储迁移服务”窗格中，创建名为 SVR1toSVR2 的迁移作业，并将源设备设置为“Windows 服务器和群集”   。
1. 在“存储迁移服务 > SVR1toSVR2”窗格的“清单服务器”选项卡上，查看“检查先决条件”窗格  。
1. 在“清单服务器”选项卡上的“输入凭据”窗格中，如有必要，输入 CONTOSO\\Administrator 用户帐户的凭据，清除“从故障转移群集迁移”复选框   。
1. 在“库存服务器”选项卡的“添加和扫描设备”窗格中，使用以下凭据添加 SEA-SVR1.contoso.com 服务器  ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

   > 注意：若要执行单一登录，你需要设置 Kerberos 约束委派。

1. 从设备列表中，选择新添加的“SEA-SVR1.contoso.com”条目并启动其扫描。

   >注意：请等待扫描成功完成。 这大约需要 1 分钟。

   >注意：扫描完成后，请进入可通过“存储迁移服务 > SVR1toSVR2”窗格中的“传输数据”选项卡访问的迁移作业的第二阶段  。

1. 验证 CONTOSO\\Administrator 用户帐户是否用于数据传输。
1. 将目标设备设置为 SEA-SVR2.contoso.com。

   >注意：请等待扫描成功完成。 这大约需要 1 分钟。

   >注意：在混合方案中，还可以选择自动创建充当迁移作业目标的 Azure VM。

1. 扫描完成后，在“指定此项的目标: sea-svr1.contoso.com”窗格中，查看“将每个源卷映射到目标卷”部分，并确保“S:”源卷映射到“S:”目标卷   。
1. 在“指定此项的目标: sea-svr1.contoso.com”窗格中，查看“选择要传输的共享”部分，并确保传输中包含 Data 源共享  。
1. 在“传输数据”选项卡上的“调整传输设置”窗格中，指定以下设置（将其他设置保留为默认值） ：

   | 设置 | 值 | 
   | --- | --- |
   | 备份将被覆盖的文件夹（不备份启用了 Azure 文件同步的共享） | enabled |
   | 验证方法 | CRC 64 |
   | 最长持续时间（分钟） | **60** |
   | 迁移用户和组 | “重复使用同名帐户” |
   | 最多重试次数 | **3** |
   | 重试之间的延迟（秒） | **60** |

   >注意：在“传输数据”选项卡上的“安装所需的功能”窗格中，等待 SEA-SVR2.contoso.com 上的 SMS-Proxy 安装完成    。

1. 扫描完成后，在“传输数据”选项卡上的“验证源和目标设备”窗格中，启动验证并等待验证成功完成 。
1. 在“传输数据”选项卡上的“启动传输”窗格中，启动数据传输 。

   >注意：请等待传输成功完成。 此过程应该会在 1 分钟内完成。

   >注意：这会过渡到可通过“存储迁移服务 > SVR1toSVR2”窗格中的“切换到新服务器”选项卡访问的迁移作业的第三阶段  。

1. 在“切换到新服务器”选项卡上的“输入源设备的凭据”和“输入目标设备的凭据”部分中，接受 CONTOSO\\Administrator 用户帐户的存储凭据   。
1. 在“切换到新服务器”选项卡上，在“配置从 sea-svr1.contoso.com 到 sea-svr2.contoso.com 的切换”窗格的“源网络适配器”部分中，指定以下设置  ：

   | 设置 | 值 | 
   | --- | --- |
   | 使用 DHCP | disabled |
   | IP 地址 | 172.16.10.111 |
   | 子网 | **255.255.0.0** |
   | 网关 | 172.16.10.1 |

1. 在“切换到新服务器”选项卡上，在“配置从 sea-svr1.contoso.com 到 sea-svr2.contoso.com 的切换”窗格的“目标网络适配器”下拉列表中，选择“西雅图”   ：
1. 在“切换到新服务器”选项卡上，在“配置从 sea-svr1.contoso.com 到 sea-svr2.contoso.com 的切换”窗格的“切换后重命名源设备”部分中，选择“选择新名称”选项，然后在“新的源计算机名称”对话框中输入“SEA-SVR1-OLD”     。
1. 在“切换到新服务器”选项卡上，在“调整切换设置”窗格的“切换超时(分钟)”文本框中，输入“30”，然后在“输入 AD 凭据”部分，将“存储凭据”选项保留为启用     。
1. 在“切换到新服务器”选项卡的“验证源和目标设备”窗格中，启动验证 。
1. 验证成功完成后，在“切换到新服务器”选项卡上，启动切换阶段。

   >注意：切换将触发 SEA-SVR1 和 SEA-SVR2 的两次连续重启  。

#### <a name="task-4-validate-migration-outcome"></a>任务 4：验证迁移结果

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell 。
1. 若要确定分配给 SEA-SVR2 的网络接口的 IPv4 地址，请在 Windows PowerShell 控制台中运行以下命令 ：
    
   ```powershell
   Get-NetIPAddress | Where-Object AddressFamily -eq 'IPv4' | Select-Object IPAddress
   ```

   >注意：确认输出同时包含 172.16.10.11 和 172.16.10.12  。

1. 若要确定分配给 SEA-SVR2 的 NetBIOS 名称，请在 Windows PowerShell 控制台中运行以下命令 ：
    
   ```powershell
   nbtstat -n
   ```

   >注意：确认输出同时包含 SEA-SVR1 和 SEA-SVR2  。

1. 若要确定 SEA-SVR2 上的本地共享，请在 Windows PowerShell 控制台中运行以下命令 ：
    
   ```powershell
   Get-SMBShare
   ```

   >注意：确认输出包含 S:\Data 文件夹中托管的 Data 共享  。

1. 若要确定 SEA-SVR2 上 Data 共享的内容，请在 Windows PowerShell 控制台中运行以下命令  ：
    
   ```powershell
   Get-ChildItem -Path 'S:\Data'
   ```

#### <a name="review"></a>审阅

在此实验室中，你执行了以下操作：

- 在 Azure 中部署 AD DS 域控制器。
- 使用存储迁移服务迁移文件服务器。
