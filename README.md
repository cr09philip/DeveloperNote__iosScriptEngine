Notebooks for ios jailbreak developer
==========================================


NOTE:this is a project to record my notebooks during developer an lua script engine.
only text, no code in this project.

##1.How To Create a suspend Window like AssistiveTouch
  & How to make the window globally
  & How to let the window Autorotate with current App
##2.Research the implementation of other apps like touchelf,touchsprite,and xxAssistant.
##3.ios IPCs
##4.socket programming 
##5.native implementation
##6.lua source and how communication between lua and c
##7.error handle
##8.notification observer

## 9.工程打包  
使用dpkg-deb命令，将编译后的文件打包成deb。 [DEBIAN包管理系统](./DEBIAN_FORMAT.MD)  

- 需要注意的是，打包前需要先设置好文件的权限及所有者。  
``` sh
chown -R root:wheel ./dir/*  
chmod -R +x ./dir/files  
chmod u+s ./dir/file  
```
chown 是用来更改文件的所有者和所属组。  
chown +x 是用来为可执行文件设置属性。  
chown u+x 是用来为文件添加[设置用户ID属性](./unix_file_mode.md)。

- 然后，使用dpkg-deb 打包
``` sh
dpkg-deb -Zgzip -b path dest.deb
```
可以通过ssh连接iOS，使用dpkg命令安装deb包。
``` sh
dpkg -i dest.deb
```
