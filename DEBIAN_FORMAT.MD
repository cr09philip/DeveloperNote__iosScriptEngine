#DEBIAN包管理系统

DEBIAN软件包，有两种类型：二进制包和源代码包。这里我们只讨论二进制包。

##二进制包结构
包含可执行文件, 配置文件, man/info 手册, 版权信息, 以及其它文档. 以一种 Debian 特有的格式分发, 通常以 '.deb' 作为后缀. 可以使用 Debian 的 dpkg 工具解包(安装)。

相关命令介绍：
    dpkg 是处理 .deb 文件的命令，尤其是解压、分析和解包它们。 
    ar   可以用来创建、修改库，也可以从库中提出单个模块。
    apt 是一组允许对系统进行高级的修改的程序：安装或者删除一个软件包（并且保持依赖关系满足），更新系统，列出可用的软件包等。 

Debian 包的 ar 存档格式由三个文件组成： 
- debian-binary 是一个文本文件，简单的标识deb文件使用的deb版本。
- control.tar.gz 包含所有有效的元信息，即DEBIAN文件夹下的文件。这些文件可以标识包的名称版本等。
- data.tar.gz 这个存档文件包含所有要被从软件包里解压缩出来的文件；这是可执行文件、文档等存储的地方。有些软件包可能会使用其它压缩格式(-Z)，那样的话文件名会有所不同（bzip2 是 data.tar.bz2，XZ 是 data.tar.xz for XZ，LZMA 是 data.tar.lzma）。 


##DEBIAN包元信息

Debian 软件包不仅仅只是一些待安装的文件的存档，它是一个更大的结构的一部分，它描述与其它 Debian 软件包的关系（依赖、冲突、建议）。它还提供脚本供软件包的生命周期的不同阶段（安装、删除、升级）使用。这些供包管理工具使用的数据不是被打包的软件的一部分，但它们存在于软件包里，被称作“元信息”（关于其它信息的信息）。

###Debian二进制软件包的命名格式：
Debian 二进制软件包的命名格式:
	`` <软件包名>_<版本号>-<Debian修订号>.deb`` 

NOTE:在iOS越狱开发过程中，发布的deb包名通常如下：
	``<软件包名>_<版本号>_<平台名>.deb``

###1.control文件
- Package: 包名
- Name:    显示名称
- Version: 给出上游开发者的版本号和修正版本号
- Architecture: 指定这个二进制包的编译硬件平台。iOS为iphoneos-arm
- Depends: 给出依赖的包的列表。此软件包仅当它依赖的软件包均已安装后才可以安装。firmware (>= 8.0)表示依赖iOS8.0以上系统。
- Installed-Size:说明安装这个包所需的磁盘空间，用于安装前端显示是否有足够的空间安装此程序。iOS一般忽略该域。
- Section: 给出此包在ftp上的存储位置。在iOS来说，用来是区分在源中的存储位置，通常用来作分类。
- Priority: 优先级，对应安装的重要程度。Required Important Standard Optional Extra。
- Maintainer: 给出当前维护此包的负责人的电子邮箱。
- Description: 给出此软件包的简要说明。
- Conflicts: 有冲突的软件包，仅当所有冲突的软件包都已经删除后此软件包才可以安装。
- Replaces:当你的程序要替换其他软件包的某些文件，或是完全地替换另一个软件包(与 Conflicts 一起使用)。列出的软件包中的某些文件会被你软件包中的文件所覆盖。  
- Essential: yes
- Author: 该软件包的作者信息。
- Homepage: 软件包主页。
- Depiction: 描述信息。
- Icon:  在cydia中显示的图标。

###2.控制脚本
在软件安装前后自动运行的可执行脚本，统称为控制文件，是Debian软件包的控制部分。

- preinst    Debian软件包(".deb")解压前执行的脚本, 为正在被升级的包停止相关服务,直到升级或安装完成.(成功后执行 'postinst' 脚本).
- postinst   主要完成软件包(".deb")安装完成后所需的配置工作. 通常, postinst 脚本要求用户输入, 和/或警告用户如果接受默认值, 应该记得按要求返回重新配置这个软件. 一个软件包安装或升级完成后, postinst 脚本驱动命令, 启动或重起相应的服务.

- prerm      停止一个软件包的相关进程, 要卸载软件包的相关文件前执行.
- postrm     修改相关文件或连接, 和/或卸载软件包所创建的文件.

当前的所有配置文件都可在 /var/lib/dpkg/info 目录下找到, 与 foo 软件包相关的命名以 "foo" 开头,以 "preinst", "postinst", 等为扩展.这个目录下的 foo.list 文件列出了软件包安装的所有文件.

###3.校验，配置文件列表
control.tar.gz包含一些其他东西。

- md5sums 包含这个包所有文件的md5校验值。
- conffiles 列举了必须作为配置文件保留的文件。

###4.依赖关系

 这里有一篇关于软件包关系的简述：

    Depends

    此软件包仅当它依赖的软件包均已安装后才可以安装。此处请写明你的程序所必须的软件包，如果没有要求的软件包该软件便不能正常运行（或严重抛锚）的话。

    Recommends

     这项中的软件包不是严格意义上必须安装才可以保证程序运行。当用户安装软件包时，所有前端工具都会询问是否要安装这些推荐的软件包。aptitude 和 apt-get 会在安装你的软件包的时候自动安装推荐的软件包(用户可以禁用这个默认行为)。dpkg 则会忽略此项。

    Suggests

    此项中的软件包可以和本程序更好地协同工作，但不是必须的。当用户安装程序时，所有的前端程序可能不会询问是否安装建议的软件包。aptitude 可以被配置为安装软件时自动安装建议的软件>包，但这不是默认。dpkg 和 apt-get 将忽略此项。

    Pre-Depends

    此项中的依赖强于 Depends 项。软件包仅在预依赖的软件包已经安装并且 正确配置 后才可以正常安装。在使用此项时应 非常慎重，仅当在 debian-devel@lists.debian.org 邮件列表讨论后才>能使用。记住：根本就不要用这项。 :-)

    Conflicts

    仅当所有冲突的软件包都已经删除后此软件包才可以安装。当程序在某些特定软件包存在的情况下根本无法运行或存在严重问题时使用此项。

    Breaks

    此软件包安装后列出的软件包将会受到损坏。通常 Breaks 要附带一个“版本号小于多少”的说明。这样，软件包管理工具将会选择升级被损坏的特定版本的软件包作为解决方案。

    Provides

    某些类型的软件包会定义有多个备用的虚拟名称。你可以在 virtual-package-names-list.txt.gz文件中找到完整的列表。如果你的程序提供了某个已存在的虚拟软件包的功能则使用此项。

    Replaces

    当你的程序要替换其他软件包的某些文件，或是完全地替换另一个软件包(与 Conflicts 一起使用)。列出的软件包中的某些文件会被你软件包中的文件所覆盖。


 Debian 的软件包管理系统有一套包"依赖性"概念, 用以标示(一个标志符号)系统中程序 A 对于现存程序 B 的依赖程度:

    软件包 A depends 软件包 B, 指运行 A 必须安装 B, 某些情况下 A 不仅依赖于 B, 还依赖于它的版本. 这种情况通常有最低版限制, A 更依赖于 B 的最新版而不是特定版.

    软件包 A recommends 软件包 B, 如果软件维护者认为用户更喜欢 B 提供功能支持的 A.

    软件包 A suggests 软件包 B, 如果 B 的某些软件与 A 的功能有关(通常是增强).

    软件包 A 与软件包 B conflicts 指如果系统中有 B 则 A 不能运行. 通常是因为 A 包含了 B 中文件的改进而出现冲突, "Conflicts" "replaces" 经常同时出现.

    软件包 A replaces 软件包 B, 指 A 安装时, B 中的文件会被 A 的删除和覆盖.

    软件包 A provides 软件包 B, 指 A 会提供 B 所有的功能和文件, 这种机制为那些磁盘空间受限用户提供了一个方法, 即只安装 A 中需要的部分.



##参考资料
1.[https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html#control](https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html#control)

2.[https://www.debian.org/doc/manuals/debian-faq/ch-pkg_basics.zh-cn.html](https://www.debian.org/doc/manuals/debian-faq/ch-pkg_basics.zh-cn.html)
