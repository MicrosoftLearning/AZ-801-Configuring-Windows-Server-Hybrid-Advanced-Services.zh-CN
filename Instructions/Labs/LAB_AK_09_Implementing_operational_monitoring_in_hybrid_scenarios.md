---
lab:
  title: 实验室：在混合场景中实现操作监视
  type: Answer Key
  module: Module 9 - Implementing operational monitoring in hybrid scenarios
---

# 实验室答案：在混合场景中实现操作监视

## 练习 1：准备监视环境

#### 任务 1：部署 Azure 虚拟机

1. 连接到 **SEA-SVR2**，如果需要，请使用讲师提供的凭据登录。
1. 在 SEA-SVR2 上，启动 Microsoft Edge，转到 Azure 门户 (`https://portal.azure.com/`)，然后使用具有要在此实验室中使用的订阅的“所有者”角色的用户帐户的凭据登录。
1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Azure 门户中的 Cloud Shell 按钮打开“Azure Cloud Shell”窗格。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。

   > **备注：** 如果这是第一次启动 Cloud Shell，且显示“**未装载任何存储**”消息，请选择要在此实验室中使用的订阅，然后选择“**应用**”。

1. 在 Cloud Shell 窗格的工具栏中，选择“上传/下载文件”图标，在下拉菜单中选择“上传”，并将文件“C:\\Labfiles\\Lab09\\L09-rg_template.json”上传到 Cloud Shell 主目录中  。
1.  重复上一步骤，将 **C:\Labfiles\Lab09\L09-rg_template.parameters.json** 文件上传到 Cloud Shell 主目录中。
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
1. 出现提示时，插入讲师提供的凭据。

   >注意：请不要等待部署完成，而是继续执行下一个任务。 部署大约需要 3 分钟的时间完成。

#### 任务 2：注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序

1. 若要注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序，在 SEA-SVR2 上，在 Cloud Shell 窗格中输入以下命令，并在输入每个命令后按 Enter。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
   Register-AzResourceProvider -ProviderNamespace Microsoft.AlertsManagement
   ```

   >注意：若要验证注册状态，可以使用 Get-AzResourceProvider cmdlet 。

1. 关闭 Cloud Shell。

   >注意：请不要等待注册过程完成，而是继续执行下一项任务。 注册大约需要 3 分钟的时间完成。

#### 任务 3：创建并配置 Azure Log Analytics 工作区

1. 在 SEA-SVR2 上，在 Azure 门户的工具栏中的“搜索资源、服务和文档”文本框中，搜索并选择“Log Analytics 工作区”，然后从“Log Analytics 工作区”页中选择“+ 创建”    。
1. 在“创建 Log Analytics 工作区”页的“基本信息”选项卡上，输入以下设置，选择“查看 + 创建”，然后选择“创建”   ：

   | 设置 | 值 |
   | --- | --- |
   | 订阅 | **你在此实验室中使用的 Azure 订阅的名称** |
   | 资源组 | 新资源组 AZ801-L0902-RG 的名称 |
   | Log Analytics 工作区 | **任何唯一名称** |
   | 区域 | **在上一个任务中，将虚拟机部署到的 Azure 区域的名称** |

   >备注：请等待部署完成。 部署大约需要 1 分钟的时间完成。

## 练习 2：配置本地服务器的监视

#### 任务 1：安装 Azure Connect Machine 代理

1. 在  **SEA-SVR2** 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在 Azure 门户的搜索栏中，键入  **ARC**，然后选择  **Azure Arc**。 
1. 在“ **Azure Arc 资源**”下的导航窗格中选择“ **计算机**”
1.  选择“ **+ 添加/创建**”，然后在下拉列表中选择“ **添加计算机**”。 
1.  在“**添加单一服务器**”部分选择“ **生成脚本**” 。 
1.  在“ **使用 Azure Arc 添加服务器**” 页中，在“ **项目详细信息**”下，选择之前创建的资源组 (**AZ801-L0901-RG**)。 
1.  在**服务器详细信息**下，选择在上一个任务中部署虚拟机的 Azure 区域的名称。
1.  查看 SQL Server 和连接选项。 取消选中“**连接 SQL Server**”，接受剩余的默认值，然后选择“ **下一步**”。 
1.  在“**标记**”选项卡中，查看默认可用标记，然后选择“ **下一步**”。 
1.  在“**使用 Azure Arc 添加服务器**”选项卡中，向下滚动并选择“ **下载**” 按钮。

      >**备注**：如果浏览器阻止下载，请在 Microsoft Edge 浏览器中允许下载；选择省略号按钮 (…)，然后选择“ **保留**”。 

1. 右键单击“**Windows 开始**”按钮，然后选择“ **Windows PowerShell (管理员)**”。

   >如果收到 UAC 提示，请输入讲师提供的凭据。 
 
1. 键入 **cd C:\Users\Administrator.CONTOSO\Downloads** 或导航到下载脚本的文件夹。  
1. 输入以下命令，更改执行策略：

   ```powershell 
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted
   ```
1. 输入“A”，表示全部“是”，然后按 Enter。 
1. 键入以下命令并按 **Enter**。 

   ```powershell 
   .\OnboardingScript.ps1
   ```
1. 输入“ **R**” 以**运行一次**，然后按 **Enter**（这可能需要几分钟时间）。
安装过程将打开新的 Microsoft Edge 浏览器标签页，以对 Azure Arc 代理进行身份验证。 选择管理员帐户，等待“**身份验证完成**”消息。 返回到 Windows PowerShell 并等待安装完成，然后再关闭窗口。
1. 返回到下载脚本的 Azure 门户页面，然后选择“ **关闭**”。
1. 关闭“ **使用 Azure Arc 添加服务器**” 页，然后导航回“**Azure Arc 计算机**” 页。
1. 选择“ **刷新**”，直到出现 **SEA-SVR2** 服务器名称，并且 ARC 控制台中的状态为“ **已连接**”。

#### 任务 2：使用“见解”启用“监视”

1.  从Azure 门户搜索窗口导航到 Azure Arc，选择 **SEA-SVR2** Azure Arc 计算机，然后打开 **SEA-SVR2** Arc 计算机。
1.  在导航窗格中，在“**监视**”下选择“**见解**”，然后选择“**启用**”。
1.  在“**监视配置**”页的“**数据收集规则**”下，选择“**新建**”。
1.  在“**创建新规则**”页中，输入以下设置，然后选择“**创建**”：

      | 设置 | 值 |
      | --- | --- |
      | **数据收集规则名称** | Arc |
      | **启用进程和依赖项（映射）** | 已启用 |
      | **订阅** | 你在此实验室中使用的 Azure 订阅的名称 |
      | **Log Analytics 工作区** | 在此实验室中创建的 Log Analytics 工作区的名称 |

1.  选择**配置**。

   >**备注**：此部署可能需要几分钟的时间。 等到部署完成，然后继续练习。

#### 任务 3：启用监视和诊断设置

1.  在  **SEA-SVR2** 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在Azure 门户的“搜索”栏中，搜索并选择“ **数据收集规则**”。
1.  在“**数据收集规则**”页中，选择前面创建的 **MSVM1-Arc** 数据收集规则。
1.  在“**配置**”下，选择“**数据源**”，然后选择“**性能计数器**”。
1.  在“**添加数据源**”页的“**基本**”选项卡中，选择**所有**性能计数器。
1.  将每个计数器的采样率更改为 10 秒，然后选择“**保存**”。
1.  在“**监视**”下，选择“**诊断设置**”，然后选择“**+添加诊断设置**”。
1.  键入 **ArcDiagSettings**，作为“**诊断设置**”的名称。
1.  在“**指标**”下，选择“**所有指标**”，然后在“**目的地详细信息**”下，选择“**发送到 Log Analytics工作区**”。
1.  使用此实验室中使用的 Azure 订阅的名称和前面创建的 Log Analytics 工作区。 选择“**保存**”，并关闭“**诊断设置**”窗口。

## 练习 3：评估监视服务

#### 任务 1：查看 Azure Monitor 监视和警报功能

1.  在 **SEA-SVR2** 上，在 Azure 门户中，浏览到“**监视 | 警报**”页。
1.  选择“**+创建**”，然后选择“**预警规则**”。
1.  在“**选择资源**”页的“**浏览**”选项卡中，展开 **AZ801-L0901-RG**，然后选择 **SEA-SVR2** Azure Arc 计算机。 
1. 选择“应用”。
1.  选择“**下一步: 条件>**”，然后在“**创建预警规则**”页的“**选择信号**”下拉列表中，选择“**自定义日志搜索**”。
1.  在查询窗口中，复制并粘贴以下KQL 查询：

      ```kql
      // Chart CPU usage trends by computer
      // Calculate CPU usage patterns over the last hour, chart by percentiles.
      InsightsMetrics
      | where TimeGenerated > ago(1h)
      | where Origin == "vm.azm.ms"
      | where Namespace == "Processor"
      | where Name == "UtilizationPercentage"
      | summarize avg(Val) by bin(TimeGenerated, 5m), Computer //split up by computer
      | render timechart
      ```

1.  选择“**运行**”，并查看“结果”和“**图表**”选项卡中的数据。
1.  选择“继续编辑警报”。
1.  在“**创建预警规则**”页的“**度量**”下，指定以下设置，其他设置保留默认值：

      | 设置 | 值 |
      | --- | --- |
      | **聚合粒度** | 1 分钟 |

1.  在“**警报逻辑**”下，输入以下设置，然后选择“**下一步: 操作 >**。

      | 设置 | 值 |
      | --- | --- |
      | **“运算符”** | 大于 |
      | **阈值** | 10 |
      | **评估频率** | 1 分钟 |

1. 在“**创建预警规则**”页的“**操作**”选项卡中，在“**选择操作**”部分，选择“**使用操作组**”，然后在“**选择操作组**”页中选择“**创建操作组**”。
1. 在“**创建操作组**”页中，输入以下设置，然后选择“**下一步: 通知 >**”。

      | 设置 | 值 |
      | --- | --- |
      | **订阅** | 你在此实验室中使用的 Azure 订阅的名称 |
      | **资源组** | AZ801-L0901-RG |
      | **区域** | 保留默认设置 |
      | **操作组名称** | 任何唯一名称 |
      | **显示名称** | 任何唯一名称（12 个字符或更少） |

1. 在“**创建操作组**”页的“**通知类型**”下，选择“**电子邮件 Azure 资源管理器角色**”，然后在“**名称**”字段中，键入 **“管理员电子邮件**”。
1. 在“**电子邮件 Azure 资源管理器角色**”页中，从下拉列表中选择“**负责人**”，然后单击“**确定**”。
1.  在“**创建操作组**”页中，选择“**查看 + 创建**”，然后选择“**创建**”。

      >注意：指标警报规则可能需要 10 分钟才能激活。

1.  在“**创建预警规则**”页中，选择“**下一步: 详细信息 >**”。
1.  保留默认设置，但在“**预警规则详细信息**”下，键入预警规则名称和说明的“**高 CPU 警报**”。
1.  依次选择“查看 + 创建”、“创建” 。
1.  浏览到 **SEA-SVR2** Azure Arc 计算机，右键单击“**Windows 开始**”按钮并选择“ **Windows PowerShell（管理员）**”。
1. 输入以下命令并按 **Enter**：

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

1.  在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开另一个选项卡，浏览到“监视”页，然后选择“警报”  。

      >**备注**：可能需要等待几分钟，然后刷新“监视器 | 警报”页。

## 练习 4：配置对 Azure VM 的监视

#### 任务 1：配置诊断设置和 VM 见解

1.  在  **SEA-SVR2** 上，在显示 Azure 门户的 Microsoft Edge 窗口中，在Azure 门户的搜索栏中，搜索“**虚拟机**”并选择 **az801l09-vm0**。
1.  在“**az801l09-vm0**”页上的“**监视**”部分下，选择“**诊断设置**”。
1.  在“**诊断设置**”选项卡上，从下拉列表中选择诊断存储帐户，然后选择“**启用来宾级监视**”。

      >注意：请等待操作生效。 这可能需要大约 3 分钟。

1.  切换到“**az801l09-vm0 | 诊断设置**”页的“**性能计数器**”选项卡，并查看可用的计数器 。

      >**备注**：默认情况下，CPU、内存、磁盘和网络计数器均已启用。 可以切换到“**自定义视图**”，以获取更详细的列表。

1.  在“**az801l09-vm0 | 诊断设置**”页上的“**日志**”选项卡上，查看可用的事件日志收集选项。

      >注意：默认情况下，日志集合包含应用程序日志和系统日志中的严重、错误和警告条目，以及安全日志中的审核失败条目。 你可以从“日志”选项卡自定义它们。

#### 任务 2：启用 VM 见解

1.  从左侧垂直菜单的“**监视**”部分，浏览到“**az801l09-vm0 | 见解**”页。
1.  在“**az801l09-vm0 | 见解**”页上，选择“**启用**”。

      >注意：此设置提供 Azure VM 见解功能。 VM 见解是一种 Azure Monitor 解决方案，可帮助监视 Azure VM 和运行 Windows 或 Linux 的本地计算机的性能和运行状况。

1.  在“**监视配置**”页的“**数据收集规则**”下，选择“**新建**”，然后在名称字段中输入 **AZ801vm0**。
1.  在“**进程和依赖项**”下，选择“**启用进程和依赖项（映射）**”。
1.  保留你在此实验室中使用的 Azure 订阅的名称。
1.  从“**Log Analytics 工作区**”下拉菜单中，选择之前创建的 Log Analytics 工作区。
1.  选择“**创建**”，然后选择“**配置**”。

      >注意：此选项使用运行状况模型来启用监视和警报功能，此模型包含使用适用于 VM 的 Azure Monitor 发出的指标而生成的运行状况监视器层次结构。
      此部署可能需要花一些时间。 等待部署完成并返回到进程“映射”，然后结束实验室。 这样就可以查看进程“映射”数据。 

## 练习 5：评估监视服务

#### 任务 1：查看 Azure Monitor 监视和警报功能

1. 在 **SEA-SVR2** 的Azure 门户中，浏览到“**监视\|见解**”页，然后在“**见解**”下选择“**虚拟机**”。 在“**监视器 | 虚拟机**”页上，选择“**性能**”选项卡，应会看到 **CPU/内存**利用率和其他类别。
1.  在“**监视器 | 警报**”页上，依次选择“**+ 创建**”、“**预警规则**”。
1.  在“**选择资源**”页中，展开 **AZ801-L0901-RG** 资源组，选择 **az801l09-vm0**，然后单击“**应用**”。 
1. 选择“**下一步: 条件 >**”。
1.  从“**信号**”下拉列表中选择“**CPU 百分比**”。
1.  在“**警报逻辑**”部分中，指定以下设置（将其他人保留其默认值），然后选择“**下一步: 操作 >**：

      | 设置 | 值 |
      | --- | --- |
      | 阈值类型 | **静态** |
      | 聚合类型 | **平均值** |
      | 值为 | **大于** |
      | 阈值 | **10** |
      | 检查间隔 | **1 分钟** |
      | 回溯期间 | **1 分钟** |

1.  在“**创建预警规则**”页的“**操作**”选项卡上，选择“**+ 创建操作组**”按钮。
1.  在“创建操作组”页的“基本信息”选项卡上，指定以下设置（将其他设置保留为默认值），然后选择“下一步: 通知 >”  ：

      | 设置 | 值 |
      | --- | --- |
      | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
      | 资源组 | **AZ801-L0902-RG** |
      | 操作组名称 | az801l09-ag1 |
      | 显示名称 | **az801l09-ag1** |

1.  在“**创建操作组**”页的“**通知**”选项卡上，在“**通知类型**”下拉列表中，选择“**电子邮件/Azure 资源管理器角色**”。 在“**名称**”文本框中，键入“**管理员电子邮件**”，然后选择“**编辑详细信息**”（铅笔）图标。
1.  在“**电子邮件 Azure 资源管理器角色**”中选择“**参与者**”，单击“**确定**”。
1.  在“**创建操作组**”页中，选择“**查看 + 创建**”，然后选择“**创建**”。

      >注意：指标警报规则可能需要 10 分钟才能激活。

1.  返回到“**创建预警规则**”页，选择“**下一页: 详细信息**”，在“**预警规则详细信息**”部分，指定以下设置（将其他设置保留为默认值）：

      | 设置 | 值 |
      | --- | --- |
      | 资源组 | **AZ801-L0902-RG** |
      | Severity | **3 - 信息性** |
      | 预警规则名称 | **CPU 百分比高于测试阈值** |
      | 说明 | **CPU 百分比高于测试阈值** |
      | 创建时启用 | **是** |

      >**备注**：如果没有看到“**创建时启用**”字段，请展开“**高级选项**”部分。

1.  依次选择“查看 + 创建”、“创建” 。
1.  在 Azure 门户中，搜索并选择“虚拟机”，然后在“虚拟机”页上，选择“az801l09-vm0”  。
1. 在 az801l09-vm0 页的“操作”部分，选择“运行命令”，然后选择 RunPowerShellScript   。
1. 在“**运行命令**”页上，选择 **RunPowerShellScript**，输入以下命令并选择“**运行**”，以提高目标操作系统中的 **CPU利用率**。

   >**备注**：先将脚本复制到记事本，再将其粘贴到脚本窗口中，然后再执行脚本。

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

   >注意：这会将 CPU 使用率增加到新创建的警报规则的阈值以上。 这可能需要几分钟的时间。

1.  在 **SEA-SVR2** 上，在显示 Azure 门户的 Microsoft Edge 窗口中，打开另一个选项卡，浏览到 **az801l09-vm0** 虚拟机页，然后浏览到“**监视 | 警报**”页。 记下“Sev 3”警报的数量，然后选择“Sev 3”行 。

      >注意：可能需要等待几分钟并选择“刷新” 。 如果耗时较长，可以继续进行实验，稍后返回“**监视 | 警报**”页查看数据。

1.  在“**CPU 百分比超过测试阈值**”页上，查看生成的警报。 完成审阅后关闭页面。


#### 任务 2：查看 Azure Monitor VM 见解功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回 az801l09-vm0 虚拟机页 。
1. 在 az801l09-vm0 虚拟机页的左侧垂直菜单中，在“监视”部分选择“见解”  。
1. 在“az801l09-vm0 \| 见解”页上的“性能”选项卡上，查看默认指标集，包括逻辑磁盘性能、CPU 利用率、可用内存以及发送和接收的字节的比率 。
1. 在“az801l09-vm0 \| 见解”页上，选择“映射”选项卡，并查看自动生成的映射 。
1. 在“az801l09-vm0 \| 见解”页上，选择“运行状况”选项卡，并查看其内容 。

   >注意：运行状况信息的可用性取决于工作区升级的完成情况。

#### 任务 3：查看 Azure Log Analytics 功能

1. 在 SEA-SVR2 上，在 Azure 门户中，浏览回“监视”页并选择“日志”  。
1. 在“**新建查询 1**”页中，单击“**选择范围**”。 
1. 在“**选择范围**”页上，选择“**浏览**”选项卡，选择之前在此实验室中创建的唯一工作区，然后选择“**应用**”。
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

1. 在“**新建查询 1**”选项卡上，选择“**表**”标题，然后查看“**Azure 资源**”部分中的表列表。

   >注意：多个表的名称与之前在此实验室中安装的解决方案相对应。 具体而言，Azure VM 见解使用 InsightMetrics 存储性能指标。

1. 将光标移动到 VMComputer 条目上，选择“查看预览数据”图标，并查看结果 。

   >注意：验证数据是否包含 az801l09-vm0 和 SEA-SVR2.contoso.com 的条目  。

   >注意：可能需要等待几分钟，更新数据才可用。

## 练习 6：取消预配 Azure 环境

#### 任务 1：在 Cloud Shell 中启动 PowerShell 会话

1. 在 SEA-SVR2 上，在显示 Azure 门户的 Microsoft Edge 窗口中，通过选择 Cloud Shell 图标打开 Cloud Shell 窗格 。

#### 任务 2：删除实验室中预配的所有 Azure 资源

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
