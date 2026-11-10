# 下载时 UI 卡顿修复（待 Windows 实测）

适用于本仓库的 .NET Framework 4.8 WPF 启动器，不修改 N_m3u8DL-CLI 下载器本体。

## 改动

- 子进程标准输出改由后台线程持续读取，不在 UI 线程同步检查 `EndOfStream`。
- 日志使用有限队列缓存；界面约 150 ms 批量刷新一次，只显示最新 100 行（最新在最上方）。
- 停止任务后不在 UI 线程执行 `WaitForExit()` 或关闭后台正在使用的输出流；进程退出等待交给后台线程。
- 子进程启动失败或返回非零退出码时，任务状态改为 `Failed`，不再误显示 `Done`。

## Windows 编译

需要 Visual Studio / Build Tools 的 MSBuild、.NET Framework 4.8 Developer Pack 和 NuGet 还原能力。在 Developer Command Prompt 中运行：

```bat
msbuild N_m3u8DL-CLI-SimpleG_List.sln /restore /p:Configuration=Release
```

输出默认位于 `N_m3u8DL-CLI-SimpleG\bin\Release`。测试前请备份原 EXE 和 `config.txt`、`saved_list.json` 等个人数据文件。

## 必测场景

1. 高并发分片下载：下载时拖动窗口、切换任务、滚动列表；日志应约 150 ms 一次刷新，不应每行输出都卡一下。
2. 高速日志输出：日志区始终显示最后约 100 行，最新一行在顶部；内存不随输出行数持续增加。
3. 下载中点「停止」：窗口应能立即继续操作，当前任务显示 `Stopped`，下载进程退出后可重新开始任务。
4. 正常下载完成：任务显示 `Done`，并继续执行队列中的下一个任务。
5. 无效 URL、缺少下载器 EXE、下载器返回非零退出码：任务显示 `Failed`，不应被误标为完成。

此版本已通过源码差异和基本静态检查；尚未在 Windows 上编译，也没有真实下载性能实测数据。
