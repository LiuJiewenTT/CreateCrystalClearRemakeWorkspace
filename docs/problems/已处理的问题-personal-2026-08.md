# 已处理的问题（个人主机） - 2026-08

> 本文档记录仅在本机环境下遇到、不具有普适性的问题。

## 1. 本机开发环境加载脚本缺失

**日期**：2026-08-22

**问题描述**：本机 Java 21 和 GCC 未加入系统 PATH，每次执行 Gradle 或编译命令前需手动配置环境变量。PowerShell 会话级环境变量无法跨会话持久化。

**修复**：创建 `keep_local/init/dev-env.ps1` 脚本，通过 dot-source 加载，临时设置当前会话的 JAVA_HOME 和 PATH，不污染系统环境变量。

**用法**：
```powershell
. "keep_local/init/dev-env.ps1"
```

**文件**：`keep_local/init/dev-env.ps1`