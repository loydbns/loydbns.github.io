# Kali WSL2 MSF数据库无法启动.md
# 问题描述：

在启动postgresql及后续初始化时出现如下报错：

```
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

# 问题原因：

所有报错信息都指向关键的`System has not been booted with systemd as init system (PID 1). Can't operate.`,这个报错指的是我们没有启动systemd,使用`service --status-all`命令可以看出,我们的数据库的确没有启动。根据我网上搜索的结果systemd主要用来对数据库进行一些操作,但wsl系统中并不能使用systemd,所有我们可以大胆的在此推测,在创建阶段就失败了。这里我们的解决方案就出来了,我们可以尝试手动建立数据库。<br />

# 解决方法：

## 手动创建数据库

根据`db_status`信息(上面报错信息中最后一张图中我使用了这个命令),我们可以看到我们需要连接的数据库管理系统叫postgresql,一个类似与MySQL、mssql这类的关系型数据库。首先我们在kali中启动这个数据库

```
sudo service postgresql start
sudo service postgresql status
```

出现如下所示则代表postgresql数据库管理系统启动成功。(如果报错,可以往下翻,我在一次部署的时候也发生了报错)

![image.png](assets/image-20211127180056-r01xnvu.png)

## 进入数据库并修改密码

```
# 先进入数据库安装目录里面有数据库的配置文件`postgresql.conf`可以改其中的配件,我当前msf使用的数据库为12版本,对应的下面的12,如果你的msf和我版本不同,可能数据库版本也不同,可以尝试先进入cd /etc/postgresql,使用ls查看有哪些版本,在进入。
cd /etc/postgresql/12/main
# 进入数据库
sudo -u postgres psql
# 修改管理员密码(当前密码为root),可以自行修改为自己想要的
alter user postgres password 'root';
# 创建一个新数据库用户,用户名为 msf 用户密码为 root
create user msf with password 'root' createdb;
# 为msf用户创建一个msf数据库
create database msf with owner=msf;
# 退出数据库
quit
```

尝试登陆数据库是看上述步骤是否配置成功,这一步可以省略掉

```
# 尝试登陆数据库
psql -U postgres -h 127.0.0.1
# 如图报错,则可能是数据库数据库端口并非默认端口(5432)
# 可以尝试指定端口登陆,比如我的端口就是5958,我这需要使用-p 5958
psql -U postgres -p 5958 -h 127.0.0.1
# 登陆后就可以退出了
/q

```

进入msf数据库配置文件，将数据库密码修改为所设的密码

```
/usr/share/metasploit-framework/config/database.yml
```

## 进入msf测试连接

```
# 进入msf
msfconsole
# 测试数据库是否连接成功
db_status
```

![image.png](assets/image-20211127184318-uzx2mu8.png)
