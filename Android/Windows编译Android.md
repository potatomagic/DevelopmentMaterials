# 引言
​        通常情况下，我们都是使用Linux系统进行整个AOSP的编译，但也有特殊情况，如Windows系统来编译，在Windows下编译，如果电脑性能特别特别特别的好，重要的事情说三遍！这时可以使用虚拟机来运行Ubuntu等Linux系统，直接进行编译。当然，这种情况对于大多数人来说是不可能的，这时候我们就要使用WSL来编译。
### WSL（Windows Subsystem for Linux）
WSL是一个在Windows 10上能够运行原生Linux二进制可执行文件（ELF格式）的兼容层。它是由微软与Canonical公司合作开发，其目标是使纯正的Ubuntu映像能下载和解压到用户的本地计算机，并且映像内的工具和实用工具能在此子系统上原生运行。

安装步骤参考官网：https://ubuntu.com/tutorials/ubuntu-on-windows 。

这里也记录几个重要的步骤

- enable WSL 1 in PowerShell as Administrator:
  - dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
- enable WSL 2 in PowerShell as Administrator:
  - dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
- Restart Windows 10
- Install Ubuntu for Windows 10（提供几个地址，不保证有效，最好直接Microsoft Store商店搜索Ubuntu下载）
  - https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6
  - https://wsldownload.azureedge.net/Ubuntu_1604.2019.523.0_x64.appx

### 使用WSL的注意事项
- 修改WSL软件源（做好备份）（示例为Ubuntu 16.04的软件源）
  - sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
  - sudo vi /etc/apt/sources. List
    ```xml
      #deb包
      deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
      deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
      deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
      deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
      ##测试版源
      deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
      # 源码
      deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
      deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
      deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
      deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
      ##测试版源
      deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
      # Canonical 合作伙伴和附加
      deb http://archive.canonical.com/ubuntu/ xenial partner
      deb http://extras.ubuntu.com/ubuntu/ xenial main
    ```
- Build AOSP nedd at least 300 GB of free disk space is required （默认情况下，WSL是安装在系统盘，即C盘，如果空间不够，需要进行WSL迁移，建议尽早迁移，不要等C盘炸了再迁移，容易迁移失败）
  - https://github.com/DDoSolitary/LxRunOffline/releases 下载Releases Zip包
  - 在cmd.exe中查询wsl名称，“wsl -l”，假如查询到：“Ubuntu-16.04 (默认)”，wsl名字为Ubuntu-16.04
  - 停止WSL服务，（CTRL + SHIFT + ESC）进入服务，找到 LxssManager 右键停止
  - .\LxRunOffline.exe move -n Ubuntu-16.04 -d D:\ubuntu  （将你已安装的WSL迁移到指定目录）
- 建议使用 WSL 2，WSL 2 需要高版本的 Windows 10 来支持，
  - https://devblogs.microsoft.com/commandline/wsl-2-support-is-coming-to-windows-10-versions-1903-and-1909/
  - https://docs.microsoft.com/en-us/windows/wsl/install-win10#update-to-wsl-2

### 配置WSL的Display



### 编译 AIO-3399J Android7.1
- 环境搭建 http://wiki.t-firefly.com/zh_CN/AIO-3399J/prepare_compile_android.html
  - 值得注意的是，可能会有一些依赖被移除，可以去Ubuntu官网查看并添加安装源
    - https://packages.ubuntu.com/
    - 16.04查看的软件版本是Xenial Xerus
    - 18.04查看的软件版本是Bionic Beaver
    - 20.04查看的软件版本是Focal Fossa
  - 设置Ubuntu 64位系统执行32位文件
    ```cmd
      sudo dpkg --add-architecture i386
      sudo apt install libc6:i386
    ```
  - 设置Bash支持32位程序运行，需要安装依赖binfmt
      ```cmd
         sudo apt update
         sudo apt install qemu-user-static
         sudo update-binfmts --install i386 /usr/bin/qemu-i386-static --magic '\x7fELF\x01\x01\x01\x03\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\x03\x00\x01\x00\x00\x00' --mask '\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xf8\xff\xff\xff\xff\xff\xff\xff'
      ```
    - 启动binfmt服务，每次都要启动，建议设置成脚本
      - sudo service binfmt-support start
      - 否则会报错：-bash: ./XXXXX: cannot execute binary file: Exec format error
- 可能需要把源码目录设置setCaseSensitiveInfo标志，因为 Linux 是区分大小写的，而 Windows 10 默认是不区分大小写的，避免因为在同一路径下同时建立 ABC 和 abc 文件夹因重名失败。
    ``` cmd
    C:\Windows\system32> fsutil.exe file SetCaseSensitiveInfo D:\ubuntu\AOSP enable
    ```
- 编译过程中，可能会报出 Permission denied 的错误，这可能需要使用系统管理员权限来运行Bash，当然需要仔细区分是何种原因没有权限
- 报错：ftruncate(fd_out, GetSize()): Invalid argument，原因ijar版本问题，
  - 方法一：修改 build/tools/ijar/zip.cc 文件

      ```text
        void *zipdata_out = mmap(NULL, mmap_length, PROT_WRITE,
        -                        MAP_SHARED, fd_out, 0);
        +                        MAP_SHARED | MAP_ANONYMOUS, fd_out, 0);  
      ```

  - 方法二：使用 https://github.com/bazelbuild/bazel/tree/master/third_party/ijar 来编译替换
- 报错：dex2oat did not finish after 2850 seconds，原因可能是odex生成失败，通常使用单线程编译即可
  - 修改  build/core/dex_preopt_libart_boot.mk

      ```text
        +SINGLE_THREAD := "-j1"
        # Use dex2oat debug version for better error reporting

        -       $(hide) ANDROID_LOG_TAGS="*:e" $(DEX2OAT) --runtime-arg -Xms$(DEX2OAT_IMAGE_XMS) \
        +       $(hide) ANDROID_LOG_TAGS="*:e" $(DEX2OAT) $(SINGLE_THREAD) --runtime-arg -Xms$(DEX2OAT_IMAGE_XMS) \
                                                --runtime-arg -Xmx$(DEX2OAT_IMAGE_XMX) \
                                                --image-classes=$(PRELOADED_CLASSES) \
      ```
  - 修改 build/core/dex_preopt_libart.mk

      ```text
        +SINGLE_THREAD := "-j1"
         define dex2oat-one-file
         $(hide) rm -f $(2)
         $(hide) mkdir -p $(dir $(2))
        -$(hide) ANDROID_LOG_TAGS="*:e" $(DEX2OAT) \
        +$(hide) ANDROID_LOG_TAGS="*:e" $(DEX2OAT) $(SINGLE_THREAD) \
                --runtime-arg -Xms$(DEX2OAT_XMS) --runtime-arg -Xmx$(DEX2OAT_XMX) \
                --runtime-arg -classpath --runtime-arg $(DEX2OAT_CLASSPATH) \
      ```

  - 如果设置了单线程编译，还是有报错，则需要在报错app或者jar包中Android.mk添加Flag：LOCAL_DEX_PREOPT := false
- 报错：File not found，主要涉及的文件有如下几个:
    ```text
      cts/tests/tests/security/res/raw/cve_2015_1538_1.mp4
      cts/tests/tests/security/res/raw/cve_2015_1538_2.mp4
      cts/tests/tests/security/res/raw/cve_2015_1538_3.mp4
      cts/tests/tests/security/res/raw/cve_2015_1538_4.mp4
      cts/tests/tests/security/res/raw/cve_2015_1539.mp4
      cts/tests/tests/security/res/raw/cve_2015_3824.mp4
      cts/tests/tests/security/res/raw/cve_2015_3828.mp4
      cts/tests/tests/security/res/raw/cve_2015_3829.mp4
      cts/tests/tests/security/res/raw/cve_2015_3864.mp4
      cts/tests/tests/security/res/raw/cve_2015_3864_b23034759.mp4
    ```
    - 经过调研，发现Windows系统判定这几个文件具有 Exploit:AndroidOS/StageFright 和 Exploit:AndroidOS/StageFright!rfn 威胁，将这些文件隔离起来，解决办法是在Windows Defender中通过这些文件即可。
    - 关于Stagefright漏洞的介绍，可以查阅以下链接
      - https://www.freebuf.com/articles/terminal/73517.html
      - https://www.freebuf.com/vuls/73791.html
- 报错：ninja: error: 'art/runtime/interpreter/mterp/out/mterp_x86_64.S', needed by 'out/host/linux-x86/obj/SHARED_LIBRARIES/libart_intermediates/interpreter/mterp/out/mterp_x86_64.o', missing and no known rule to make it
  ```cmd
    cd art/runtime/interpreter/mterp/
    ./rebuild.sh
  ```
- 报错：You have tried to change the API from what has been previously approved.
  - 该错误解决办法在错误提示中已经给出，有两个方法
    - 将该接口加上 非公开的标签：/*{@hide}/；
    - 在修改后执行：make update-api 命令，将修改内容与API的doc文件更新到一致即可。


### 搭建 AOSP Mirror 服务器



### 编译 AOSP
- 报错：Build sandboxing disabled due to nsjail error.
  - 原因，WSL 未升级到 WSL 2，升级 WSL 2 需要 Windows 高版本
  -

### Ubuntu 20.04 编译 AOSP
- http://source.android.com/source/requirements.html
- http://source.android.com/source/downloading.html
##### 配置环境（以下为必须，编译看情况）
``` cmd
sudo apt-get install git
sudo apt-get install openjdk-8-jdk
sudo apt-get install libncurses5
sudo apt-get install libncurses5:i386

git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
##### 下载源码
参考网址
- 清华大学开源软件镜像站 https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/
- 北京外国语大学开源软件镜像站: https://mirrors.bfsu.edu.cn/help/AOSP/
- 中国科学技术大学: https://lug.ustc.edu.cn/wiki/mirrors/help/aosp/

##### 下载方法
先要获取 repo ，以清华大学开源软件镜像站为例
```cmd
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > /usr/bin/repo
chmod a+x /usr/bin/repo

/*
 * repo的运行过程中会尝试访问官方的git源更新自己，可能会因为墙的问题访问不了
 * 使用国内镜像源来配置 REPO_URL 变量即可，如：
 */
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

值得注意的是，现在repo运行需要 python 3.6 版本及以上，如果默认的 python指向的是 python 其他版本，需要修改python指向的版本，或者直接指定路径：
```cmd
/usr/bin/python3.6 /usr/bin/repo init ......
```
同步代码通常有两种方法：

方法一：

直接获取AOSP的git地址，使用 repo init 和 repo sync 即可，例如：
```cmd
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
repo sync
```

方法二：

直接获取 .repo 压缩包，然后使用 repo sync -l 命令 checkout 出 AOSP 代码即可，各大镜像站都会给出响应的 .repo 压缩包，或者直接和其他电脑拷贝即可。

##### 编译运行命令
```cmd
source build/envsetup.sh
lunch
make
emulator
```

##### 运行报错汇总以及解决方案
```cmd
emulator : Error : no init encryptionkey.img .
emulator : Error : encryptionkey is requested but failed to create encrypt partition .

// 解决办法
emulator -encryption-key ./device/generic/goldfish/data/etc/encryptionkey.img
```
