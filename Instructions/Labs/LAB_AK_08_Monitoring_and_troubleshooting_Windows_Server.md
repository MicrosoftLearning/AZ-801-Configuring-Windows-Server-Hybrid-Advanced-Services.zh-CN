---
lab:
  title: 实验室：监视 Windows Server 并对其进行故障排除
  type: Answer Key
  module: 'Module 8: Monitoring, performance, and troubleshooting'
---

# <a name="lab-answer-key-monitoring-and-troubleshooting-windows-server"></a>实验室参考答案：监视 Windows Server 并对其进行故障排除

### <a name="exercise-1-establishing-a-performance-baseline"></a>练习 1：建立性能基线

> 注意：启动数据收集器集后，结果可能会延迟 10 分钟显示。

#### <a name="task-1-create-and-start-a-data-collector-set"></a>任务 1：创建和启动数据收集器集

1. 连接到 SEA-SVR2，然后根据需要，以 CONTOSO\\Administrator 的身份，使用密码 Pa55w.rd 登录  。
1. 在 SEA-SVR2 上，在“开始”按钮旁边的“在此键入搜索内容”文本框中，输入 Perf，然后在“最佳匹配”列表中选择“性能监视器”     。
1. 在性能监视器中，展开导航窗格中的“数据收集器集”节点，然后选择“用户定义” 。
1. 右键单击或访问“用户定义”的上下文菜单，选择“新建”，然后选择“数据收集器集”  。 随即将启动“新建数据收集器集”向导。
1. 在“新建数据收集器集”向导的“希望如何新建数据收集器集?”页中，在“名称”框中输入“SEA-SVR2 性能”   。
1. 选择“手动创建(高级)”选项，然后选择“下一步” 。
1. 在“要包括哪种类型的数据?”页中，选中“性能计数器”复选框，然后选择“下一步”  。
1. 在“要记录哪些性能计数器?”页中，选择“添加” 。
1. 在“可用计数器”列表中，展开“处理器”，选择“处理器时间百分比”，确保在“所选对象的实例”部分中选中“_Total”条目，然后选择“添加”     。
1. 在“可用计数器”列表中，展开“内存”，选择“页数/秒”，然后选择“添加”   。
1. 在“可用计数器”列表中，展开“PhysicalDisk”，选择“磁盘时间百分比”，确保在“所选对象的实例”部分中选中“_Total”条目，然后选择“添加”     。
1. 选择“平均磁盘队列长度”，确保在“所选对象的实例”部分中选中“_Total”条目，然后选择“添加”   。
1. 在“可用计数器”列表中，展开“系统”，选择“处理器队列长度”，然后选择“添加”   。
1. 在“可用计数器”列表中，展开“网络接口”，选择“总字节数/秒”，确保在“所选对象的实例”部分中选中 <All instances> 条目，选择“添加”，然后选择“确定”      。
1. 在“要记录哪些性能计数器?”页中，在“示例间隔”框中输入“1”，然后选择“下一步”   。
1. 在“希望将数据保存到何处?”页中，选择“下一步” 。
1. 在“创建数据收集器集?”页中，确保选中“保存并关闭”选项，然后选择“完成”  。
1. 在性能监视器的结果窗格中，右键单击或访问“SEA-SVR2 性能”的上下文菜单，然后选择“开始” 。

#### <a name="task-2-create-a-typical-workload-on-the-server"></a>任务 2：在服务器上创建典型工作负载

1. 在 SEA-SVR2 上，选择“开始”，右键单击或访问“Windows PowerShell”的上下文菜单，然后选择“以管理员身份运行”  。
1. 若要创建特定大小的文件，请在 Windows PowerShell 命令提示符下，输入以下命令并按 Enter：

   ```powershell
   fsutil file createnew bigfile 104857600
   ```

1. 若要将新创建的文件复制到 SEA-DC1 C 盘的根目录，请输入以下命令并按 Enter ：

   ```powershell
   Copy-Item -Path .\bigfile -Destination \\SEA-DC1.contoso.com\c$\ -Force
   ```

1. 若要将文件从 SEA-DC1 C 盘的根目录复制到当前的工作目录，请输入以下命令并按 Enter ：

   ```powershell
   Copy-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Destination .\bigfile2 -Force
   ```

1. 若要删除当前工作目录中的这两个文件，请输入以下命令并按 Enter：

   ```powershell
   Remove-Item -Path .\bigfile* -Force
   ```

1. 若要删除 SEA-DC1 C 盘根目录下的文件，请输入以下命令并按 Enter ：

   ```powershell
   Remove-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Force
   ```

1. 让 Windows PowerShell 窗口保持打开状态。

#### <a name="task-3-analyze-the-collected-data"></a>任务 3：分析收集的数据

1. 在 SEA-SVR2 上，切换到性能监视器。
1. 在导航窗格中，右键单击或访问“SEA-SVR2 性能”的上下文菜单，然后选择“停止” 。
1. 在性能监视器的导航窗格中，依次展开“报表”、“用户定义”、“SEA-SVR2 性能”，选择“SEA-SVR2\_DateTime-000001”（其中 DateTime 指定当前的时间戳），然后查看报表数据   。
1. 在菜单栏上，选择“更改图形类型”或按 Ctrl+G，然后选择“报表” 。
1. 记录报表中列出的值，供以后分析之用。 记录的值包括：

   - Memory\\Pages/sec
   - Network Interface\\Bytes Total/sec
   - PhysicalDisk\\% Disk Time
   - PhysicalDisk\\Avg. Disk Queue Length
   - Processor\\% Processor Time
   - System\\Processor Queue Length

### <a name="exercise-2-identifying-the-source-of-a-performance-problem"></a>练习 2：确定性能问题的根源

#### <a name="task-1-create-additional-workload-on-the-server"></a>任务 1：在服务器上创建其他工作负载

1. 在 SEA-SVR2 上，打开“文件资源管理器”。
1. 在文件资源管理器中，浏览到 C:\Labfiles\Lab08。
1. 双击或选择“CPUSTRES64.EXE”，然后按 Enter。

   > 注意：CPUSTRES64.EXE 是一个 SysInternals 实用工具，它通过在一个循环中最多运行 64 个线程来模拟 CPU 活动 。

1. 在“CPUSTRES 许可协议”对话框中，选择“同意” 。
1. 在“CPU 压力”对话框中，右键单击或访问正在运行的线程列表顶部突出显示的线程的上下文菜单，选择“活动级别”，然后选择“忙碌(75%)”  。

#### <a name="task-2-capture-performance-data-by-using-a-data-collector-set"></a>任务 2：使用数据收集器集捕获性能数据

1. 在 SEA-SVR2 上，切换到性能监视器。
1. 在性能监视器中，展开“数据收集器集”，然后选择“用户定义” 。
1. 在结果窗格中，右键单击或访问“SEA-SVR2 性能”的上下文菜单，然后选择“开始” 。

   > 注意：等待 1 分钟，以便进行数据捕获。

#### <a name="task-3-remove-the-workload-and-review-the-performance-data"></a>任务 3：删除工作负载并查看性能数据

1. 在 SEA-SVR2 上，切换到“CPU 压力”对话框并关闭它 。 
1. 切换到性能监视器。
1. 在导航窗格中，右键单击或访问“SEA-SVR2 性能”的上下文菜单，然后选择“停止” 。
1. 在性能监视器的导航窗格中，依次展开“报表”、“用户定义”、“SEA-SVR2 性能”，选择“SEA-SVR2\_DateTime-000002”（其中 DateTime 指定当前的时间戳），然后查看报表数据   。
1. 在菜单栏上，选择“更改图形类型”或按 Ctrl+G，然后选择“报表” 。
1. 和之前一样，记录报表中列出的值。 记录的值包括：

   - Memory\\Pages/sec
   - Network Interface\\Bytes Total/sec
   - PhysicalDisk\\% Disk Time
   - PhysicalDisk\\Avg. Disk Queue Length
   - Processor\\% Processor Time
   - System\\Processor Queue Length

### <a name="exercise-3-viewing-and-configuring-centralized-event-logs"></a>练习 3：查看和配置集中式事件日志

#### <a name="task-1-configure-subscription-prerequisites"></a>任务 1：配置订阅先决条件

1. 在 SEA-SVR2 上，切换到 Windows PowerShell。
1. 若要创建和管理转发到 SEA-SVR2 的事件订阅，请输入以下命令并按 Enter：

   ```powershell
   WECUtil qc /q
   ```

1. 若要确保事件源和收集器同步其本地日期和时间，请输入以下命令并按 Enter：

   ```powershell
   w32tm /resync /computer:SEA-DC1.contoso.com
   ```

1. 若要在出现 Kerberos 身份验证问题时允许 WinRM 连接，请输入以下命令并按 Enter：

   ```powershell
   Set-Item WSMan:localhost\client\trustedhosts -Value *.contoso.com -Force
   ```

1. 若要建立与 SEA-DC1 的 PowerShell 远程会话，请输入以下命令并按 Enter：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. 若要确保在 SEA-DC1 上启用 Windows 远程管理 (WinRM)，请输入以下命令并按 Enter：

   ```powershell
   winrm qc
   ```

   > 注意：验证 WinRM 服务是否已在运行，并已针对远程管理进行了设置。

1. 若要确保在 SEA-DC1 上启用具有高级安全规则的相关 Windows Defender 防火墙，请输入以下命令并按 Enter：

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Event Log Management' -Enabled True -Profile Domain -PassThru
   ```

   > 注意：让 Windows PowerShell 窗口保持打开状态。

1. 在 SEA-SVR2 上，在“开始”按钮旁边的“在此键入搜索内容”文本框中，输入“可用”，然后在“最佳匹配”列表中选择“Active Directory 用户和计算机”     。
1. 在“Active Directory 用户和计算机”控制台中，选择“Builtin”容器 。 
1. 在“Builtin”容器中，选择“事件日志读取者”组 。
1. 右键单击或访问“事件日志读取者”组的上下文菜单，选择“属性”，然后在“事件日志读取器属性”对话框中，选择“成员”选项卡   。
1. 在“成员”选项卡上，选择“添加”。
1. 在“选择用户、联系人、计算机、服务帐户或组”对话框中，选择“对象类型” 。
1. 在“对象类型”对话框中，选中“计算机”复选框，然后选择“确定”  。
1. 在“选择用户、计算机、服务帐户或组”对话框中，在“输入要选择的对象名称”框中输入 SEA-SVR2，然后选择“确定”   。
1. 在“事件日志读取者属性”对话框中，选择“确定” 。

#### <a name="task-2-create-a-subscription-and-verify-the-results"></a>任务 2：创建订阅并验证结果

1. 在 SEA-SVR2 上，在“开始”按钮旁边的“在此键入搜索内容”文本框中，输入“事件”，然后在“最佳匹配”列表中选择“事件查看器”     。
1. 在“事件查看器”中，选择导航窗格中的“订阅” 。
1. 右键单击或访问“订阅”的上下文菜单，然后选择“创建订阅” 。
1. 在“订阅属性”对话框中，在“订阅名称”框中输入“SEA-DC1 事件”  。
1. 确保选中“收集器发起的”选项，然后选择“选择计算机” 。
1. 在“计算机”对话框中，选择“添加域计算机” 。
1. 在“选择计算机”对话框中，在“输入要选择的对象名称”框中输入 SEA-DC1，然后选择“确定”   。
1. 在“计算机”对话框中，选择“确定” 。
1. 在“订阅属性 - SEA-DC1 事件”对话框中，选择“选择事件” 。
1. 在“已记录”下拉列表中，选择“最近 24 小时” 。
1. 在“查询筛选器”对话框中，选中“关键”、“错误”、“警告”和“信息”复选框    。
1. 在“事件日志”下拉列表中，展开“Windows 日志”，然后选择“系统”和“应用程序”复选框   。
1. 在“查询筛选器”对话框中，选择“确定” 。
1. 返回“订阅属性 - SEA-DC1 事件”对话框，选择“确定” 。
1. 在 SEA-SVR2 上，切换到“事件查看器”窗口，然后在导航窗格中展开“Windows 日志”  。
1. 选择“转发的事件”并验证转发的事件是否包含在 SEA-DC1 上生成的那些事件 。
