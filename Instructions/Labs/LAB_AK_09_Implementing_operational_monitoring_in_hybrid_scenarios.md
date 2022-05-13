---
lab:
  title: 实验室：在混合场景中实现操作监视
  type: Answer Key
  module: Module 9 - Implementing operational monitoring in hybrid scenarios
ms.openlocfilehash: 103013bf6732ecd022c20fac9e64ac25cbb0c1d5
ms.sourcegitcommit: e31cab525d98deeb444bd10846cbb38215bcdb29
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/01/2022
ms.locfileid: "141347662"
---
# <a name="lab-answer-key-implementing-operational-monitoring-in-hybrid-scenarios"></a>实验室答案：在混合场景中实现操作监视

## <a name="exercise-1-preparing-a-monitoring-environment"></a>练习 1：准备监视环境

#### <a name="task-1-deploy-an-azure-virtual-machine"></a>任务 1：部署 Azure 虚拟机

1. 连接到 SEA-SVR2，并根据需要，以 CONTOSO\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-SVR2 上，启动 Microsoft Edge，转到 [Azure 门户](https://portal.azure.com)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录 。
1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Azure 门户中的 Cloud Shell 按钮打开“Azure Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。

   > 注意：如果这是你第一次启动 Cloud Shell 且向你显示了“未装载任何存储”消息，请选择要在此实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，并将文件“C:\\Labfiles\\Lab09\\L09-rg_template.json”上传到 Cloud Shell 主目录中  。
1. 重复上一步骤，将“C:\\Labfiles\\Lab09\\L09-rg_template.parameters.json”文件上传到 Cloud Shell 主目录中。
1. 若要创建将托管实验室环境的资源组，请在 Cloud Shell 窗格中的 PowerShell 会话中，输入以下命令，并在输入每个命令后按 Enter（将 `<Azure_region>` 占位符替换为在此实验室中要向其中部署资源的 Azure 区域的名称）：

   >备注：可以使用 (Get-AzLocation).Location 命令列出可用 Azure 区域的名称 ：

   ```powershell 
   $location = '<Azure_region>'
   $rgName = 'AZ801-L0901-RG'
   New-AzResourceGroup -ResourceGroupName $rgName -Location $location
   ```

1. 若要将 Azure 虚拟机 (VM) 部署到新创建的资源组中，请输入以下命令并按 Enter：

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l0901deployment -ResourceGroupName $rgName -TemplateFile ./L09-rg_template.json -TemplateParameterFile ./L09-rg_template.parameters.json -AsJob
   ```

   >注意：请不要等待部署完成，而是继续执行下一个任务。 部署大约需要 3 分钟的时间完成。

#### <a name="task-2-register-the-microsoftinsights-and-microsoftalertsmanagement-resource-providers"></a>任务 2：注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序

1. 若要注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序，在 SEA-SVR2 上，在 Cloud Shell 窗格中输入以下命令，并在输入每个命令后按 Enter。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
   Register-AzResourceProvider -ProviderNamespace Microsoft.AlertsManagement
   ```

   >注意：若要验证注册状态，可以使用 Get-AzResourceProvider cmdlet 。

   >注意：请不要等待注册过程完成，而是继续执行下一项任务。 注册大约需要 3 分钟的时间完成。

1. 关闭 Cloud Shell。

#### <a name="task-3-create-and-configure-an-azure-log-analytics-workspace"></a>任务 3：创建并配置 Azure Log Analytics 工作区

1. 在 SEA-SVR2 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“Log Analytics 工作区”，然后从“Log Analytics 工作区”页中选择“+ 创建”    。
1. 在“创建 Log Analytics 工作区”页的“基本信息”选项卡上，输入以下设置，选择“查看 + 创建”，然后选择“创建”   ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | 新资源组 AZ801-L0902-RG 的名称 |
   | Log Analytics 工作区 | 任何唯一名称 |
   | 区域 | 在上一个任务中将虚拟机部署到的 Azure 区域的名称 |

   >注意：请确保指定在上一个任务中向其中部署了虚拟机的同一区域。

   >注意：请等待部署完成。 部署大约需要 1 分钟的时间完成。

## <a name="exercise-2-configuring-monitoring-of-on-premises-servers"></a>练习 2：配置本地服务器的监视

#### <a name="task-1-register-windows-admin-center-with-azure"></a>任务 1：向 Azure 注册 Windows Admin Center

1. 在 SEA-SVR2 上，选择“开始”，然后选择“Windows PowerShell (管理员)”  。

   >注意：如果尚未在 SEA-SVR2 上安装 Windows Admin Center，请执行后面两个步骤 。

1. 在 Windows PowerShell 控制台中，输入以下命令，然后按 Enter 下载最新版本的 Windows Admin Center：
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 输入以下命令，然后按 Enter 安装 Windows Admin Center：
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > 注意：请等待安装完成。 这大约需要 2 分钟。

   > **注意**：完成 Windows Admin Center 安装后，可能会收到错误 ERR_CONNECTION_REFUSED。 如果发生这种情况，请重启 SEA-SVR2，然后再继续。

1. 在 SEA-SVR2 上，启动 Microsoft Edge，然后浏览至 https://SEA-SVR2.contoso.com 。 
1. 如果出现提示，请在“Windows 安全”对话框中输入以下凭据，然后选择“确定” ：

   - 用户名：CONTOSO\\Administrator
   - 密码：Pa55w.rd

1. 在“所有连接”窗格中，选择页面右上角的“设置”（齿轮）图标 。
1. 在 Windows Admin Center 中，在“设置”页上的“Azure 帐户”部分中，选择“注册 Azure”，然后选择“注册”   。
1. 在“Windows Admin Center 中的 Azure 入门”窗格中，选择“复制”以复制注册过程步骤列表中显示的代码 。 
1. 在注册过程的步骤列表中，选择“输入代码”链接。

   >注意：这将在 Microsoft Edge 窗口中打开显示“输入代码”页的另一个选项卡 。

1. 在“输入代码”文本框中，将复制的代码粘贴到剪贴板，然后选择“下一步” 。
1. 在“登录”页上，提供在上一个练习中用于登录到 Azure 订阅的用户名，选择“下一步”，提供相应的密码，然后选择“登录”  。
1. 出现提示“是否尝试登录到 Windows Admin Center?”时，选择“继续” 。
1. 在 Windows Admin Center 中，验证登录是否成功，并关闭 Microsoft Edge 窗口中新打开的选项卡。
1. 在“Windows Admin Center 中的 Azure 入门”窗格中，确保“Azure Active Directory 应用程序”设置为“新建”，然后选择“连接”   。
1. 在注册过程的步骤列表中，选择“登录”。 这将打开一个标有“请求的权限”的弹出窗口。
1. 在“请求的权限”弹出窗口中，选择“代表你的组织同意”，然后选择“接受”  。

#### <a name="task-2-integrate-an-on-premises-windows-server-with-azure-monitor"></a>任务 2：将本地 Windows Server 与 Azure Monitor 集成

1. 在 SEA-SVR2 上，在显示 Windows Admin Center 的 Microsoft Edge 窗口中，浏览到“所有连接”窗格 。
1. 在“所有连接”窗格中，选择“sea-svr2.contoso.com”条目 。 
1. 在 sea-svr2.contoso.com 页上的“工具”菜单中，选择“Azure Monitor”，然后选择“登录到 Azure 并设置”   。
1. 在“设置 Azure Monitor”页上，指定以下设置，并选择“设置” 。

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0902-RG** |
   | 资源组区域 | 在上一个练习中将虚拟机部署到的 Azure 区域的名称 |
   | Log Analytics 工作区 | 在上一个练习中创建的工作区的名称 |
   | 启用 Azure Arc | 已选定 |

   >注意：请不要等待安装完成，而是继续执行下一个练习。 安装过程大约需要 3 分钟。

   >注意：此过程会自动安装 Log Analytics Agent 和 Dependency Agent。

## <a name="exercise-3-configuring-monitoring-of-azure-vms"></a>练习 3：配置 Azure VM 的监视

#### <a name="task-1-review-host-based-monitoring"></a>任务 1：查看基于主机的监视

1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”页上，选择“az801l09-vm0”  。
1. 在“az801l09-vm0”页上的“监视”部分中，选择“指标”  。
1. 在“az801l09-vm0 \| 指标”页上，在默认图表上的“指标命名空间”下拉列表中，请注意，仅虚拟机主机度量值可用  。

   >注意：这是意料之中的，因为尚未配置来宾级别的诊断设置。 不过，你可以选择直接从“指标命名空间”下拉列表中启用来宾内存度量。 稍后将在本练习中启用它。

1. 在“指标”下拉列表中，查看可用指标的列表。

   >注意：此列表包含可从虚拟机主机收集的一系列 CPU、磁盘和网络相关指标，无需访问来宾级指标。

1. 在“指标”下拉列表中，选择“CPU 百分比”，在“聚合”下拉列表中，确保选择“平均”条目，并查看生成的图表   。

#### <a name="task-2-configure-diagnostic-settings-and-vm-insights"></a>任务 2：配置诊断设置和 VM 见解

1. 在“az801l09-vm0”页上的“监视”部分中，选择“诊断设置”  。
1. 在“az801l09-vm0 \| 诊断设置”页的“概述”选项卡上，选择“启用来宾级别监视”  。

   >注意：请等待操作生效。 这可能需要大约 3 分钟。

1. 切换到“az801l09-vm0 \| 诊断设置”页的“性能计数器”选项卡，并查看可用的计数器 。

   >注意：默认情况下，CPU、内存、磁盘和网络计数器均已启用。 可以切换到“自定义”视图以获取更详细的列表。

1. 切换到“az801l09-vm0 \| 诊断设置”页的“日志”选项卡，并选择“启用来宾级别监视”按钮  。 

   >注意：请等待来宾级监视诊断启用。 这大约需要 3 分钟。

1. 在“az801l09-vm0 \| 诊断设置”页上的“概述”选项卡上，查看可用的事件日志收集选项 。

   >注意：默认情况下，日志集合包含应用程序日志和系统日志中的严重、错误和警告条目，以及安全日志中的审核失败条目。 你可以从“日志”选项卡自定义它们。

1. 在“az801l09-vm0 \| 诊断设置”页上，选择“日志”选项卡并查看可用的配置设置 。
1. 在“az801l09-vm0 \| 日志”页的左侧垂直菜单中，在”监视”部分选择”指标”  。
1. 在“az801l09-vm0 \| 指标”页的默认图表上，请注意，此时除了“虚拟机主机”条目外，“指标命名空间”下拉列表还包括“来宾(经典)”条目   。

   >注意：这是意料之中的，因为已启用来宾级别的诊断设置。 你还可以选择“启用新的来宾内存指标”。

1. 在“指标命名空间”下拉列表中，选择“来宾(经典)”条目 。
1. 在“指标”下拉列表中，查看可用指标的列表，并注意它们包含一系列与内存和逻辑磁盘相关的指标。

   >注意：此列表包括仅依赖于主机级监视时不可用的其他来宾级指标。

1. 在“指标命名空间”下拉列表中，选择“启用新的来宾内存指标”条目 。
1. 在“启用来宾指标(预览)”窗格中，查看提供的信息。
1. 在“az801l09-vm0 \| 诊断设置”页上，选择“接收器”选项卡，在“Azure Monitor (预览)”部分中，选择“已启用”，然后选择“保存”    。
1. 浏览回“az801l09-vm0 \| 指标”页的默认图表，请注意，此时除了“虚拟机主机”和“来宾(经典)”条目外，“指标命名空间”下拉列表还包括“虚拟机来宾”条目    。

   >注意：可能需要刷新页面才会显示“虚拟机来宾”条目 。

1. 在“az801l09-vm0 \| 指标”页的左侧垂直菜单中，在”监视”部分选择“日志”  。
1. 在“az801l09-vm0 \| 日志”页上，选择“启用” 。
1. 在“选择 Log Analytics 工作区”下拉列表中，选择你之前在此实验室中创建的 Log Analytics 工作区，然后选择“启用” 。
1. 在“az801l09-vm0 \| 日志”页的左侧垂直菜单中，在”监视”部分选择”见解”  。
1. 根据需要，在“az801l09-vm0 \| 见解”页上，选择“启用” 。

   >注意：此设置提供 Azure VM 见解功能。 VM 见解是一种 Azure Monitor 解决方案，可帮助监视 Azure VM 和运行 Windows 或 Linux 的本地计算机的性能和运行状况。

1. 在 SEA-SVR2 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“监视”，然后在“监视 \| 概述”页的“见解”下，选择“VM 见解”     。
1. 在“监视 \| 虚拟机”页上，选择“性能”选项卡，然后选择“升级”  。
1. 在“升级”窗格中，选择“升级” 。
1. 在“监视 \| 虚拟机”页上，选择“映射”选项卡，然后选择“立即试用”  。
1. 在“管理覆盖范围”页上，选择“配置工作区” 。
1. 在“Azure Monitor”页上，从“选择 Log Analytics 工作区”下拉菜单中选择之前在此实验室中创建的工作区，然后选择“配置”  。

   >注意：此选项使用运行状况模型来启用监视和警报功能，此模型包含使用适用于 VM 的 Azure Monitor 发出的指标而生成的运行状况监视器层次结构。

## <a name="exercise-4-evaluating-monitoring-services"></a>练习 4：评估监视服务

#### <a name="task-1-review-azure-monitor-monitoring-and-alerting-functionality"></a>任务 1：查看 Azure Monitor 监视和警报功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回“监视 \| 指标”页 。
1. 在“选择范围”页上的“浏览”选项卡上，浏览至“AZ801-L0901-RG”资源组，展开该资源组，选中该资源组中 az801l09-vm0 虚拟机条目旁边的复选框，然后选择“应用”    。

   >注意：如果未在列表中看到 az801l09-vm0，请在“搜索以筛选项...”框中搜索“az801l09-vm0”   。
   
   >注意：其中的视图和选项与“az801l09-vm0 \| 指标”页中提供的相同 。

1. 在“指标”下拉列表中，选择“CPU 百分比”，确保“平均”显示在“聚合”下拉列表中，并查看生成的图表   。
1. 在“监视 \| 指标”页上，在“az801l09-vm0 的平均 CPU 百分比”窗格中选择“新建警报规则”  。

   >注意：来宾（经典）指标命名空间中的指标不支持从指标创建警报规则。 这可以通过使用“Azure 资源管理器”模板来完成，如[使用 Windows 虚拟机的资源管理器模板将来宾 OS 指标发送到 Azure Monitor 指标存储](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/collect-custom-metrics-guestos-resource-manager-vm)文档中所述。

1. 在“创建警报规则”页上的“条件”部分中，选择现有条件条目 。
1. 在“配置信号逻辑”页上，在“警报逻辑”部分的信号列表中，指定以下设置（将其他设置保留为默认值），然后选择“完成”  ：

   | 设置 | 值 |
   | --- | --- |
   | 阈值 | **静态** |
   | 运算符 | 大于 |
   | 聚合类型 | **平均值** |
   | 阈值 | **2** |
   | 聚合粒度（期限） | **1 分钟** |
   | 评估频率 | **每 1 分钟** |

1. 在“创建警报规则”页上的“操作”选项卡上，选择“+ 创建操作组”按钮  。
1. 在“创建操作组”页的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值），然后选择“下一步: 通知 >”  ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
   | 资源组 | **AZ801-L0902-RG** |
   | 操作组名称 | **az801l09-ag1** |
   | 显示名称 | az801l09-ag1 |

1. 在“创建操作组”页的“通知”选项卡上，在“通知类型”下拉列表中，选择“电子邮件/短信/推送/语音”   。 在“名称”文本框中，键入“管理员电子邮件”，然后选择“编辑详细信息”（铅笔）图标  。
1. 在“电子邮件/短信/推送/语音”页上，选中“电子邮件”复选框，在“电子邮件”文本框中键入电子邮件地址，将其他设置保留为默认值，然后选择“确定”   。 返回到“创建操作组”页上的“通知”选项卡，选择“下一步: 操作  >”  。
1. 在“创建操作组”页的“操作”选项卡上，查看“操作类型”下拉列表中的可用项，而不进行任何更改，然后选择“查看 + 创建”   。
1. 在“创建操作组”页的“查看 + 创建”选项卡上，选择“创建”  。
1. 返回到“创建警报规则”页，在“警报规则详细信息”部分，指定以下设置（将其他设置保留为默认值） ：

   | 设置 | 值 |
   | --- | --- |
   | 预警规则名称 | **CPU 百分比高于测试阈值** |
   | 说明 | **CPU 百分比高于测试阈值** |
   | 资源组 | **AZ801-L0902-RG** |
   | 严重性 | **Sev 3** |
   | 创建时启用规则 | **是** |

1. 选择“创建警报规则”。

   >注意：指标警报规则可能需要 10 分钟才能激活。

1. 在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”页上，选择“az801l09-vm0”  。
1. 在 az801l09-vm0 页的“操作”部分，选择“运行命令”，然后选择 RunPowerShellScript   。
1. 在“运行命令脚本”页上的“PowerShell 脚本”部分中，输入以下命令，并选择“运行”以增加目标操作系统内的 CPU 使用率  ：

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

   >注意：这会将 CPU 使用率增加到新创建的警报规则的阈值以上。

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开另一个选项卡，浏览到“监视”页，然后选择“警报”  。
1. 记下“Sev 3”警报的数量，然后选择“Sev 3”行 。

   >注意：可能需要等待几分钟并选择“刷新” 。

1. 在“所有警报”页上，查看生成的警报。

#### <a name="task-2-review-azure-monitor-vm-insights-functionality"></a>任务 2：查看 Azure Monitor VM 见解功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回 az801l09-vm0 虚拟机页 。
1. 在 az801l09-vm0 虚拟机页的左侧垂直菜单中，在“监视”部分选择“见解”  。
1. 在“az801l09-vm0 \| 见解”页上的“性能”选项卡上，查看默认指标集，包括逻辑磁盘性能、CPU 利用率、可用内存以及发送和接收的字节的比率 。
1. 在“az801l09-vm0 \| 见解”页上，选择“映射”选项卡，并查看自动生成的映射 。
1. 在“az801l09-vm0 \| 见解”页上，选择“运行状况”选项卡，并查看其内容 。

   >注意：运行状况信息的可用性取决于工作区升级的完成情况。

#### <a name="task-3-review-azure-log-analytics-functionality"></a>任务 3：查看 Azure Log Analytics 功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回“监视”页并选择“日志”  。

   >注意：如果这是你第一次访问 Log Analytics，可能需要关闭“欢迎使用 Log Analytics”窗格 。

1. 在“选择范围”页上，选择“最近”选项卡，选择“az801l09-vm0”，然后选择“应用”   。
1. 在查询窗口中，粘贴以下查询，选择“运行”，然后查看生成的图表：

   ```kql
   // Virtual Machine available memory
   // Chart the VM's available memory over the last hour.
   InsightsMetrics
   | where TimeGenerated > ago(1h)
   | where Name == "AvailableMB"
   | project TimeGenerated, Name, Val
   | render timechart
   ```

1. 在工具栏中选择“查询”，在“查询”窗格中，展开“可用性”节点，选择“跟踪 VM 可用性”磁贴，选择“运行”按钮，并查看结果    。
1. 在“新建查询 1”选项卡上，选择“表”标头，并查看“虚拟机”部分中的表列表  。

   >注意：多个表的名称与之前在此实验室中安装的解决方案相对应。 具体而言，Azure VM 见解使用 InsightMetrics 存储性能指标。

1. 将光标移动到 VMComputer 条目上，选择“查看预览数据”图标，并查看结果 。

   >注意：验证数据是否包含 az801l09-vm0 和 SEA-SVR2.contoso.com 的条目  。

   >注意：可能需要等待几分钟，更新数据才可用。

1. 在 SEA-SVR2 上，在 Azure 门户的工具栏上的“搜索资源、服务和文档”文本框中，搜索并选择“Log Analytics 工作区”，然后从“Log Analytics 工作区”页中选择表示你之前在此实验室中创建的工作区的条目   。
1. 在工作区页的左侧垂直菜单的“常规”部分中，选择“解决方案” 。
1. 在解决方案列表中，选择“ServiceMap”，然后在“摘要”页上选择“服务映射”磁贴  。
1. 在“ServiceMap”页上的“计算机”选项卡上，选择“SEA-SVR2”以显示服务映射  。
1. 放大查看说明 SEA-SVR2 上的可用网络端口的图，选择不同的端口并查看相应的连接信息。
1. 对于每个连接，在“摘要”和“属性”视图之间切换，后者提供了有关连接目标的更详细的信息 。

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>练习 5：取消预配 Azure 环境

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>任务 1：在 Cloud Shell 中启动 PowerShell 会话

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格 。

#### <a name="task-2-delete-all-azure-resources-provisioned-in-the-lab"></a>任务 2：删除实验室中预配的所有 Azure 资源

1. 在 Cloud Shell 窗格中，运行以下命令，列出你在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L09*'
   ```

   > 备注：验证输出是否仅包含你在此实验室中创建的资源组。 此任务中将删除该组。

1. 运行以下命令，删除在此实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L09*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >注意：该命令异步执行（由 -AsJob 参数确定）。 因此，尽管可以在同一 PowerShell 会话中立即运行另一个 PowerShell 命令，但在实际删除资源组之前将需要几分钟的时间。
