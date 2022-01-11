## macOS 前端开发环境配置指南

最近公司配的 **MacBook Pro** 不知道为啥抽风了，小风扇一直转，要重装一下系统，然后就是要重新配置我前端的开发环境了，这篇文章记录一下这次配置 **macOS** 的步骤，需要配合我的 [dotfiles](https://github.com/oddii/dotfiles) 仓库使用

## 系统设置

### 更改计算机名称

- **macOS** 会有一个默认的计算机名称，但可以自己去修改，只需要：

  修改 "系统偏好设置" -> "共享" -> "电脑名称"

---

### 触控板

- 设置轻点来点按（单指轻点）：

  勾选 "系统偏好设置" -> "触控板" -> "光标与点按" -> "轻点来点按"

- 不设置滚动方向：自然：

  不勾选："系统偏好设置" -> "触控板" -> "滚动缩放" -> "滚动方向：自然"

---

### Finder

- 设置开启新 **Finder** 窗口时打开 "桌面"：

  修改 "**Finder**" -> "偏好设置" -> "通用" -> "开启新 "**Finder**" 窗口时打开" -> "桌面"

- 显示所有文件扩展名：

  勾选："**Finder**" -> "偏好设置" -> "高级" -> "显示所有文件扩展名"

## 开发环境设置

### Command Line Tools

简单来说 **Command Line Tools** 就是一个小型独立包，为 **mac** 终端用户提供了许多常用的工具，实用编程器和编译器。包括 **svn**，**git**，**make**，**GCC**，**clang**，**perl**，**size**，**strip**，**strings**，**cpp** 以及其他很多能够在 **Linux** 默认安装中找到的有用的命令，可通过以下命令进行安装：

```sh
$ xcode-select --install
```

#### Command Line Tools 安装在哪？

安装在 **mac** 的根目录中的 `/Library/Developer/CommandLineTools/`

> 注意：是在 `/` 根目录下，不是 `~/.` 用户目录

而可用的新命令，都在根目录中的 `/Library/Developer/CommandLineTools/usr/bin/`

可以通过刚安装的命令之一来确定是否安装成功：

```sh
$ gcc -v
$ git version
```

---

### bootstrap

这里的 **bootstrap** 不是前端开发的 `Bootstrap UI`，而是一个终端自动化脚本，直接 `.bootstrap` 执行这个文件，就可以在终端安装我常用的前端开发配置环境和软件

```sh
$ xcode-select --install
$ git clone --recursive https://github.com/oddii/dotfiles.git
# 在执行 Brewfile 之前要使用 Apple Id 登陆 AppStore，并确保该 Apple Id 已购买过需要通过 mas 安装的软件，否则会失败
$ cd dotfiles
$ ./bootstrap
```

> 注意：需要进行科学上网，因为源都是用的国外的，如果速度慢的话可以换个代理节点

#### bootstrap 里面有什么？

- 设置 **HostName**：

  ```sh
  echo "Setup hostname"
  sudo scutil --set HostName odd-macbook
  ```

- 判断是否有安装 **macOS** 的包管理工具 [Homebrew](https://brew.sh/)，如果没有就安装，它可以让你基于命令行安装任何软件：

  ```sh
  echo "Install Homebrew"
  if test ! $(which brew); then
    echo "Installing homebrew..."
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  fi
  ```

- 使用 [Brew Bundle](https://github.com/Homebrew/homebrew-bundle)，它可以帮你解决所有软件依赖，包括官方的 formula 和 cask，甚至还包括 macOS AppStore 中的应用，我们只需要一个 **Brewfile**，就可以配置好所有所需的应用

  **Homebrew** 默认安装了 **Homebrew Bundle**，可以执行 `brew bundle dump` 来生成现有依赖：

  ```sh
  $ brew bundle dump

  brew "yarn"	# 命令行类应用
  ...
  cask "typora" # 非命令行类应用
  ...
  mas "微信", id: 836500024 # macOS AppStore 应用
  ...
  ```

  然后执行 `cat Brewfile` 输出到 **Brewfile**：

  ```sh
  $ cat Brewfile
  ```

  当然也可以自己编写 **Brewfile** 文件，将自己需要的软件名根据 **Brewfile** 的语法添加即可，我自己的 Brewfile 文件已包括一些个人常用的前端软件，还有其他需求的可以根据我的 Brewfile 然后个性化修改

  最后在 **bootstrap** 中会执行 `brew bundle` 即可完成配置应用的自动化安装：

  ```sh
  echo "Install with Brew Bundle"
  set +e
  brew bundle
  set -e
  ```

- 设置 **work** 目录和日常 **demo** 目录（个人习惯）：

  ```sh
  echo "Setup workspace"
  mkdir -p ~/demo
  mkdir -p ~/work
  ```

- 设置 `.gitconfig`：

  ```sh
  echo "Setup Git"
  cp ./git/work.gitconfig ~/work/.gitconfig
  cp ./git/global.gitconfig ~/.gitconfig
  ```

- 安装 [Zsh](https://www.zsh.org/)，并设置为默认的 **shell**：

  ```sh
  echo "Setup Zsh"
  sudo sh -c 'echo /usr/local/bin/zsh >> /etc/shells'
  sudo chsh -s $(which zsh)
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
  cp ./zsh/crispy.zsh-theme ~/.oh-my-zsh/themes/
  cp ./zsh/.zshrc ~/
  ```

- 安装 [OhMyZsh](https://github.com/ohmyzsh/ohmyzsh)：

  ```sh
  echo "Setup ohmyzsh"
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
  ```

- 终端连接 **vscode**，连接后可直接在终端执行 `code ./`，然后将当前目录在 **vscode** 中打开：

  ```sh
  echo "Link editors"
  mkdir -p ~/Applications/
  sudo ln -s /Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code /usr/local/bin/code
  ```

- 安装 [fzf](https://github.com/junegunn/fzf)，它可以在终端快速进行模糊搜索，具体使用方法可以查看官方文档：

  ```sh
  echo "Setup applications"
  echo "- fzf"
  $(brew --prefix)/opt/fzf/install
  ```

---

### Zsh / Oh My Zsh

这两个应用在上文 **Brewfile** 中已经进行了配置安装

#### 什么是 Zsh？

**Z shell**（**Zsh**）是一款可用作交互式登录的 **shell** 及脚本编写的命令解释器。**Zsh** 对 **Bourne shell** 做出了大量改进，同时加入了 **Bash**、**ksh** 及 **tcsh** 的某些功能。

自 2019 年起，**macOS** 的默认 **shell** 已从 **Bash** 改为 **Zsh**

#### 什么是 Oh My Zsh？

**Oh My Zsh** 是一个令人愉快的、开源的、社区驱动的框架，用于管理你的 **Zsh** 配置，它捆绑了数以千计的有用功能、助手、插件、主题，还有一些让你大呼过瘾的东西

它可以安装一些便于开发的插件，如 **zsh-autosuggestions**，**zsh-syntax-highlighting**，**zsh-completions**，当然这些插件安装后都需要进行在 `.zshrc` （**Oh My Zsh** 配置文件）中进行相关配置

#### zshrc 常用配置

- 第三方插件配置：

  ```sh
  # zsh 开发插件配置
  # zsh-autosuggestions 自动补全 需配置信息
  source /usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh

  # zsh-syntax-highlighting 高亮语法 需配置信息
  source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

  # zsh-completions 增强自动补全 需配置信息
  if type brew &>/dev/null; then
      FPATH=$(brew --prefix)/share/zsh-completions:$FPATH

      autoload -Uz compinit
      compinit
  fi

  # 还需执行以下命令

  # rm -f ~/.zcompdump; compinit

  # chmod go-w '/usr/local/share'
  ```

- 个人使用的主题：

  ```sh
  ZSH_THEME="powerlevel10k/powerlevel10k"
  ```

  可通过执行以下命令进行下载，然后按步骤执行即可：

  ```sh
  $ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
  ```

- 常用插件：

  ```sh
  plugins=(git z web-search)
  ```

- 设置编码：

  ```sh
  export LC_ALL=en_US.UTF-8
  export LANG=en_US.UTF-8
  ```

- 对常用命令起别名，可以让我们使用 **vscode** 快速修改 **Oh My Zsh** 的配置文件，并快速更新配置文件：

  ```sh
  alias config="code .zshrc"
  alias restart="source ~/.zshrc"
  ```

---

### Item2

这个应用在上文 **Brewfile** 中已经进行了配置安装

使用 **Item2** 来代替系统默认终端

在 **item2 / themes** 中有自定义配色的相关文件，可在 "**Item2**" -> "**Perferences**" -> "**Profiles**" -> "**Colors**" -> "**Color Persets**" 选择喜欢的配色，也可以在 https://iterm2colorschemes.com/ 中选择喜欢的配色下载然后选择设置

---

### Typora

在 **typora / themes** 中有自定义主题的相关文件，将对应主题文件夹与 `.css` 文件放到 **Typora** 的默认主题文件夹，然后重启 **Typora** 即可

**Typora** 默认主题文件夹可在 "**Tpyora**" -> "偏好设置" -> "外观" -> "主题" -> "打开主题文件夹" 中打开

## 其他应用

基本应用清单可以查看 Brewfile 里列举的应用，这里是记录一些个人常用的其他应用：

- [DeepL](https://www.deepl.com/home)：一款快速翻译软件。

- [ClashX Pro](https://install.appcenter.ms/users/clashx/apps/clashx-pro/distribution_groups/public)：一款适用于 macOS 用爱发电的代理管理软件。

- [Figma](https://www.figma.com)：一款设计效率工具。

- [字由](https://www.hellofont.cn)：一款无需安装字体即可在设计工具上使用字体的软件。

- [itsycal](https://www.mowglii.com/itsycal)：一款颜值较高的日历。

- [HapiGo](https://hapigo.com/)：一款脑力工作者必备工具。

- [Notion](https://www.notion.so/)：一款需要科学上网的笔记软件。

- [阿里云盘](https://www.aliyundrive.com)：一款不限速的网盘。
