## 安装自定义软件

### 自定义软件

这里自定义软件定义为只有可执行文件的一个程序，可以是自己编写的、他人提供的、网络下载的。

### 最终效果

1. **自定义的可执行文件存放目录。** 大部分由deb安装的程序总会安装在 **/opt** 目录下，而一些软件只给了一个可执行文件，比如 **.o、 .x86_64** 等各种类型的可执行文件。为了更加清晰可控的管理这些软件，我们需要自己指定一个目录来存放它。
2. **自定义图标**。自定义一个图标并且在 **应用程序菜单** 显示图标，同时可以点击图标运行。
3. **卸载软件直接删除可执行文件即可。**

### 安装方式

以 Godot 为例，官网下载的程序只有一个可执行文件。

1. 我希望可执行文件安装在 `/home/<user_name>/Godot` 中。在 `/home/<user_name>` 目录下创建 Godot 文件夹 `mkdir Godot`，然后进入这个文件夹。
2. 把下载的可执行文件移动到 `/home/<user_name>/Godot`，完成软件的安装。
3. 找一个图标放在 `/home/<user_name>/Godot` 中。
4. 创建一个 .desktop 格式的文件： `gedit Godot.desktop` 在其中完成配置。具体配置方式见下一节。
5. 为 `.desktop` 文件授予执行权限，以便它可以被当作应用程序启动：`sudo chmod +x Godot.desktop` 。
6. 在 `/home/<user_name>/.local/share/applications` 中创建一个指向第4步中创建的文件的软链接：`ln -s /home/<user_name>/Godot/Godot.desktop /home/<user_name>/.local/share/applications/Godot.desktop`。

此时打开应用程序菜单即可看到我们刚刚创建的应用程序图标了，点击即可启动。

### desktop 配置

一般来说至少包含如下的内容：

```
[Desktop Entry]
Version=1.0
Name=Godot
Comment=Godot Game Engine
Exec=/home/<user_name>/Godot/Godot_v4.3-stable_linux.x86_64
Icon=/home/<user_name>/Godot/icon.png
Terminal=false
Type=Application
Categories=Game;Development;
```

- Version: 可有可无，直接放在这里不影响。
- Name：应用程序的名称，显示在启动器中。
- Comment：描述应用程序，可以简要介绍其功能。
- Exec：可执行文件的完整路径。要从根目录开始写完整。
- Icon：应用程序的图标路径，可以是一个图标文件（例如 .png 或 .svg），路径可以是绝对路径或相对路径。
- Terminal：如果应用程序需要在终端中运行，设置为 true。如果不需要终端，设置为 false。
- Type：类型设置为 Application，表示这是一个应用程序。
- Categories：你可以指定应用程序的类别，以便它显示在适当的类别中（例如，Utility、Development 等）。

Categories 可以自己根据程序的情况定义，可以添加多个，每一个都要以英文分号结束。

- Application    基本类别，用于所有应用程序。
- Utility    系统工具和实用程序。
- Development    开发工具，如编程语言、IDE、调试器等。
- Education    教育类应用程序。
- Game    游戏类应用程序。
- Graphics    图形和设计工具。
- Internet    网络应用程序，如浏览器、邮件客户端等。
- Multimedia    多媒体应用程序，如音频、视频播放器、编辑工具等。
- Office    办公软件，如文字处理、电子表格、演示文稿等。
- Science    科学类应用程序，通常用于数据分析、建模、计算等。
- System    系统工具和管理应用程序，如设置、磁盘管理、文件管理等。
- Accessibility    辅助功能应用程序，针对有特殊需求的用户。
- Settings    配置和设置相关的应用程序。
- Hardware    硬件管理和控制工具。
- AudioVideo    与音频和视频相关的工具或应用。
