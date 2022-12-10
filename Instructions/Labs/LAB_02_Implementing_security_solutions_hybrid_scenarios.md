---
lab:
  title: 实验室：在混合场景中实现安全解决方案
  module: 'Module 2: Implementing Security Solutions in Hybrid Scenarios'
---

# <a name="lab-implementing-security-solutions-in-hybrid-scenarios"></a>实验室：在混合场景中实现安全解决方案

## <a name="scenario"></a>场景

为了确定 Microsoft Azure 安全性相关的集成功能，以便进一步增强你的本地和云安全环境，你已决定将你概念证明环境中的 Windows 服务器载入 Microsoft Defender for Cloud。 还需要将运行 Windows Server 的本地服务器和 Azure VM 与基于 Azure 自动化的解决方案相集成，包括清单、更改跟踪以及更新管理。

                **注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Implementing%20security%20solutions%20in%20hybrid%20scenarios)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 创建 Azure Log Analytics 工作区和 Azure 自动化帐户。
- 配置 Defender for Cloud。
- 预配运行 Windows Server 的 Azure VM。
- 将本地 Windows Server 载入 Defender for Cloud 和 Azure 自动化。
- 验证 Defender for Cloud 和 Azure 自动化解决方案的混合功能。

## <a name="estimated-time-75-minutes"></a>估计时间：75 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 必须运行  。 其他 VM 可以运行，但本实验室不需要这些 VM。

> 备注：AZ-801T00A-SEA-DC1、AZ-801T00A-SEA-SVR1 和 AZ-801T00A-SEA-SVR2 虚拟机分别托管 SEA-DC1、SEA-SVR1 和 SEA-SVR2 的安装      。

1. 选择 SEA-SVR2。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境和 Azure 订阅。 在开始实验室之前，请确保拥有 Azure 订阅以及具有该订阅中“所有者”或“参与者”角色的用户帐户。

## <a name="exercise-1-creating-an-azure-log-analytics-workspace-and-an-azure-automation-account"></a>练习 1：创建 Azure Log Analytics 工作区和 Azure 自动化帐户

### <a name="scenario"></a>场景

若要为评估做好准备，首先需要创建 Azure Log Analytics 工作区和 Azure 自动化帐户，这将提供安全相关解决方案的核心功能，包括清单、更改跟踪以及更新管理。

此练习的主要任务如下：

1. 创建一个 Azure Log Analytics 工作区 
1. 创建并配置一个 Azure 自动化帐户

#### <a name="task-1-create-an-azure-log-analytics-workspace"></a>任务 1：创建一个 Azure Log Analytics 工作区 

1. 在 SEA-SVR2 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录 。
1. 在 Azure 门户中，使用以下设置创建 Log Analytics 工作区：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0201-RG 的名称 |
   | Log Analytics 工作区 | 任何唯一名称 |
   | 区域 | 在上一个任务中将虚拟机部署到的 Azure 区域的名称 |

   >备注：请等待部署完成。 部署大约需要 1 分钟的时间完成。

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>任务 2：创建并配置一个 Azure 自动化帐户

1. 在 SEA-SVR2 上的 Azure 门户中，使用以下设置创建 Azure 自动化帐户：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | AZ801-L0201-RG |
   | 名称 | 任何唯一名称 |
   | 区域 | 根据[工作区映射文档](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)确定的 Azure 区域的名称 |

   >备注：请确保根据[工作区映射文档](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)指定 Azure 区域。

   >备注：请等待部署完成。 部署可能需要大约 3 分钟的时间完成。

1. 在 Azure 门户中，浏览到新创建的 Azure 自动化帐户。 
1. 在“自动化帐户”页面中，浏览到它的“清单”页 。
1. 在“清单”页中，通过将它与你之前在此任务中创建的 Log Analytics 工作区相关联来启用解决方案。

   >备注：请等待相应 Log Analytics 解决方案的安装完成。 这可能需要大约 3 分钟。 

   >备注：这还会自动安装“更改跟踪”解决方案 。

1. 在“自动化帐户”页面中，浏览到它的“更新管理”页 。
1. 在“自动化帐户”页上，通过将它与你之前在此任务中创建的 Log Analytics 工作区相关联来启用解决方案。

   >备注：请等待安装完成。 这可能需要大约 5 分钟。

## <a name="exercise-2-configuring-microsoft-defender-for-cloud"></a>练习 2：配置 Microsoft Defender for Cloud

### <a name="scenario"></a>场景

接下来，需要确保你能够利用 Defender for Cloud 提供的增强安全功能。 这使你能够正确评估应用于 Windows Server 混合场景的特性和功能。

此练习的主要任务如下：

1. 启用 Defender for Cloud 和自动代理安装。
1. 启用 Defender for Cloud 增强的安全性。

#### <a name="task-1-enable-defender-for-cloud-and-automatic-agent-installation"></a>任务 1：启用 Defender for Cloud 和自动代理安装

在此任务中，你将连接到你的 Azure 订阅并启用 Defender for Cloud 增强的安全性。

1. 在 SEA-SVR2 上的 Azure 门户中，浏览到“Microsoft Defender for Cloud”页面 。
1. 将你的订阅升级到 Defender for Cloud 并启用自动代理安装。 

   > 备注：你的订阅可能已启用了 Defender for Cloud 增强的安全性，在这种情况下，请继续进行下一个任务。

#### <a name="task-2-enable-enhanced-security-of-defender-for-cloud"></a>任务 2：启用 Defender for Cloud 增强的安全性

在此任务中，你将启用 Defender for Cloud 增强的安全性。

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，从“Microsoft Defender for Cloud | 概述”页浏览到“环境设置”页  。
1. 在“环境设置”页上，查看 Azure 订阅中可用的 Defender for Cloud 计划。

   > 备注：注意，可以选择性地禁用同一页面上列出的单个 Microsoft Defender 计划。

1. 启用“Microsoft Defender for Servers”计划。 
1. 浏览到“设置 \| 自动预配”页。 
1. 在扩展列表中，执行以下任务：

   - 利用在上一练习中创建的 Log Analytics 工作区启用“用于 Azure VM 的 Log Analytics 代理”。
   - 利用在上一练习中创建的 Log Analytics 工作区启用“用于 Azure Arc 计算机的 Log Analytics 代理(预览版)”。
   - 使用“Microsoft 威胁和漏洞管理”选项启用“计算机的漏洞评估” 。


1. 浏览到 Defender for Cloud 中的“云环境设置”页。
1. 在“环境设置”页上，展开表示你的 Azure 订阅的条目，并查看表示你在上一练习中创建的 Log Analytics 工作区的条目。
1. 在“设置 \| Defender 计划”页上，启用工作区中可用的所有 Defender for Cloud 计划。

   > 备注：若要启用所有 Defender for Cloud 功能（包括威胁防护功能），必须在包含适用工作负载的订阅上启用增强的安全性功能。 如果在工作区级别启用它，则不会为 Azure 资源启用实时 VM 访问、自适应应用程序控制和网络检测功能。 此外，只在工作区级别提供两种 Microsoft Defender 计划：适用于服务器的 Microsoft Defender，以及适用于计算机上的 SQL Server 的 Microsoft Defender。

1. 对于在上一练习中创建的工作区，将“数据收集”的范围设置为“所有事件” 。

   > 备注：在 Defender for Cloud 中选择数据收集层只会影响 Log Analytics 工作区中安全事件的存储。 无论你选择在工作区中存储哪一级别的安全事件，Log Analytics 代理仍将收集和分析 Defender for Cloud 的威胁防护所需的安全事件。 选择存储安全事件可以在工作区中调查、搜索和审核这些事件。

## <a name="exercise-3-provisioning-azure-vms-running-windows-server"></a>练习 3：预配运行 Windows Server 的 Azure VM

### <a name="scenario"></a>场景

必须在混合场景中测试 Defender for Cloud 功能，包括其适用于运行 Windows Server 的 Azure VM 的权益。 为实现此目的，你将使用 Azure 资源管理器模板来预配运行 Windows Server 的 Azure VM。

此练习的主要任务如下：

1. 启动 Azure Cloud Shell。
1. 使用 Azure 资源管理器模板部署 Azure VM。

#### <a name="task-1-start-azure-cloud-shell"></a>任务 1：启动 Azure Cloud Shell

在此任务中，你将启动 Azure Cloud Shell。

1. 在 SEA-SVR2 上的 Azure 门户中，在 Azure Cloud Shell 窗格中打开一个 PowerShell 会话。
1. 如果这是第一次启动 Cloud Shell，请接受其默认配置设置。

#### <a name="task-2-deploy-an-azure-vm-by-using-an-azure-resource-manager-template"></a>任务 2：使用 Azure 资源管理器模板部署 Azure VM

在此任务中，你将使用 Azure 资源管理器模板部署 Azure VM。

1. 在 SEA-SVR2 上的 Azure 门户中，在 Cloud Shell 窗格中，将文件 C:\\Labfiles\\Lab02\\L02-sub_template.json、C:\\Labfiles\\Lab02\\L02-rg_template.json 和 C:\\Labfiles\\Lab02\\L02-rg_template.parameters.json 文件上传到 Cloud Shell 主目录中   。
1. 若要创建将托管实验室环境的资源组，在 Cloud Shell 窗格中的 PowerShell 会话中，运行以下命令（将 `<Azure_region>` 占位符替换为在此实验室中要向其中部署资源的 Azure 区域的名称）：

   >备注：可以使用 (Get-AzLocation).Location 命令列出可用 Azure 区域的名称 ：

   ```powershell 
   $location = '<Azure_region>'
   New-AzSubscriptionDeployment -Location $location -Name az801l2001deployment -TemplateFile ./L02-sub_template.json -rgLocation $location -rgName 'AZ801-L0202-RG'
   ```

1. 若要将 Azure 虚拟机 (VM) 部署到新创建的资源组中，请运行以下命令：

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l2002deployment -ResourceGroupName AZ801-L0202-RG -TemplateFile ./L02-rg_template.json -TemplateParameterFile ./L02-rg_template.parameters.json
   ```

   >备注：请等待部署完成。 这大约需要 3 分钟。

1. 关闭 Cloud Shell。

## <a name="exercise-4-onboarding-on-premises-windows-server-into-microsoft-defender-for-cloud-and-azure-automation"></a>练习 4：将本地 Windows Server 载入 Microsoft Defender for Cloud 和 Azure 自动化

### <a name="scenario"></a>场景

你需要将 SEA-SVR1 和 SEA-SVR2 载入 Defender for Cloud 来确定 Defender for Cloud 功能，这些功能可用于增强在本地环境中运行的服务器的安全性 。

此练习的主要任务如下：

1. 执行 Log Analytics 代理的手动安装。
1. 执行 Log Analytics 代理的无人参与安装。
1. 为 Azure VM 启用 Azure 自动化解决方案。
1. 为本地服务器启用 Azure 自动化解决方案。

#### <a name="task-1-perform-manual-installation-of-the-log-analytics-agent"></a>任务 1：执行 Log Analytics 代理的手动安装

在此任务中，你将执行 Log Analytics 代理的手动安装。

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，浏览到 Defender for Cloud 的“Microsoft Defender for Cloud \| 清单”页 。
1. 在“Microsoft Defender for Cloud \| 清单”页上，选择“+ 添加非 Azure 服务器” 。
1. 在“将服务器载入 Defender for Cloud”页上，升级 Log Analytics 工作区 az801-l02-workspace 。

   > 备注：升级成功完成后，“升级”按钮的标签将更改为“+ 添加服务器”  。

1. 使用“+ 添加服务器”按钮浏览到“az801-l02-workspace \| 代理管理”页，你可以从中下载 Log Analytics 代理安装程序并确定完成代理安装所需的工作区 ID 和密钥 。
1. 在“az801-l02-workspace \| 代理管理”页上，记录“工作区 ID”和“主密钥”的值  。 此任务后面部分会用到它们。
1. 从“az801-l02-workspace \| 代理管理”页中，下载 Windows 代理（64 位）。
1. 使用下载的可执行文件启动“Microsoft Monitoring Agent 安装”向导。
1. 在“Microsoft Monitoring Agent 安装”向导的“Azure Log Analytics”页上，输入你之前在此任务中记录的“工作区 ID”和“主密钥”的值，并使用默认设置完成 Microsoft Monitoring Agent 的安装   。

#### <a name="task-2-perform-unattended-installation-of-the-log-analytics-agent"></a>任务 2：执行 Log Analytics 代理的无人参与安装

在此任务中，你将执行 Log Analytics 代理的无人参与安装。

1. 在 SEA-SVR2 上，以管理员身份启动 Windows PowerShell 。
1. 若要提取 MMASetup-AMD64.exe 文件的内容，请在 Windows PowerShell 控制台中运行以下命令 ：
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Labfiles\L02\' -Force
   Copy-Item -Path $env:USERPROFILE\Downloads\MMASetup-amd64.exe -Destination 'C:\Labfiles\L02\' -Force
   Set-Location -Path C:\Labfiles\L02
   .\MMASetup-amd64.exe /c /t:C:\Labfiles\L02
   Remove-Item -Path .\MMASetup-amd64.exe
   ```

1. 若要将安装文件复制到目标 SEA-SVR1，请在 Windows PowerShell 控制台中运行以下命令 ：
    
   ```powershell
   New-Item -ItemType Directory -Path '\\SEA-SVR1\c$\Labfiles\L02' -Force
   Copy-Item -Path 'C:\Labfiles\L02\*' -Destination '\\SEA-SVR1\c$\Labfiles\L02' -Recurse -Force
   ```

1. 若要在 SEA-SVR1 上执行 Log Analytics 代理的安装，请在 Windows PowerShell 控制台中运行以下命令（将 `<WorkspaceID>` 和 `<PrimaryKey>` 占位符替换为你在此练习的上一个任务中记录的“工作区 ID”和“工作区密钥”的值）   ：

   ```powershell
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Labfiles\L02\setup.exe -ArgumentList '/qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<WorkspaceID>" OPINSIGHTS_WORKSPACE_KEY="<PrimaryKey>" AcceptEndUserLicenseAgreement=1' -Wait }
   ```

   > 备注：请等待安装完成。 这大约需要 1 分钟。

#### <a name="task-3-enable-azure-automation-solutions-for-azure-vms"></a>任务 3：为 Azure VM 启用 Azure 自动化解决方案

1. 在 SEA-SVR2 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后浏览到之前你在此实验室中预配的 Azure 自动化帐户页。 
1. 在“自动化帐户”页面中，浏览到它的“清单”页。
1. 在“清单”页上，使用工具栏中的“+ 添加 Azure VM”按钮为 az801l02-vm0 Azure VM 启用清单和更改跟踪  。 

   > 备注：VM 必须连接到与自动化帐户解决方案关联的 Log Analytics 工作区才能被列为“准备启用” 。

1. 浏览到自动化帐户的“更新管理”页。
1. 在“更新管理”页上，使用工具栏中的“+ 添加 Azure VM”按钮为 az801l02-vm0 Azure VM 启用更新管理  。 

   > 备注：与清单解决方案一样，VM 必须连接到与自动化帐户解决方案关联的 Log Analytics 工作区才能被列为“准备启用” 。

   > 备注：本地 VM 是随着安装 Log Analytics 代理并注册到与托管“清单”、“更改跟踪”和“更新管理”解决方案的 Azure 自动化帐户关联的 Azure Log Analytics 工作区一同载入的，你在此练习的上一任务中执行了这些操作。

#### <a name="task-4-enable-azure-automation-solutions-for-on-premises-servers"></a>任务 4：为本地服务器启用 Azure 自动化解决方案

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，浏览到之前你在此实验室中预配的“Azure 自动化帐户”页 。 
1. 在“自动化帐户”页面中，浏览到它的“清单”页 。
1. 在“清单”页上，使用“单击以管理计算机”链接为本地服务器启用清单和更改跟踪 。

   > 备注：此选项适用于已安装 Log Analytics 代理并在 Azure Log Analytics 工作区注册的本地服务器，该工作区与托管清单、更改跟踪和更新管理解决方案的 Azure 自动化帐户关联。

1. 浏览到自动化帐户的“更新管理”页。
1. 在“更新管理”页上，使用“单击以管理计算机”链接为本地服务器启用更新管理 。

   > 备注：与清单和更改跟踪解决方案一样，此选项适用于已安装 Log Analytics 代理并在 Azure Log Analytics 工作区注册的本地服务器，该工作区与托管清单、更改跟踪和更新管理解决方案的 Azure 自动化帐户关联。

## <a name="exercise-5-verifying-the-hybrid-capabilities-of-microsoft-defender-for-cloud-and-azure-automation-solutions"></a>练习 5：验证 Microsoft Defender for Cloud 和 Azure 自动化解决方案的混合功能

### <a name="scenario"></a>场景

混合使用运行 Windows Server 的本地 Azure VM 和服务器时，需要验证在这两种情况下均可用的 Defender for Cloud 和 Azure 自动化安全相关解决方案的功能。 你将模拟针对这两种资源发起网络攻击，并在 Defender for Cloud 中识别警报。

此练习的主要任务如下：

1. 验证 Azure VM 的威胁检测功能。
1. 验证本地服务器的威胁检测功能。
1. 查看应用于混合场景的特性和功能。
1. 验证 Azure 自动化解决方案。

#### <a name="task-1-validate-threat-detection-capabilities-for-azure-vms"></a>任务 1：验证 Azure VM 的威胁检测功能

在此任务中，你将验证 Azure VM 的威胁检测功能。

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“az801l02-vm0”虚拟机的页面 。
1. 在“az801l02-vm0”页上，使用“运行命令”功能运行以下 PowerShell 命令以触发威胁检测警报 ：

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   Start-Process -FilePath powershell.exe -ArgumentList '-nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"' -Wait
   ```

1. 在 SEA-ADM1 上的 Azure 门户中，浏览到“Microsoft Defender for Cloud \| 安全警报”页面 。
1. 在“Microsoft Defender for Cloud \| 安全警报”页上，注意高严重性警报，这表明在 az801l02-vm0 上存在 PowerShell 可疑使用 。
1. 选择安全警报，在“安全警报”页上，选择“采取措施”，然后查看可行的措施 。

   > 备注：为了最大限度地减少未来攻击的可能性，应该考虑实施安全建议。

#### <a name="task-2-validate-the-threat-detection-capabilities-for-on-premises-servers"></a>任务 2：验证本地服务器的威胁检测功能

在此任务中，你将验证本地服务器的威胁检测功能。

1. 在 SEA-ADM1 上，切换到 Windows PowerShell 控制台 。
1. 若要触发威胁检测警报，请在 Windows PowerShell 控制台中运行以下命令：
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   powershell -nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"
   ```

1. 在 SEA-ADM1 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后浏览回“Microsoft Defender for Cloud \| 安全警报”页 。
1. 在“Microsoft Defender for Cloud \| 安全警报”页上，注意高严重性警报，这表明在 SEA-SVR2 上存在 PowerShell 可疑使用 。
1. 选择安全警报，在“安全警报”页上，选择“采取措施”，然后查看可行的措施 。

   > 备注：为了最大限度地减少未来攻击的可能性，应该考虑实施安全建议。

#### <a name="task-3-review-the-features-and-capabilities-that-apply-to-hybrid-scenarios"></a>任务 3：查看应用于混合场景的特性和功能

在此任务中，你将查看应用于混合场景的 Defender for Cloud 特性和功能。

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，浏览到“Microsoft Defender for Cloud \| 清单”页 。
1. 在“清单”页的资源列表中，标识代表 az801l02-vm0 Azure VM 以及 SEA-SVR1.contoso.com 和 SEA-SVR2.contoso.com 本地服务器的条目   。

   > 备注：可能需要几分钟时间，代表 Azure 和本地 VM 的条目才会出现在“清单”页上 。

   > 备注：如果 az801l02-vm0 在监视代理列中报告“未安装”状态，请选择 az801l02-vm0 链接    。 在“资源运行状况(预览版)”页上，查看“建议”，然后选择“应在虚拟机上安装 Log Analytics 代理”条目  。 在“应在虚拟机上安装 Log Analytics 代理”页上，选择“修复” 。 在“修复资源”页的“工作区 ID”下拉列表中，选择 Defender for Cloud 创建的默认工作区，然后选择“修复 1 个资源”  。 

1. 浏览到 az801l02-vm0 的“资源运行状况(预览版)”页并查看“建议”  。
1. 浏览到 SEA-SVR2 的“资源运行状况(预览版)”页并查看“建议”  。

#### <a name="task-4-validate-azure-automation-solutions"></a>任务 4：验证 Azure 自动化解决方案

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，浏览回之前你在此实验室中预配的 Azure 自动化帐户的“清单”页 。
1. 在“清单”页面上，查看“计算机”选项卡并验证它是否包括 Azure VM 和之前你在此实验室中向 Log Analytics 工作区注册的本地服务器 。

   >备注：如果“计算机”选项卡上未列出 Azure VM 服务器或本地服务器或者两者均未列出，你可能需要等待更长的时间 。

1. 在“清单”页上，查看其余选项卡，包括“软件”、“文件”、“Windows 注册表”和“Windows 服务”选项卡    。

   >备注：收集的数据和文件可通过“清单”页工具栏中的“编辑设置”选项进行配置  。

   >备注：这还会自动安装“更改跟踪”解决方案 。

1. 浏览到同一 Azure 自动化帐户的“更改跟踪”页。 
1. 标识与“事件”、“文件”、“注册表”、“软件”和“Windows 服务”条目关联的编号    。 如果其中任何一个大于 0，你可以在页面底部的“更改”和“事件”选项卡上找到有关相应更改的更多详细信息 。

   >备注：在这种情况下，跟踪的更改也可以通过“更改跟踪”页工具栏中的“编辑设置”选项进行配置  。

1. 浏览到同一 Azure 自动化帐户的“更新管理”页。
1. 在“更新管理”页上，查看“计算机”选项卡并验证它是否包括 Azure VM 和之前你在此实验室中向 Log Analytics 工作区注册的本地服务器 。
1. 标识“计算机”选项卡上每个条目的合规状态，然后浏览到“缺少的更新”选项卡 。

   >备注：可以选择为本地服务器和 Azure VM 安排缺少的更新的自动部署。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>练习 6：取消预配 Azure 环境

### <a name="scenario"></a>场景

为了最大限度地减少与 Azure 相关的费用，需要取消预配整个实验室中预配的 Azure 资源。

此练习的主要任务是：

1. 在 Cloud Shell 中启动一个 PowerShell 会话。
1. 标识并删除在实验室中预配的所有 Azure 资源。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-SVR2 上显示 Azure 门户的 Microsoft Edge 窗口中，打开 Cloud Shell 窗格。

#### <a name="task-2-identify-and-remove-all-azure-resources-that-were-provisioned-in-the-lab"></a>任务 2：标识并删除在实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*'
   ```

   > 备注：验证输出中是否仅包含你在此实验室中创建的资源组。 将在此任务中删除这些组。

1. 若要删除你在此实验室中创建的所有资源组，请运行以下命令：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >备注：该命令异步执行（由 -AsJob 参数决定），因此，虽然你可以随后立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。

## <a name="results"></a>结果

完成此实验室后，你已经：

- 创建了 Azure Log Analytics 工作区和 Azure 自动化帐户。
- 配置了 Defender for Cloud。
- 预配了运行 Windows Server 的 Azure VM。
- 将本地 Windows Server 载入了 Defender for Cloud 和 Azure 自动化。
- 验证了 Defender for Cloud 和 Azure 自动化解决方案的混合功能。
