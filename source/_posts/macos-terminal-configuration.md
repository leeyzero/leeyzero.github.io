---
title: 配置macOS终端环境
date: 2022-08-07 23:30:40
categories:
- 开发工具
tags:
- 开发环境
- macOS
- iTerm2
- oh-my-zsh
- powerlevel10k
---

最近换了一台mac，配置了一下终端（terminal）环境，在此记录一下，以便后续查阅，同时给分享给网友作为参考。本文不会细无具细，只会列举出主要步骤和相关配置参考资料。主要包括以下四个部分：

- 安装 [iTerm2](https://iterm2.com/)
- 安装 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
- 配置 [powerlevel10k](https://github.com/romkatv/powerlevel10k#extremely-customizable)
- 配置插件

终端效果:
![terminal](/images/macOS-terminal-configuration/terminal.png)

<!--more-->

vim效果:
![vim](/images/macOS-terminal-configuration/vim.png)

## 安装iTerm2

如果你安装了[homebrew](https://brew.sh/)的话，可以使用brew安装：

```shell
$ brew install --cask iterm2
```

你也可以在iTerm2官网[下载](http://www.iterm2.com/downloads.html)安装。

安装好iTerm2后，可以配置你喜欢的主题：

```plaintext
iTerm → preferences → profiles → colors → load presets
```

也可以在[iterm2colorschemes](https://iterm2colorschemes.com/)找到更多主题。我使用的是GitHub Dark。

## 安装oh-my-zsh

使用curl安装：

```shell
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

安装好后，编辑`~/.zshrc`更改配置。可以在[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)中找到更多配置项说明。

**安装字体补丁**

1. 下载以下4个字体文件：
- [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
- [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
- [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
- [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

2. 点击每个字体文件，逐一安装。字体文件将会安装到系统中。
3. 在终端中使用字体

```plaintext
open iTerm2 → Preferences → Profiles → Text and set Font to MesloLGS NF.
```

## 配置powerlevel10k

下载powerlevel10k：

```shell
$ git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```

编辑`~/.zshrc`，设置`ZSH_THEME="powerlevel10k/powerlevel10k"`。配置完成后，重启iTerm2终端，你将看到powerlevel10k的配置引导，将引导一步步完成配置。
powerlevel10k提供很多配置项，更多配置项说明请参考[这里](https://github.com/romkatv/powerlevel10k#extremely-customizable)。

## 配置插件

zsh有很多插件可配置，在此我只配置了以下两个插件：
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md#oh-my-zsh)
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

clone上述两个插件至本地：

```shell
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在`~/.zshrc`中添加插件：

```shell
plugins=( 
    # other plugins...
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```

## 参考资料

[1] [iterm2-solarized_instructions](https://gist.github.com/kevin-smets/8568070)
[2] [Configuration of a beautiful (efficient) terminal and prompt on OSX in 7minutes](https://medium.com/@Clovis_app/configuration-of-a-beautiful-efficient-terminal-and-prompt-on-osx-in-7-minutes-827c29391961#id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjE1NDllMGFlZjU3NGQxYzdiZGQxMzZjMjAyYjhkMjkwNTgwYjE2NWMiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJuYmYiOjE2NTk3NjQ0MTYsImF1ZCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsInN1YiI6IjExNTQ5OTE5Nzc2MjA0Nzg2Njk4NSIsImVtYWlsIjoic3VjdW55dW5AZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImF6cCI6IjIxNjI5NjAzNTgzNC1rMWs2cWUwNjBzMnRwMmEyamFtNGxqZGNtczAwc3R0Zy5hcHBzLmdvb2dsZXVzZXJjb250ZW50LmNvbSIsIm5hbWUiOiJZb25nIExlZSIsInBpY3R1cmUiOiJodHRwczovL2xoMy5nb29nbGV1c2VyY29udGVudC5jb20vYS0vQUZkWnVjcVNnT1BRUmtubnBNUGZFd2daZk95NXE4T3hBTUVZZjFfdy1OSFQ9czk2LWMiLCJnaXZlbl9uYW1lIjoiWW9uZyIsImZhbWlseV9uYW1lIjoiTGVlIiwiaWF0IjoxNjU5NzY0NzE2LCJleHAiOjE2NTk3NjgzMTYsImp0aSI6IjExOWMwNjMyOGQ2MDg4NjFlZGEzOWI1MGRhYzNlNTg3NmViMWFmNDIifQ.QMm3HvMaZ_W_kpefHdp9zlLwr5kb2jp8vGks8NAGraxm98N2BuUteiJAdxhNkBMRxCV3jgsEZb1HOAw8hbpxr5RK0OoZZuOlipcWFpIsciXVaLOM_ni-9lK9091-0e8bYEypU7zCLi30OUFlrSBo5ZPPZP4cnwl7biPcUUqO3H45If1g3e3zFSlWQoFZk4Wi2keq3GhIFbosdtKo8OK18syyZdeq3v1hYUTpSa1ZQSKRSBbcvObiwOdKPXoJwbQKHBJlMY-C-rfxpxIWtZqW_fXxPqS2mMwtOfpafvOdtJGlzGpQLkvWyi2eGnEAlqPMuyAIXV4gEhRPgto6toS5SQ)
