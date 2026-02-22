---
title: Win11下MySQL安装和Workbench汉化教程
date: 2025-1-29T20:10:08+08:00
categories: "技术分享"
id: 20251129
cover: /blog/images/posts/my_sql.png
tags:
  - 数据库
  - MySQL
  - 2025
  - 汉化
  - 教程
recommend: false
top: false
hide: false
author: "YEYUbaka"
description: MySQL 8.0.44.0 安装配置、Workbench汉化教程 
showToc: true
---

:::note{type="success"}
最近弄的一个项目需要使用到MySQL，正好记录总结一下Win11的安装汉化工程
:::

# Windows 11 安装 MySQL 8.0.44.0 详细教程

适用版本：Windows 11
MySQL 版本：8.0.44.0（Community Server）
安装方式：MSI 安装程序（图形化界面）

---

### 一、准备工作

下载 MySQL 安装包

1. 打开 MySQL 官方下载页面

2. 选择 “MySQL Community (GPL) Downloads”

3. 在 “MySQL Community Server” 页面，点击 “Looking for previous GA versions?”

4. 在下拉菜单中选择 8.0.44（注意：截至 2025 年，8.0.44 可能已不是最新版）

5. 下载适用于 Windows 的 MSI 安装包（推荐 mysql-installer-community-8.0.44.0.msi）

   > 备份: 通过网盘分享的文件：8.0.44.0MySQL安装包及汉化文件 链接:https://pan.baidu.com/s/1f8slnx_5KGXwhp7lE-IHrA?pwd=hwvw 提取码: hwvw
   > 建议选择 MSI Installer（带图形界面），比 ZIP 压缩包更简单。

### 二、安装步骤

1. 运行安装程序

   - 双击下载好的 .msi 文件（如 mysql-installer-community-8.0.44.0.msi）
   - 若弹出用户账户控制（UAC）提示，点击 “是”
2. 选择安装类型
  在 Choosing a Setup Type 界面：

  - 推荐选择 “Developer Default”（开发者默认）——包含 MySQL Server + Workbench + Shell 等常用工具
  - 如果仅需数据库服务，可选 “Server only”
  - 点击 Next

3. 检查依赖项（Requirements）

   - 安装程序会自动检测系统是否缺少依赖（如 Visual C++ Redistributable）
   - 如有缺失，点击 Execute 自动下载并安装
   - 安装完成后点击 Next
4. 安装产品

   -  Execute 开始安装所选组件
   - 进度条完成（可能需要几分钟）
   - 后点击 Next
### 三、配置MySQL
1. 配置类型（Configuration Type）
	
	- 通常选择 “Development Computer”（开发用）
	
	- 点击 Next
	
2. 连接设置（Connectivity）
	
	- TCP/IP：默认启用（端口 3306）
	
	- 可勾选 “Open firewall port for network access”（如需远程连接）
	
	- 建议保留默认端口 3306
	
	- 点击 Next
	
3. 认证方式（Authentication Method）
	⚠️ **重要选择**！
	
	- 推荐选择 “Use Strong Password Encryption”（即 caching_sha2_password）
	  这是 MySQL 8.0 默认认证插件，兼容性更好
	- 不要选 “Use Legacy Authentication Method”，除非你有旧客户端必须兼容
	    点击 Next
	
4. 配置root密码
    - 强密码（至少 8 位，含大小写字母、数字、符号）
    - 必记住此密码！
    - 击 Add User 添加其他用户（可选）
    -  Next

5. Windows 服务配置

    - 名称默认为 MySQL80
    -  “Start the MySQL Server at System Startup”（开机自启）
    -  Next

6. 应用配置

    -  Execute 应用所有配置

    - 绿色对勾出现

    -  Finish
### 四、验证安装

方法 1：命令行验证
1. 按 Win + R，输入 cmd 打开命令提示符
2. 输入以下命令：
```bash
mysql -u root -p
```
3. 输入你设置的 root 密码
4. 成功进入 MySQL 命令行（出现 mysql> 提示符）即表示安装成功
```bash
mysql> SELECT VERSION();
+-----------+
| VERSION() |
+-----------+
| 8.0.44    |
+-----------+
1 row in set (0.00 sec)
```

方法2：使用 MySQL Workbench（如已安装）
    - 打开 MySQL Workbench
    - 默认会有一个 Local instance MySQL80 连接
    - 双击并输入密码即可连接

### 五、MySQL Workbench
>启动Workbench后，发现为纯英文，对很大一部分人来说并不友好，下面将进行汉化版教程（注意，以下教程只针对于Work bench版本8.0.39及以上的版本）

**汉化MySQL Workbench**

下载我网盘中的汉化文件将其中的**main_menu.xml**文件复制到**C:\Program Files\MySQL\MySQL Workbench 8.0\data**或者你自己的**MySQL Workbench**安装目录，替换即可。怕出错可以先将原来的文件先备份，以免重装
替换后重新打开**Work bench**
汉化完毕
当然，如何大家有其他不懂的，可以看看原文：

[MySQL8 Workbench 中文汉化详细教程（附汉化文件）](https://blog.csdn.net/zp8126/article/details/141359391)

