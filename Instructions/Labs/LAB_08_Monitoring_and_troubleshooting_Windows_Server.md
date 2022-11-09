---
lab:
  title: 实验室：监视 Windows Server 并对其进行故障排除
  module: 'Module 8: Monitoring, performance, and troubleshooting'
---

# <a name="lab-monitoring-and-troubleshooting-windows-server"></a>实验室：监视 Windows Server 并对其进行故障排除

## <a name="scenario"></a>场景

Contoso, Ltd. 是一家全球工程和制造公司，其总部位于美国华盛顿州西雅图市。 IT 办公室和数据中心位于西雅图，以支持西雅图位置和其他位置。 Contoso 最近部署了一个 Windows Server 服务器和客户端基础结构。

由于组织部署了新服务器，因此，为这些新服务器建立具有典型负载的性能基线非常重要。 公司要求你负责此项目。 此外，为了简化监视和故障排除过程，你决定对事件日志执行集中式监视。

                **注意：** 我们提供 **[交互式实验室模拟](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Monitoring%20and%20troubleshooting%20Windows%20Server)** ，让你能以自己的节奏点击浏览实验室。 你可能会发现交互式模拟与托管实验室之间存在细微差异，但演示的核心概念和思想是相同的。 

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 建立性能基线。
- 确定性能问题的根源。
- 查看和配置集中式事件日志。

## <a name="estimated-time-40-minutes"></a>估计时间：40 分钟

## <a name="lab-setup"></a>实验室设置

虚拟机：AZ-801T00A-SEA-DC1 和 AZ-801T00A-SEA-SVR2 必须正在运行 。 其他 VM 可以运行，但本实验室不需要这些 VM。

> 注意： AZ-801T00A-SEA-DC1 和 AZ-801T00A-SEA-SVR2 虚拟机托管 SEA-DC1 和 SEA-SVR2 的安装    。

1. 选择“SEA-SVR2”。
1. 使用以下凭据登录：

   - 用户名：Administrator
   - 密码：Pa55w.rd
   - 域名：CONTOSO

对于本实验室，你将使用可用的 VM 环境。

## <a name="exercise-1-establishing-a-performance-baseline"></a>练习 1：建立性能基线

### <a name="scenario"></a>场景

在此练习中，你将在服务器上使用性能监视器，并使用典型性能计数器创建基线。

本练习的主要任务是：

1. 创建和启动数据收集器集。
1. 在服务器上创建典型工作负载。
1. 分析收集的数据。

> **注意**：启动数据收集器集后，结果可能会延迟 10 分钟显示。

#### <a name="task-1-create-and-start-a-data-collector-set"></a>任务 1：创建和启动数据收集器集

1. 在 SEA-SVR2 上，打开性能监视器。
1. 使用以下设置通过性能监视器创建新的用户定义的数据收集器集：

   - 名称：SEA-SVR2 性能
   - 创建：手动创建（高级）
   - 数据类型：性能计数器
   - 性能计数器（使用默认实例选项）：

      - Memory\\Pages/sec
      - Network Interface\\Bytes Total/sec
      - PhysicalDisk\\% Disk Time
      - PhysicalDisk\\Avg. Disk Queue Length
      - Processor\\% Processor Time
      - System\\Processor Queue Length

   - 采样间隔：1 秒

1. 在性能监视器中，启动 SEA-SVR2 性能数据收集器。 

#### <a name="task-2-create-a-typical-workload-on-the-server"></a>任务 2：在服务器上创建典型工作负载

1. 在 SEA-SVR2 上，以管理员身份启动 Windows PowerShell。
1. 若要模拟典型的工作负载，请在 Windows PowerShell 命令提示符下，运行以下命令：


   ```powershell
   fsutil file createnew bigfile 104857600
   Copy-Item -Path .\bigfile -Destination \\SEA-DC1.contoso.com\c$\ -Force
   Copy-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Destination .\bigfile2 -Force
   Remove-Item -Path .\bigfile* -Force
   Remove-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Force
   ```

1. 让 Windows PowerShell 窗口保持打开状态。

#### <a name="task-3-analyze-the-collected-data"></a>任务 3：分析收集的数据

1. 在 SEA-SVR2 上，切换到性能监视器。
1. 停止 SEA-SVR2 性能数据收集器集。
1. 在性能监视器的导航窗格中，浏览到“报表”、“用户定义”、SEA-SVR2、SEA-SVR2\_DateTime-000001，然后使用“报表”视图查看报表数据   。
1. 记录报表中列出的值，供以后分析之用。 记录的值包括：

   - Memory\\Pages/sec
   - Network Interface\\Bytes Total/sec
   - PhysicalDisk\\% Disk Time
   - PhysicalDisk\\Avg. Disk Queue Length
   - Processor\\% Processor Time
   - System\\Processor Queue Length

### <a name="results"></a>结果

完成本练习后，应已建立用于性能比较的基线。

## <a name="exercise-2-identifying-the-source-of-a-performance-problem"></a>练习 2：确定性能问题的根源

### <a name="scenario"></a>场景

本练习将模拟负载来表示实时使用中的系统，使用数据收集器集收集性能数据，然后确定性能问题的潜在原因。

本练习的主要任务是：

1. 在服务器上创建其他工作负载。
1. 使用数据收集器集捕获性能数据。
1. 删除工作负载并查看性能数据。

#### <a name="task-1-create-additional-workload-on-the-server"></a>任务 1：在服务器上创建其他工作负载

1. 在 SEA-SVR2 上，打开“文件资源管理器”。
1. 在文件资源管理器，浏览到 C:\Labfiles\Lab08 并启动 CPUSTRES64 。
1. 配置第一个突出显示的任务以运行 BUSY (75%)。

   > **注意**：CPUSTRES64.EXE 是一个 SysInternals 实用工具，它通过在一个循环中最多运行 64 个线程来模拟 CPU 活动。

#### <a name="task-2-capture-performance-data-by-using-a-data-collector-set"></a>任务 2：使用数据收集器集捕获性能数据

1. 在 SEA-SVR2 上，切换到性能监视器。
1. 在性能监视器中，浏览到“数据收集器集”和“用户定义”，然后在结果窗格中启动 SEA-SVR2 性能数据收集器集  。

   > **注意**：等待 1 分钟，以便进行数据捕获。

#### <a name="task-3-remove-the-workload-and-review-the-performance-data"></a>任务 3：删除工作负载并查看性能数据

1. 在 SEA-SVR2 上，关闭 CPUSTRES64。
1. 切换到性能监视器。
1. 停止 SEA-SVR2 性能数据收集器集。
1. 在性能监视器的导航窗格中，浏览到“报表”、“用户定义”、SEA-SVR2、SEA-SVR2\_DateTime-000002，然后查看报表数据   。 
1. 记录报表中列出的值：

   - Memory\\Pages/sec
   - Network Interface\\Bytes Total/sec
   - PhysicalDisk\\% Disk Time
   - PhysicalDisk\\Avg. Disk Queue Length
   - Processor\\% Processor Time
   - System\\Processor Queue Length

### <a name="results"></a>结果

完成本练习后，应已使用性能工具来识别潜在的性能瓶颈。

### <a name="exercise-3-viewing-and-configuring-centralized-event-logs"></a>练习 3：查看和配置集中式事件日志

### <a name="scenario"></a>场景

本练习将使用 SEA-SVR2 从 SEA-DC1 收集应用程序事件日志条目 。 

本练习的主要任务是：

1. 配置订阅先决条件。
1. 创建订阅并验证结果。

#### <a name="task-1-configure-subscription-prerequisites"></a>任务 1：配置订阅先决条件

1. 在 SVR2 上，切换到 Windows PowerShell。
1. 若要创建和管理转发到 SEA-SVR2 的事件订阅，请运行以下命令：

   ```powershell
   WECUtil qc /q
   ```

1. 若要确保事件源和收集器同步其本地日期和时间，请运行以下命令：

   ```powershell
   w32tm /resync /computer:SEA-DC1.contoso.com
   ```

1. 若要在出现 Kerberos 身份验证问题时允许 WinRM 连接，运行以下命令：

   ```powershell
   Set-Item WSMan:localhost\client\trustedhosts -Value *.contoso.com -Force
   ```

1. 若要建立与 SEA-DC1 的 PowerShell 远程会话，请运行以下命令：

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. 若要确保在 SEA-DC1 上启用 Windows 远程管理 (WinRM)，请运行以下命令：

   ```powershell
   winrm qc
   ```

   > **注意**：验证 WinRM 服务是否已在运行，并已针对远程管理进行了设置。

1. 若要确保在 SEA-DC1 上启用具有高级安全规则的相关 Windows Defender 防火墙，请运行以下命令：

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Event Log Management' -Enabled True -Profile Domain -PassThru
   ```

   > **注意**：让 Windows PowerShell 窗口保持打开状态。

1. 在 SEA-SVR2 上，打开“Active Directory 用户和计算机” 。
1. 在“Active Directory 用户和计算机”控制台中，将 SVR2 计算机帐户添加到“事件日志读取器”组  。

#### <a name="task-2-create-a-subscription-and-verify-the-results"></a>任务 2：创建订阅并验证结果

1. 在 SVR2上，打开“事件查看器” 。
1. 使用以下属性创建新订阅：

   - 名称：SEA-DC1 事件
   - 计算机：SEA-DC1
   - 收集器：已启动
   - 事件：“严重”、“警告”、“信息”和“错误”   
   - 记录时间：过去 24 小时
   - 日志：应用程序和系统 Windows 日志  

1. 在 SEA-SVR2 上，切换到“事件查看器”窗口，然后在导航窗格中展开“Windows 日志”  。
1. 选择“转发的事件”并验证转发的事件是否包含在 SEA-DC1 上生成的那些事件 。
