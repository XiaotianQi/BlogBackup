列举一系列，python使用环境相关。

## pip

```text
pip list --outdate	# 检查更新

pip install --upgrade xxx	# 升级

pip uninstall xx	# 卸载

pip freeze > d:\requirements.txt	# 记录所有第三方模块

pip install -r requirements.txt		# 通过第三方模块安装记录安装模块

pip install xxx -i http://pypi.douban.com/simple/ # 使用豆瓣源安装
```

pip升级所有的已安装Python包：

```text
pip freeze --local | grep -v '^\-e' | cut -d = -f 1  | xargs -n1 pip install -U

pip freeze --local : 列出所有本地安装的python包
grep -v '^-e' : 过滤掉以编辑模式安装的python包
cut -d = -f 1 : 剪切取得安装包名列表
xargs -n1 pip install -U : 把前面得到的软件包列表以参数形式传给pip，进行安装升级，xargs选项-n1是防止一个软件包安装失败后终止安装后面的软件包
```

用python代码来执行升级：

```python
import pip
from subprocess import call

for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

***

## 虚拟环境

虚拟环境是程序执行时的独立执行环境，在同一台服务器中可以创建不同的虚拟环境供不同的系统使用，项目之间的运行环境保持独立性而相互不受影响。

涉及第三方包：

* virtualenv
* virtualenvwrapper（Windows版本：virtualenvwrapper-win）

virtualenvwrapper 提供了一些便捷的命令。

```
# 使用virtualenv创建指定版本虚拟环境
λ virtualenv -p C:\Python\Python37\python.exe env37

# 使用virtualenvwrapper创建指定版本虚拟环境
λ mkvirtualenv --python=C:\Python\Python27\python.exe env27

# 列出所有虚拟环境
λ lsvirtualenv
λ workon

# 进入虚拟环境
λ env27\Scripts\activate	(linux:source env27\Scripts\activate)
λ workon env27

# 退出虚拟环境
(env27) λ deactivate

# 删除虚拟环境
λ rmvirtualenv env27
```

***

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

***

## vscode

使用 setting sync 第三方插件，同步配置信息。

```text
ALT+SHITF+U:上传配置

ALT+SHITF+D:下载配置

需要GitHub-Personal Access Token
以及第一次使用时，插件自身提供的GIST ID
进行认证
```

***

## git

```text
配置git用户名和邮箱
git config --global user.name "用户名"
git config --global user.email "邮箱"

生成ssh key
ssh-keygen -t rsa -C “邮箱”
按3个回车，密码为空。

上传key到github
cat < ~/.ssh/id_rsa.pub
windows 在C:\Users\username\.ssh 文件夹中
Github--Accounting settings--SSH key--Add SSH key

测试是否配置成功
ssh -T git@github.com
输入yes
```



