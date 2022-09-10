---
title: 从命令行启动 Windows 应用商店的 APP
date: 2022-09-10T20:37:42+08:00
tags: ['Windows']
categories: ['Windows']
draft: false
---

1. 找到 APP 的名字

- <Win>+<R> 打开运行窗口
- 输入 `shell:AppsFolder`
- 找到你的 APP
- 右键-建立快捷方式
- 系统会提示无法再次建立快捷方式，选择确认然后快捷方式会创建到桌面
- 右键桌面的快捷方式，选择详情，目标栏开头就是 APP 的名字

2. 获取 APP 的 PackageFamilyName 和 InstallLocation

- 打开 PowerShell 
- 输入 `Get-AppxPackage <APPNAME>` 将其中 `<APPNAME>` 换为第一步获得的名字
- 输出中就有 PackageFamilyName 和 InstallLocation

3. 获取 APP 的 ID

- 进入 InstallLocation 的目录
- 打开 InstallLocation 目录下的 AppxManifest.xml 文件
- 全局寻找 `<Application` 并确保那一行有 Id 和 Executable
- ID 即为此行的 Id

4. 拼接命令
最终命令如下,替换其中的 <PackageFamilyName> 及 <ID> 为上文所获得的 PackageFamilyName 和 ID
```
explorer.exe shell:appsFolder\<PackageFamilyName>!<ID>
```

## 参考资料
- [Starting Windows 10 "Store App" from the command line.](https://answers.microsoft.com/en-us/windows/forum/all/starting-windows-10-store-app-from-the-command/836354c5-b5af-4d6c-b414-80e40ed14675)