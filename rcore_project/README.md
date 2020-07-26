操作系统暑期项目 rCore 实验报告。

阅读了lab1至lab6的实验指导书。实现了lab1至lab3的复现。完成了lab1至lab6的实验题。

# rCore

> [视频：半个世纪过去了，是时候用Rust重写操作系统了吗？（CC字幕）](https://www.bilibili.com/video/av44834267?from=search&seid=4162693380754135939)
>
> [视频+PPT：金枪鱼之夜：陈嘉杰同学介绍 rCore v0.2.0 实现历程和进展, 2019](https://tuna.moe/event/2019/rcore-os/)
>
> [王润基：RUST OS开发历程与心得体会](https://cloud.tsinghua.edu.cn/f/530d03a556394fc6882c/)
>
> > [Rust语言操作系统的设计与实现,王润基本科毕设论文,2019](https://github.com/rcore-os/zCore/wiki/files/wrj-thesis.pdf)
>
> [【TUNA】rCore v0.2.0 实现历程与进展](https://www.bilibili.com/video/av47855780)
>
> [zCore操作系统内核的设计与实现,潘庆霖本科毕设论文,2020](https://github.com/rcore-os/zCore/wiki/files/pql-thesis.pdf)
>
> [Rust-OS-comparison](https://github.com/LearningOS/rcore_step_by_step/wiki/Rust-OS-comparison)
>
> [PPT: 尝试用RUST写教学操作系统, 2018](https://s4plus.ustc.edu.cn/_upload/article/files/57/c6/a2ce9bd84b2ab411967842a1334d/27730908-ef69-4827-98a7-8e387875b39b.pdf)
>
> [操作系统(RISC-V)清华在线课程,2020春季](https://next.xuetangx.com/course/thu08091002729/3175284?fromArray=search_result)
>
> [**Writing an OS in Rust**](https://os.phil-opp.com)
>
> [Redox](https://github.com/redox-os/redox) 开源界完成度最高的RustOS
>
> [从零开始写OS](https://zhuanlan.zhihu.com/c_1086573713289347072)，[rCore step by step](https://learningos.github.io/rcore_step_by_step_webdoc/)
>
> [rCore Tutorial 2020](https://github.com/rcore-os/rCore-Tutorial-deploy)（[gitpage](https://rcore-os.github.io/rCore-Tutorial-deploy/)）

## 简介

rCore是用Rust语言实现的小型操作系统。

- 兼容Alpine Linux(musl libc)：Busybox，GCC，Nginx……
- 支持四种指令集：x86_64，ARM64，RISC-V，MIPS32。

rCore社区：https://github.com/rcore-os。

![image-20200713004923144](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200713004923144.png)

### uCore回顾

uCore是C语言实现的小型操作系统。主要参考了：[xv6](https://github.com/jserv/xv6-x86_64)([xv6中文文档](https://th0ar.gitbooks.io/xv6-chinese/content/))，OS161，Linux。分为两个版本：

- [uCore Lab](https://github.com/chyyuu/ucore_os_lab)：用于OS课程实验
- uCore Plus：用于OS课程设计

### 从uCore到rCore

C语言：内存不安全（SegmentFault），缺少现代语言特性和好用的工具链。

而Rust：内存+线程安全，高层语言特性，友好的工具链，蓬勃发展的社区生态。

> https://github.com/rcore-os/rCore/blob/e1f93a179a4b2798247da9dd35277c2aa5f4ef14/docs/1_OS/FinalReport.md

在此项目（rCore）开始时，开源界已经有不少RustOS的项目：

- Redox：这是目前完成度最高的RustOS，微内核架构，平台x86_64
- 《Writing an OS in Rust》& blog_os：这是一个从零开始写RustOS的教程，平台x86_64
- rv6：这是一个xv6的Rust移植，然而它止步于内存管理，并且是完全C风格的
- CS140e：这是斯坦福2018年新开的实验性课程，用Rust写的教学OS，平台arm/RaspberryPi

### 反思rCore

![image-20200713004511979](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200713004511979.png)

#### 经验杂谈

![image-20200713010321508](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200713010321508.png)

![image-20200713010510052](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200713010510052.png)

![image-20200713010551131](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200713010551131.png)

## 环境安装

> https://rcore-os.github.io/rCore-Tutorial-deploy/docs/pre-lab/env.html

linux环境安装总览：（在这之前**<font color=red>先将WSL2和Ubuntu环境装好</font>**）

```bash
# # bash功能增强（可选，注意！！！发现此功能会导致rustup环境变量索引失效。建议先不使用。）
# sudo apt install fish
# chsh -s $(which fish)
# # 解决方案（手动添加rust到环境变量）： export PATH="$HOME/.cargo/bin:$PATH"
# # 发现fish的设置容易产生bug，尽量不用把

# 前置软件
sudo apt install gcc g++ git make
sudo apt install libglib2.0-dev libpixman-1-dev
sudo apt install pkg-config
sudo apt install flex bison

# qemu
wget https://download.qemu.org/qemu-5.0.0.tar.xz
tar xvJf qemu-5.0.0.tar.xz
cd qemu-5.0.0
./configure --target-list=riscv32-softmmu,riscv64-softmmu
make -j$(nproc)             # 如果你安装了fish，这里运行：make -j(nproc)
sudo make install
qemu-system-riscv64 --version
# 或：sudo apt install qemu（保证版本够新的情况下）

# rust
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
# 若不行，可采用清华源，见：https://mirrors.tuna.tsinghua.edu.cn/help/rustup/
# 即：export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup
# 注意：在终端输入【export】只是【临时】命令，终端退出即失效，需要手动永久添加语句到 ~/.bashrc
curl https://sh.rustup.rs -sSf | sh
# WSL采用：curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# 或者：curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --profile complete（获取最近的【完全】安装包）
export PATH="$HOME/.cargo/bin:$PATH"
rustup install nightly      # rCore默认使用nightly版本
# 若出现缺失，比如rustfmt，运行：rustup component add rustfmt
rustup default nightly      # 或：rustup override set nightly
rustc --version

# rCore
sudo git clone https://github.com/rcore-os/rCore-Tutorial.git
cd rCore-Tutorial
git checkout master

# 版本更新加速（在首次make run的过程中可能会下载大量依赖）
vi ~/.cargo/config     # 修改该文件（内容见下述链接）
# 详情见：https://rcore-os.github.io/rCore-Tutorial-deploy/docs/pre-lab/env.html

# 执行工具集
rustup target add riscv64imac-unknown-none-elf
cargo install cargo-binutils
rustup component add llvm-tools-preview
rust-objdump --version

# 编译运行
make run
```

### Windows WSL && Ubuntu

> [Dev on Windows with WSL - 在 Windows 上用 WSL 优雅开发](https://dowww.spencerwoo.com/)
>
> https://www.jianshu.com/p/3e627ff45ccb
>
> > Windows Subsystem for Linux（简称WSL）是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。它是由微软与Canonical公司合作开发，目标是使纯正的Ubuntu 14.04 "Trusty Tahr"映像能下载和解压到用户的本地计算机，并且映像内的工具和实用工具能在此子系统上原生运行。
> > 我们简单的认为它在 Windows 上安装了一个 Linux 环境就好了。

[WSL](https://docs.microsoft.com/zh-cn/windows/wsl/)（Windows Subsystem for Linux）是指 Windows 下构建 Linux 环境。你可以在使用 Windows 的同时，方便地进行 Linux 下的开发，并且 Linux 子系统上可以访问 Windows 的文件系统。但是，WSL 在安装rust时会出现环境配置方面的问题，因此这里我们采用新版的 WSL，即 WSL 2。

WSL 2 和 Ubuntu 环境安装步骤：（ **https://docs.microsoft.com/en-us/windows/wsl/install-win10**）

- 升级 Windows 10 到最新版（ **Windows 10 版本 18917 或以后的<font color=red>内部</font>版本**）
  - 如果不是 Windows 10 专业版，可能需要手动更新，在微软官网上下载。否则，可能 WSL 功能不能启动。
  - 在 Powershell 中输入 `winver` 查看**内部版本**号。
- 「Windows 设置 > 更新和安全 > Windows 预览体验计划」处选择加入，Dev开发者模式
- 打开 **PowerShell** 终端（**管理员**），输入：

```powershell
# 启用windows功能：“适用于Linux的Windows子系统”
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用windows功能：“已安装的虚拟机平台”
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 设置默认为WSL2，如果内部版本不够，这条命令会出错
wsl --set-default-version 2
```

- 如果先装了Ubuntu，则运行：

```powershell
# <Distro>改为对应版本名，比如： `wsl --set-version Ubuntu 2`
wsl --set-version <Distro> 2
```

- 在**微软商店**（Microsoft Store）中搜索 Ubuntu，安装第一个（或者你想要的版本）
  - 在 [此处](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-kernel) 下载 WSL 2 的 Linux 内核更新包
  - 安装完成后，打开 Ubuntu，进行**初始化**
- 回到 PowerShell 终端，输入：

```powershell
# 查看 WSL 的版本是否为 2
# 可简写为 `wsl -l -v`
wsl --list --verbose
```

- 若得到的版本信息正确，结束。**WSL 2 和 Ubuntu 环境安装完毕**。

在构建完成 WSL 2 + Ubuntu 环境后，可以在 Windows 的 Linux 子系统下便捷地部署 Linux 环境。

> 注意为了装rust，须**<font color=red>启用WSL 2</font>**！！！（https://docs.microsoft.com/zh-cn/windows/wsl/install-win10）
>
> > 详见 [*rust工具链*](https://vel.life/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/#rust%E5%B7%A5%E5%85%B7%E9%93%BE) 小节。

---

微软商店加载页面失败：https://jingyan.baidu.com/article/c45ad29cf41577441753e2db.html。

> 如果想在 Linux 查看其他分区，WSL 将其它盘符挂载在 **`/mnt`** 下。
>
> 如果想在 Windows 下查看 WSL 文件位置，文件位置在：`C:\Users\用户名\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs` 下。

---

WSL的linux命令使用似乎与纯ubuntu有些差异：

```sh
$ ls ./            # 查看当前目录下内容。在根目录须使用 ls /
$ cd folder        # 进入文件夹，比如： cd /mnt进入windows目录
```

为了方便访问windows磁盘文件夹，可能需要创建一个或多个软链接放到根目录。

比如：`sudo ln -s /mnt/c/Users/your_name/Desktop/ /`。

> 删除软链接：http://c.biancheng.net/view/744.html

#### Windows Terminal

> https://dowww.spencerwoo.com/1.1/2-cli/2-1-terminal.html#windows-terminal

WSL项目组开发，可以方面统一管理WSL、Power Shell、Command Prompt等环境。

Windows Terminal 已经**可以从 Microsoft Store 中直接下载**。

> 使用该软件还可以解决WSL的配色问题。
>

---

> https://github.com/microsoft/WSL/issues/5092
>
> https://github.com/microsoft/WSL/issues/4904 因为更新导致的4294967295

solve "**process exited with code 4294967295**" , run `netsh winsock reset` as Administrator, then reboot your computer.
 The result like below:

```
❯ netsh winsock reset

Sucessfully reset the Winsock Catalog.
You must restart the computer in order to complete the reset.
```

#### bash / zsh / fish

> https://dowww.spencerwoo.com/1.1/2-cli/2-2-shell.html#bash
>
> [Linux修改默认shell](https://blog.csdn.net/liubenq/article/details/78446648)

下载安装的 Windows Subsystem for Linux 默认就是 `bash` 的 Shell 环境。`bash` 是 Unix shell 的一种，是我们开发环境的基础。不过 `bash` 本身仅提供一个非常基础的命令行交互功能，没有类似 `zsh` 或 `fish` 等 Shell 的自动补全、命令提示等高阶功能。

`zsh` 和 `fish`，都是 Unix-like 系统中不可或缺的好 Shell，它们都极大的拓展了我们命令行界面的交互体验。在命令行的世界中：

- `fish` 更加注重「**开箱即用**」的体验，让我们安装完成即拥有一个包含了命令高亮、自动补全等强大功能的 Shell 环境
- `zsh` 则更加重视**拓展性**，借助于社区中优秀的 `zsh` 插件系统 oh-my-zsh 以及无数优秀的**插件**，`zsh` 同样能有比肩 `fish` 甚至比 `fish` 更高阶的功能和体验

```sh
sudo apt install zsh

chsh -s $(which zsh)    # 作为默认的 Shell 环境
```

zsh还需要单独安装自定义扩展才能达到较好的效果。也可以安装fish：

```sh
sudo apt install fish

chsh -s $(which fish)
```

切换回bash：

```sh
chsh -s /bin/bash
```

**<font color=red>此功能会导致rustup索引失效</font>**。（https://github.com/rust-lang/rustup/issues/686，https://github.com/rust-lang/vscode-rust/issues/675）

- 解决方案（手动添加rust到环境变量）： `export PATH="$HOME/.cargo/bin:$PATH"`
- 注意：export只能临时生效。需要**修改环境变量文件**。（[设置环境变量永久生效和临时生效 export PS1](https://blog.csdn.net/zhouyong0/article/details/8005520)）
  - [linux下ls、pwd等命令显示command not found](https://www.cnblogs.com/swlip/p/11768651.html)
  - 太烦了，~~fish这什么辣鸡语法~~，逼着我用zsh。
  - 还是zsh好，添加环境变量的语法跟bash一致。省得折腾。

> [为什么说 zsh 是 shell 中的极品？](https://www.zhihu.com/question/21418449)
>
> [加速你的 zsh —— 最强 zsh 插件管理器 zplugin/zinit 教程](http://www.aloxaf.com/2019/11/zplugin_tutorial/)
>
> [oh-my-zsh,让你的终端从未这么爽过](https://www.jianshu.com/p/d194d29e488c)
>
> [给 Zsh 添加主题和插件](https://linux.cn/article-11426-1.html)

### XServer for windows（可选）

> https://dowww.spencerwoo.com/1.1/4-advanced/4-1-gui.html#%E5%AE%89%E8%A3%85-xserver-for-windows GUI 图形化界面
>
> [**在 WSL（Windows Subsystem for Linux） 2 中运行 Linux 图形界面应用**](https://blog.csdn.net/qq_20084101/article/details/106423595)
>
> [Windows 10 WSL2 安装Linux Xfce图形界面](https://blog.csdn.net/kfeng632/article/details/102856346)

首先安装：[VcXsrv Windows X Server](https://sourceforge.net/projects/vcxsrv/)[ ](https://sourceforge.net/projects/vcxsrv/)。并按上述GUI教程配置打开。

```sh
sudo apt-get remove --purge openssh-server
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y openssh-server
sudo vi /etc/ssh/sshd_config
# Port 222
# X11Forwarding yes
# X11DisplayOffset 10
sudo service ssh start

sudo vi ~/zshrc  # 或者bashrc之类的
# export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf 2>/dev/null):0
# export LIBGL_ALWAYS_INDIRECT=1

sudo apt install libgtk2.0-0 libxss1 libasound2

vi ~/.profile
# # 添加以下内容：
# export DISPLAY=`cat /etc/resolv.conf | grep nameserver | awk '{print $2}'`:0
# export PULSE_SERVER=`cat /etc/resolv.conf | grep nameserver | awk '{print $2}'`
# # PULSE_SERVER一句是关于 PulseAudio 声音支持的，不需要可以删掉。

sudo apt install x11-apps -y  # 小眼睛工具

xeyes # 启动，看到小眼睛说明成功了
```

> 对于IP地址的设置：https://zhuanlan.zhihu.com/p/51270874。

#### VSCode

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'

sudo apt update && sudo apt upgrade
sudo apt install code

# 输入`code`启动VS Code到XServer的GUI上。
code
```

![image-20200718003257896](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718003257896.png)

VSCode命令行：https://code.visualstudio.com/docs/editor/command-line

> `code -h`获得帮助。

---

[VSCode设置中文语言显示](https://blog.csdn.net/qq_30068487/article/details/82589347)：

- Ctrl+Shift+p，在搜索框中输入“configure display language”
- 选择安装更多语言 `->` 中文简体
- [vs code中使用remote wsl中文乱码问题](https://blog.csdn.net/qq_28120673/article/details/102087006)

---

[Skip WLS check if env var DONT_PROMPT_WSL_INSTALL is set.](https://github.com/microsoft/vscode/pull/80529/commits/08580a467153d620964c2b17d7a6c556ab13334e):

![image-20200718033436446](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718033436446.png)

- VSCode启动时总是有提示，专门去分析了一下此处的源码。了解了grep命令和if的用法：
  - [linux grep命令](https://www.cnblogs.com/end/archive/2012/02/21/2360965.html)
  - [Shell Script if / else 條件判斷式](https://www.opencli.com/linux/shell-script-if-else-elseif)
- 对于`if grep -qi Microsoft /proc/version && [ -z "$DONT_PROMPT_WSL_INSTALL" ]; then`
  - `grep -qi Microsoft /proc/version`：模式匹配，若在文件vesion中搜索到Microsoft则为true。不区分大小写。
  - `[ -z "$DONT_PROMPT_WSL_INSTALL" ]`：當 $str 是 null, 回傳 true.
- 修改并添加了环境变量`DONT_PROMPT_WSL_INSTALL=233`。没有起到预想中的效果。暂时**放弃**。
- **<font color=red>卸掉完事</font>**。

---

**<font color=red>事实证明，根本不需要额外安装WSL里面的VSCode，WSL可以自动启动windows里面已经装好的VSCode！</font>**

### qemu

下载地址：https://qemu.weilnetz.de/w64/

> https://download.qemu.org/

windows安装后须配置环境变量：将安装目录添加到`path`中。

运行：

```c++
qemu-system-riscv64 --version
// QEMU emulator version 5.0.0 (v5.0.0-11810-g8846fa22bb-dirty)
// Copyright (c) 2003-2020 Fabrice Bellard and the QEMU Project developers
```

表明RISC-V 64 虚拟器安装成功。

---

linux版本按照实验指导书安装即可。

> 若tar.xz文件下载较慢，可以在https://download.qemu.org/手动科学下载。

`ERROR: "cc" either does not exist or does not work`：

- 说明没有安装`gcc`。（https://blog.csdn.net/feiyangyongran/article/details/46414517）
  - 更换ubuntu软件源镜像：[Here](https://dowww.spencerwoo.com/1.1/2-cli/2-2-shell.html#%E6%9B%B4%E6%8D%A2%E8%BD%AF%E4%BB%B6%E6%BA%90%E9%95%9C%E5%83%8F)
- 运行：`sudo apt install gcc`

> 接下来可能还会有一堆not found和required。按提示依次`sudo apt install ...`即可。

`ERROR: glib-2.40 gthread-2.0 is required to compile`：

- 使用`apt-cache search glib2`**查看应该安装哪个库**。（https://blog.csdn.net/fuxy3/article/details/104732541）
  - `sudo apt-get install libglib2.0-dev`
    - 注：新版ubuntu（Ubuntu 16.04）引入了`apt`代替`apt-get`命令
- **<font color=red>注 - 软件包查找方法</font>**：`apt-cache search pixman`。
  - `sudo apt-get install libpixman-1-dev`

---

QEMU 可以使用 `ctrl+a` （macOS 为 `control+a`） 再按下 `x` 键退出。

### make

> [Make 命令教程](http://www.ruanyifeng.com/blog/2015/02/make.html)
>
> [Windows安装GNU编译器使用makefile](https://blog.csdn.net/Nicholas_Liu2017/article/details/78323391)，[【杂谈】windows10配置make命令](https://blog.csdn.net/C2681595858/article/details/85554359)

自动化编译工具。

- `sudo apt install make`。

> `make[1]: rust-objcopy: Command not found`：
>
> - 缺少binutils 工具集。
> - `cargo install cargo-binutils`
>   `rustup component add llvm-tools-preview`

### rust工具链

首先安装 Rust 版本管理器 rustup 和 Rust 包管理器 cargo，这个windows之前已经安装。

linux版可能再装一次。

```sh
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
curl https://sh.rustup.rs -sSf | sh
```

---

`thread panicked while panicking. aborting.
Illegal instruction (core dumped)`：（似乎是用WSL装rust的特有错误）

- https://stackoverflow.com/questions/61603982/thread-main-panicked-at-assertion-failed-left-right-left-22-right
- https://github.com/rust-lang/rustup/issues/2245 WLS 2才能正常装。。

发现自己可能WSL装错了版本。

> **只有 Windows 10 版本 18917 或以后的版本才能够正常运行 WSL 2**。需要明确，WSL 2  目前依旧只能在 Windows 10 预览体验计划的版本中使用，因此你需要在「Windows 设置 > 更新和安全 >  Windows 预览体验计划」处选择加入 Fast ring 或 Slow ring，这样才能使用正确的 Windows 10 版本安装 WSL 2。（[Here](https://dowww.spencerwoo.com/1.1/1-preparations/1-1-installation.html#windows-10)）
>
> https://docs.microsoft.com/zh-cn/windows/wsl/install-win10
>
> https://github.com/Lincyaw/Rust_os_summer/blob/master/readme.md

更新windows。装上了WSL2。发现linux无法切换到WSL2。卸载重装报错：

```
Installing, this may take a few minutes...
WslRegisterDistribution failed with error: 0x800701bc
Error: 0x800701bc WSL 2 ?????????????????? https://aka.ms/wsl2kernel
```

解决方案：https://github.com/microsoft/WSL/issues/5393

> 在此处下载WSL2 Linux内核更新包：https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-kernel
>
> 更新后问题解决。

然后重装一切。。

---

---

---

在经过漫长的鏖战以后，凌晨三点半，终于，运行成功了！

![image-20200717032528029](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200717032528029.png)

![image-20200717032609113](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200717032609113.png)

### gdb

运行GDB架构：

```sh
gdb --configuration    # --target指定可以debug的类型
```

安装依赖：

```sh
sudo apt-get install libncurses5-dev python python-dev texinfo libreadline-dev
```

按照教程走即可。

---

`error: *** A compiler with support for C++11 language features is required.`：

- `sudo apt install g++`

---

#### GDB调试语法

- `b <函数名>` ：在函数进入时设置断点，例如 `b rust_main` 或 `b os::memory::heap::init`
- `cont` ：继续执行
- `n` ：执行下一行代码，不进入函数
- `ni` ：执行下一条指令（跳转指令则执行至返回）
- `s` ：执行下一行代码，进入函数
- `si` ：执行下一条指令，包括跳转指令
- `layout`：如果没有安装 `gdb-dashboard`，可以通过 `layout` 指令来呈现寄存器等信息，具体查看 `help layout`
- `x/<格式> <地址>` ：使用 `x/<格式> <地址>` 来查看内存，例如 `x/8i 0x80200000` 表示查看 `0x80200000` 起始的 8 条指令。具体格式查看 `help x`

#### gdb-dashboard（可选）

```sh
wget -P ~ git.io/.gdbinit
```

> GDB will automatically load `./.gdbinit` for current debugging.

`Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|0.0.0.0|:443... failed: Connection refused.`：（被墙了）

- https://blog.csdn.net/littlehaes/article/details/103638711
- [WSL2来了！但是能正常使用并不简单](https://zhuanlan.zhihu.com/p/144583887)【WSL和V2ray的防火墙设置】
- [记录一次WSL2的网络代理配置](https://jiayaoo3o.github.io/2020/06/23/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1WSL2%E7%9A%84%E7%BD%91%E7%BB%9C%E4%BB%A3%E7%90%86%E9%85%8D%E7%BD%AE/)【V2ray】

`Error parsing proxy URL socks5://172.27.208.1:10808: Unsupported scheme ‘socks5’.`：

- wget不支持socks5。。。
- 搞来搞去。逼得没办法了。分析了一下`wget -P ~ git.io/.gdbinit`的含义，如下
  - **用wget将网络文件`git.io/.gdbinit`保存到`~`根目录下**
  - ` -P,  --directory-prefix=PREFIX   save files to PREFIX/..
           --cut-dirs=NUMBER           ignore NUMBER remote directory components`
  - 好了，这就好办，直接手动下载`.gdbinit`放到根目录！
    - `mv ./.gdbinit ~/`（https://gitee.com/dongbo_89/gdb-dashboard）
    - 安装pip：https://pip.pypa.io/en/stable/installing/

```sh
pip install pygments # Optionally install Pygments to enable syntax highlighting
```

> https://stackoverflow.com/questions/42870537/zsh-command-cannot-found-pip
>
> - `python -m pip install pygments`

## Lab0：了解写RUST写OS的相关综述信息

> [**Lab0 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-0/guide/intro.html)
>
> [操作系统(RISC-V)清华在线课程,2020春季](https://next.xuetangx.com/course/thu08091002729/3175284?fromArray=search_result) 了解一下RISC-V、rCore的知识
>
> [Rust语言操作系统的设计与实现,王润基本科毕设论文,2019](https://github.com/rcore-os/zCore/wiki/files/wrj-thesis.pdf)
>
> [zCore操作系统内核的设计与实现,潘庆霖本科毕设论文,2020](https://github.com/rcore-os/zCore/wiki/files/pql-thesis.pdf)
>
> [Rust-OS-comparison](https://github.com/LearningOS/rcore_step_by_step/wiki/Rust-OS-comparison)
>
> [视频：半个世纪过去了，是时候用Rust重写操作系统了吗？（CC字幕）](https://www.bilibili.com/video/av44834267?from=search&seid=4162693380754135939)
>
> [视频+PPT：金枪鱼之夜：陈嘉杰同学介绍 rCore v0.2.0 实现历程和进展, 2019](https://tuna.moe/event/2019/rcore-os/)
>
> [PPT: 尝试用RUST写教学操作系统, 2018](https://s4plus.ustc.edu.cn/_upload/article/files/57/c6/a2ce9bd84b2ab411967842a1334d/27730908-ef69-4827-98a7-8e387875b39b.pdf)

### 创建项目

```sh
mkdir ./rcore_project
cd rcore_project
echo "nightly-2020-06-27" >> ./rust-toolchain
cargo new os
```

项目结构到此创建完毕。

运行测试：

```sh
cd os
cargo run
```

![image-20200718024308463](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718024308463.png)

### 移除标准库依赖

```rust
#![no_std]
fn main() {
    println!("Hello, world!");
}
```

利用`#![no_std]`禁用标准库。产生三个**error**：

```rust
error: cannot find macro `println` in this scope
 --> src/main.rs:3:5
  |
7 |     println!("Hello, rCore-Tutorial!");
  |     ^^^^^^^
error: `#[panic_handler]` function required, but not found
error: language item required, but not found: `eh_personality`
```

**error1**，删去println!宏即可。

**error2**，自主实现panic函数（ `panic_handler` ）：

```rust
/// 当 panic 发生时会调用该函数
use core::panic::PanicInfo; // 核心库 core，与标准库 std 不同，这个库不需要操作系统的支持
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}                 // 我们暂时将它的实现为一个死循环
}
```

**error3**，语义项（Language Item）缺失。（ `panic_handler` 也是一个语义项）

- `eh_personality`：eh 是 Exception Handling 的缩写，它是一个标记某函数用来实现**堆栈展开**处理功能的语义项。
- 在`os/Cargo.toml`中：将 dev 配置和 release 配置的 panic 的处理策略设为直接终止，也就是直接调用我们的 `panic_handler` 而不是先进行堆栈展开等处理再调用。

```toml
...

# panic 时直接终止，因为我们没有实现堆栈展开的功能
[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

运行：

![image-20200718200833153](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718200833153.png)

### 移除运行时环境依赖

**运行时系统**（Runtime System）可能导致 `main` 函数并不是实际执行的第一个函数。

Rust 的运行时入口点被 `start` 语义项标记。Rust 运行时环境的**入口点结束之后才会调用 `main` 函数**进入主程序。

- 重写覆盖整个 `crt0` 入口点。
  - 加上 `#![no_main]` 告诉编译器我们不用常规的入口点。
  - 实现一个 **`_start`** 函数来代替 `crt0`，并加上 `#[no_mangle]` 告诉编译器对于此函数禁用编译期间的名称重整（Name Mangling）——确保编译器生成一个名为 `_start` 的函数。

```rust
//! # 全局属性
//! - `#![no_std]`  
//!   禁用标准库
#![no_std]
//!
//! - `#![no_main]`  
//!   不使用 `main` 函数等全部 Rust-level 入口点来作为程序入口
#![no_main]

/// 当 panic 发生时会调用该函数
use core::panic::PanicInfo; // 核心库 core，与标准库 std 不同，这个库不需要操作系统的支持
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}                 // 我们暂时将它的实现为一个死循环
}

/// 覆盖 crt0 中的 _start 函数
/// 我们暂时将它的实现为一个死循环
#[no_mangle]
pub extern "C" fn _start() -> ! {
    loop {}
}
```

![image-20200718214654610](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718214654610.png)

### 编译为裸机目标

**链接错误**：链接器的默认配置假定程序依赖于 C 语言的运行时环境，但我们的程序并不依赖于它。

> 为了解决这个错误，我们需要告诉链接器，它不应该包含 C 语言运行时环境。我们可以选择**提供特定的链接器参数（Linker  Argument）**，也可以选择**编译为裸机目标（Bare Metal  Target）**，我们将沿着后者的思路在后面解决这个问题，即直接编译为裸机目标不链接任何运行时环境。

`rustc --version --verbose`：查看当前系统的目标三元组。

![image-20200718221104763](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718221104763.png)

> host 字段的值为三元组 x86_64-unknown-linux-gnu，它包含了 CPU 架构 x86_64、供应商 unknown、操作系统 linux 和二进制接口 gnu。

裸机环境：（底层没有操作系统的运行环境。这个其实之前已经装了）

```sh
rustup target add riscv64imac-unknown-none-elf
# 目标三元组 riscv64imac-unknown-none-elf 描述了一个 RISC-V 64 位指令集的系统。
```

`cargo build --target riscv64imac-unknown-none-elf`：

![image-20200718224323452](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718224323452.png)

编译后结果放在了 `os/target/riscv64imac-unknown-none-elf/debug` 文件夹中。其中有一个名为 `os` 的可执行文件。它的目标平台是 RISC-V 64，暂时还不能通过我们的开发环境执行它。

在 `os` 文件夹中创建一个 `.cargo` 文件夹，并在其中创建一个名为 `config` 的文件，在其中填入以下内容：

```toml
# 编译的目标平台
[build]
target = "riscv64imac-unknown-none-elf"
```

这指定了此项目编译时默认的目标。

以后可以直接使用 `cargo build` 来编译了。

![image-20200718225757450](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718225757450.png)

### 生成内核镜像

为了查看和分析生成的可执行文件，我们首先需要安装一套名为 binutils 的命令行工具集，其中包含了 objdump 和 objcopy 等常用工具。这在之前已经安装完毕。

查看编译好的`os`可执行文件：

```sh
file target/riscv64imac-unknown-none-elf/debug/os
```

![image-20200718230007596](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200718230007596.png)

> 它是一个 64 位的 elf 格式的可执行文件，架构是 RISC-V；链接方式为静态链接；not stripped 指的是里面符号表的信息未被剔除，而这些信息在调试程序时会用到，程序正常执行时通常不会使用。

使用刚刚安装的工具链中的 rust-objdump 工具看看它的具体信息：

```sh
vel@LAPTOP-OD50F928 /Linux/rcore_project/os
 % rust-objdump target/riscv64imac-unknown-none-elf/debug/os -x --arch-name=riscv64

target/riscv64imac-unknown-none-elf/debug/os:   file format ELF64-riscv

architecture: riscv64
start address: 0x0000000000011120

Program Header:
    PHDR off    0x0000000000000040 vaddr 0x0000000000010040 paddr 0x0000000000010040 align 2**3
         filesz 0x00000000000000e0 memsz 0x00000000000000e0 flags r--
    LOAD off    0x0000000000000000 vaddr 0x0000000000010000 paddr 0x0000000000010000 align 2**12
         filesz 0x0000000000000120 memsz 0x0000000000000120 flags r--
    LOAD off    0x0000000000000120 vaddr 0x0000000000011120 paddr 0x0000000000011120 align 2**12
         filesz 0x0000000000000004 memsz 0x0000000000000004 flags r-x
   STACK off    0x0000000000000000 vaddr 0x0000000000000000 paddr 0x0000000000000000 align 2**64
         filesz 0x0000000000000000 memsz 0x0000000000000000 flags rw-

Dynamic Section:
Sections:
Idx Name            Size     VMA              Type
  0                 00000000 0000000000000000
  1 .text           00000004 0000000000011120 TEXT
  2 .debug_str      00000403 0000000000000000
  3 .debug_abbrev   00000113 0000000000000000
  4 .debug_info     0000053c 0000000000000000
  5 .debug_aranges  00000040 0000000000000000
  6 .debug_ranges   00000030 0000000000000000
  7 .debug_pubnames 000000a4 0000000000000000
  8 .debug_pubtypes 00000308 0000000000000000
  9 .debug_frame    00000050 0000000000000000
 10 .debug_line     0000005b 0000000000000000
 11 .comment        00000013 0000000000000000
 12 .symtab         00000108 0000000000000000
 13 .shstrtab       000000a5 0000000000000000
 14 .strtab         0000002d 0000000000000000

SYMBOL TABLE:
0000000000000000 l    df *ABS*  00000000 3gqd1qcioyc9uzqc
0000000000011120         .text  00000000
0000000000011120         .text  00000000
0000000000011120         .text  00000000
0000000000011124         .text  00000000
0000000000000000         .debug_info    00000000
0000000000000000         .debug_ranges  00000000
0000000000000000         .debug_frame   00000000
0000000000000000         .debug_line    00000000 .Lline_table_start0
0000000000011120 g     F .text  00000004 _start
```

按顺序逐个查看：

- `start address`：程序的**入口地址**
- `Sections`：从这里我们可以看到程序**各段的各种信息**。后面以 debug 开头的段是调试信息
- `SYMBOL TABLE`：符号表，从中我们可以看到程序中**所有符号的地址**。例如 `_start` 函数就位于入口地址上
- `Program Header`：程序加载时所需的**段信息**
  - 其中的 **off** 是它在文件中的位置，**vaddr 和 paddr** 是要加载到的虚拟地址和物理地址，**align**  规定了地址的对齐，**filesz 和 memsz** 分别表示它在文件和内存中的大小，**flags** 描述了相关权限（r 表示可读，w 表示可写，x  表示可执行）

对于`rust-objdump`，`-x` 来、可以查看程序的元信息，下面我们用 `-d` 来对代码进行反汇编：

```bash
vel@LAPTOP-OD50F928 /Linux/rcore_project/os
 % rust-objdump target/riscv64imac-unknown-none-elf/debug/os -d --arch-name=riscv64

target/riscv64imac-unknown-none-elf/debug/os:   file format ELF64-riscv


Disassembly of section .text:

0000000000011120 _start:
   11120: 09 a0                         j       2
   11122: 01 a0                         j       0
```

可以看到其中只有一个 `_start` 函数，里面什么都不做，就一个死循环。

> 并没有看到类似的东西。

#### 生成镜像

```sh
rust-objcopy target/riscv64imac-unknown-none-elf/debug/os --strip-all -O binary target/riscv64imac-unknown-none-elf/debug/kernel.bin
```

这里 `--strip-all` 表明丢弃所有符号表及调试信息，`-O binary` 表示输出为二进制文件。

至此，我们编译并生成了内核镜像 `kernel.bin` 文件。接下来，我们将使用 QEMU 模拟器真正将我们的内核镜像跑起来。

### 调整内存布局

一般来说，一个程序按照功能不同会分为下面这些段：

- .text 段：代码段，存放汇编代码
- .rodata 段：只读数据段，顾名思义里面存放只读数据，通常是程序中的常量
- .data 段：存放被初始化的可读写数据，通常保存程序中的全局变量
- .bss 段：存放被初始化为 0 的可读写数据，与 .data 段的不同之处在于我们知道它要被初始化为 0，因此在可执行文件中只需记录这个段的大小以及所在位置即可，而不用记录里面的数据，也不会实际占用二进制文件的空间
- Stack：栈，用来存储程序运行过程中的局部变量，以及负责函数调用时的各种机制。它从高地址向低地址增长
- Heap：堆，用来支持程序**运行过程中**内存的**动态分配**，比如说你要读进来一个字符串，在你写程序的时候你也不知道它的长度究竟为多少，于是你只能在运行过程中，知道了字符串的长度之后，再在堆中给这个字符串分配内存

内存布局，也就是指这些段各自所放的位置。一种典型的内存布局如下：

![image-20200719222922888](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200719222922888.png)

#### 编写链接脚本

使用**链接脚本（Linker Script）**来指定程序的内存布局。创建文件 `os/src/linker.ld`：

```sh
touch ./src/linker.ld
cd ./src
```

写入下述内容：

```rust
/* 有关 Linker Script 可以参考：https://sourceware.org/binutils/docs/ld/Scripts.html */

/* 目标架构 */
// 使用 OUTPUT_ARCH 指定了架构
OUTPUT_ARCH(riscv)

/* 执行入口 */
// 使用 ENTRY 指定了入口点为 _start 函数，即程序第一条被执行的指令所在之处
// 在这个链接脚本中我们并未看到 _start ，回忆上一章，我们为了移除运行时环境依赖，重写了入口 _start 。所以，链接脚本宣布整个程序会从那里开始运行。
ENTRY(_start)

/* 数据存放起始地址 */
BASE_ADDRESS = 0x80200000;

// 链接脚本的整体写在 SECTION{ } 中，里面有多个形如 output section: { input section list } 的语句，每个都描述了整个程序内存布局中的一个输出段 output section 是由各个文件中的哪些输入段 input section 组成的。
SECTIONS
{
    /* . 表示当前地址（location counter） */
    . = BASE_ADDRESS;

    /* start 符号表示全部的开始位置 */
    kernel_start = .;

    text_start = .;

    /* .text 字段 */
    .text : {
        // 我们可以用 *( ) 来表示将各个文件中所有符合括号内要求的输入段放在当前的位置。而括号内，你可以直接使用段的名字，也可以包含通配符 *。
        /* 把 entry 函数放在最前面 */
        *(.text.entry)
        /* 要链接的文件的 .text 字段集中放在这里 */
        *(.text .text.*)
    }
	
    // 单独的一个 . 为当前地址（Location Counter），可以对其赋值来从设置的地址继续向高地址放置各个段。如果不进行赋值的话，则默认各个段会紧挨着向高地址放置。将一个符号赋值为 . 则会记录下这个符号的地址。
    rodata_start = .;

    /* .rodata 字段 */
    .rodata : {
        /* 要链接的文件的 .rodata 字段集中放在这里 */
        *(.rodata .rodata.*)
    }

    data_start = .;

    /* .data 字段 */
    .data : {
        /* 要链接的文件的 .data 字段集中放在这里 */
        *(.data .data.*)
    }

    bss_start = .;

    /* .bss 字段 */
    .bss : {
        /* 要链接的文件的 .bss 字段集中放在这里 */
        *(.sbss .bss .bss.*)
    }

    /* 结束地址 */
    kernel_end = .;
}

// 首先是从 BASE_ADDRESS 即 0x80200000 开始向下放置各个段，依次是 .text，.rodata，.data，.stack 和 .bss。同时我们还记录下了每个段的开头和结尾地址，如 .text 段的开头、结尾地址分别就是符号 stext 和 etext 的值
```

在 `.cargo/config` 文件中加入以下配置：

```sh
# 使用我们的 linker script 来进行链接
# 在链接时传入一个参数 -T 来指定使用哪个链接脚本
[target.riscv64imac-unknown-none-elf]
rustflags = [
    "-C", "link-arg=-Tsrc/linker.ld",
]
```

重新编译：

```sh
cargo build
rust-objdump target/riscv64imac-unknown-none-elf/debug/os -h --arch-name=riscv64
rust-objdump target/riscv64imac-unknown-none-elf/debug/os -d --arch-name=riscv64
```

![image-20200721142034340](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200721142034340.png)

### 重写程序入口点 `_start`

在 `_start` 中设置内核的运行环境：（`os/src/entry.asm`）

```asm
# 操作系统启动时所需的指令以及字段
#
# 我们在 linker.ld 中将程序入口设置为了 _start，因此在这里我们将填充这个标签
# 它将会执行一些必要操作，然后跳转至我们用 rust 编写的入口函数
#
# 关于 RISC-V 下的汇编语言，可以参考 https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md

    .section .text.entry
    .globl _start
# 目前 _start 的功能：将预留的栈空间写入 $sp，然后跳转至 rust_main
_start:
    la sp, boot_stack_top
    call rust_main

    # 回忆：bss 段是 ELF 文件中只记录长度，而全部初始化为 0 的一段内存空间
    # 这里声明字段 .bss.stack 作为操作系统启动时的栈
    .section .bss.stack
    .global boot_stack
boot_stack:
    # 16K 启动栈大小
    .space 4096 * 16
    .global boot_stack_top
boot_stack_top:
    # 栈结尾
```

将 `os/src/main.rs` 里面的 `_start` 函数删除，并换成 `rust_main`。

### 使用 QEMU 运行内核

运行命令：

```sh
$ qemu-system-riscv64 \
  --machine virt \
  --nographic \
  --bios default
```

![image-20200721143738441](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200721143738441.png)

加入输出代码，以及Makefile。

![image-20200721150522668](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200721150522668.png)

---

`Makefile:16: *** missing separator.  Stop.`：

- https://stackoverflow.com/questions/16931770/makefile4-missing-separator-stop
  - makefile has a very stupid relation with tabs, all actions of every rule are identified by tabs. And no, 4 spaces don't make a tab, only a tab  makes a tab.
- Makefile语法不支持4个空格代替Tab。

### 接口封装和代码整理

#### 使用 OpenSBI 提供的服务

OpenSBI 实际上不仅起到了 **bootloader**  的作用，还为我们提供了一些底层系统服务供我们在编写内核时使用，以简化内核实现并提高内核跨硬件细节的能力。这层底层系统服务接口称为  **SBI**（Supervisor Binary Interface），是 S Mode 的 OS 和 M Mode 执行环境之间的标准接口约定。

> 参考 [OpenSBI 文档](https://github.com/riscv/riscv-sbi-doc/blob/master/riscv-sbi.adoc#legacy-sbi-extension-extension-ids-0x00-through-0x0f) ，里面包含了一些以 C 函数格式给出的我们可以调用的接口。

建立`os/src/sbi.rs`：

```rust
//! 调用 Machine 层的操作
// 目前还不会用到全部的 SBI 调用，暂时允许未使用的变量或函数
#![allow(unused)]

/// SBI 调用
#[inline(always)]
fn sbi_call(which: 4usize, arg0: usize, arg1: usize, arg2: usize) -> usize {
    let ret;
    unsafe {
        llvm_asm!("ecall"
            : "={x10}" (ret)
            : "{x10}" (arg0), "{x11}" (arg1), "{x12}" (arg2), "{x17}" (which)
            : "memory"      // 如果汇编可能改变内存，则需要加入 memory 选项
            : "volatile");  // 防止编译器做激进的优化（如调换指令顺序等破坏 SBI 调用行为的优化）
    }
    ret
}

const SBI_SET_TIMER: usize = 0;
const SBI_CONSOLE_PUTCHAR: usize = 1;
const SBI_CONSOLE_GETCHAR: usize = 2;
const SBI_CLEAR_IPI: usize = 3;
const SBI_SEND_IPI: usize = 4;
const SBI_REMOTE_FENCE_I: usize = 5;
const SBI_REMOTE_SFENCE_VMA: usize = 6;
const SBI_REMOTE_SFENCE_VMA_ASID: usize = 7;
const SBI_SHUTDOWN: usize = 8;

/// 向控制台输出一个字符
///
/// 需要注意我们不能直接使用 Rust 中的 char 类型
pub fn console_putchar(c: usize) {
    sbi_call(SBI_CONSOLE_PUTCHAR, c, 0, 0);
}

/// 从控制台中读取一个字符
///
/// 没有读取到字符则返回 -1
pub fn console_getchar() -> usize {
    sbi_call(SBI_CONSOLE_GETCHAR, 0, 0, 0)
}

/// 调用 SBI_SHUTDOWN 来关闭操作系统（直接退出 QEMU）
pub fn shutdown() -> ! {
    sbi_call(SBI_SHUTDOWN, 0, 0, 0);
    unreachable!()
}
```

把整个 `print` 和 `println` 宏按照逻辑写出：

```rust
//! 实现控制台的字符输入和输出
//! 
//! # 格式化输出
//! 
//! [`core::fmt::Write`] trait 包含
//! - 需要实现的 [`write_str`] 方法
//! - 自带实现，但依赖于 [`write_str`] 的 [`write_fmt`] 方法
//! 
//! 我们声明一个类型，为其实现 [`write_str`] 方法后，就可以使用 [`write_fmt`] 来进行格式化输出
//! 
//! [`write_str`]: core::fmt::Write::write_str
//! [`write_fmt`]: core::fmt::Write::write_fmt

use crate::sbi::*;
use core::fmt::{self, Write};

/// 一个 [Zero-Sized Type]，实现 [`core::fmt::Write`] trait 来进行格式化输出
/// 
/// ZST 只可能有一个值（即为空），因此它本身就是一个单件
struct Stdout;

impl Write for Stdout {
    /// 打印一个字符串
    ///
    /// [`console_putchar`] sbi 调用每次接受一个 `usize`，但实际上会把它作为 `u8` 来打印字符。
    /// 因此，如果字符串中存在非 ASCII 字符，需要在 utf-8 编码下，对于每一个 `u8` 调用一次 [`console_putchar`]
    fn write_str(&mut self, s: &str) -> fmt::Result {
        let mut buffer = [0u8; 4];
        for c in s.chars() {
            for code_point in c.encode_utf8(&mut buffer).as_bytes().iter() {
                console_putchar(*code_point as usize);
            }
        }
        Ok(())
    }
}

/// 打印由 [`core::format_args!`] 格式化后的数据
/// 
/// [`print!`] 和 [`println!`] 宏都将展开成此函数
/// 
/// [`core::format_args!`]: https://doc.rust-lang.org/nightly/core/macro.format_args.html
pub fn print(args: fmt::Arguments) {
    Stdout.write_fmt(args).unwrap();
}

/// 实现类似于标准库中的 `print!` 宏
/// 
/// 使用实现了 [`core::fmt::Write`] trait 的 [`console::Stdout`]
#[macro_export]
macro_rules! print {
    ($fmt: literal $(, $($arg: tt)+)?) => {
        $crate::console::print(format_args!($fmt $(, $($arg)+)?));
    }
}

/// 实现类似于标准库中的 `println!` 宏
/// 
/// 使用实现了 [`core::fmt::Write`] trait 的 [`console::Stdout`]
#[macro_export]
macro_rules! println {
    ($fmt: literal $(, $($arg: tt)+)?) => {
        $crate::console::print(format_args!(concat!($fmt, "\n") $(, $($arg)+)?));
    }
}
```

将 `main.rs` 中处理 panic 的语义项抽取并完善到 `panic.rs` 中：

```rust
//! 代替 std 库，实现 panic 和 abort 的功能

use core::panic::PanicInfo;
use crate::sbi::shutdown;

/// 打印 panic 的信息并 [`shutdown`]
///
/// ### `#[panic_handler]` 属性
/// 声明此函数是 panic 的回调
#[panic_handler]
fn panic_handler(info: &PanicInfo) -> ! {
    // `\x1b[??m` 是控制终端字符输出格式的指令，在支持的平台上可以改变文字颜色等等，这里使用红色
    // 参考：https://misc.flogisoft.com/bash/tip_colors_and_formatting
    //
    // 需要全局开启 feature(panic_info_message) 才可以调用 .message() 函数
    println!("\x1b[1;31mpanic: '{}'\x1b[0m", info.message().unwrap());
    shutdown()
}

/// 终止程序
/// 
/// 调用 [`panic_handler`]
#[no_mangle]
extern "C" fn abort() -> ! {
    panic!("abort()")
}
```

运行：

![image-20200721152846175](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200721152846175.png)

## Lab1：boot与中断

> [**Lab1 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-1/guide/intro.html)
>
> https://github.com/chyyuu/ucore_os_lab/blob/riscv64-priv-1.10/docs/riscv-overview.md
>
> https://github.com/chyyuu/ucore_os_lab/blob/riscv64-priv-1.10/docs/toolchain-overview.md
>
> https://github.com/chyyuu/ucore_os_lab/blob/riscv64-priv-1.10/docs/lab1.md

广义的中断包括异常、系统调用（软中断）、硬件中断。

关于中断的分类：

> > https://github.com/rcore-os/rCore-Tutorial/issues/97
>
> interrupt
>
> - **hardware** interrupt (external, async)
> - **software** interrupt (internal, sync)
>   - **syscall**/**trap** (voluntarily yield to os)
>   - **exception** (involuntarily caught by os)

### 中断CSR

发生中断时，硬件**<font color=red>自动填写</font>**的寄存器：

- **`sepc`**：即 Exception Program Counter，用来**记录触发中断的指令的地址**。

  > 和我们之前学的 MIPS 32 系统不同，RISC-V 中不需要考虑延迟槽的问题。但是 RISC-V 中的指令不定长，如果中断处理需要恢复到异常指令后一条指令执行，就**需要正确判断将 `pc` 寄存器加上多少字节**。

- **`scause`**：记录中断**是否是硬件中断**，以及具体的**中断原因**。

- **`stval`**：scause 不足以存下中断所有的必须信息。例如**缺页异常**，就会将 `stval` 设置成需要访问但是不在内存中的地址，以便于操作系统将这个地址所在的页面加载进来。

指导硬件**<font color=red>处理中断</font>**的寄存器：

- **`stvec`**：设置**内核态中断处理流程的入口地址**。存储了一个基址 BASE 和模式 MODE：
  - MODE 为 0 表示 Direct 模式，即遇到中断便跳转至 BASE 进行执行。
  - MODE 为 1 表示 Vectored 模式，此时 BASE 应当指向一个向量，存有不同处理流程的地址，遇到中断会跳转至 `BASE + 4 * cause` 进行处理流程。
- **`sstatus`**：具有许多状态位，**控制全局中断使能**等。
- **`sie`**：即 Supervisor Interrupt Enable，用来**控制具体类型中断的使能**，
  - 例如其中的 STIE 控制时钟中断使能。
- **`sip`**：即 Supervisor Interrupt Pending，和 `sie` 相对应，**记录每种中断是否被触发**。
  - 仅当 `sie` 和 `sip` 的对应位都为 1 时，意味着开中断且已发生中断，这时中断最终触发。
- **`sscratch`**：在用户态，`sscratch` 保存**内核栈的地址**；在内核态，`sscratch` 的值为 0。
  - 在内核态中，`sp` 可以认为是一个安全的栈空间，`sscratch` 便不需要保存任何值。此时将其设为 0，可以在遇到中断时通过 `sscratch` 中的值判断中断前程序是否处于内核态。

### 中断指令

- **`ecall`**：**触发中断**，进入更高一层的中断处理流程之中。用户态进行系统调用进入内核态中断处理流程，内核态进行 SBI 调用进入机器态中断处理流程，使用的都是这条指令。
- **`sret`**：**从内核态返回用户态**，同时将 `pc` 的值设置为 `sepc`。（如果需要返回到 `sepc` 后一条指令，就需要在 `sret` 之前修改 `sepc` 的值）
- **`ebreak`**：**触发一个断点**。
- **`mret`**：**从机器态返回内核态**，同时将 `pc` 的值设置为 `mepc`。
- **`csrrw dst, csr, src`**（CSR Read Write）
  **同时读写**的原子操作，将指定 CSR 的值写入 `dst`，同时将 `src` 的值写入 CSR。
- **`csrr dst, csr`**（CSR Read）
  仅**读取**一个 CSR 寄存器。
- **`csrw csr, src`**（CSR Write）
  仅**写入**一个 CSR 寄存器。
- **`csrc(i) csr, rs1`**（CSR Clear）
  将 CSR 寄存器中**指定的位 清零**，`csrc` 使用通用寄存器作为 mask，`csrci` 则使用立即数。
- **`csrs(i) csr, rs1`**（CSR Set）
  将 CSR 寄存器中**指定的位 置 1**，`csrc` 使用通用寄存器作为 mask，`csrci` 则使用立即数。

### 上下文

设计Context的结构：

```rust
use riscv::register::{sstatus::Sstatus, scause::Scause};

#[repr(C)]
pub struct Context {
    pub x: [usize; 32],     // 32 个通用寄存器
    pub sstatus: Sstatus,
    pub sepc: usize
}
```

在`os/Cargo.toml`添加依赖：

```toml
[dependencies]
riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }
```

上下文的保存与恢复：

```asm
# 我们将会用一个宏来用循环保存寄存器。这是必要的设置
.altmacro
# 寄存器宽度对应的字节数
.set    REG_SIZE, 8
# Context 的大小
.set    CONTEXT_SIZE, 34

# 宏：将寄存器存到栈上
.macro SAVE reg, offset
    sd  \reg, \offset*8(sp)
.endm

.macro SAVE_N n
    SAVE  x\n, \n
.endm

# 宏：将寄存器从栈中取出
.macro LOAD reg, offset
    ld  \reg, \offset*8(sp)
.endm

.macro LOAD_N n
    LOAD  x\n, \n
.endm

    .section .text
    .globl __interrupt
# 进入中断
# 保存 Context 并且进入 Rust 中的中断处理函数 interrupt::handler::handle_interrupt()
__interrupt:
    # 在栈上开辟 Context 所需的空间
    addi    sp, sp, -34*8

    # 保存通用寄存器，除了 x0（固定为 0）
    SAVE    x1, 1
    # 将原来的 sp（sp 又名 x2）写入 2 位置
    addi    x1, sp, 34*8
    SAVE    x1, 2
    # 保存 x3 至 x31
    .set    n, 3
    .rept   29
        SAVE_N  %n
        .set    n, n + 1
    .endr

    # 取出 CSR 并保存
    csrr    s1, sstatus
    csrr    s2, sepc
    SAVE    s1, 32
    SAVE    s2, 33

    # 调用 handle_interrupt，传入参数
    # context: &mut Context
    mv      a0, sp
    # scause: Scause
    csrr    a1, scause
    # stval: usize
    csrr    a2, stval
    jal  handle_interrupt

    .globl __restore
# 离开中断
# 从 Context 中恢复所有寄存器，并跳转至 Context 中 sepc 的位置
__restore:
    # 恢复 CSR
    LOAD    s1, 32
    LOAD    s2, 33
    csrw    sstatus, s1
    csrw    sepc, s2

    # 恢复通用寄存器
    LOAD    x1, 1
    # 恢复 x3 至 x31
    .set    n, 3
    .rept   29
        LOAD_N  %n
        .set    n, n + 1
    .endr

    # 恢复 sp（又名 x2）这里最后恢复是为了上面可以正常使用 LOAD 宏
    LOAD    x2, 2
    sret
```

### 中断处理流程

在`os/src/interrupt/handler.rs`中初始化处理器：

```rust
use super::context::Context;
use riscv::register::{stvec, scause::Scause};

global_asm!(include_str!("./interrupt.asm"));

pub fn init(){
    unsafe {
        extern "C" {
            fn __interrupt();  // 调用interrupt.asm的接口
        }
        stvec::write(__interrupt as usize, stvec::TrapMode::Direct);
    }
}

#[no_mangle]
pub fn handle_interrupt(context: &mut Context, scause: Scause, stval: usize) {
    panic!("Interrupted: {:?}", scause.cause());
}
```

并将之前的所有函数封装：

```rust
mod handler;
mod context;
pub fn init() {
    handler::init();
    println!("mod interrupt initialized");
}
```

在main函数中设置触发器：

```rust
...
mod interrupt;
...

pub extern "C" fn rust_main() -> ! {
    interrupt::init();
    unsafe {
        llvm_asm!("ebreak"::::"volatile");
    };
    unreachable!();
}
```

运行：

![image-20200722194730988](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200722194730988.png)

### 时钟中断

设计时钟中断处理器：

```rust
use crate::sbi::set_timer;
use riscv::register::{time, sie, sstatus};

pub fn init() {
    unsafe {
        sie::set_stimer(); 
        sstatus::set_sie();  // 开启 sstatus 寄存器中的 SIE 位
        // SIE 位决定中断是否能够打断 supervisor 线程。在这里我们需要允许时钟中断打断 内核态线程，因此置 SIE 位为 1。
    }
    set_next_timeout();
}

static INTERVAL: usize = 100000;

fn set_next_timeout() {
    set_timer(time::read() + INTERVAL);
}

/// 触发时钟中断计数
pub static mut TICKS: usize = 0;

pub fn tick() {
    set_next_timeout();
    unsafe {
        TICKS += 1;
        if TICKS % 100 == 0 {
            println!("{} tick", TICKS);
        }
    }
}
```

进行教程中所示的微小调整，引入mod。

```rust
mod handler;
mod context;
pub mod timer;   // 为了在main函数中测试，这里设置为pub
pub fn init() {
    handler::init();
    timer::init();
    println!("mod interrupt initialized");
}
```

为了在main函数中临时调用timer，我暂时将timer设置为了pub库。得到如下效果：

![image-20200722210015868](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200722210015868.png)

对比lab1关于context的内容，似乎lab1中的描述少了一部分关于Debug格式的代码。对比之后，我将此部分代码添加到了本地lab1的代码中。

### 实验一实验题

> https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-1/practice.html

#### 原理

原理：在 `rust_main` 函数中，执行 `ebreak` 命令后至函数结束前，`sp` 寄存器的值是怎样变化的？

> `ebreak` 命令就是设置断点。它会调用中断服务例程。也就是说，涉及到 `Context` 上下文的存取。我们注意到，`sp` 是栈指针，如果需要存取 `Context` 上下文时，`sp` 的值便会发生变化。
>
> - 首先在中断保存 `Context` 的过程中，`sp` 减去一个 `Context` 的大小，从而执行中断服务例程将 `Context` 保存到栈中；
> - 执行中断的过程中，`sp` 可能因为局部变量等操作有一些加加减减，但最后仍然总的来说保持不变；
> - 从中断返回时，执行`_restore`，`sp` 加上一个 `Context` 的大小，并恢复上下文。

#### 分析

分析：如果去掉 `rust_main` 后的 `panic` 会发生什么，为什么？

> 实际运行时发生如下错误：
>
> ![image-20200724223712653](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724223712653.png)
>
> 这说明 `panic!` 的返回值是必要的。
>
> 另外按照实验书的解释，`rust_main` 返回后，程序并没有停止。其执行完后会回到 `entry.asm` 中。但是，`entry.asm` 并没有在后面写任何指令，这意味着程序将接着向后执行内存中的任何指令。
>
> 执行 `rust-objdump -d -S os/target/riscv64imac-unknown-none-elf/debug/os | less` 来查看汇编代码，可以发现之后还有很长很长的各种函数。
>
> ![image-20200724224316427](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724224316427.png)

#### 实验

如果程序访问不存在的地址，会得到 `Exception::LoadFault`。模仿捕获 `ebreak` 和时钟中断的方法，捕获 `LoadFault`（之后 `panic` 即可）。

> 添加如下match：
>
> ![image-20200724224822935](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724224822935.png)

在处理异常的过程中，如果程序想要非法访问的地址是 `0x0`，则打印 `SUCCESS!`。

> 用 `fault` 函数类似的机制，单独实现对 LoadFault 的处理函数：
>
> ![image-20200724225553596](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724225553596.png)

添加或修改少量代码，使得运行时触发这个异常，并且打印出 `SUCCESS!`。

- 要求：不允许添加或修改任何 unsafe 代码

> 这个实在是没什么经验，不过，看了看解答，可以通过汇编代码的方式实现（修改 `Context` 调用的那个方法虽然有效，但是具有破坏性）。
>
> 但是，不允许添加或修改任何 unsafe 代码，这个就有点过分。那这样就用不了跳转指令了。
>
> > - 解法 1：在 `interrupt/handler.rs` 的 `breakpoint` 函数中，将 `context.sepc += 2` 修改为 `context.sepc = 0`（则 `sret` 时程序会跳转到 `0x0`）
> > - 解法 2：去除 `rust_main` 中的 `panic` 语句，并在 `entry.asm` 的 `jal rust_main` 之后，添加一行读取 `0x0` 地址的指令（例如 `jr x0` 或 `ld x1, (x0)`）
>
> 照着解法2搞了搞没搞出来。照着解法1搞：
>
> ![image-20200724232808726](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724232808726.png)
>
> 没有如预料中的出现 `LoadFault`。。
>
> 可能是因为我用了lab3的代码吧。。**用lab1的代码过了**。（两种解法均有效）
>
> ![image-20200724235036288](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724235036288.png)
>
> ![image-20200725000104386](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725000104386.png)

## Lab2：物理内存管理

> [**Lab2 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-2/guide/intro.html)

### 动态内存分配

内核中需要**动态内存分配**。典型的应用场景有：

- `Box<T>` ，你可以理解为它和 `malloc` 有着相同的功能；
- 引用计数 `Rc<T>`，原子引用计数 `Arc<T>`，主要用于在引用计数清零，即某对象不再被引用时，对该对象进行自动回收；
- 一些 Rust std 标准库中的数据结构，如 `Vec` 和 `HashMap` 等。

动态内存分配需要操作系统的支持，也就需要手动实现。在 Rust 语言中，我们需要实现 `Trait GlobalAlloc`，并将这个类实例化，并使用语义项 `#[global_allocator]` 进行标记。这样的话，编译器就会知道如何使用我们提供的内存分配函数进行动态内存分配。

为了实现`Trait GlobalAlloc`，就需要实现以下两个方法：

```rust
unsafe fn alloc(&self, layout: Layout) -> *mut u8;            // 分配一块虚拟内存
unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout);       // 回收一块虚拟内存
```

> `Layout`：分配一块连续的、大小至少为 `size` 字节的虚拟内存，且对齐要求为 `align` 。它有两个字段：`size` 表示要分配的字节数，`align` 则表示分配的虚拟地址的最小对齐要求，即分配的地址要求是 `align` 的倍数。这里的 `align` 必须是 2 的幂次。
>
> https://doc.rust-lang.org/nightly/core/alloc/trait.GlobalAlloc.html
>
> https://doc.rust-lang.org/nightly/core/alloc/struct.Layout.html
>
> Layout的结构摘录如下：

```rust
pub struct Layout {
    // size of the requested block of memory, measured in bytes.
    size_: usize,

    // alignment of the requested block of memory, measured in bytes.
    // we ensure that this is always a power-of-two, because API's
    // like `posix_memalign` require it and it is a reasonable
    // constraint to impose on Layout constructors.
    //
    // (However, we do not analogously require `align >= sizeof(void*)`,
    //  even though that is *also* a requirement of `posix_memalign`.)
    align_: NonZeroUsize,
}
```

建立`config.rs`，设置堆空间大小：

```rust
// 开辟堆空间（8M）
pub const KERNEL_HEAP_SIZE: usize = 0x80_0000;
```

建立`heap.rs`，实现堆空间的管理：

（关于buddy_sysytem：https://github.com/rcore-os/buddy_system_allocator）

```rust
use super::config::KERNEL_HEAP_SIZE;
use buddy_system_allocator::LockedHeap;

// 堆空间，放在 bss 段
static mut HEAP_SPACE: [u8; KERNEL_HEAP_SIZE] = [0; KERNEL_HEAP_SIZE];

#[global_allocator]
static HEAP: LockedHeap = LockedHeap::empty();
// [`LockedHeap`] 实现了 [`alloc::alloc::GlobalAlloc`] trait

pub fn init() {
    unsafe {
        HEAP.lock().init(
            HEAP_SPACE.as_ptr() as usize, KERNEL_HEAP_SIZE
        )
    }
}

#[alloc_error_handler]
fn alloc_error_handler(_: alloc::alloc::Layout) -> ! {
    panic!("alloc error")
}
```

注意：[`LockedHeap`] 已经实现了 [`alloc::alloc::GlobalAlloc`] trait（Buddy System Allocator）。查看lab2源代码，发现 `heap2.rs` 实现了其他分配算法。但是 Trait 就要相应地自己去实现。

注意：这个`buddy_system_allocator`要在 `Cargo.toml` 中引入：（https://github.com/rcore-os/rCore-Tutorial/blob/master/os/Cargo.toml#L13）。我的建议是直接把这部分的配置搬过来，省得之后麻烦：

```toml
[dependencies]
# algorithm = { path = 'src/algorithm' }
bit_field = "0.10.0"
bitflags = "1.2.1"
buddy_system_allocator = "0.3.9"        # 【就是这里】了
hashbrown = "0.7.2"
lazy_static = { version = "1.4.0", features = ["spin_no_std"] }
riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }
spin = "0.5.2"
device_tree = { git = "https://github.com/rcore-os/device_tree-rs" }
virtio-drivers = { git = "https://github.com/rcore-os/virtio-drivers" }
rcore-fs = { git = "https://github.com/rcore-os/rcore-fs"}
rcore-fs-sfs = { git = "https://github.com/rcore-os/rcore-fs"}
xmas-elf = "0.7.0"

# panic 时直接终止，因为我们没有实现堆栈展开的功能
[profile.dev]
panic = "abort"
[profile.release]
panic = "abort"
```

然后就是更新依赖。。。

然后把`#![feature(alloc_error_handler)]`添加到main.rs里面，启用相关特性。

---

![image-20200723172942494](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200723172942494.png)

### 分配算法*

操作系统的分配算法当然是很多的。操作系统课上就学了不少了。。

这部分有时间可以写一个看看。有时间可以参考https://github.com/rcore-os/buddy_system_allocator看看。

### 物理内存探测

发现此处需要用到 `address.rs`，但是却没有提到。从终代码中获取了该部分的源码。

若出现以下的引用错误：

![image-20200723223643024](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200723223643024.png)

注意到super所指的对象是当前目录下的`mod.rs`文件，在`mod.rs`引用address模块即可。

---

最终效果：

![image-20200723223921092](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200723223921092.png)

### 物理内存管理

注意动态内存分配，管理的是堆中的内存分配问题。而物理内存管理，是整个物理内存的页式存储管理。

实验指导写得不全，导致各种错误。和lab2得代码对比着调了半天过了。

![image-20200723235210795](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200723235210795.png)

注意到测试代码的内容：

![image-20200723235450099](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200723235450099.png)

这说明了内存的分配和自动回收是有效的。

### 实验二实验题

> https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-2/practice.html

#### 原理

原理：.bss 字段是什么含义？为什么我们要将动态分配的内存（堆）空间放在 .bss 字段？

> .bss 段：存放被初始化为 0 的可读写数据，与 .data 段的不同之处在于我们知道它要被初始化为 0，因此在可执行文件中只需记录这个段的大小以及所在位置即可，而不用记录里面的数据，也不会实际占用二进制文件的空间。
>
> **并不是必须**要将动态分配的内存（堆）空间放在 .bss 字段。任何一个其他的段也都是可以的。但是这样做可能在代码实现上会比较简单。并且保证堆空间在内核的二进制数据之中。

#### 分析

分析：我们在动态内存分配中实现了一个堆，它允许我们在内核代码中使用动态分配的内存，例如 `Vec` `Box` 等。那么，如果我们在实现这个堆的过程中使用 `Vec` 而不是 `[u8]`，会出现什么结果？

- 无法编译？
- 运行时错误？
- 正常运行？

> 没有看懂这个题。。不过看了解答之后。明白了是递归定义的锅。实现堆的过程中如果又用了堆，那么就会一直递归下去。。

#### 实验

回答：`algorithm/src/allocator` 下有一个 `Allocator` trait，我们之前用它实现了物理页面分配。这个算法的时间和空间复杂度是什么？

> 这说的哪个算法？`stacked_allocator` 吧应该。。那么对于栈来说，时间复杂度O(1)，空间复杂度O(n)。

二选一：实现基于线段树的物理页面分配算法（不需要考虑合并分配）；或尝试修改 `FrameAllocator`，令其使用未被分配的页面空间（而不是全局变量）来存放页面使用状态。

> 线段树感觉之前的代码好像已经不小心搬运过来了。。
>
> lab2的线段树应该是用位图的方式维护的。我不妨先将这个算法理解一般好了。~~虽然肯定比不上自己实现~~了。在大概理解了线段树的思路之后，随后自己手动实现了一遍：（能通过编译和测试，但是正确性不太好验证）

```rust
use super::Allocator;
use alloc::{vec, vec::Vec};

pub struct MySegmentTreeAllocator {
    tree: Vec<bool>,  // 二叉堆标记法
    capacity: usize,
}


impl Allocator for MySegmentTreeAllocator {
    fn new(capacity: usize) -> Self {
        assert!(capacity >= 1);

        // next_power_of_two: 上取整为 最近的 2的幂
        // https://github.com/mattdesl/next-power-of-two
        let leaf_num = capacity.next_power_of_two();

        // 初始所有节点为空
        let mut tree : Vec<bool> = vec![false; 2*leaf_num];

        // 所有超出capacity的节点为满
        for i in (leaf_num - 2 + capacity)..tree.len(){
            tree[i] = true;
        }
        Self { tree, capacity }
    }

    fn alloc(&mut self) -> Option<usize> {
        if self.tree[0] == true {
            None
        } else{
            let mut id = 0; id = 100;
            loop{
                if id * 2 + 2 < self.tree.len(){
                    if self.tree[id * 2 + 1] == false{
                        id = id * 2 + 1;
                    } else{
                        id = id * 2 + 2;
                    }
                } else {break;}
            }

            self.tree[id] = true;
            let ret = id - self.tree.len() / 2;

            // 向上回溯
            while id > 0 {
                id = (id - 1) / 2;
                self.tree[id] = self.tree[id * 2 + 1] && self.tree[id * 2 + 2];
            }
            Some(ret)
        }
    }

    fn dealloc(&mut self, index: usize) {
        let mut id = self.capacity.next_power_of_two() + index;
        while id > 0{
            self.tree[id] = false;
            id = (id - 1) / 2;
        }
    }
}
```

> https://github.com/yunwei37/os-summer-of-code-daily/blob/master/daily_documents/Day15_lab2_practice.md。这位老哥好像用time这个模块实现了性能测试，mark一下。
>
> 我看了看栈分配器，又对比实现了一个队列分配器（虽然实际上没有怎么改动）：

```rust
//! 提供队列结构实现的分配器 [`QueueAllcator`]

use super::Allocator;
use alloc::{vec, vec::Vec};

/// 使用队列实现分配器
pub struct QueueAllcator {
    list: Vec<(usize, usize)>,
}

impl Allocator for QueueAllcator {
    fn new(capacity: usize) -> Self {
        Self {
            list: vec![(0, capacity)],
        }
    }

    // 队列从 左边 出队，从 右边 进队
    fn alloc(&mut self) -> Option<usize> {
        if let Some((start, end)) = Some(self.list[0]) {   // 这里改动了一下
            self.list.remove(0);
            if end - start > 1 {
                self.list.push((start + 1, end));
            }
            Some(start)
        } else {
            None
        }
    }

    fn dealloc(&mut self, index: usize) {
        self.list.push((index, index + 1));
    }
}
```

#### 挑战实验（选做）

> 既然是选做，那我就暂时不做了。。。QAQ

挑战实验（选做）

1. 在 `memory/heap2.rs` 中，提供了一个手动实现堆的方法。它使用 `algorithm::VectorAllocator` 作为其根本分配算法，而我们目前提供了一个非常简单的 bitmap 算法（而且只开了很小的空间）。请在 `algorithm` crate 中利用伙伴算法实现 `VectorAllocator` trait。
2. 前面说到，堆的实现本身不能完全使用动态内存分配。但有没有可能让堆能够利用动态分配的空间，这样做会带来什么好处？

## Lab3：虚拟内存管理

> [**Lab3 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-3/guide/intro.html)

虚拟内存这一块的东西，包括各种映射、TLB之类的，我还是比较熟悉的。 

在实现虚拟地址结构后，调整为虚拟地址空间。

然后是各种映射的函数。。

运行：（注意到输出的是 VirtualAddress，虚拟地址生效了）

![image-20200724185713088](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200724185713088.png)

### 页面置换*

页面置换的部分暂时跳过了。

### 实验三实验题

> https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-3/practice.html

#### 原理

原理：在 `os/src/entry.asm` 中，`boot_page_table` 的意义是什么？当跳转执行 `rust_main` 时，不考虑缓存，硬件通过哪些地址找到了 `rust_main` 的第一条指令？

> `boot_page_table` 的意义自然是页表，具体来说，`boot_page_table` 指的是根页表。第一部分是低地址的恒等映射，用于维护 pc 的值，保证程序正常运行。第二部分是将高地址映射到低地址。
>
> ![image-20200725143528166](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725143528166.png)
>
> 然后就不会了。看了看解答，真的多。。
>
> > 我们在 `linker.ld` 中指定了起始地址为 `0xffff_ffff_8020_0000`，操作系统执行文件会认为所有的符号都是在这个高地址上的。但是我们在硬件上只能将内核加载到 `0x8020_0000` 开始的内存空间上，此时的 `pc` 也会调转到这里。
> >
> > 执行 `jal rust_main` 时，硬件需要加载 `rust_main` 对应的地址，大概是 `0xffff_ffff_802x_xxxx`。
> >
> > - **页表已经启用**，硬件先从 `satp` 高位置读取内存映射模式，再从 `satp` 低位置读取根页表页号，即 `boot_page_table` 的物理页号
> > - 对于 Sv39 模式，页号有三级共 27 位。对于 `rust_main` 而言，一级页号是其 [30:38] 位，即 510。硬件此时定位到根页表的第 510 项
> > - 这一项的标志为 XWR，说明它指向一个大页而不是指向下一级页表；目标的页号为 `0x8_0000`，即物理地址 `0x8000_0000` 开始的区间；这一项的 V 位为 1，说明目标在内存中。因此，硬件寻址到页基址 + 页内偏移，即 `0x8000_0000 + 0x2x_xxxx`，找到 `rust_main`
>
> 总的来说，就是硬件获取到初始的内存映射模式以后，就通过 `rust_main` 的虚拟地址，进行一级一级的查表，最后查到了 `rust_main` 的物理地址所在的帧，这样就可以读取 `rust_main` 了。重点就是，这里需要进行虚实转换。
>
> 我通过反汇编 ` rust-objdump -d -S ./target/riscv64imac-unknown-none-elf/debug/os >> ../debug.file`，将输出保存文件中，查找到了 `rust_main` 的地址：（左侧即虚拟地址）
>
> ![image-20200725145413476](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725145413476.png)

#### 分析

分析：为什么 `Mapping` 中的 `page_tables` 和 `mapped_pairs` 都保存了一些 `FrameTracker`？二者有何不同？

> `FrameTracker` 的作用：方便管理所有的物理页，我们需要实现一个分配器可以进行分配和回收的操作。这个 `Tracker` 就是一个管理存储页的结构。
>
> ![image-20200725150421443](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725150421443.png)
>
> 保存了一些 `FrameTracker`，就是保存了一些物理页，也就是使用了一些内存。显然 `page_tables` 和 `mapped_pairs` 使用内存的目的是不同的，`page_tables` 存放了所有页表所用到的页面，而 `mapped_pairs` 则存放了进程所用到的页面。 

分析：假设某进程需要虚拟地址 A 到物理地址 B 的映射，这需要操作系统来完成。那么操作系统在建立映射时有没有访问 B？如果有，它是怎么在还没有映射的情况下访问 B 的呢？

> 建立映射不需要访问B，这是显然的，因为访问B必然要在得到B的物理地址以后进行。而我们只需要物理地址就可以建立映射，因此后来的访问步骤是不必要的——我们暂时不需要访问页面内的具体内容。
>
> > 不过，通常程序都会需要操作系统建立映射的同时向页面中加载一些数据。
> >
> > > 那么实操来说，还是需要访问B的。
> >
> > 尽管 A→B 的映射尚不存在，因为我们将**整个可用物理内存都建立了内核映射**，所以操作系统仍然可以通过线性偏移量来访问到 B。
> >
> > > 都有物理地址了，有没有映射根本不影响访问嘛！

#### 实验

实验：了解并实现时钟页面置换算法（或任何你感兴趣的算法），可以自行设计样例来比较性能

- 置换算法只需要修改 `os/src/memory/mapping/swapper.rs`，你可能需要在其中访问页表项
- 在 `main.rs` 中调用 `start_kernel_thread` 来创建线程，你可以任意修改其中运行的函数，以达到测试效果

> 怎么感觉这个东西需要lab4的内容。。。暂时做不了。
>
> 有点难度。。有时间再考虑。。
>
> 看了看[Here](https://github.com/chibinz/rCoreSummerOfCode/blob/master/DailySchedule.md)，确实虽然时间提前了，但也不必过于搞突击。尽力就好。不过我目前算是没有什么整理总结的压力吧。

## Lab4：内核线程&用户进程&调度

> [**Lab4 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-4/guide/intro.html)

### 线程与进程

线程与进程的一些东西。。之前学过。

每个线程都有自己独立的运行栈是，但它们可以在进程的尺度上共享资源，比如CPU时间、物理内存等等。

线程的表示需要用到控制块，线程如果我没记错的是线程控制块，进程则是PCB（进程控制块），每个控制块里面保存了识别一个线程、进程的关键信息。

比如实验书上提到的：

- **线程 ID**：用于唯一确认一个线程，它会在系统调用等时刻用到。
- **运行栈**：每个线程都必须有一个独立的运行栈，保存运行时数据。
- **线程执行上下文**：当线程不在执行时，我们需要保存其上下文（其实就是一堆**寄存器**的值），这样之后才能够将其恢复，继续运行。和之前实现的中断一样，上下文由 `Context` 类型保存。（注：这里的**线程执行上下文**与前面提到的**中断上下文**是不同的概念）
- **所属进程的记号**：同一个进程中的多个线程，会共享页表、打开文件等信息。因此，我们将它们提取出来放到线程中。
- **内核栈**：除了线程运行必须有的运行栈，中断处理也必须有一个单独的栈。之前，我们的中断处理是直接在原来的栈上进行（我们直接将 `Context` 压入栈）。但是在后面我们会引入用户线程，这时就只有上帝才知道发生了什么——栈指针、程序指针都可能在跨国（**国 == 特权态**）旅游。为了确保中断处理能够进行（让操作系统能够接管这样的线程），中断处理必须运行在一个准备好的、安全的栈上。这就是内核栈。不过，内核栈并没有存储在线程信息中。（注：**它的使用方法会有些复杂，我们会在后面讲解**。）

![image-20200725152929938](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725152929938.png)

注意这里的Range就是分配虚拟地址的范围：

![image-20200725153007458](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725153007458.png)

> 注意到，因为线程一般使用 `Arc<Thread>` 来保存，它是不可变的，所以其中再用 `Mutex` 来包装一部分，让这部分可以修改。

同理，进程的结构：

![image-20200725153252404](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725153252404.png)

在完成了一些工作以后，可以看到输出：（由于各种依赖关系BUG太多，我直接用了最终版的rcore代码来进行测试，不再一个一个文件地修改了）

```rust
pub fn test_restore_thread(){
    extern "C" {
        fn __restore(context: usize);
    }
    // 获取第一个线程的 Context，具体原理后面讲解
    println!("获取第一个线程的 Context，具体原理后面讲解.");
    let context = PROCESSOR.lock().prepare_next_thread();
    
    // 启动第一个线程
    println!("启动第一个线程");
    unsafe { __restore(context as usize) };

    unreachable!()
}
```

![image-20200725163544551](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725163544551.png)

在重新部署了lab4的代码之后：

```sh
    Finished dev [unoptimized + debuginfo] target(s) in 6.98s

OpenSBI v0.6
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : QEMU Virt Machine
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 8
Current Hart           : 0
Firmware Base          : 0x80000000
Firmware Size          : 120 KB
Runtime SBI Version    : 0.2

MIDELEG : 0x0000000000000222
MEDELEG : 0x000000000000b109
PMP0    : 0x0000000080000000-0x000000008001ffff (A)
PMP1    : 0x0000000000000000-0xffffffffffffffff (A,R,W,X)
Hello rCore-Tutorial !!!
mod interrupt initialized
mod memory initialized
测试
hello from kernel thread 1
hello from kernel thread 2
hello from kernel thread 3
hello from kernel thread 4
hello from kernel thread 5
hello from kernel thread 6
hello from kernel thread 7
hello from kernel thread 8
Thread {
    thread_id: 0x7,
    stack: Range {
        start: VirtualAddress(
            0x1300000,
        ),
        end: VirtualAddress(
            0x1380000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 7
Thread {
    thread_id: 0x5,
    stack: Range {
        start: VirtualAddress(
            0x1200000,
        ),
        end: VirtualAddress(
            0x1280000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 5
Thread {
    thread_id: 0x4,
    stack: Range {
        start: VirtualAddress(
            0x1180000,
        ),
        end: VirtualAddress(
            0x1200000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 4
Thread {
    thread_id: 0x2,
    stack: Range {
        start: VirtualAddress(
            0x1080000,
        ),
        end: VirtualAddress(
            0x1100000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 2
Thread {
    thread_id: 0x8,
    stack: Range {
        start: VirtualAddress(
            0x1380000,
        ),
        end: VirtualAddress(
            0x1400000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 8
Thread {
    thread_id: 0x3,
    stack: Range {
        start: VirtualAddress(
            0x1100000,
        ),
        end: VirtualAddress(
            0x1180000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 3
Thread {
    thread_id: 0x6,
    stack: Range {
        start: VirtualAddress(
            0x1280000,
        ),
        end: VirtualAddress(
            0x1300000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 6
Thread {
    thread_id: 0x1,
    stack: Range {
        start: VirtualAddress(
            0x1000000,
        ),
        end: VirtualAddress(
            0x1080000,
        ),
    },
    context: None,
} terminated: unimplemented interrupt type
cause: Exception(InstructionPageFault), stval: 1
src/process/processor.rs:87: 'all threads terminated, shutting down'
```

和实验指导上的输出结果不太一样。。不过算勉强可以了吧。。

又用lab-4分支的代码实验了一下，发现直接多了一个user目录。。（这个是lab6的内容。。）最终效果：

![image-20200725184502172](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725184502172.png)

### 实验四实验题

#### 原理

原理：线程切换之中，页表是何时切换的？页表的切换会不会影响程序 / 操作系统的运行？为什么？

> 页表是在 `Process::prepare_next_thread()` 中调用 `Thread::prepare()`，其中换入了新线程的页表。
>
> ![image-20200725192355942](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725192355942.png)
>
> 下面是线程中对应方法的实现，可以看到页表的切换过程：
>
> ![image-20200725192243627](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725192243627.png)
>
> 页表的切换**不会**影响程序 / 操作系统的运行。因为切换过程是通过中断完成的，而中断是操作系统实现的。同时页表切换后，只要之前保存的映射关系有效，程序也可以恢复到之前的状态，从而正确运行。

#### 分析

> https://www.bookstack.cn/read/ucore_os_docs/lab6-lab6_3_6_1_basic_method.md

分析：

- 在 Stride Scheduling 算法下，如果一个线程进入了一段时间的等待（例如等待输入，此时它不会被运行），会发生什么？

> 它被调用的可能性增加。（stride相对降低，因为其他线程会升高）
>
> Stride Scheduling 算法的**核心公式**是：P.pass =BigStride / P.priority。也就是优先级越高，步长pass越小。
>
> 因为Stride Scheduling 算法的**核心策略**是：重新调度当前stride最小的进程。这样累加的stride越慢，就更加容易被调用。
>
> 参考：https://github.com/yunwei37/os-summer-of-code-daily/blob/master/daily_documents/Day21_lab4_practice.md。可以得到，这里的线程实际上是阻塞的，所以如果它的stride值很低时会抢占CPU而不能运行，导致其他线程不能获得资源。

- 对于两个优先级分别为 9 和 1 的线程，连续 10 个时间片中，前者的运行次数一定更多吗？

> 不一定。比如，前者的stride现在是1000，而后者的stride是100。假设BigStride = 90; 这样，即使优先级更高，但总的stride仍然太大，后者仍然会被更多地运行。
>
> 也可能优先级分别为 9 的线程运行一次就结束了。

- 你认为 Stride Scheduling 算法有什么不合理之处？可以怎样改进？

> stride累计对旧进程可能不太友好。如果stride累计太久了，那么新加入的进程将在一个时间段内长期占据CPU，从而让其他的进程无法运行。应该设计一种抑制措施：
>
> - 当一个进程等待时长每超过 T 秒时，此进程的 stride 累计值减半
>
> 这样，等待过久的进程的stride值会很快恢复正常。
>
> Stride Scheduling 算法不支持对进程状态的应对，比如优先级高的可能正处于阻塞状态。

#### 设计

设计：如果不使用 `sscratch` 提供内核栈，而是像原来一样，遇到中断就直接将上下文压栈，请举出（思路即可，无需代码）：

- 一种情况不会出现问题
- 一种情况导致异常无法处理（指无法进入 `handle_interrupt`）
- 一种情况导致产生嵌套异常（指第二个异常能够进行到调用 `handle_interrupt`，不考虑后续执行情况）
- 一种情况导致一个用户进程（先不考虑是怎么来的）可以将自己变为内核进程，或以内核态执行自己的代码

> 这个真是不会。。看了解答。
>
> > - 只运行一个非常善意的线程，比如 `loop {}`
> >   - `jr 0` + `jr 2` 的那种，这样程序始终运行在局部，当然不会出现问题了
> > - 线程把自己的 `sp` 搞丢了，比如 `mv sp, x0`。此时无法保存寄存器，也没有能够支持操作系统正常运行的栈
> >   - 这说明需要一个用户程序不能直接修改的栈来存储中断上下文
> > - 运行两个线程。在两个线程切换的时候，会需要切换页表。但是此时操作系统运行在前一个线程的栈上，一旦切换，再访问栈就会导致缺页，因为每个线程的栈只在自己的页表中
> >   - 每个栈的访问需要借助虚拟地址
> >   - 如果使用同一个栈，那么切换页表后的映射关系就不对了
> > - 用户进程巧妙地设计 `sp`，使得它恰好落在内核的某些变量附近，于是在保存寄存器时就修改了变量的值。这相当于任意修改操作系统的控制信息
> >   - 还是不安全的问题，因为没有对系统信息进行隔离，同一个栈里面访问控制不好实施

#### 实验

实验：当键盘按下 Ctrl + C 时，操作系统应该能够捕捉到中断。实现操作系统捕获该信号并结束当前运行的线程（你可能需要阅读一点在实验指导中没有提到的代码）

> > 参考：https://github.com/yunwei37/os-summer-of-code-daily/blob/master/daily_documents/Day21_lab4_practice.md
>
> 这一题和下一题需要捕捉键盘输入。
>
> 首先 `handler.rs` 中找到外部中断：
>
> ![image-20200725213235014](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725213235014.png)
>
> 先了解 `Ctrl + C` 对应的键值：https://blog.csdn.net/softimite_zifeng/article/details/53259542。从中可以知道，Ctrl+C（3）。
>
> 故，我们需要特别处理键盘输入键值为3的情况。
>
> ![image-20200725214512851](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725214512851.png)

实验：实现线程的 `clone()`。目前的内核线程不能进行系统调用，所以我们先简化地实现为“按 C 进行 clone”。clone 后应当为目前的线程复制一份几乎一样的拷贝，新线程与旧线程同属一个进程，公用页表和大部分内存空间，而新线程的栈是一份拷贝。

> 首先，实现键盘中断响应：
>
> ![image-20200725215011781](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725215011781.png)
>
> 然后在进程控制中实现 clone 函数，
>
> ![image-20200725215351382](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725215351382.png)
>
> 最后在线程中具体实现clone功能，（和线程里面的new类似地实现即可）注意：新线程与旧线程**同属一个进程**，**公用页表和大部分内存空间**，而新线程的**栈是一份拷贝**。
>
> ![image-20200725221124229](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200725221124229.png)
>
> BUG：注意不能直接命名为 clone 函数，否则会与现有的克隆函数冲突。重命名为 `clone_`。

实验：了解并实现 Stride Scheduling 调度算法，为不同线程设置不同优先级，使得其获得与优先级成正比的运行时间。

> > https://www.bookstack.cn/read/ucore_os_docs/lab6-lab6_3_6_2_priority_queue.md
>
> Stride Scheduling 调度算法似乎需要用到优先队列结构。不妨先实现朴素的stride算法（**无优先队列**）。
>
> 新建`stride_scheduler.rs`：（有瑕疵）

```rust
//! Stride Scheduling的调度器 [`StrideSchduler`]

use super::Scheduler;
use alloc::vec::Vec;

/// 将线程和调度信息打包
struct StrideThread<ThreadType: Clone + Eq> {
    stride: usize,
    BigStride: usize,
    priority: usize,  // pass = BigStride / priority
    /// 线程数据
    pub thread: ThreadType,
}

/// 采用 Stride Scheduling 算法的线程调度器
pub struct StrideScheduler<ThreadType: Clone + Eq> {
    /// 带有调度信息的线程池
    pool: Vec::<StrideThread<ThreadType>>,
}

/// `Default` 创建一个空的调度器
impl<ThreadType: Clone + Eq> Default for StrideScheduler<ThreadType> {
    fn default() -> Self {
        Self {
            pool:  Vec::<StrideThread<ThreadType>>::new(),
        }
    }
}

impl<ThreadType: Clone + Eq> Scheduler<ThreadType> for StrideScheduler<ThreadType> {
    type Priority = ();

    fn add_thread(&mut self, thread: ThreadType) {
        self.pool.push(StrideThread {
            stride: 0,
            BigStride: 23333,
            priority: 100,    // 暂时定为常数,因为ThreadType时抽象的，无法使用线程中保存的优先级
            thread,
        })
    }
    fn get_next(&mut self) -> Option<ThreadType> {
        // 遍历线程池，返回stride最小者
        if let Some(best) = self.pool.iter_mut().min_by(|x, y| {
            x.stride.cmp(& y.stride)
        }) {
            best.stride += best.BigStride / best.priority;
            Some(best.thread.clone())
        } else {
            None
        }
    }
    fn remove_thread(&mut self, thread: &ThreadType) {
        // 移除相应的线程并且确认恰移除一个线程
        let mut removed = self.pool.drain_filter(|t| t.thread == *thread);
        assert!(removed.next().is_some() && removed.next().is_none());
    }
    fn set_priority(&mut self, _thread: ThreadType, _priority: ()) {}
}
```

## Lab5：块设备和文件系统

> [**Lab5 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-5/guide/intro.html)

### 设备树

在 RISC-V 中，操作系统通过 bootloader，即 OpenSBI 固件完成以设备树的格式管理全部已接入设备信息的功能。它来完成对于包括物理内存在内的各外设的扫描，将扫描结果以**设备树二进制对象（DTB，Device Tree Blob）**的格式保存在物理内存中的某个地方。

![image-20200726110115158](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200726110115158.png)

这个结构其实跟文件系统很像。

> 每个设备在物理上连接到了父设备上最后再通过总线等连接起来构成一整个设备树，在每个节点上都描述了对应设备的信息，如支持的协议是什么类型等等。而操作系统就是通过这些节点上的信息来实现对设备的识别的。

因为整个是一个树结构，所以在解析设备树获取节点信息时，可以直接采用简单的递归函数，即树遍历算法。如下代码所示：（其余代码参考实验书）

```rust
/// 递归遍历设备树
fn walk(node: &Node) {
    // 检查设备的协议支持并初始化
    if let Ok(compatible) = node.prop_str("compatible") {
        if compatible == "virtio,mmio" {
            // 在遍历过程中，一旦发现了一个支持 "virtio,mmio" 的设备
            // （其实就是 QEMU 模拟的存储设备），就进入下一步加载驱动的逻辑。
            virtio_probe(node);
        }
    }
    // 遍历子树
    for child in node.children.iter() {
        walk(child);        // 【递归】
    }
}
```

#### 挂载

QEMU支持挂载设备树。只不过用了一个 `virtio` 半虚拟化技术架构：

![image-20200726180311493](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200726180311493.png)

> 以 virtio 为中心的总线下又挂载了 virtio-blk（块设备）总线、virtio-net（网络设备）总线、virtio-pci（PCI 设备）总线等，本身就构成一个设备树。

当然，要启用QEMU挂载功能，需要调用相应的命令：

```makefile
# 运行 QEMU
qemu: build
    @qemu-system-riscv64 \
            -machine virt \
            -nographic \
            -bios default \
            -device loader,file=$(BIN_FILE),addr=0x80200000 \
            -drive file=$(TEST_IMG),format=raw,id=sfs \      # 模拟存储设备，TEST_IMG 是特定文件系统格式的磁盘镜像
            -device virtio-blk-device,drive=sfs              # 以 virtio Block Device 的形式挂载到 virtio 总线上
```

接着便是一些驱动代码细节。。

#### 思考

为什么物理地址到虚拟地址转换直接线性映射，而虚拟地址到物理地址却要查表？

> 答案中提到：在内核线程里面，只要一个物理地址加上偏移得到的虚拟地址肯定是可以访问对应的物理地址的。所以，把物理地址转为虚拟地址加个偏移既可。
>
> 这说明，如果物理地址有效，那么**线性映射是肯定存在的**，这时虚拟地址也一定有效（是可能的物理地址对应的虚拟地址之一）。但是反过来，虚拟地址却不一定是最初的线性映射形成的，因此不能直接读写，必须通过页表查询。

### 驱动、块设备

**块设备**，即以块为单位进行单次读写操作，这样每次读取的效率更高。因为硬盘的读取具有局部性，如果每次只读一个字节，那么花在寻道时间等等上的成本就会很大。所以，一次读一波~

**驱动**，就是负责对设备的管理和访问，在驱动中要实现诸如：获取设备类型信息，读取某个块到缓冲区，将缓冲区的数据写入某个块，获取设备树上的设备信息并保存等等。

**抽象块设备**，其实就是提供给文件系统的一个高级接口，在这一层隐去了驱动、块设备的诸多细节，只保留了几个封装好的函数，供上层文件系统调用。

![image-20200726183637116](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200726183637116.png)

### 文件系统

> 之前我们在加载 QEMU 的时候引入了一个磁盘镜像文件，这个文件的打包是由 [rcore-fs-fuse 工具](https://github.com/rcore-os/rcore-fs/tree/master/rcore-fs-fuse) 来完成的，它会根据不同的格式把目录的文件封装成到一个文件系统中，并把文件系统封装为一个磁盘镜像文件。然后我们把这个镜像文件像设备一样挂载在 QEMU 上，QEMU 就把它模拟为一个块设备了。接下来我们需要让操作系统理解块设备里面的文件系统。

由上述可知，我们需要在块设备中分析文件系统。

文件系统已经有了大量前人的实现。所以，采用了一个模板  Simple File System。（[前人的分析](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-5/files/rcore-fs-analysis.pdf)）

最后加入测试代码，试着运行一下，看看效果：（`PROCESSOR.lock().run()` 这个方法并没有任何地方实现了，所以要改成其他的方法）

![image-20200726185907886](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200726185907886.png)

### 实验五实验题

> 实验五暂时没有实验题

## Lab6：加载执行文件形成用户进程

> [**Lab6 实验指导--rcore tutorial教程第三版**](https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-6/guide/intro.html)

我们成功单独生成 ELF 格式的用户程序，并打包进文件系统中；同时，从中读取，创建并运行用户进程；而为了可以让用户程序享受到操作系统的功能，我们使用系统调用为用户程序提供服务。

### 打包

用户程序框架和实验准备中为操作系统「去除依赖」的工作十分类似。只不过需要新建一个 `user` 专用文件，与 `os` 文件夹相互独立、并列。

```sh
$ cargo new --bin user
```

> 和操作系统一样，我们需要为用户程序移除 std 依赖，并且补充一些必要的功能。
>
> `lib.rs`：
>
> - `#![no_std]` 移除标准库
> - `#![feature(...)]` 开启一些不稳定的功能
> - `#[global_allocator]` 使用库来实现动态内存分配
> - `#[panic_handler]` panic 时终止
>
> 其他文件：
>
> - `.cargo/config` 设置编译目标为 RISC-V 64
> - `console.rs` 实现 `print!` `println!` 宏

安装 `rcore-fs-fuse` 工具：

```sh
$ cargo install rcore-fs-fuse --git https://github.com/rcore-os/rcore-fs
```

在 `user/Makefile` 里面设置打包的命令：

```makefile
build: dependency
    # 编译
    @cargo build
    @echo Targets: $(patsubst $(SRC_DIR)/%.rs, %, $(SRC_FILES))
    # 移除原有的所有文件
    @rm -rf $(OUT_DIR)
    @mkdir -p $(OUT_DIR)
    # 复制编译生成的 ELF 至目标目录
    @cp $(BIN_FILES) $(OUT_DIR)
    # 使用 rcore-fs-fuse 工具进行打包
    @rcore-fs-fuse --fs sfs $(IMG_FILE) $(OUT_DIR) zip
    # 将镜像文件的格式转换为 QEMU 使用的高级格式
    @qemu-img convert -f raw $(IMG_FILE) -O qcow2 $(QCOW_FILE)
    # 提升镜像文件的容量（并非实际大小），来允许更多数据写入
    @qemu-img resize $(QCOW_FILE) +1G
```

### 用户进程

> 在之前实现内核线程时，我们只需要为线程指定一个起始位置就够了，因为所有的代码都在操作系统之中。但是现在，我们需要从 ELF 文件中加载用户程序的代码和数据信息，并且映射到内存中。

在 lab6 中，用户程序需要从文件中获取，而不是之前的手动创建了。对应的是 ELF 文件解析器，因为有 `xmas-elf` 这个 crate 替我们实现了 ELF 的解析，所以直接调用就行了。

读取文件内容：（将文件整个读到一个向量里面然后返回）

```rust
fn readall(&self) -> Result<Vec<u8>> {
    // 从文件头读取长度
    let size = self.metadata()?.size;
    // 构建 Vec 并读取
    let mut buffer = Vec::with_capacity(size);
    unsafe { buffer.set_len(size) };
    self.read_at(0, buffer.as_mut_slice())?;
    Ok(buffer)
}
```

然后解析各个字段。。代码有点长就不贴了。

思考：我们在为用户程序建立映射时，虚拟地址是 ELF 文件中写明的，那物理地址是程序在磁盘中存储的地址吗？这样做有什么问题吗？

> 肯定是不行的，这样搞的话，每次物理地址解析完还要访问磁盘，众所周知，磁盘的读写炒鸡慢。所以这样搞，系统的性能就太低了。所以要将文件内容加载进入内存，并以内存中的物理地址为准。这样，便也就涉及到了页面置换等等优化的问题。

对于一个页面，有其**物理地址**、**虚拟地址**和**待加载数据的地址**。此时，是不是直接从**待加载数据的地址**拷贝到页面的**虚拟地址**，如同 `memcpy` 一样就可以呢？

> 当然是不行的。因为首先一个页面可能具有多个虚拟地址！那么你要拷贝到那个虚拟地址呢？如果页表加载的不同，那么同一个虚拟地址可能有不同的映射，可能就访问不到页面所在的真正的物理地址了。所以这里必须用物理地址来写入数据，确保正确性。

### 系统调用

系统调用通过一些中断性的设计来完成一些功能。在后面实验题的部分也有做到。

首先系统调用底层需要用到一定的汇编。可以看指导书。系统调用通常会返回三类处理结果：一个数值、程序进入等待、程序被终止。

后面的条件变量暂时先这样把。。条件变量在这里的作用就是：

> 为输入流加入条件变量后，就可以使得调用 `sys_read` 的线程在等待期间保持休眠，不被调度器选中，消耗 CPU 资源。

这似乎是可以解决之前 Stride Scheduling 算法的缺陷，因为之前的调度算法没有对等待期的线程做处理，因此可能出现所有高 stride 的线程被迫等待，CPU 资源闲置的情况。

### 实验六实验题

> https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-6/practice.html

#### 原理

原理：使用条件变量之后，分别从线程和操作系统的角度而言读取字符的系统调用是阻塞的还是非阻塞的？

> 跟解答差不多。对于线程而言，是阻塞的，因为需要等待系统调用结束。对于操作系统，等待输入的时间完全分配给了其他线程，所以对于操作系统来说是非阻塞的（操作系统似乎很难发生阻塞，除非所有的进程都阻塞了，否则总是可以通过调度实现运行）。

#### 设计*

设计：如果要让用户线程能够使用 `Vec` 等，需要做哪些工作？如果要让用户线程能够使用大于其栈大小的动态分配空间，需要做哪些工作？

> 首先需要支持用户态的堆空间预分配，然后让 `Vec` 访问这个堆空间。
>
> > 应当要在用户部分实现 #[global_allocator] ：包含 [`alloc::alloc::GlobalAlloc`] trait等
>
> 要让用户线程能够使用大于其栈大小的动态分配空间，需要设计一个相应的用户进程的堆空间实现。

#### 实验

> 参考：https://github.com/yunwei37/os-summer-of-code-daily/blob/master/daily_documents/Day23_lab6_practice.md。

实验：实现 `get_tid` 系统调用，使得用户线程可以获取自身的线程 ID。

> 随便设定一个获取自身的线程 ID 的系统调用号。
>
> 在 `user/src/syscall.rs` 添加对应的系统调用：

```rust
const SYSCALL_GETTID: usize = 233;

pub fn sys_get_tid() -> isize {
    syscall(SYSCALL_GETTID, 0, 0, 0)
}
```

> 然后在 `os/src/kernel/syscall.rs` 中实现具体的系统调用接口：

```rust
pub const SYSCALL_GETTID: usize = 233;
...
let result = match syscall_id {
        ...
        SYS_GETTID => sys_get_tid(),
        ...
    };
```

> 最后在 `os/src/kernel/process.rs` 中具体实现系统调用：

```rust
use super::*;

pub(super) fn sys_exit(code: usize) -> SyscallResult {
    println!(
        "thread {} exit with code {}",
        PROCESSOR.lock().current_thread().id,
        code
    );
    SyscallResult::Kill
}

pub(super) fn sys_get_tid() -> SyscallResult {   // sys_exit 已经描述了 id 如何获取
    SyscallResult::Proceed(
        PROCESSOR.lock().current_thread().id.clone()
    )
}
```

实验：将你在实验四（上）实现的 `clone` 改进成为 `sys_clone` 系统调用，使得该系统调用为父进程返回自身的线程 ID，而为子线程返回 0。

> 同理。随便设定一个 `sys_clone` 系统调用号。
>
> 在 `user/src/syscall.rs` 添加对应的系统调用：

```rust
const SYS_CLONE: usize = 110;

pub fn sys_clone() -> isize {
    syscall(SYS_CLONE, 0, 0, 0)
}
```

> 然后在 `os/src/kernel/syscall.rs` 中实现具体的系统调用接口：

```rust
pub const SYS_CLONE: usize = 110;
...
let result = match syscall_id {
        ...
        SYS_CLONE => sys_clone(context),
        ...
    };
```

> 最后在 `os/src/kernel/process.rs` 中具体实现系统调用：

```rust
pub(super) fn sys_clone(context: &Context) -> SyscallResult {
    let id = PROCESSOR.lock().current_thread().id.clone();
    PROCESSOR.lock().clone_current_thread(context);
    if PROCESSOR.lock().current_thread().id.clone() == id {
        SyscallResult::Proceed(id)
    }else{
        SyscallResult::Proceed(0)
    }
}
```

实验：将一个文件打包进用户镜像，并让一个用户进程读取它并打印其内容。需要实现 `sys_open`，将文件描述符加入进程的 `descriptors` 中，然后通过 `sys_read` 来读取。

> 随便设定一个 `sys_open` 系统调用号。
>
> 在 `user/src/syscall.rs` 添加对应的系统调用：

```rust
const SYSCALL_OPEN: usize = 120;

pub fn sys_open(file: &str) -> isize {
    syscall(
        SYSCALL_OPEN,
        0,
        file.as_ptr() as *const u8 as usize,
        file.len(),
    )
}
```

> 然后在 `os/src/kernel/syscall.rs` 中实现具体的系统调用接口：

```rust
pub const SYSCALL_OPEN: usize = 120;
...
let result = match syscall_id {
        ...
        SYSCALL_OPEN => sys_open(context),
        ...
    };
```

> 最后在 `os/src/kernel/fs.rs` 文件系统中具体实现系统调用：（不太会。。）

```rust
use crate::ROOT_INODE;

pub(super) fn sys_open(buffer: *mut u8, size: usize) -> SyscallResult {
    let file_name = unsafe {
        let slice = slice::from_raw_parts(buffer, size);
        str::from_utf8(slice).unwrap()
    };
    // 从文件系统中找到文件描述符
    let file = ROOT_INODE.find(file_name).unwrap();

    let process = PROCESSOR.lock().current_thread().process.clone();
    process.inner().descriptors.push(file);   // 加入文件描述符
    
    SyscallResult::Proceed(
        (PROCESSOR
            .lock()
            .current_thread()
            .process
            .clone()
            .inner()
            .descriptors
            .len()
            - 1) as isize,
    )
}
```

> 将一个文件打包进用户镜像：（根据实验指导书：https://rcore-os.github.io/rCore-Tutorial-deploy/docs/lab-6/guide/part-2.html），编写MAkefile：

```makefile
TEST_FILE	:= test.file

build: dependency
    # 编译
    @cargo build
    @echo Targets: $(patsubst $(SRC_DIR)/%.rs, %, $(SRC_FILES))
    # 移除原有的所有文件
    @rm -rf $(OUT_DIR)
    @mkdir -p $(OUT_DIR)
    # 复制编译生成的 ELF 至目标目录
    @cp $(BIN_FILES) $(OUT_DIR)
    @cp $(TEST_FILE) $(OUT_DIR)
    # 使用 rcore-fs-fuse 工具进行打包
    @rcore-fs-fuse --fs sfs $(IMG_FILE) $(OUT_DIR) zip
    # 将镜像文件的格式转换为 QEMU 使用的高级格式
    @qemu-img convert -f raw $(IMG_FILE) -O qcow2 $(QCOW_FILE)
    # 提升镜像文件的容量（并非实际大小），来允许更多数据写入
    @qemu-img resize $(QCOW_FILE) +1G
```

![image-20200726014415664](%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%9A%91%E6%9C%9F%E9%A1%B9%E7%9B%AE/image-20200726014415664.png)

> 最后在 main 函数中调试，遇到一些格式上的困难。暂时放弃了。

#### 挑战实验

挑战实验：实现 `sys_pipe`，返回两个文件描述符，分别为一个管道的读和写端。用户线程调用完 `sys_pipe` 后调用 `sys_fork`，父线程写入管道，子线程可以读取。读取时尽量避免忙等待。

> 先放着了。。

## OS实习第三次交流会

- 老师介绍第二阶段鹏城实验室实习的准备工作
  - 15到20人的规模左右
  - 28号左右完成一个check（26号提交问卷，27号修改完毕）
  - 没有拿到《**复课证明**》的折衷方案：
    - 只要本人被同意而做好规划，但是只有**<font color=red>一周</font>**的时间（黑客马拉松的形式）
      - 有《复课证明》可以待一个月，发工牌
    - 每个同学签署自我安全协议书
    - 来回车票、食宿**报销**，有实习劳务费
      - 高铁/航班（二等座、经济舱+登机牌）。
      - 宿舍两人一间。
      - 深圳天气较热。
      - 实验室有食堂。
  - **总结报告**：今天至少提交一个版本，可以继续更新
  - 以后的实习机会优先考虑
- 学生提问与交流

# ==第一阶段总结==

> 博客记录：[操作系统暑期项目](https://vel.life/操作系统暑期项目/)。

简要自身情况介绍：我是计算机科学爱好者，学生，机缘凑巧听说了 rCore 的暑期实习项目，本身也没有别的要紧的事情，于是决定来参与这个活动。可以说在很大的程度上达到了我想要的效果吧，虽然离群里的大佬还有很大的差距，但我对于我自己的收获还是比较认可的。虽然少，但是实在。

总的来说，第一阶段确实学了一些东西，但是相对来说，又学得偏少；Rust 只是掌握了最基础的一些语法，大概是那种能通过编译、有一定正确性的程度，但是离熟练掌握 Rust 还有一定距离。Rust的编译检查在最开始可能确实有些“反人类”，可是做完 rustlings，做完 15 道编程题，慢慢地通过编译变得容易多了，也会更加注重编译出错时地提示，通常这些提示都会很贴心。在这样的“与编译器作斗争”的过程中，编程水平也许有了无形的提高也说不定。虽然我选的编程题比较简单，都是从 LeetCode 上摘取的简单题、中等题。Rust的语法特性有一些确实很好用，比如模式匹配系列（`match`，`if let`，`while let`，……），用得好便会有奇妙的逻辑效果，尤其是与 `Option` 这种枚举类型配合时，更让人感觉到语言的有美感。Go 语言作为另一门现代系统级编程语言，通常有着非常固定的、专属的编程范式，那么 Rust 语言会不会在将来形成自己的编程范式呢？至少现在来看，`fmt` 相关的工具只是对代码的样式风格做了标准化，离代码的逻辑风格还差一些。具体的lab实验中，也会用到 Rust 的各种特性。在以后，如果有机会继续深入学习的话，可能会对语言的设计产生更深刻的理解。但是目前，到此为止也还不错。

而RISC-V方面我也只是粗看了皮毛，仗着自己在MIPS和x86汇编方面的知识，倒也暂时没有遇到太多的困难；但是，要说显著的进步，确实没有了。RISC-V作为精简指令集，在设计思路上其实跟 MIPS 的区别不大，可能它的优势就在于历史包袱小、开源，但说实话真正从技术层面上来分析却没有什么特别之处。RISC-V 更像是一个开源运动的产物，就像 Linux 一样，具有广泛的社区和生命力。RISC-V 在陈渝老师这边强调的是特权架构，不过也没感觉到什么特别的地方，跟 x86 的特权级体系还是蛮像的，也许是多了一两个特权级？我这里想到了网络的分层体系，有七层的 OSI 体系，也有五层的 TCP/IP 体系，其实操作系统的特权架构和网络系统的分层两者还是挺像的，从中也可以看出特权级究竟分多少级其实完全取决于现实需求，而没有什么理论上的特别限制，只要做到对不同层级之间的功能的清晰划分便足矣。

在做 rCore 实验方面，由于时间的提前，打乱了之前的计划，在研究了前几个 lab 之后（lab1 - lab3），只好匆匆地跑通后面几个 lab（然后做实验题去了），而实验书的对应章节却几乎来不及看了（得知延期一天，不妨抽点时间浅浅地看一番）。所以在实验方面，有大量的代码细节，没有时间去看，这可能会对第二阶段的 zCore（如果可能的话）产生比较严重的影响。在学习之余，我也参与了少量的微信群讨论和 issue 上的提问与回答，还提交了几个简单的 Pull Request，目前都已经被合并了。因为之前自学过操作系统和 ucore 系列实验，所以在实验的理论方面没有遇到太大的障碍，反倒是代码细节上能力有些捉襟见拙——操作系统真是一个注重实践大于理论的学科。目前来说，我感觉到操作系统的编写过程中，DEBUG 是一个非常要命的事情，我至今还不是很会用 gdb，由于禁用 std，在单元测试的时候也经常被迫只能选择 `assert!` 之类的断言的形式。至少以我以前维护 Java 项目和 Python 项目的经验来看，Rust 项目的测试功能还是比较麻烦的（也可能是我不熟悉或者不知道更好的 DEBUG 方法）。

这一个月，说快也快。中途还摸鱼划水了一段时间。整体来说，花在操作系统暑期实习上的时间并不是特别多，因此收获相对来说还是对得起付出的时间的。也许多花点精力，可以把实验做完；或者做一点微小的贡献和改进；或者帮助解决更多的问题……但是，时间都已经过去了。现在也只能唏嘘。不管怎么说，这一个阶段也总算结束了。不管之后能不能入选第二个阶段的实习，我都已经很满足了（即使不能入选，也有其它的事情要做，所以并不慌张~）。像我这样佛系的态度来面对科研可能差点火候，但是对于喜欢的、有兴趣的定西，这样的态度不也可能成为长燃的火烛么。许多年以后，我会庆幸自己曾经在某个夏伏天，写过一些文字，写过一些代码……那已经极好了。