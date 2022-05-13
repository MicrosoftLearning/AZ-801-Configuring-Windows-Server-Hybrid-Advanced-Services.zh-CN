---
lab:
  title: 实验室：在混合场景中实现安全解决方案
  type: Answer Key
  module: 'Module 2: Implementing Security Solutions in Hybrid Scenarios'
ms.openlocfilehash: 34c38131c3d92d6250260fd98da506dec1d42cda
ms.sourcegitcommit: e31cab525d98deeb444bd10846cbb38215bcdb29
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/01/2022
ms.locfileid: "141347650"
---
# <a name="lab-answer-key-implementing-security-solutions-in-hybrid-scenarios"></a>实验室解答：在混合场景中实现安全解决方案

## <a name="exercise-1-creating-an-azure-log-analytics-workspace-and-an-azure-automation-account"></a>练习 1：创建 Azure Log Analytics 工作区和 Azure 自动化帐户

#### <a name="task-1-create-an-azure-log-analytics-workspace"></a>任务 1：创建 Azure Log Analytics 工作区 

1. 连接到 SEA-SVR2，然后根据需要，以 CONTOSO\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-SVR2 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录 。
1. 在 SEA-SVR2 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“Log Analytics 工作区”，然后从“Log Analytics 工作区”页中选择“+ 创建”    。
1. 在“创建 Log Analytics 工作区”页的“基本信息”选项卡上，输入以下设置，选择“查看 + 创建”，然后选择“创建”   ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0201-RG 的名称 |
   | Log Analytics 工作区 | 任何唯一名称 |
   | 区域 | 选择你附近的区域 |

   >备注：请等待部署完成。 部署大约需要 1 分钟的时间完成。

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>任务 2：创建并配置 Azure 自动化帐户

1. 在 SEA-SVR2 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“自动化帐户”，然后从“自动化帐户”页中选择“+ 创建”    。
1. 在“创建自动化帐户”页上，指定以下设置，然后选择“查看 + 创建” 。 验证后，选择“创建”：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | AZ801-L0201-RG |
   | 名称 | 任何唯一名称 |
   | 区域 | 根据[工作区映射文档](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)确定的 Azure 区域的名称 |

   >备注：请确保根据[工作区映射文档](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)指定 Azure 区域 。

   >备注：请等待部署完成。 部署可能需要大约 3 分钟。

1. 在“部署”页上，选择“前往资源”。
1. 在“自动化帐户”页的“配置管理”部分，选择“清单”  。
1. 在“清单”页的“Log Analytics 工作区”下拉列表中，选择你之前在此任务中创建的 Log Analytics 工作区，然后选择“启用”  。

   >备注：请等待相应 Log Analytics 解决方案的安装完成。 这可能需要大约 3 分钟的时间。 

   >备注：这还会自动安装“更改跟踪”解决方案 。

1. 在“自动化帐户”页的“更新管理”部分，选择“更新管理”并选择“启用”   。

   >备注：请等待安装完成。 这可能需要大约 5 分钟。

## <a name="exercise-2-configuring-microsoft-defender-for-cloud"></a>练习 2：配置 Microsoft Defender for Cloud

#### <a name="task-1-enable-defender-for-cloud-and-automatic-agent-installation"></a>任务 1：启用 Defender for Cloud 和自动代理安装

1. 在 SEA-SVR2 上，在 Azure 门户的“搜索资源、服务和文档”文本框中的工具栏上，搜索并选择 Microsoft Defender for Cloud  。
1. 在“Microsoft Defender for Cloud \| 开始”页上，选择“升级”，然后选择“安装代理”  。

   > 备注：你的订阅可能已经启用 Defender for Cloud 的增强安全性，在这种情况下，请继续执行下一个任务。

#### <a name="task-2-enable-enhanced-security-of-defender-for-cloud"></a>任务 2：启用 Defender for Cloud 的增强安全性

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在“Microsoft Defender for Cloud | 概览”页左侧垂直菜单的“管理”部分，选择“环境设置”   。
1. 在“环境设置”页上，选择代表你的 Azure 订阅的条目。
1. 在“设置 \| Defender 计划”页上，选择磁贴“启用所有 Microsoft Defender for Cloud 计划” 。

   > 备注：注意，可以选择性地禁用同一页面上列出的单个 Microsoft Defender 计划。

1. 将所有计划（第一个计划除外，它代表 Microsoft Defender for Servers）设置为“关闭”，然后选择“保存”  。
1. 在“设置 \| Defender 计划”页左侧垂直菜单的“设置”部分，选择“自动预配”  。
1. 在“设置 \| 自动预配”页的扩展列表中，在“Azure VM 的 Log Analytics 代理”条目右侧，选择“编辑配置”链接  。
1. 在“扩展部署配置”页的“工作区配置”部分，选择“将 Azure VM 连接到其他工作区”选项  。 在下拉菜单中，选择代表在上一个练习中创建的工作区的条目，然后选择“应用”。
1. 返回“设置 \| 自动预配”页，将“Azure Arc Machine 的 Log Analytics 代理(预览版)”设置为“开”  。 这将自动显示“扩展部署配置”页。 
1. 在“扩展部署配置”页的“选择 Log Analytics 工作区”下拉列表中，选择代表你在上一个练习中创建的工作区的条目，然后选择“应用”  。
1. 返回“设置 \| 自动预配”页面，将“计算机的漏洞评估”设置为“开”  。 在“扩展部署配置”页中，确保选中“Microsoft 威胁和漏洞管理”选项，然后选择“应用”  。
1. 返回“设置 \| 自动预配”页，将“来宾配置代理(预览版)”设置为“开”  。
1. 返回“设置 \| 自动预配”页，将“Microsoft Dependency Agent(预览版)”设置为“开”  。
1. 在页面顶部选择“保存”。
1. 浏览回“Microsoft Defender for Cloud | 概览”页，然后在左侧垂直菜单的“管理”部分，选择“环境设置”  。
1. 在“环境设置”页上，展开代表你的 Azure 订阅的条目，并选择代表你在上一个练习中创建的 Log Analytics 工作区的条目。
1. 在“设置 \| Defender 计划”页上，选择磁贴“启用所有 Microsoft Defender for Cloud 计划”，然后选择“保存”  。

   > 备注：若要启用所有 Defender for Cloud 功能（包括威胁防护功能），必须在包含适用工作负载的订阅上启用增强的安全性功能。 如果在工作区级别启用它，则不会为 Azure 资源启用实时 VM 访问、自适应应用程序控制和网络检测功能。 此外，只在工作区级别提供两种 Microsoft Defender 计划：适用于服务器的 Microsoft Defender，以及适用于计算机上的 SQL Server 的 Microsoft Defender。

1. 在“设置 \| Defender 计划”页左侧垂直菜单的“设置”部分，选择“数据收集”  。
1. 在“设置 \| 数据收集”上，选择“所有事件”，然后选择“保存”  。

   > 备注：在 Defender for Cloud 中选择数据收集层只会影响 Log Analytics 工作区中安全事件的存储。 无论你选择在工作区中存储哪一级别的安全事件，Log Analytics 代理仍将收集和分析 Defender for Cloud 的威胁防护所需的安全事件。 选择存储安全事件可以在工作区中调查、搜索和审核这些事件。

## <a name="exercise-3-provisioning-azure-vms-running-windows-server"></a>练习 3：预配运行 Windows Server 的 Azure VM

#### <a name="task-1-start-azure-cloud-shell"></a>任务 1：启动 Azure Cloud Shell

1. 在 SEA-SVR2 上，在 Azure 门户中，通过选择搜索文本框旁边的工具栏图标打开“Cloud Shell”窗格。
1. 如果系统提示选择 Bash 或 PowerShell，请选择 PowerShell  。

   >备注：如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

#### <a name="task-2-deploy-an-azure-vm-by-using-an-azure-resource-manager-template"></a>任务 2：使用 Azure 资源管理器模板部署 Azure VM

1. 在 Cloud Shell 窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，并将文件 C:\\Labfiles\\Lab02\\L02-sub_template.json 上传到 Cloud Shell 主目录中  。
1. 重复上一步两次，将 C:\\Labfiles\\Lab02\\L02-rg_template.json 和 C:\\Labfiles\\Lab02\\L02-rg_template.parameters.json 文件上传到 Cloud Shell 主目录中 。
1. 若要创建将托管实验室环境的资源组，请在 Cloud Shell 窗格中的 PowerShell 会话中，输入以下命令，并在输入每个命令后按 Enter（将 `<Azure_region>` 占位符替换为在此实验室中要向其中部署资源的 Azure 区域的名称）：

   >备注：可以使用 (Get-AzLocation).Location 命令列出可用 Azure 区域的名称 ：

   ```powershell 
   $location = '<Azure_region>'
   New-AzSubscriptionDeployment -Location $location -Name az801l2001deployment -TemplateFile ./L02-sub_template.json -rgLocation $location -rgName 'AZ801-L0202-RG'
   ```

1. 要将 Azure 虚拟机 (VM) 部署到新创建的资源组中，请输入以下命令并按 Enter：

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l2002deployment -ResourceGroupName AZ801-L0202-RG -TemplateFile ./L02-rg_template.json -TemplateParameterFile ./L02-rg_template.parameters.json
   ```

   >备注：请等待部署完成。 这大约需要 3 分钟。

1. 关闭 Cloud Shell。

## <a name="exercise-4-onboarding-on-premises-windows-server-into-microsoft-defender-for-cloud-and-azure-automation"></a>练习 4：将本地 Windows Server 载入 Microsoft Defender for Cloud 和 Azure 自动化

#### <a name="task-1-perform-manual-installation-of-the-log-analytics-agent"></a>任务 1：执行 Log Analytics 代理的手动安装

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在“Microsoft Defender for Cloud \| 概览”页左侧垂直菜单的“常规”部分，选择“清单”   。
1. 在“Microsoft Defender for Cloud \| 清单”页上，选择“+ 添加非 Azure 服务器” 。
1. 在“将服务器载入 Defender for Cloud”页上，在代表你在此实验室前面预配的 Log Analytics 工作区的条目旁边，选择“升级” 。

   > 备注：升级成功完成后，“升级”按钮的标签将更改为“+ 添加服务器”  。

1. 选择“+ 添加服务器”按钮。 这将自动显示“代理管理”页，你可以从中下载 Log Analytics 代理安装程序并确定完成代理安装所需的工作区 ID 和密钥。
1. 在“代理管理”页上，记录“工作区 ID”和“主密钥”的值  。 此任务后面部分会用到它们。
1. 在“代理管理”页上，选择“下载 Windows 代理(64 位)”链接 。
1. 下载完成后，选择“打开文件”。 这将启动“Microsoft Monitoring Agent 安装”向导。
1. 在“Microsoft Monitoring Agent 安装”向导的“欢迎使用 Microsoft Monitoring Agent 安装向导”页上，选择“下一步”  。
1. 在 Microsoft Monitoring Agent 安装向导的“重要通知”页上，选择“我同意”  。
1. 在 Microsoft Monitoring Agent 安装向导的“目标文件夹”页上，选择“下一步”  。
1. 在“Microsoft Monitoring Agent 安装”导向的“代理安装选项”页上，依次选择“将代理连接到 zure Log Analytics (OMS) 复选框”和“下一步”   。
1. 在“Microsoft Monitoring Agent 安装”向导的 Azure Log Analytics 页上，输入你之前在此任务中记录的“工作区 ID”和“工作区密钥”的值，然后选择“下一步”    。
1. 在“Microsoft Monitoring Agent 安装”向导的“Microsoft 更新”页上，选择“下一步”  。
1. 在“Microsoft Monitoring Agent 安装”向导的“准备安装”页上，选择“下一步”  。
1. 安装完成后，在“Microsoft Monitoring Agent 配置成功完成”页上，选择“完成” 。

#### <a name="task-2-perform-unattended-installation-of-the-log-analytics-agent"></a>任务 2：执行 Log Analytics 代理的无人参与安装

1. 在 SEA-SVR2 上，选择“开始”，然后选择 Windows PowerShell (Admin)  。
1. 要提取 MMASetup-AMD64.exe 文件的内容，请在 Windows PowerShell 控制台中输入以下命令，并在输入每个命令后按 Enter ：
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Labfiles\L02\' -Force
   Copy-Item -Path $env:USERPROFILE\Downloads\MMASetup-amd64.exe -Destination 'C:\Labfiles\L02\' -Force
   Set-Location -Path C:\Labfiles\L02
   .\MMASetup-amd64.exe /c /t:C:\Labfiles\L02
   Remove-Item -Path .\MMASetup-amd64.exe
   ```

1. 要将安装文件复制到目标 SEA-SVR1，请在 Windows PowerShell 控制台中输入以下命令，并在输入每个命令后按 Enter ：
    
   ```powershell
   New-Item -ItemType Directory -Path '\\SEA-SVR1\c$\Labfiles\L02' -Force
   Copy-Item -Path 'C:\Labfiles\L02\*' -Destination '\\SEA-SVR1\c$\Labfiles\L02' -Recurse -Force
   ```

1. 要在 SEA-SVR1 上执行 Log Analytics 代理的安装，请在 Windows PowerShell 控制台中输入以下命令并按 Enter（将 `<WorkspaceID>` 和 `<PrimaryKey>` 占位符替换为你在此练习的上一个任务中记录的“工作区 ID”和“工作区密钥”的值）   ：

   ```powershell
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Labfiles\L02\setup.exe -ArgumentList '/qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<WorkspaceID>" OPINSIGHTS_WORKSPACE_KEY="<PrimaryKey>" AcceptEndUserLicenseAgreement=1' -Wait }
   ```

   > 备注：请等待安装完成。 这大约需要 1 分钟。

#### <a name="task-3-enable-azure-automation-solutions-for-azure-vms"></a>任务 3：为 Azure VM 启用 Azure 自动化解决方案

1. 在 SEA-SVR2 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后浏览到你在此实验室前面预配的 Azure 自动化帐户页。 
1. 在“自动化帐户”页的“配置管理”部分，选择“清单”  。
1. 在“清单”页的工具栏中，选择“+ 添加 Azure VM” 。
1. 在“启用清单”页上的 VM 列表中，确保选中 az801l02-vm0 条目旁边的复选框，然后选择“启用”  。

   > 备注：VM 必须连接到与自动化帐户解决方案关联的 Log Analytics 工作区，才能列为“准备启用” 。

1. 浏览回“自动化帐户”页并在“更新管理”部分，选择“更新管理”  。
1. 在“更新管理”页的工具栏中，选择“+ 添加 Azure VM” 。
1. 在“启用更新管理”页上的 VM 列表中，确保选中 az801l02-vm0 条目旁边的复选框，然后选择“启用”  。

   > 备注：与清单和更改跟踪解决方案一样，VM 必须连接到与自动化帐户解决方案关联的 Log Analytics 工作区，才能列为“准备启用” 。

#### <a name="task-4-enable-azure-automation-solutions-for-on-premises-servers"></a>任务 4：为本地服务器启用 Azure 自动化解决方案

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览回“自动化帐户”页，然后在“配置管理”部分，选择“库存”   。
1. 在“清单”页面上，选择“单击以管理计算机”链接 。
1. 在“管理计算机”页上，选择“在所有可用和未来计算机上启用”选项，然后选择“启用”  。

   > 备注：此选项适用于已安装 Log Analytics 代理并在 Azure Log Analytics 工作区注册的本地服务器，该工作区与托管清单、更改跟踪和更新管理解决方案的 Azure 自动化帐户关联。

1. 浏览回“自动化帐户”页并在“更新管理”部分，选择“更新管理”  。
1. 在“更新管理”页上，选择“单击以管理计算机”链接 。
1. 在“管理计算机”页上，选择“在所有可用和未来计算机上启用”选项，然后选择“启用”  。

   > 备注：与清单和更改跟踪解决方案一样，此选项适用于已安装 Log Analytics 代理并在 Azure Log Analytics 工作区注册的本地服务器，该工作区与托管清单、更改跟踪和更新管理解决方案的 Azure 自动化帐户关联。

## <a name="exercise-5-verifying-the-hybrid-capabilities-of-microsoft-defender-for-cloud-and-azure-automation-solutions"></a>练习 5：验证 Microsoft Defender for Cloud 和 Azure 自动化解决方案的混合功能

#### <a name="task-1-validate-threat-detection-capabilities-for-azure-vms"></a>任务 1：验证 Azure VM 的威胁检测功能

1. 在 SEA-SVR2 上，切换到显示 Azure 门户的 Microsoft Edge。 
1. 在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“虚拟机” 。
1. 在“虚拟机”页上，选择 az801l02-vm0 。
1. 在 az801l02-vm0 页的“操作”部分，选择“运行命令”，然后选择 RunPowerShellScript   。
1. 在“运行命令脚本”页的“PowerShell 脚本”部分，输入以下命令，然后选择“运行”以触发威胁检测警报  ：

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   Start-Process -FilePath powershell.exe -ArgumentList '-nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"' -Wait
   ```

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览到“Microsoft Defender for Cloud \| 概览”页 。 
1. 在“Microsoft Defender for Cloud \| 概览”页上，在左侧垂直菜单的“常规”部分，选择“安全警报”  。
1. 在“Microsoft Defender for Cloud \| 安全警报”页上，注意高严重性警报，表明在 az801l02-vm0 上存在 PowerShell 可疑使用 。
1. 选择安全警报，在“安全警报”页上，选择“采取措施”，然后查看可行的措施 。

   > 备注：为了最大限度地减少未来攻击的可能性，应该考虑实施安全建议。

#### <a name="task-2-validate-the-threat-detection-capabilities-for-on-premises-servers"></a>任务 2：验证本地服务器的威胁检测功能

1. 在 SEA-SVR2 上，切换到 Windows PowerShell 控制台 。
1. 若要触发威胁检测警报，请在 Windows PowerShell 控制台中输入以下命令，然后在输入每个命令后按 Enter：
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   powershell -nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"
   ```

1. 在 SEA-SVR2 上，切换到显示 Azure 门户的 Microsoft Edge 窗口，然后浏览回“Microsoft Defender for Cloud \| 安全警报”页 。
1. 在“Microsoft Defender for Cloud \| 安全警报”页上，注意高严重性警报，表明在 SEA-SVR2 上存在 PowerShell 可疑使用 。
1. 选择安全警报，在“安全警报”页上，选择“采取措施”，然后查看可行的措施 。

   > 备注：为了最大限度地减少未来攻击的可能性，应该考虑实施安全建议。

#### <a name="task-3-review-the-features-and-capabilities-that-apply-to-hybrid-scenarios"></a>任务 3：查看适用于混合场景的特性和功能

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在“Microsoft Defender for Cloud \| 概览”页左侧垂直菜单的“常规”部分，选择“清单”   。
1. 在“清单”页的资源列表中，标识代表 az801l02-vm0 Azure VM 以及 SEA-SVR1.contoso.com 和 SEA-SVR2.contoso.com 本地服务器的条目   。

   > 备注：可能需要几分钟时间，代表 Azure 和本地 VM 的条目才会出现在“清单”页上 。

   > 备注：如果 az801l02-vm0 在监视代理列中报告“未安装”状态，请选择 az801l02-vm0 链接    。 在“资源运行状况(预览版)”页上，查看建议，然后选择“应在虚拟机上安装 Log Analytics 代理”条目  。 在“应在虚拟机上安装 Log Analytics 代理”页上，选择“修复” 。 在“修复资源”页的“工作区 ID”下拉列表中，选择 Defender for Cloud 创建的默认工作区，然后选择“修复 1 个资源”  。 

1. 在“清单”页上，选择 az801l02-vm0 链接，然后在“资源运行状况(预览版)”页上查看建议   。
1. 浏览回“清单”页，选择代表 SEA-SVR2 的链接，然后在“资源运行状况(预览版)”页上查看建议   。

#### <a name="task-4-validate-azure-automation-solutions"></a>任务 4：验证 Azure 自动化解决方案

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，浏览回你在此实验室中前面预配的 Azure 自动化帐户的页面，然后在“配置管理”部分，选择“清单”  。
1. 在“清单”页面上，查看“计算机”选项卡并验证它是否包括 Azure VM 和你在此实验室前面向 Log Analytics 工作区注册的本地服务器 。

   >备注：如果“计算机”选项卡上未列出其中一种或两种类型的系统，你可能需要等待更长时间 。

1. 在“清单”页上，查看其余选项卡，包括“软件”、“文件”、“Windows 注册表”和“Windows 服务”选项卡    。

   >备注：收集的数据和文件可通过“清单”页工具栏中的“编辑设置”选项进行配置  。

   >备注：这还会自动安装“更改跟踪”解决方案 。

1. 浏览回你在此实验室中前面预配的 Azure 自动化帐户的页面，然后在“配置管理”部分，选择“更改跟踪” 。 
1. 标识与“事件”、“文件”、“注册表”、“软件”和“Windows 服务”条目关联的编号    。 如果其中任何一个大于 0，你可以在页面底部的“更改”和“事件”选项卡上找到有关相应更改的更多详细信息 。

   >备注：在这种情况下，跟踪的更改也可以通过“更改跟踪”页工具栏中的“编辑设置”选项进行配置  。

1. 浏览回你在此实验室中前面预配的 Azure 自动化帐户的页面，然后在“更新管理”部分，选择“更新管理” 。
1. 在“更新管理”页上，查看“计算机”选项卡并验证它是否包括 Azure VM 和你在此实验室前面向 Log Analytics 工作区注册的本地服务器 。
1. 标识“计算机”选项卡上每个条目的合规状态，然后浏览到“缺少的更新”选项卡 。

   >备注：可以选择为本地服务器和 Azure VM 安排缺少的更新的自动部署。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>练习 6：取消预配 Azure 环境

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>任务 2：标识实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出你在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*'
   ```

   > 备注：验证输出是否仅包含你在此实验室中创建的资源组。 此任务中将删除该组。

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >备注：该命令异步执行（由 -AsJob 参数决定），因此，虽然你可以随后立即在同一个 PowerShell 会话中运行另一个 PowerShell 命令，但需要几分钟才能实际删除资源组。