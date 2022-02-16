---
lab:
  title: 实验室：在混合场景中实现操作监视
  module: 'Module 9: Implementing operational monitoring in hybrid scenarios'
ms.openlocfilehash: c4f2f64b9027140c85099da700b4cdda49d55715
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907058"
---
# <a name="lab-implementing-operational-monitoring-in-hybrid-scenarios"></a>实验室：在混合场景中实现操作监视

## <a name="lab-scenario"></a>实验室方案

你需要评估 Microsoft Azure 功能，以便深入了解 Azure 资源的性能和配置，特别是 Azure 虚拟机 (VM)。 若要实现此目的，你打算检查 Azure Monitor 的功能，包括 Log Analytics。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 准备监视环境。
- 配置对本地服务器的监视。
- 配置对 Azure VM 的监视。
- 评估监视服务。

## <a name="estimated-time-60-minutes"></a>估计时间：60 分钟

## <a name="lab-environment"></a>实验室环境
  
虚拟机：AZ-801T00A-SEA-DC1 和 AZ-801T00A-SEA-SVR2 必须正在运行 。 其他 VM 可以运行，但本实验室不需要这些 VM。

> 注意： AZ-801T00A-SEA-DC1 和 AZ-801T00A-SEA-SVR2 虚拟机托管 SEA-DC1 和 SEA-SVR2 的安装    。

1. 选择“SEA-SVR2”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境和 Azure 订阅。 在开始实验室之前，请确保拥有 Azure 订阅以及具有该订阅中“所有者”角色的用户帐户。

## <a name="exercise-1-preparing-a-monitoring-environment"></a>练习 1：准备监视环境

本练习的主要任务是：

1. 部署 Azure 虚拟机。
1. 注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序。
1. 创建并配置 Azure Log Analytics 工作区。

#### <a name="task-1-deploy-an-azure-virtual-machine"></a>任务 1：部署 Azure 虚拟机

在此任务中，你将部署用于测试监视方案的虚拟机。

1. 在 SEA-SVR2 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录 。
1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在 Azure Cloud Shell 中打开一个 PowerShell 会话。
1. 在 Cloud Shell 窗格中，将 C:\\Labfiles\\Lab02\\L02-sub_template.json、C:\\Labfiles\\Lab09\\L09-rg_template.json 和 C:\\Labfiles\\Lab09\\L09-rg_template.parameters.json 文件上传到 Cloud Shell 主目录  。
1. 若要创建将托管实验室环境的资源组，在 Cloud Shell 窗格中的 PowerShell 会话中，运行以下命令（将 `<Azure_region>` 占位符替换为在此实验室中要向其中部署资源的 Azure 区域的名称）：

   >注意：可以使用 (Get-AzLocation).Location 命令列出可用 Azure 区域的名称 ：

   ```powershell 
   $location = '<Azure_region>'
   $rgName = 'AZ801-L0901-RG'
   New-AzResourceGroup -ResourceGroupName $rgName -Location $location
   ```

1. 若要将 Azure VM 部署到新创建的资源组中，请运行以下命令：

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l0901deployment -ResourceGroupName $rgName -TemplateFile ./L09-rg_template.json -TemplateParameterFile ./L09-rg_template.parameters.json -AsJob
   ```

   >注意：请不要等待部署完成，而是继续执行下一个任务。 部署大约需要 3 分钟的时间完成。

#### <a name="task-2-register-the-microsoftinsights-and-microsoftalertsmanagement-resource-providers"></a>任务 2：注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序

1. 若要注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序，在 SEA-SVR2 上的 Cloud Shell 窗格中运行以下命令：

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
   Register-AzResourceProvider -ProviderNamespace Microsoft.AlertsManagement
   ```

   >注意：若要验证注册状态，可以使用 Get-AzResourceProvider cmdlet 。

   >注意：请不要等待注册过程完成，而是继续执行下一项任务。 注册大约需要 3 分钟的时间完成。

1. 关闭 Cloud Shell。

#### <a name="task-3-create-and-configure-an-azure-log-analytics-workspace"></a>任务 3：创建并配置 Azure Log Analytics 工作区

在此任务中，你将创建并配置 Azure Log Analytics 工作区和基于 Azure 自动化的解决方案。

1. 在 SEA-SVR2 上，在 Azure 门户中使用以下设置创建 Log Analytics 工作区：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0902-RG 的名称 |
   | Log Analytics 工作区 | 任何唯一名称 |
   | 区域 | 在上一个任务中将虚拟机部署到的 Azure 区域的名称 |

   >注意：请确保指定在上一个任务中向其中部署了虚拟机的同一区域。

   >注意：请等待部署完成。 部署大约需要 1 分钟的时间完成。

## <a name="exercise-2-configuring-monitoring-of-on-premises-servers"></a>练习 2：配置对本地服务器的监视

本练习的主要任务是：

1. 向 Azure 注册 Windows Admin Center。
1. 将本地 Windows Server 与 Azure Monitor 集成。

#### <a name="task-1-register-windows-admin-center-with-azure"></a>任务 1：向 Azure 注册 Windows Admin Center

1. 在 SEA-ADM1 上，以管理员身份启动 Windows PowerShell。

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

1. 在 SEA-ADM1 上，启动 Microsoft Edge 并连接到 Windows Admin Center 的本地实例 (https://SEA-ADM1.contoso.com ) 。 
1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在 Windows Admin Center，浏览到“设置”页面，并将 Windows Admin Center 注册到在此实验室中使用的 Azure 订阅。 

#### <a name="task-2-integrate-an-on-premises-windows-server-with-azure-monitor"></a>任务 2：将本地 Windows Server 与 Azure Monitor 集成

1. 在 SEA-SVR2 上，转到 sea-svr2.contoso.com 页面 。 
1. 在 sea-svr2.contoso.com 页面上，使用“工具”菜单将本地服务器载入 Azure Monitor 并采用以下设置  ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0901-RG** |
   | 资源组区域 | 在上一个练习中将虚拟机部署到的 Azure 区域的名称 |
   | Log Analytics 工作区 | 在上一个练习中创建的工作区的名称 |
   | 启用 Azure Arc | 已选定 |

   >注意：请不要等待安装完成，而是继续执行下一个练习。 安装过程大约需要 3 分钟。

   >注意：此过程会自动安装 Log Analytics Agent 和 Dependency Agent。

## <a name="exercise-3-configuring-monitoring-of-azure-vms"></a>练习 3：配置对 Azure VM 的监视

本练习的主要任务是：

1. 查看基于主机的监视。
1. 配置诊断设置和 VM 见解。

#### <a name="task-1-review-host-based-monitoring"></a>任务 1：查看基于主机的监视

在此任务中，你将查看 Azure 虚拟机的默认监视设置。

1. 在 Azure 门户中，浏览到 az801l09-vm0 虚拟机页。
1. 在“az801l09-vm0”页上的“监视”部分中，选择“指标”  。
1. 在“az801l09-vm0 \| 指标”页上，在默认图表上的“指标命名空间”下拉列表中，请注意，仅虚拟机主机度量值可用  。

   >注意：这是意料之中的，因为尚未配置来宾级别的诊断设置。 不过，你可以选择直接从“指标命名空间”下拉列表中启用来宾内存度量。 稍后将在本练习中启用它。

1. 在“指标”下拉列表中，查看可用指标的列表。

   >注意：此列表包含可从虚拟机主机收集的一系列 CPU、磁盘和网络相关指标，无需访问来宾级指标。

1. 在“az801l09-vm0 \| 指标”页上，显示说明 az801l09-vm0 的 CPU 平均使用率的图表 。

#### <a name="task-2-configure-diagnostic-settings-and-vm-insights"></a>任务 2：配置诊断设置和 VM 见解

在此任务中，将配置 Azure 虚拟机诊断设置。

1. 在“az801l09-vm0 ”页上的“监视”部分，选择“诊断设置”并启用来宾级别监视  。

   >注意：请等待操作生效。 这可能需要大约 3 分钟。

1. 切换到“az801l09-vm0 \| 诊断设置”页的“性能计数器”选项卡，并查看可用的计数器 。

   >注意：默认情况下，CPU、内存、磁盘和网络计数器均已启用。 可以切换到“自定义”视图以获取更详细的列表。

1. 切换到“az801l09-vm0 \| 诊断设置”页的“日志”选项卡，并启用来宾级别监视 。 

   >注意：请等待来宾级监视诊断启用。 这大约需要 3 分钟。

1. 在“az801l09-vm00 \| 诊断设置”页上的“概述”选项卡上，查看可用的事件日志收集选项 。

   >注意：默认情况下，日志集合包含应用程序日志和系统日志中的严重、错误和警告条目，以及安全日志中的审核失败条目。 你可以从“日志”选项卡自定义它们。

1. 在“az801l09-vm0 \| 诊断设置”页上，选择“日志”选项卡并查看可用的配置设置 。
1. 在“az801l09-vm0 \| 日志”页的左侧垂直菜单中，在”监视”部分选择”指标”  。
1. 在“az801l09-vm0 \| 指标”页的默认图表上，请注意，此时除了“虚拟机主机”条目外，“指标命名空间”下拉列表还包括“来宾(经典)”条目   。

   >注意：这是意料之中的，因为已启用来宾级别的诊断设置。 你还可以选择“启用新的来宾内存指标”。

1. 在“指标命名空间”下拉列表中，选择“来宾(经典)”条目 。
1. 在“指标”下拉列表中，查看可用指标的列表，并注意它们包含一系列与内存和逻辑磁盘相关的指标。

   >注意：此列表包括仅依赖于主机级监视时不可用的其他来宾级指标。

1. 使用“指标命名空间”下拉列表中的选项，按照 Azure 门户中提供的说明启用新的来宾内存指标。
1. 浏览回“az801l09-vm0 \| 指标”页的默认图表，请注意，此时除了“虚拟机主机”和“来宾(经典)”条目外，“指标命名空间”下拉列表还包括“虚拟机来宾”条目    。

   >注意：可能需要刷新“虚拟机来宾”条目的页面才能显示 。

1. 在“az801l09-vm0 \| 指标”页的左侧垂直菜单中，在”监视”部分选择“日志”  。
1. 在“az801l09-vm0 \| 日志”页上，对你之前在此实验室中创建的 Log Analytics 工作区启用日志收集。
1. 浏览到“az801l09-vm0 \| 见解”页面，启用 Azure Monitor VM 见解功能。

   >注意：VM Insights 是一种 Azure Monitor 的解决方案，可帮助监视 Azure VM 和运行 Windows 或 Linux 的本地计算机的性能和运行状况。

1. 在 SEA-SVR2 上，浏览到“监视 \| 虚拟机”页，并升级之前在此实验室中创建的工作区的性能和映射功能   。

   >注意：此选项使用运行状况模型来启用监视和警报功能，此模型包含使用适用于 VM 的 Azure Monitor 发出的指标而生成的运行状况监视器层次结构。

## <a name="exercise-4-evaluating-monitoring-services"></a>练习 4：评估监视服务

本练习的主要任务是：

1. 查看 Azure Monitor 监视和警报功能。
1. 查看 Azure Monitor VM 见解功能。
1. 查看 Azure Log Analytics 功能。

#### <a name="task-1-review-azure-monitor-monitoring-and-alerting-functionality"></a>任务 1：查看 Azure Monitor 监视和警报功能

1. 在 SEA-SVR2 上，在 Azure 门户中，回到“监视 \| 指标”边栏选项卡 。
1. 将范围设置为 az801l09-vm0 虚拟机。 

   >注意：其中的视图和选项与“az801l09-vm0 \| 指标”页中提供的相同 。

1. 在“az801l09-vm0 \| 指标”页上，显示说明 az801l09-vm0 的 CPU 平均使用率的图表 。
1. 在“监视 \| 指标”页上的“az801l09-vm0 的平均 CPU 百分比”窗格中，使用以下信号逻辑设置为 az801l09-vm0 的平均 CPU 利用率创建新的警报规则  ：

   | 设置 | 值 |
   | --- | --- |
   | 阈值 | **静态** |
   | 运算符 | 大于 |
   | 聚合类型 | **平均值** |
   | 阈值 | **2** |
   | 聚合粒度（期限） | **1 分钟** |
   | 评估频率 | **每 1 分钟** |

   >注意：来宾（经典）指标命名空间中的指标不支持从指标创建警报规则。 这可以通过使用“Azure 资源管理器”模板来完成，如[使用 Windows 虚拟机的资源管理器模板将来宾 OS 指标发送到 Azure Monitor 指标存储](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/collect-custom-metrics-guestos-resource-manager-vm)文档中所述。

1. 使用以下设置为新警报规则创建操作组（其他设置保留默认值）并选择“下一步：通知 >”：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0902-RG** |
   | 操作组名称 | **az801l09-ag1** |
   | 显示名称 | **az801l09-ag1** |
   | 通知类型 | **电子邮件/短信/推送/语音** |
   | 通知名称 | **admin email** |
   | 通知电子邮件地址 | 你的电子邮件地址 | 

1. 根据以下设置配置警报规则详细信息（其他设置保留默认值）：

   | 设置 | 值 |
   | --- | --- |
   | 预警规则名称 | **CPU 百分比高于测试阈值** |
   | 说明 | **CPU 百分比高于测试阈值** |
   | 资源组 | **AZ801-L0902-RG** |
   | 严重性 | **Sev 3** |
   | 创建时启用规则 | **是** |

   >注意：指标警报规则可能需要 10 分钟才能激活。

1. 在 Azure 门户中，浏览到 az801l09-vm0 虚拟机页。
1. 在“az801l09-vm0”页上的“操作”部分中，使用“运行命令”RunPowerShellScript 功能运行以下命令，这些命令应该会增加目标操作系统内的 CPU 使用率，并基于新配置的规则触发警报   ：

   ```powershell
   $vCpuCount = Get-WmiObject Win32_Processor | Select-Object -ExpandProperty NumberOfLogicalProcessors
   ForEach ($vCpu in 1..$vCpuCount){ 
      Start-Job -ScriptBlock{
         $result = 1;
         ForEach ($loopCount in 1..2147483647){
            $result = $result * $loopCount
         }
      }
   }
   ```

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开另一个选项卡，浏览到 Azure Monitor 的“警报”页 。
1. 查看警报列表，并显示所创建的警报规则生成的“严重性 3”警报的详细信息。

   >注意：可能需要等待几分钟并选择“刷新” 。

#### <a name="task-2-review-azure-monitor-vm-insights-functionality"></a>任务 2：查看 Azure Monitor VM 见解功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回 az801l09-vm0 虚拟机页 。
1. 在 az801l09-vm0 虚拟机页的左侧垂直菜单中，在“监视”部分选择“见解”  。
1. 在“az801l09-vm0 \| 见解”页上的“性能”选项卡上，查看默认指标集，包括逻辑磁盘性能、CPU 利用率、可用内存以及发送和接收的字节的比率 。
1. 在“az801l09-vm0 \| 见解”页上，选择“映射”选项卡，并查看自动生成的映射 。
1. 在“az801l09-vm0 \| 见解”页上，选择“运行状况”选项卡，并查看其内容 。

   >注意：运行状况信息的可用性取决于工作区升级的完成情况。

#### <a name="task-3-review-azure-log-analytics-functionality"></a>任务 3：查看 Azure Log Analytics 功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回“监视”页并选择“日志”  。
1. 在“选择范围”页上，使用“最近”选项卡将范围设置为“az801l09-vm0”  。
1. 在查询窗口中，运行以下查询并查看生成的图表：

   ```kql
   // Virtual Machine available memory
   // Chart the VM's available memory over the last hour.
   InsightsMetrics
   | where TimeGenerated > ago(1h)
   | where Name == "AvailableMB"
   | project TimeGenerated, Name, Val
   | render timechart
   ```

1. 在工具栏中选择“查询”，在“查询”窗格中，展开“可用性”节点，查看并运行“跟踪 VM 可用性”查询，并查看结果   。
1. 在“新建查询 1”选项卡上，选择“表”标头，并查看“虚拟机”部分中的表列表  。

   >注意：多个表的名称与之前在此实验室中安装的解决方案相对应。 具体而言，Azure VM 见解使用 InsightMetrics 存储性能指标。

1. 将光标移动到 VMComputer 条目上，选择“查看预览数据”图标，并查看结果 。

   >注意：验证数据是否包含 az801l09-vm0 和 SEA-SVR2.contoso.com 的条目  。

   >注意：可能需要等待几分钟，更新数据才可用。

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览到你之前在此实验室中创建的 Log Analytics 工作区的页面。
1. 从工作区页面，浏览到其“解决方案”页。
1. 从“解决方案”页，浏览到“服务映射”解决方案的页面 。
1. 在“ServiceMap”页上的“计算机”选项卡上，选择“SEA-SVR2”以显示服务映射  。
1. 放大查看说明 SEA-SVR2 上的可用网络端口的图，选择不同的端口并查看相应的连接信息。
1. 对于每个连接，在“摘要”和“属性”视图之间切换，后者提供了有关连接目标的更详细的信息 。

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>练习 5：取消预配 Azure 环境

本练习的主要任务是：

1. 在 Cloud Shell 中启动一个 PowerShell 会话。
1. 删除实验室中预配的所有 Azure 资源。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动一个 PowerShell 会话

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格。

#### <a name="task-2-delete-all-azure-resources-provisioned-in-the-lab"></a>任务 2：删除实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L09*'
   ```

   > 注意：验证输出中是否仅包含你在此实验室中创建的资源组。 将在此任务中删除这些组。

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L09*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >注意：该命令异步执行（由 -AsJob 参数确定）。 因此，尽管可以在同一 PowerShell 会话中立即运行另一个 PowerShell 命令，但在实际删除资源组之前将需要几分钟的时间。


#### <a name="review"></a>审阅

在此实验室中，你执行了以下操作：

- 准备监视环境。
- 配置对本地服务器的监视。
- 配置对 Azure VM 的监视。
- 评估监视服务。