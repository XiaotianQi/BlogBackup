## cmder

Windows 中使用方便的命令行。

```text
# 设置环境变量:系统变量--新建
变量名:CMDER_HOME
变量值:C:\everything\cmder

# 设置环境变量:系统变量--Path--编辑
%CMDER_HOME%

# 添加右键菜单:以管理员权限打开cmd
Cmder.exe /REGISTER ALL

# 添加中文支持:Cmder设置--环境
set LANG=zh_CN.UTF-8
alias ll=ls -al --show-control-chars --color $*
```

------

## 

清屏：

clear命令，或者 ctrl+l



ctrl + a：光标跳至起始

ctrl + e：光标跳至结尾



创建并切换至文件夹

mkdir testdir && cd testdir