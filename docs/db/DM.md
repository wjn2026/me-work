---
title: 达梦
---

## [安装](https://docs.dameng.com/zh-CN/docs/1/470/113664){target='_blank'}

### 创建dmdba用户

```
useradd dmdba
```

### 安装

```
/mnt/DMInstall.bin -i
```

### 初始化实例

```
[dmdba@localhost bin]$ ./dminit PATH=/opt/dmdbms/data PAGE_SIZE=32 LOG_SIZE=2048 CHARSET=0 CASE_SENSITIVE=Y SYSDBA_PWD=****** SYSAUDITOR_PWD=******
initdb V8
db version: 0x7000c
file dm.key not found, use default license!
License will expire on 2025-09-18
Normal of FAST
Normal of DEFAULT
Normal of RECYCLE
Normal of KEEP
Normal of ROLL

 log file path: /opt/dmdbms/data/DAMENG/DAMENG01.log


 log file path: /opt/dmdbms/data/DAMENG/DAMENG02.log

write to dir [/opt/dmdbms/data/DAMENG].
create dm database success. 2025-04-03 11:10:57
```

```
[dmdba@dmdsc03 ~]$  su - root
密码：<输入密码>
[root@~]# /opt/dmdbms/script/root/dm_service_installer.sh -t dmserver -dm_ini /opt/dmdbms/data/DAMENG/dm.ini -p DMSERVER
Created symlink /etc/systemd/system/multi-user.target.wants/DmServiceDMSERVER.service → /usr/lib/systemd/system/DmServiceDMSERVER.service.
创建服务(DmServiceDMSERVER)完成

```

```
[dmdba@dmdsc03 ~]$ /opt/dmdbms/bin/DmServiceDMSERVER start
Starting DmServiceDMSERVER:                                [ OK ]
```

### 开启归档

!!! tip

    1. 相关路径及归档大小上限参数需按照实际要求配置，建议以生产环境每日增量数据的 1 至 2 倍设置归档上限；
    2. 不论使用以上哪种方法，均需要对数据库进行 mount 或重启操作。

=== "sql"

    ```
    ALTER DATABASE MOUNT;
    ALTER DATABASE ARCHIVELOG;
    ALTER DATABASE ADD ARCHIVELOG 'DEST=/dmarch, TYPE=LOCAL, FILE_SIZE=2048, SPACE_LIMIT=102400';
    ALTER DATABASE OPEN;
    ```


=== "修改配置"

    ```
    ##修改数据库实例的 /dmdata/DAMEGN/dm.ini文件中 ARCH_INI 参数值
    vi /dmdata/DAMENG/dm.ini
    ##将 ARCH_INI 值改为 1，保存后退出
    ARCH_INI = 1 #开启归档功能

    ##新增文件dmarch.ini
    vi /dmdata/DAMENG/dmarch.ini
    ##新增如下内容
    [ARCHIVE_LOCAL1]
    ARCH_TYPE = LOCAL
    ARCH_DEST = /dmarch
    ARCH_FILE_SIZE = 2048
    ARCH_SPACE_LIMIT = 102400

    ##最后重启数据库完成归档配置
    ```

### 归档查看

查看归档状态

```
SELECT ARCH_TYPE, ARCH_DEST, ARCH_STATUS
FROM V$ARCH_STATUS;
```
查已生成的归档文件

```
SELECT DB_MAGIC, PATH, CREATE_TIME, CLOSE_TIME, ARCH_LSN, CLSN, STATUS, LEN
FROM V$ARCH_FILE
ORDER BY CREATE_TIME;
```
查归档日志目录
```
SELECT ARCH_NAME, ARCH_TYPE, ARCH_DEST, ARCH_FILE_SIZE, ARCH_SPACE_LIMIT
FROM V$DM_ARCH_INI;
```

### 数据备份

=== "命令行"

    ```
    call SP_INIT_JOB_SYS(1);
    call SP_CREATE_JOB('bakall_delall',1,0,'',0,0,'',0,'每天 23:00 全量备份、删除 30 天前的全量备份');
    call SP_JOB_CONFIG_START('bakall_delall');
    call SP_ADD_JOB_STEP('bakall_delall', 'bakall', 6, '01020000/opt/dmdbms/data/DAMENG/bak', 3, 1, 0, 0, NULL, 0);
    call SP_ADD_JOB_STEP('bakall_delall', 'delall', 0, 'SF_BAKSET_BACKUP_DIR_ADD(''DISK'',''/opt/dmdbms/data/DAMENG/bak'');
    CALL SP_DB_BAKSET_REMOVE_BATCH(''DISK'',SYSDATE-30);', 1, 1, 0, 0, NULL, 0);
    call SP_ADD_JOB_SCHEDULE('bakall_delall', 'bakall_delall_time01', 1, 1, 1, 0, 0, '23:00:00', NULL, '2019-01-01 01:01:01', NULL, '');
    call SP_JOB_CONFIG_COMMIT('bakall_delall');
    ```



---


./disql SYSDBA/'"SYSDBA"':5236

SELECT * FROM V$VERSION;


## FAQ

* 备份出现 IO操作超时间间隔内未响应
sp_set_para_value(1,'BAK_TIMEOUT',300)

select * from sysjob.sysjobs查出你全备的sess_id
然后sp_dbms_job_run(sess_id)


针对达梦数据库因超长 `IN` 列表 SQL 触发 `VM locate variable with incorrect size` 导致的宕机问题，以下是排查肇事 SQL 与修改底层配置的完整操作总结：

### 一、 如何查看导致宕机的问题 SQL

当达梦数据库触发 `dm_sys_halt` 保护性停机时，系统会生成 core 文件。由于 Linux 系统的 `systemd-coredump` 机制接管了 core 文件，我们需要将其提取并解析：

1. **定位并解压 Core 文件**
   进入 systemd 的 coredump 目录，找到对应时间点的 `.lz4` 压缩文件，并使用 `lz4` 工具解压：
   ```bash
   cd /var/lib/systemd/coredump/
   sudo lz4 -d core.dmserver.xxxx.xz4
   ```
2. **使用 dmrdc 提取 SQL**
   切换到 `dmdba` 用户，进入达梦 `bin` 目录，使用 `dmrdc` 工具解析解压后的 core 文件，提取崩溃瞬间的 SQL：
   ```bash
   ./dmrdc sfile=/var/lib/systemd/coredump/core.dmserver.xxxx dfile=/home/dmdba_pro/crash_sql.txt
   ```
3. **查看提取结果**
   打开生成的 `crash_sql.txt` 文件，搜索 `sql:` 关键字即可看到肇事 SQL。
   *注：如果提取出的仅为 `select status$, mode$ from v$instance;` 等心跳语句，说明工作线程已崩溃，此时需使用 `gdb ./dmserver [core文件路径]` 进入调试模式，通过 `thread apply all bt` 查看底层函数调用栈来进一步定位。*

---

### 二、 如何查看与修改数据库配置

在达梦数据库中，参数分为动态参数（修改后即时生效）和静态参数（必须重启生效）。

#### 1. 查看当前配置
*   **查询视图（推荐）**：通过 `V$DM_INI` 视图可以查看参数的当前内存值、配置文件值以及参数类型（`IN FILE` 代表静态参数，必须重启）：
    ```sql
    SELECT PARA_NAME, PARA_VALUE, FILE_VALUE, PARA_TYPE FROM V$DM_INI WHERE PARA_NAME IN ('BUFFER', 'VM_STACK_SIZE');
    ```
*   **使用系统函数**：直接获取内存中生效的值：
    ```sql
    SELECT SF_GET_PARA_VALUE(2, 'VM_STACK_SIZE');
    ```

#### 2. 修改配置（以扩大 VM_STACK_SIZE 为例）
由于 `VM_STACK_SIZE` 是静态参数，且在很多版本中默认不显式写在配置文件里，必须手动添加并重启：

1. **停止数据库服务**：
   ```bash
   sudo systemctl stop DmServiceDMSERVERZCGL_PRO
   ```
2. **编辑 dm.ini 文件**：
   ```bash
   vim /home/dmdba_pro/dmdata_zcgl/DAMENG/dm.ini
   ```
3. **添加或修改参数**：在文件末尾添加以下内容（单位为 KB，1024 即 1MB，可根据需求调大至 262144 即 256MB）：
   ```ini
   VM_STACK_SIZE = 1024
   ```
4. **启动数据库并验证**：
   ```bash
   sudo systemctl start DmServiceDMSERVERZCGL_PRO
   ```
   重启后，再次使用 `SF_GET_PARA_VALUE(2, 'VM_STACK_SIZE')` 验证数值是否已变更为 1024。

---

### 三、 核心避坑指南

*   **静态参数修改陷阱**：对于 `PARA_TYPE` 为 `IN FILE` 的静态参数，使用 `SP_SET_PARA_VALUE(1, ...)` 是无效的，必须直接修改 `dm.ini` 文件并重启服务。
*   **超长 IN 列表的性能隐患**：虽然扩大 `VM_STACK_SIZE` 能让数据库“扛住”超长 SQL 不崩溃，但当 `IN` 列表的值过多时，达梦优化器极易放弃索引转为全表扫描，导致 SQL 执行极慢。长期来看，建议从代码层面将超长 `IN` 列表拆分为分批查询，或存入临时表后进行 `JOIN` 关联。

---




## 全量备份异地恢复

用备份集 `DB_DAMENG_FULL_2026_11_14_01_00_50` 在目标服务器上恢复达梦数据库。全过程以 `dmdba` 用户操作，`dmrman` 需在数据库服务停止的状态下执行。

### 1. 传输备份集

把备份集目录整体传到目标服务器，并改成 `dmdba` 属主：

```shell
scp -r DB_DAMENG_FULL_2026_11_14_01_00_50 dmdba@目标服务器IP:/home/dmdba/
```

```shell
sudo chown -R dmdba:dmdba DB_DAMENG_FULL_2026_11_14_01_00_50
```

!!! warning "备份集要放在 dmdba 家目录"

    后面 `dmrman` 里用相对路径引用备份集名，把目录整个放在 `dmdba` 用户家目录（如 `/home/dmdba/`）下最省事，避免路径写错。

### 2. 校验备份集

```shell
/opt/dmdbms/bin/dmrman
```

进入 `dmrman` 交互界面后执行：

```sql
CHECK BACKUPSET 'DB_DAMENG_FULL_2026_11_14_01_00_50';
```

### 3. 停止数据库服务

```shell
/opt/dmdbms/bin/DmServiceDMSERVER stop
```

### 4. 还原并恢复

仍在 `dmrman` 中，按顺序执行三条命令：

```sql
RESTORE DATABASE '/opt/dmdbms/data/DAMENG/dm.ini' FROM BACKUPSET 'DB_DAMENG_FULL_2026_11_14_01_00_50';
RECOVER DATABASE '/opt/dmdbms/data/DAMENG/dm.ini' FROM BACKUPSET 'DB_DAMENG_FULL_2026_11_14_01_00_50';
RECOVER DATABASE '/opt/dmdbms/data/DAMENG/dm.ini' UPDATE DB_MAGIC;
```

!!! note "三条命令各自做什么"

    - `RESTORE DATABASE`：把备份集里的数据文件写回 `dm.ini` 指向的数据目录
    - `RECOVER DATABASE ... FROM BACKUPSET`：应用备份集中的日志做恢复
    - `RECOVER DATABASE ... UPDATE DB_MAGIC`：重置数据库标识，让恢复后的库能在新环境正常启动

### 5. 启动并验证

```shell
/opt/dmdbms/bin/DmServiceDMSERVER start
```

服务起来后用 `disql` 登录，抽查表数据和实例状态，确认恢复完整。


## 全量+增量备份异地恢复

SHOW BACKUPSET '/home/dmdba/cs/home/dmdba_pro/backup/DB_DAMENG_INCREMENT_2026_11_19_01_00_46';