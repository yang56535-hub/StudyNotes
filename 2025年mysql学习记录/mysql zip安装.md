# 一、准备工作

## 1.下载zip包：

从 MySQL 官网下载页 选择 MySQL 8.0.42 (x86, 64-bit), MSI Installer 下方的 ZIP Archive 版本（文件名类似 mysql-8.0.42-winx64.zip），或直接下载链接：https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.42-winx64.zip。  

## 2.系统要求

- Windows 7 SP1 或更高版本（64位）。
- 至少 4GB 内存（生产环境建议更高）。
- 至少 10GB 硬盘空间（用于数据和日志）。
- 需安装 .NET Framework 4.8（部分工具依赖）。

# 二、安装步骤

## 1.解压zip包

选择一个无中文、无空格的目录（如 C:\mysql-8.0.42-winx64），右键点击 mysql-8.0.42-winx64.zip 选择“解压到当前文件夹”。
解压后目录结构示例：
C:\mysql-8.0.42-winx64\
├─ bin\               # 可执行文件（如 mysqld、mysql 命令）
├─ data\              # 数据目录（初始化后自动生成）
├─ docs\              # 文档
├─ include\           # 头文件（开发用）
├─ lib\               # 库文件（开发用）
├─ share\             # 错误消息、字符集等配置
└─ my.ini（或 my-default.ini） # 配置文件模板

## 2. 配置环境变量（可选但推荐）

为了在命令行中直接使用 mysql 命令，需将 MySQL 的 bin 目录添加到系统环境变量 Path 中：
右键点击“此电脑”→“属性”→“高级系统设置”→“环境变量”。
在“系统变量”中找到 Path，双击编辑→“新建”→添加 C:\mysql-8.0.42-winx64\bin（根据实际解压路径调整）。
确认保存后，重启命令行窗口生效。

# 三、配置文件my.ini创建

在 C:\mysql-8.0.42-winx64 下新建文件 my.ini，用记事本编辑以下内容：
[mysqld]

- **基础配置**

  basedir=C:/mysql-8.0.42-winx64      # MySQL 安装目录
  datadir=C:/mysql-8.0.42-winx64/data # 数据存储目录
  port=3306             # 默认端口
  character-set-server=utf8mb4
  collation-server=utf8mb4_general_ci
  default-storage-engine=INNODB

  [client]
  default-character-set=utf8mb4
  **注意：路径使用 / 或 \\（如 C:\\mysql-8.0.42-winx64），避免 \ 转义问题。**



# 四、初始化数据库（关键步骤）
ZIP 包需要手动初始化数据目录，生成系统表和初始数据。
操作步骤：

以 管理员身份 打开命令提示符（CMD）（右键开始菜单→“命令提示符（管理员）”）。
切换到 MySQL 的 bin 目录：
`cd C:\mysql-8.0.42-winx64\bin`

- **执行初始化命令（根据需求选择无密码或有临时密码模式）：**
  模式 1：无密码（适合本地开发）
  使用 --initialize-insecure 参数，初始化后 root 用户无密码：
  `mysqld --initialize-insecure --user=MySQL`
  模式 2：生成随机临时密码（适合生产环境）
  使用 --initialize 参数，初始化后会生成一个临时密码（记录下来后续修改）：
  `mysqld --initialize --user=mysql`
  初始化完成后，查看命令行输出的最后几行，找到临时密码（如 A temporary password is generated for root@localhost: xxxxxx）。
  报错处理：
  若提示 缺少 MSVCR120.dll：安装 Visual C++ Redistributable。
  若提示 data 目录已存在：删除 C:\mysql\data 文件夹后重试。
- **解决方案**
  1. 清理数据目录
    步骤：
    删除 C:\mysql-8.0.42-winx64\data\ 下的所有文件和子目录：
    `rm -rf C:\mysql-8.0.42-winx64\data\`
    注意：Windows 用户可通过资源管理器手动删除，或使用 PowerShell：
    `Remove-Item -Recurse -Force C:\mysql-8.0.42-winx64\data\`
    重新创建空 data 目录：
    `mkdir C:\mysql-8.0.42-winx64\data\`
    验证：
    确保 data 目录为空后再运行初始化命令。
  2. 检查配置文件（my.ini）
    路径：
    确认 my.ini 中 datadir 和 basedir 的路径是否正确：
    `[mysqld]
    basedir=C:/mysql-8.0.42-winx64
    datadir=C:/mysql-8.0.42-winx64/data`
    注意：路径使用 / 或 \\，避免 \ 转义问题
  3. 权限修复
    Windows 权限设置：
    右键 data 目录 → 属性 → 安全 → 编辑权限，确保当前用户有 完全控制 权限。
    若使用管理员权限运行 CMD，需确保 mysqld 进程有写入权限
    4.重新初始化
    命令：
    以管理员身份运行 CMD，执行：
    `cd C:\mysql-8.0.42-winx64\bin
    mysqld --initialize --console`
    成功标志：生成临时 root 密码（如 A temporary password is generated for root@localhost: Abc123!*）

# 五、启动 MySQL 服务

初始化完成后，需启动 MySQL 服务。有两种方式：

方式 1：临时启动（每次重启后失效）
在管理员 CMD 中执行：

`mysqld --console`
控制台会输出 MySQL 启动日志，看到 Ready for connections 表示启动成功（按 Ctrl+C 可停止服务）。

方式 2：注册为系统服务（推荐，开机自启动）
在管理员 CMD 中执行以下命令，将 MySQL 注册为 Windows 服务：

`mysqld --install MySQL80 --defaults-file="C:\mysql-8.0.42-winx64\my.ini"`
MySQL80 是服务名（可自定义，避免与其他服务冲突）。
--defaults-file 指定配置文件路径（若未指定，默认读取 my.ini 或 my-default.ini）。
注册成功后，通过以下命令启动服务：

`net start MySQL80`
停止服务命令：

`net stop MySQL80`

# 六、启动服务后连接 MySQL 数据库
无论采用哪种初始化模式，都需通过命令行或工具连接 MySQL：

场景 1：初始化时使用 --initialize-insecure（无密码）
在 CMD 中执行：

~~~bash
mysql -u root -p
~~~


输入密码时直接按回车（无密码），成功进入 MySQL 命令行界面（提示符 mysql>）。

场景 2：初始化时使用 --initialize（有临时密码）
在 CMD 中执行：

~~~bash
mysql -u root -p
~~~


输入初始化时生成的临时密码（如 xxxxxx），登录后需立即修改 root 密码：

-- 修改 root 用户密码（8.0+ 推荐使用 caching_sha2_password 认证）

~~~mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
~~~


-- 刷新权限

~~~mysql
FLUSH PRIVILEGES;
~~~

# 七、配置远程访问（可选）
默认情况下，MySQL 仅允许本地连接（localhost）。若需远程访问（如其他电脑连接当前 MySQL），需修改配置：

登录 MySQL 命令行：
`mysql -u root -p`
授权远程访问（以 root 用户为例）：
-- 允许 root 用户从任意 IP 远程访问（生产环境建议指定具体 IP）

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'admin123' WITH GRANT OPTION;
```

ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'IDENTIFIED BY 'admin123' WITH GRANT OPTION' at line 1
在 MySQL 8.0 及以上版本中，GRANT 语句的语法已不再支持直接通过 IDENTIFIED BY 指定密码。您遇到的错误是由于 语法不兼容 导致的。以下是解决方法：
错误原因
MySQL 8.0 后，GRANT 语句的 IDENTIFIED BY 语法被废弃，需改用 CREATE USER 和 ALTER USER 分步操作
**解决方案**

1.分步操作：先创建用户，再授予权限
-- 步骤 1：创建用户（如果用户不存在）

~~~mysql
CREATE USER 'root'@'%' IDENTIFIED BY 'admin123';
~~~

-- 步骤 2：授予所有权限

~~~mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
~~~

-- 步骤 3：刷新权限

~~~mysql
FLUSH PRIVILEGES;
~~~

2.修改现有用户的密码（如果用户已存在）
如果 root@'%' 用户已存在，需先修改密码再授予权限：

-- 修改密码

~~~mysql
ALTER USER 'root'@'%' IDENTIFIED BY 'admin123';
~~~

-- 授予权限（若需保留原有权限）

~~~mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
~~~

-- 刷新权限

~~~mysql
FLUSH PRIVILEGES;
~~~

关键语法说明
创建用户：CREATE USER '用户名'@'主机' IDENTIFIED BY '密码';
授予权限：GRANT 权限 ON 数据库.表 TO '用户名'@'主机' [WITH GRANT OPTION];
刷新权限：FLUSH PRIVILEGES;（使权限立即生效）
其他注意事项
权限冲突：
如果用户已有权限，建议先撤销旧权限再重新授予：

~~~mysql
REVOKE ALL PRIVILEGES ON *.* FROM 'root'@'%';
DROP USER 'root'@'%';  -- 删除旧用户
~~~

再按上述步骤重新创建用户。
密码策略：
MySQL 8.0 默认要求密码复杂度（至少 8 位，含大小写字母、数字、符号）。若密码不符合要求，需调整密码格式。
远程访问配置：
若需允许远程连接，还需修改配置文件 my.ini：
[mysqld]
`bind-address = 0.0.0.0  # 允许所有 IP 访问`
并重启 MySQL 服务。
验证操作
执行以下命令确认权限是否生效：

~~~mysql
SHOW GRANTS FOR 'root'@'%';
~~~


输出应包含：

~~~mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION
~~~

-- 刷新权限

~~~mysql
FLUSH PRIVILEGES;
~~~

关闭 MySQL 服务，修改 my.ini 配置文件（路径：C:\mysql-8.0.42-winx64\my.ini）：
在 [mysqld] 部分添加/修改以下参数：
[mysqld]
`port=3306                  # 端口（默认 3306，若修改需同步防火墙）
bind-address=0.0.0.0       # 允许所有 IP 连接（默认 127.0.0.1 仅本地）
character-set-server=utf8mb4  # 字符集（推荐）`
重启 MySQL 服务：

~~~bash
net stop MySQL80
net start MySQL80
~~~

`C:\mysql-8.0.42-winx64\bin>net start MySQL80`
MySQL80 服务正在启动 .
MySQL80 服务无法启动。

服务没有报告任何错误。