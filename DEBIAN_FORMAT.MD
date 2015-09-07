#DEBIAN包管理系统

##二进制包结构
    dpkg 是处理 .deb 文件的命令，尤其是解压、分析和解包它们。 
    ar 
    apt 
Debian 包的 ar 存档格式由三个文件组成： 
**debian-binary
**control.tar.gz
**data.tar.gz


##DEBIAN包元信息


Debian 软件包不仅仅只是一些待安装的文件的存档，它是一个更大的结构的一部分，它描述与其它 Debian 软件包的关系（依赖、冲突、建议）。它还提供脚本供软件包的生命周期的不同阶段（安装、删除、升级）使用。这些供包管理工具使用的数据不是被打包的软件的一部分，但它们存在于软件包里，被称作“元信息”（关于其它信息的信息）。

###1.control文件


###2.配置脚本
preinst postinst prerm postrm


###3.校验，配置文件列表
