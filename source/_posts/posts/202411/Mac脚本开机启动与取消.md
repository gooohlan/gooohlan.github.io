---
title: Mac脚本开机启动与取消
toc_number: false
abbrlink: 55538
date: 2024-11-08 11:22:06
updated: 2024-11-08 11:22:06
tags: 
  - Mac
  - Shell
categories: 工具
cover: https://gooohlan.fishpi.cn/img/202411081148690.jpeg
keywords: 
description: Mac脚本开机启动与取消
---

# 开机运行脚本![617af3cb2ee67597c6f8fcc31c99d8a0a58b154b270f66c30306e5d7830ddc2f](https://gooohlan.fishpi.cn/img/header.png)

1. **创建一个脚本文件：**

   首先，在你的主目录下创建一个脚本文件，比如 `set_bclm.sh`：

   ```bash
   vi ~/set_bclm.sh
   ```

   在文件中输入以下内容：

   ```bash
   #!/bin/bash
   echo "your_password" | sudo -S bclm write 80
   ```

   请将 `"your_password"` 替换为你的实际密码。**注意：将密码存储在脚本中可能存在安全风险，请谨慎使用。**

2. **修改脚本权限：**

   使脚本可执行：

   ```bash
   chmod +x ~/set_bclm.sh
   ```

3. **使用 LaunchAgents 自动执行脚本：**

   创建一个 LaunchAgent plist 文件：

   ```bash
   mkdir -p ~/Library/LaunchAgents
   nano ~/Library/LaunchAgents/com.user.setbclm.plist
   ```

   在文件中输入以下内容：

   ```xml
 <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
       <key>Label</key>
       <string>com.user.setbclm</string>
       <key>ProgramArguments</key>
       <array>
           <string>/bin/bash</string>
           <string>/Users/your_username/set_bclm.sh</string>
       </array>
       <key>RunAtLoad</key>
       <true/>
   </dict>
   </plist>
   ```

   请将 `/Users/your_username/set_bclm.sh` 替换为你实际的用户名和脚本路径。

4. **加载 LaunchAgent：**

   使用以下命令加载 LaunchAgent：

   ```bash
   launchctl load ~/Library/LaunchAgents/com.user.setbclm.plist
   ```

这样，每次重启后，`sudo bclm write 80` 就会自动运行。

# 取消脚本

要取消已经设置的开机脚本，你可以按照以下步骤操作：

1. **卸载 LaunchAgent：**

   使用 `launchctl` 卸载 LaunchAgent：

   ```bash
   launchctl unload ~/Library/LaunchAgents/com.user.setbclm.plist
   ```

2. **删除 LaunchAgent 文件：**

   删除对应的 plist 文件：

   ```bash
   rm ~/Library/LaunchAgents/com.user.setbclm.plist
   ```

这样就可以取消开机时自动执行的脚本了。
