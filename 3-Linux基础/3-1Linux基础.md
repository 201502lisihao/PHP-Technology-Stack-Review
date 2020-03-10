### 3.1 Linux基础
***

#### Linux常用命令

**系统安全：**
```php
sudo
su
chmod // 修改文件或目录权限
chown // 修改文件或目录拥有者
chgrp // 修改文件或目录所属群组
getfacl // 用来在命令行里获取ACL（访问控制列表）
setfacl // 用来在命令行里设置ACL（访问控制列表）
```

**进程管理：**
```php
w // 用于显示目前登入系统的用户信息
top // 用于实时显示 process 的动态
ps // 用于显示当前进程 (process) 的状态。
kill // 用于删除执行中的程序或工作，例：kill 12345
pkill // 用于根据进程名杀死进程
pstree // 将所有行程以树状图显示
killall // 使用进程的名称来杀死一组进程，例：killall php-fpm
```

**用户管理：**
```php
id // 用于显示用户的ID，以及所属群组的ID
usermod
useradd
groupadd
userdel
```

**文件系统：**
```php
mount // 用于挂载Linux系统外的文件
umount
fsck // 用于检查与修复 Linux 档案系统
df // 用于显示目前在Linux系统上的文件系统的磁盘使用情况统计
du // 显示文件或目录所占用的磁盘空间，例：du -sh test.php
```

**关机&重启：**
```php
shutdown 
reboot
```

**网络应用：**
```php
curl // 利用URL规则在命令行下工作的文件传输工具
telnet // 用于远端登入
mail // 命令行下发送和接收电子邮件
elinks // 纯文本界面的WWW浏览器
wget // Linux系统下载文件工具，从指定url下载资源
```

**网络测试：**
```php
ping // 用于检测指定主机网络状态
netstat // 用于显示本主机网络状态
host // 常用的分析域名查询工具
```

**网络配置：**
```php
hostname // 显示和设置系统的主机名
ifconfig // 用于显示或设置网络设备
```

**常用工具：**
```php
ssh // 用于登录远程主机
screen // 是一款由GNU计划开发的用于命令行终端切换的自由软件
clear // 清除当前窗口内容
who // 显示当前用户
date // 显示当前日期
```

**软件包管理：**
```php
yum // 用于添加/删除/更新RPM包,自动解决包的依赖问题以及系统更新升级
rpm // m是一个功能十分强大的软件包管理系
apt-get // 是一个下载安装软件包的简单命令行接口
```

**文件查找和比较：**
```php
locate // 在mlocate数据库中搜索条目，例：locate ./test，查找当前目录下以test开头的文件
find // 查找目录和文件
```

**文件内容查看：**
```php
cat // 将[文件]或标准输入组合输出到标准输出，例：cat test.php
head // 输出文件开头部分，例：head -100 test.php
tail // 输出文件末尾部分，例：tail -f test.log，以此监控log
less
more
```

**文件处理：**
```php
touch // 用于创建文件，将每个文件的访问时间和修改时间改为当前时间，不存在的文件将会被创建为空文件
unlink // 用于删除文件
rename // 用于对文件进行命名管理
ln // 建立软连接，例：ln -s /home/svn/test /home/www/abc
```

**文件传输：**
```php
ftp // 连接远程文件服务器，可以上传、下载文件
scp // 从远程服务器拷贝文件到本地或从本地服务器推送文件给远程服务器
```

**压缩/解压缩：**
```php
bzip2/bunzip2
gzip/gunzip
zip/unzip
tar 
```

**目录操作：**
```php
cd
mv
rm
pwd
cp
ls
```

#### 系统定时任务

**crontab：**
```php
// crontab -e 来创建定时任务
// 每行一个命令
*    *    *    *    * xxxxx命令
-    -    -    -    -
|    |    |    |    |
|    |    |    |    +----- 星期中星期几 (0 - 7) (星期天 为0)
|    |    |    +---------- 月份 (1 - 12) 
|    |    +--------------- 一个月中的第几天 (1 - 31)
|    +-------------------- 小时 (0 - 23)
+------------------------- 分钟 (0 - 59)
```

`tip：`推荐此[定时命令解释工具](https://crontab.guru/)，生产环境创建定时任务时，可先通过此网站验证。

**at：**
```php
// 一次性定时命令
at 2:00 tomorrow
at> xxx命令
at> ctrl + d
```
#### vi/vim编辑器

**模式：**
```
一般模式：删除、复制、黏贴
编辑模式：编辑
命令行模式：保存、退出、查找
```

#### shell编程基础
```bash
#!/bin/sh
#第一行指定脚本解释器

#编写具体逻辑，以一个取余数的逻辑为例
function Mod(){
    read -p "请输入被除数：" a                                                                                                                                                                 
    if  [ $a -eq 0 ];then
        echo "被除数为0,请重新输入"
        Mod 
    fi  
    read -p "请输入除数：" b
    if  [ $b -eq 0 ];then
        echo "除数为0,请重新输入"
        Mod 
    fi  
    echo "$a % $b = "$((a%b))
    echo "余数是 "$((a%b))
}
Mod
```

***
真题：实现在每天0点重启服务器
> 请思考后写出答案。

[**下一小节：4.1 MySQL数据库基础**](https://github.com/201502lisihao/PHP-Technology-Stack-Review/blob/master/4-MySQL%E6%95%B0%E6%8D%AE%E5%BA%93/4-1MySQL%E6%95%B0%E6%8D%AE%E5%BA%93%E5%9F%BA%E7%A1%80.md)