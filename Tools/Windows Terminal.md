## oh-my-posh

* 安装到当前用户

```powershell
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
```

* 启动

```powershell
# Start the default settings
Set-Prompt
# Alternatively set the desired theme:
Set-Theme Agnoster
```

* 创建配置文件

```powershell
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
```

* 编辑配置文件

```powershell
notepad $PROFILE
```

并输入：

```powershell
Import-Module posh-git
Import-Module oh-my-posh
Set-Theme Sorin
```

***

## Windows Terminal

* 安装

由微软商店安装

* 配置

```json
"profiles":
    {
        "defaults":
        {
            "fontFace" : "Sarasa Term SC",
            "fontSize" : 14
            // Put settings here that you want to apply to all profiles.
        },
        "list":
        [
            {
                // Make changes here to the powershell.exe profile.
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false,
                "useAcrylic" : true,        //使用亚克力效果
                "acrylicOpacity" : 0.80,    //亚克力背景透明度（需启用useAcrylic）
                "background" : "#012456",   //背景颜色，PS默认为蓝色
                "closeOnExit" : true,       //关闭窗口的时候退出所有挂载的程序
                "colorScheme" : "Dracula",  //配色方案（Dracula需导入）
                "cursorColor" : "#FFFFFF",  //光标颜色
                "cursorShape" : "bar",      //光标形状（默认为bar，即条状）
                "tabTitle" : "Windows PowerShell", //在选项卡上显示的名称
                "padding" : "0, 0, 0, 0",   //内容的边框距，默认填充全部空间
                "snapOnInput" : true,       //输入的时候自动滚动到输入位置
                "startingDirectory" : "%USERPROFILE%" //初始工作目录，默认为用户目录
            },
            ......
            {	// 新增SSH
                "guid": "{e7084f3e-6488-43e0-a5d7-830692b3af5a}",
                "name": "SSH",
                "commandline": "powershell.exe ssh root@45.32.57.255",
                "icon" : "C:\\Everything\\Icons\\SSH.png",
                "hidden": false,
                "useAcrylic" : true,
                "acrylicOpacity" : 0.70,
                "closeOnExit" : true,
                "colorScheme" : "Dracula",
                "cursorColor" : "#FFFFFF",
                "cursorShape" : "bar",
                "tabTitle" : "SSH",
                "padding" : "0, 0, 0, 0",
                "snapOnInput" : true,
                "startingDirectory" : "%USERPROFILE%"
            }
        ]
    },

    // Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
    "schemes": [
        {	// 新增 Dracula 主题
            "name" : "Dracula",
            "background" : "#272935",
            "black" : "#21222C",
            "blue" : "#BD93F9",
            "cyan" : "#8BE9FD",
            "foreground" : "#F8F8F2",
            "green" : "#50FA7B",
            "purple" : "#FF79C6",
            "red" : "#FF5555",
            "white" : "#F8F8F2",
            "yellow" : "#FFB86C",
            "brightBlack" : "#6272A4",
            "brightBlue" : "#D6ACFF",
            "brightCyan" : "#A4FFFF",
            "brightGreen" : "#69FF94",
            "brightPurple" : "#FF92DF",
            "brightRed" : "#FF6E6E",
            "brightWhite" : "#F8F8F2",
            "brightYellow" : "#FFFFA5"
        }
    ],
```

| 设置项                     | 作用                                                         |
| -------------------------- | ------------------------------------------------------------ |
| backgroundImage            | 背景图片                                                     |
| backgroundImageAlignment   | 背景图片的对齐设置                                           |
| backgroundImageOpacity     | 背景图片透明度                                               |
| backgroundImageStretchMode | 背景图片拉伸设置                                             |
| colorScheme                | 使用的字体颜色方案                                           |
| cursorColor                | 光标颜色                                                     |
| cursorShape                | 光标的形状`"vintage"` ( ▃ ), `"bar"` ( ┃ ), `"underscore"` ( ▁ ), `"filledBox"` ( █ ), `"emptyBox"` ( ▯ ) |
| cursorHeight               | 光标的高度（仅当光标为vintage时生效)                         |
| fontFace                   | 字体                                                         |
| fontSize                   | 字体大小                                                     |
| historySize                | 可追述的历史记录数                                           |
| icon                       | 这个Tab显示的logo                                            |
| padding                    | 字体周围的padding，格式为“8, 8, 8, 8”                        |
| snapOnInput                | 设置为True时，在打字时会自动跳转到输入行                     |
| startingDirectory          | 开启终端时的目录                                             |
| tabTitle                   | 这个终端tab的名字                                            |
| useAcrylic                 | 亚克力背景，终端背景会像蒙着一层亚克力，半透明化             |
| acrylicOpacity             | 设置亚克力背景透明度                                         |
| foreground                 | 前景颜色                                                     |
| background                 | 背景颜色                                                     |

Schemes项中的`colors`项的顺序则对应以下的表格：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/windows%20terminal.png)

***

## Sarasa-Gothic 字体

在[Sarasa-Gothic](https://github.com/be5invis/Sarasa-Gothic)中获取，代码常用其中的 Sarasa Term SC字体（sarasa-term-sc-regular.ttf）。

下载压缩包后，直接安装即可。

***

## 常用的命令行参数

* 文件资源管理器中，进入当前路径的 Windows Terminal 终端：

```powershell
wt -d .
```

- 在运行(Win+R)中，使用名为 `SSH` 的 远程登录的终端：

  ```powershell
  wt -p "SSH" -d .
  ```

- 在运行(Win+R)中，垂直分屏打开 PowerShell 与 SSH 终端：

  ```powershell
  wt -p "Windows PowerShell" -d . ; split-pane -V -p "SSH" -d .
  ```

***

参考：

[Windows Terminal 设置]([https://sec.mrfan.xyz/2019/11/08/%E3%80%90%E6%B0%B5%E3%80%91Windows%20Terminal%20%E8%AE%BE%E7%BD%AE/](https://sec.mrfan.xyz/2019/11/08/[氵]Windows Terminal 设置/))，Fanxs