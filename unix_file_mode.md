# UNIX 文件属性与进程权限

本文主要介绍一下unix中文件的属性，关注于权限部分。  

## 文件权限
我们通过ls或者stat可以查看文件的属性。
在struct stat中，有一个域`mode_t st_mode;`,是一个16位无符号整数。

这个uint16_t的值中，高4位(bit12-15)标识了文件的类型(File Type),  
低9位分别表示该文件的所有者(owner)的读写执行权限，组(group)的读写执行权限，和其他用户(other)的读写执行权限。  
以上应该是比较常接触的。

而第10-12位则表示
|                      ||
|:---:|:---:|
|*bit12: SUID*         |	 当设置了SUID位的文件被执行时，该文件将以所有者的身份运行，也就是说无论谁来执行这个文件，他都有文件所有者的特权。|  
|*bit11: SGID*         | 当设置了SGID位的文件被执行时，文件运行时，运行者将具有文件所属组的特权。|  
|*bit10: sticky-bit*   | sticky位要求操作系统既是在可执行程序退出后，仍要在内存中保留该程序的映象。这样做是为了节省大型程序的启动时间。但是会占用系统资源。|  
 
### 理解文件权限  
- 当我们用名字打开任一类型的文件时，对改名字中包含的每一个目录，包括它可能隐含的当前工作目录都应该具有执行权限。因此对于目录其执行权限位常被称为搜索位。
>   对于目录的读权限允许我们读目录，获取在该目录中的所有文件名的列表。
> 当一个目录是我们想要访问的文件的路径名的一部分时，对该目录的执行权限使得我们可通过该目录。
> (也就是搜索该目录，寻找一个特定的文件名。)   
> 当我们需要打开某个文件时，对该文件所在的各层目录，需要具备执行权限。

- 读权限决定了我们能否打开该文件进行读操作。（O_RDONLY\O_RDWR）
- 写权限决定了我们能否打开该文件进行写操作。（O_WRONLY\O_RDWR）
- 为了在open函数中对一个文件指定O_TRUNC标志，必须对该文件具有写权限。
- 为了在一个目录中创建一个新文件，必须对该目录具备写权限和执行权限。
- 为了删除一个现有文件，必须对包含该文件的目录具备写权限和执行权限。对该文件本身不需要具备权限。
- 如果用exec系列函数执行某个文件，都必须对该文件具有执行权限。该文件还必须是一个普通文件。

### 粘滞位  
sticky-bit, 又称为保存正文位(saved-text bit)。  
如果一个可执行文件的这一位被设置了，那么在该程序第一次被执行并结束时，其程序正文部分的一个部分仍被保存在交换区，这使得瑕疵执行该程序时能较快的将其装入内存区。

### 如何设置权限：

通常使用chmod更改文件的权限。有两种方法来操作：
- 1. **符号模式**  
使用符号模式可以设置多个项目, 每个项目包含三个元素 who(用户类型），operator(操作符）和permission(权限。每个项目的设置可以用逗号隔开。  
命令chmod将修改who指定的用户类型对文件的访问权限，用户类型由一个或者多个字母在who的位置来说明,如who的符号模式表所示:  

|who|用户类型|说明|
|:---:|:---:|:---:|
|u	|user	|文件所有者|  
|g	|group	|文件所有者所在组|
|o	|others	|所有其他用户|
|a	|all	|所有用户, 相当于 ugo。如果不填who，默认为all。|
operator的符号模式表:  

|Operator	|说明|
|:--:|:-:|
|+	|为指定的用户类型增加权限|
|-	|去除指定用户类型的权限|
|=	|设置指定用户权限的设置，即将用户类型的所有权限重新设置|
permission的符号模式表:

|模式	|名字	|说明|
|:---:|:-:|:-:|
|r	|读	|设置为可读权限|
|w	|写	|设置为可写权限|
|x	|执行/搜索权限	|设置为可执行/搜索权限|
|X	|特殊执行/搜索权限	|只有当文件为目录文件，或者原始权限中有任何可执行权限位被设置时，才将文件权限设置可执行，只能与op+配合。|
|s	|setuid/gid	|当文件被执行时，根据who参数指定的用户类型设置文件的setuid或者setgid权限.  |
|t	|粘滞位	|设置粘滞位，只有超级用户可以设置该位，只有文件所有者u可以使用该位。|
|u  ||该文件原始的user权限。|
|g  ||该文件原始的组权限。|
|o  ||该文件原始的其他用户权限。|

**NOTE:** root用户是个特例，拥有对所有文件的所有权限。

- 2. **八进制语法**  
chmod命令可以使用八进制数来指定权限。文件或目录的权限位是由9个权限位来控制，每三位为一组，它们分别是文件所有者（User）的读、写、执行，用户组（Group）的读、写、执行以及其它用户（Other）的读、写、执行。文件的特殊权限位(setuid/setgid/粘滞位)也可以通过八进制来设置。

如果mode是一个3位八进制数，则只是设置文件权限位。如`chmod 755 file`;
如果mode是一个4位八进制数，则设置所有的权限。如`chmod 4755`;

### 查看权限
设置权限后, 可以用 ls -l 来查看. 

结果的第一列表示当前的mode, 如-rwxr-xr-x  
- 最左一位表示文件类型,   


    b     Block special file.  
	c     Character special file.  
	d     Directory.  
	l     Symbolic link.  
	s     Socket link.  
	p     FIFO.  
	-     Regular file.  

- 后9位表示文件的访问权限(-rwxstST)  
设置文件的特殊权限位时，如果本来在该位上有x, 则这些特殊标志显示为小写字母 (s, s, t). 否则, 显示为大写字母 (S, S, T)。  
- @  
如果在这一列的最后有@符号，表示该文件存在扩展属性。    
 
第二列表示文件个数, 如果是目录则表示目录中文件个数，包括.和.. 。普通文件一般为1。  
第三列表示文件的所有者；  
第四列表示文件所属组；  
第五列表示文件大小，或者目录项大小；  
第六七八列分别表示m d hh:mm ；  
第九列表示文件名。

### unix 用户
对每个文件或目录都有 4 类不同的用户。每类用户各有一组读、写和执行（搜索）文件的访问权限，这 4 类用户是：

	 root：系统特权用户类，既 UID = 0 的用户。
	 owner：拥有文件的用户。
	 group：共享文件的组访问权限的用户类的用户组名称。
	 other：不属于上面 3 类的所有其他用户。

	 
## 实际用户ID 有效用户ID 保存设置ID  

与一个进程关联的ID则有以下：  
***实际用户ID 实际组ID***   
	
    实际用户ID和实际组ID标识我们是谁，这两个字段在登陆时取自口令文件中的登陆项。通常在一个登陆会话期间并不改变。
***有效用户ID 有效组ID 附加组ID***   

    有效用户ID 有效组ID和附加组ID决定了我们的文件访问权限。(通俗的理解是 当前进程以哪个用户和组来运行。)
***保存的设置用户ID 保存的设置组ID***

	保存的设置用户ID和保存的设置组ID在执行一个程序时，包含了有效用户ID和有效组ID的副本。（用来保证在用户通过各种手段更改euid后，可以恢复到原始的有效ID。）

通常，有效用户ID等于实际用户ID，有效组ID等于实际组ID。  
>如果文件属性的setuid被设置了，`当执行此文件时，将进程的有效用户ID设置为文件所有者的用户ID。`  
如果文件属性的setgid被设置了，`当执行此文件时，将进程的有效组ID设置为文件组所有者ID。`

进程每次打开、创建或删除一个文件时，内核会进行文件访问测试。  
- 若进程的有效用户ID是0（超级用户），则允许访问。  
- 若进程的有效用户ID等于文件所有者（该进程拥有此文件），那么检查用户访问权限来确定是否允许。
- 若进程的有效组ID或进程的附加组ID之一等于文件的组ID，那么检查组访问权限来确定是否允许。
- 检查其他用户访问权限来确定是否允许。

### 更改用户ID和组ID

>- 若进程具有超级用户特权，则setuid函数将实际用户ID、有效用户ID，以及保存的设置用户ID设置为*uid*（setuid函数的参数）。
>- 若进程没有超级用户特权，但是*uid*等于实际用户ID或保存的设置用户ID，则setuid函数只将有效用户ID设置为*uid*。不改变实际用户ID和保存的设置用户ID。
>- 否则，return－1，errno ＝ EPERM。  

- 只有超级用户进程可以更改实际用户ID。  

		通常实际用户ID是在用户登录时，由1号进程(unix为login,iOS/OSX为launchd)设置的，而去永远不会改变它。在1号进程调用setuid时, 设置所有3个ID。
- 仅当对程序文件设置了设置用户ID位时，exec函数才会设置有效用户ID。  
如果设置用户ID位没有设置，则exec函数不会改变有效用户ID，维持原值。
- 保存的设置用户ID由exec复制有效用户ID而得来的。
		
      	所以保存的设置用户ID仅在exec时候被设置（复制有效用户ID），此时有效用户ID取决于是否设置了用户ID位。这就是为什么这个ID要叫做保存的设置用户ID。
- 一个非特权用户总能交换实际用户ID和有效用户ID。  
- 一个非特权用户可以将其有效用户ID设置为其实际用户ID或其保存的设置用户ID。
		
        以上两条规则的目的在于，允许一个设置用户ID的程序转换成只具有用户的普通权限，以后又可以转换回设置用户ID所得到的额外权限。
        
       

#### 总结：
- 对于特权用户  

		setuid(uid)   将实际用户ID、有效用户ID，以及保存的设置用户ID设置为*uid*。
        
		setreuid(ruid,euid)  将实际用户ID设置为*ruid*, 将有效用户ID设置为*euid*。
        若ruid等于euid，等同于调用setuid函数。 可以使用seteuid替代其目的。
        
		seteuid(euid) 将有效用户ID设置为*euid*。  
        
		以上的uid，ruid，euid应该可以为任意值。 (此处有待验证)

- 对于非特权用户  
		
        setuid(uid)   只将有效用户ID设置为*uid*。
        uid仅可为原实际用户ID或保存的设置用户ID。    
        
		setreuid(ruid,euid)  将实际用户ID设置为*ruid*, 将有效用户ID设置为*euid*。  
		其中ruid仅可为原有效用户ID，euid仅可为原实际用户ID或保存的设置用户ID。 
  		若ruid等于euid，等同于调用setuid函数。 可以使用seteuid替代其目的。
        
		seteuid(euid) 将有效用户ID设置为*euid* 。
        euid仅可为原实际用户ID或保存的设置用户ID。
        
        对非特权用户，seteuid函数等同于setuid函数。

        
## 参考文献  
1. [维基百科](https://zh.wikipedia.org/wiki/)
2. man
3. [APUE](http://www.apuebook.com/)


