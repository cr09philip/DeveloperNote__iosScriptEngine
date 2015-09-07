#UNIX 文件
本文主要介绍一下unix中文件的属性，关注于权限部分。

##文件权限
我们通过ls或者stat可以查看文件的属性。
在struct stat中，有一个域``mode_t st_mode;``,是一个16位无符号蒸熟。

这个uint16_t的值中，高4位(bit12-15)标识了文件的类型(File Type),
低9位分别表示该文件的所有者(owner)的读写执行权限，组(group)的读写执行权限，和其他用户(other)的读写执行权限。

以上应该是比较常接触的。访问权限通常使用chmod更改。

而第10-12位则表示 
bit12: SUID  	当设置了SUID 位的文件被执行时，该文件将以所有者的身份运行，也就是说无论谁来执行这个文件，他都有文件所有者的特权。
bit11: SGID     当设置了SGID 位的文件被执行时，文件运行时，运行者将具有所属组的特权。
bit10: sticky-bit   sticky 位要求操作系统既是在可执行程序退出后，仍要在内存中保留该程序的映象。这样做是为了节省大型程序的启动时间。但是会占用系统资源。

如何设置：

操作这些标志与操作文件权限的命令是一样的, 都是 chmod. 有两种方法来操作,
1) chmod u+s temp -- 为temp文件加上setuid标志. (setuid 只对文件有效，U=用户)
chmod g+s tempdir -- 为tempdir目录加上setgid标志 (setgid 只对目录有效，g=组名)
chmod o+t temp -- 为temp文件加上sticky标志 (sticky只对文件有效)

2) 采用八进制方式. 这一组八进制数字三位的意义如下,
abc
a - setuid位, 如果该位为1, 则表示设置setuid
b - setgid位, 如果该位为1, 则表示设置setgid
c - sticky位, 如果该位为1, 则表示设置sticky

设置后, 可以用 ls -l 来查看. 如果本来在该位上有x, 则这些特殊标志显示为小写字母 (s, s, t). 否则, 显示为大写字母 (S, S, T)
如:

rwsrw-r-- 表示有setuid标志 (rwxrw-r--:rwsrw-r--)
rwxrwsrw- 表示有setgid标志 (rwxrwxrw-:rwxrwsrw-)
rwxrw-rwt 表示有sticky标志 (rwxrw-rwx:rwxrw-rwt)

对每个文件或目录都有 4 类不同的用户。每类用户各有一组读、写和执行（搜索）文件的访问权限，这 4 类用户是：

     root：系统特权用户类，既 UID = 0 的用户。
     owner：拥有文件的用户。
     group：共享文件的组访问权限的用户类的用户组名称。
     other：不属于上面 3 类的所有其他用户。
     
     
     
     
## 实际用户ID 有效用户ID 保存设置ID