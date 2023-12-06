---
layout: default
title: Windows
parent: 安装 OpenSearch 控制面板
nav_order: 37
redirect_from: 
  - /dashboards/install/windows/
---

# 在 Windows 上运行 OpenSearch 控制面板

执行以下步骤以在 Windows 上安装 OpenSearch 控制面板。

确保你已安装 zip 实用程序。
{:.note}

1. 下载 [ `opensearch-dashboards-{{site.opensearch_version}}-windows-x64.zip`]（https://artifacts.opensearch.org/releases/bundle/opensearch-dashboards/{{site.opensearch_version}}/opensearch-dashboards--windows-x64{{site.opensearch_version}}.zip） {:target='\_blank'} 存档。

1. 要提取存档内容，请右键单击以选择**全部提取**。
   
   **注意**：某些版本的 Windows 操作系统会限制文件路径长度。如果在解压缩存档时遇到与路径长度相关的错误，请执行以下步骤以启用长路径支持：

   1. 通过在任务栏上输入旁边的**开始**搜索框来 `powershell` 打开 Powershell。
   1. 在 Powershell 中运行以下命令：
      ```bat
      Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem LongPathsEnabled -Type DWORD -Value 1 -Force
      ```
   1. 重新启动计算机。

1. 运行 OpenSearch 控制面板。

   有两种方法可以运行 OpenSearch 控制面板：

   1. 使用 Windows UI 运行批处理脚本：

      1. 导航到 OpenSearch 控制面板安装的顶部目录并打开该 `opensearch-dashboards-{{site.opensearch_version}}` 文件夹。
      1. 如果需要，请修改 `opensearch_dashboards.yml` 位于 `config` 文件夹中的位置，以更改默认的 OpenSearch 控制面板设置。
      1.  `bin` 打开文件夹，然后双击 `opensearch-dashboards.bat` 文件运行批处理脚本。这将打开一个命令提示符，其中正在运行 OpenSearch 控制面板实例。

   1. 从命令提示符或 Powershell 运行批处理脚本：

      1. 通过输入打开命令提示符，或通过在任务栏旁边的**开始**搜索框中输入 `cmd` `powershell` 来打开 Powershell。
      1. 切换到 OpenSearch 控制面板安装的顶部目录。
         ```bat
         cd \path\to\opensearch-dashboards-{{site.opensearch_version}}
         ```
      1. 如果需要，请修改 `config\opensearch_dashboards.yml`.
      1. 运行批处理脚本以启动 OpenSearch 控制面板。
         ```bat
         .\bin\opensearch-dashboards.bat
         ```

要停止 OpenSearch 控制面板，请在命令提示符或 Powershell 中按， `Ctrl+C` 或者直接关闭命令提示符或 Powershell 窗口。
{:.tip}