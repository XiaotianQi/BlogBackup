## zsh+oh my zsh

* zsh 的安装：

```bash
查看系统自带哪些 shell:
cat /etc/shells

查看当前环境 shell:
echo $SHELL

安装 zsh:
sudo apt-get install zsh

将 zsh 设置为默认 shell:
chsh -s /bin/zsh
```

* oh-my-zsh

```bash
自动安装:
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh

查看可用的 Theme：
# ls ~/.oh-my-zsh/themes

修改 zsh 主题:
编辑 ~/.zshrc文件，修改ZSH_THEME="candy"。采用的 ys。
```

* zsh 扩展

在 ~/.zshrc 中 `plugins` 关键字，可以自定义启用的插件，系统默认加载git。

* 扩展：zsh-syntax-highlighting

[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

`~/.zshrc`文件中配置：                   

```bash
plugins=(其他的插件 zsh-syntax-highlighting)
```

* 扩展：git-open

[git-open](https://github.com/paulirish/git-open) 插件可以在 git 项目下打开远程仓库浏览项目。                     

```bash
git clone https://github.com/paulirish/git-open.git $ZSH_CUSTOM/plugins/git-open
```

* 扩展：autojump

[autojump](https://github.com/wting/autojump)

```bash
sudo install autojump
```

安装好之后，需要在`~/.zshrc`中配置一下，除了在`plugins`中增加`autojump`之外，还需要添加一行：

```
. /usr/share/autojump/autojump.sh

或者
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

`j --stat`：可以查看历史路径库

***

## tldr

* 安装

```bash
sudo apt-get install nodejs
sudo apt-get install npm
sudo npm install -g tldr
```

* 使用

```bash
tldr  --update
tldr  <commandname>
```

***

## tree

```bash
sudo apt-get install tree
```



