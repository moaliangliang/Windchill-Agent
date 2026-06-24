# Windchill 系统克隆 SOP

## 概述
- **文档编号**: PISX-SOP-SYS-001
- **版本**: A6
- **用途**: 将 Windchill 系统完整克隆到新服务器

## 前置条件

### 源服务器检查
- [ ] Windchill 服务正常（MethodServer、Tomcat、Oracle）
- [ ] 有足够的磁盘空间用于 expdp 导出
- [ ] 记录当前 Windchill 版本和补丁级别：`windchill version`
- [ ] 记录 Oracle 版本：`SELECT * FROM v$version`

### 目标服务器检查
- [ ] 已安装与源服务器**完全一致**的 Windchill 版本（含相同补丁）
- [ ] 已安装 Oracle 数据库（版本与源一致）
- [ ] 已安装 WRS（Windchill REST Services）
- [ ] 磁盘空间充足（建议源 vaults 大小的 2 倍以上）
- [ ] 网络连通（能访问源服务器的 Oracle 和 vaults）

## 步骤一：导出源系统配置

在源服务器上执行：

```bash
# 1. 导出完整配置
xconfmanager -dump > D:\temp\windchill_config_$(date +%Y%m%d).txt

# 2. 记录 Windchill 版本
windchill version > D:\temp\windchill_version.txt

# 3. 记录 Oracle 配置
sqlplus -S / as sysdba @echo "
col name for a50
select name, created, log_mode from v$database;
col member for a60
select * from v$logfile;
"
```

## 步骤二：导出数据库

```bash
# expdp 全库导出（使用 knowagent 工具）
windchill_oracle_sql(sql="SELECT username FROM dba_users WHERE default_tablespace NOT LIKE 'SYSTEM%'")

# 建议 expdp 命令（在服务器上直接执行）
expdp system/manager directory=DATA_PUMP_DIR \
  dumpfile=windchill_full_$(date +%Y%m%d).dmp \
  logfile=expdp_$(date +%Y%m%d).log \
  full=y
```

## 步骤三：复制文件保险库（Vaults）

```bash
# 在目标服务器上执行（从源拉取）
# 需先配置 SSH 或使用共享存储
rsync -avz --progress \
  administrator@源IP:D:/ptc/Windchill_12.1/Windchill/vaults/ \
  D:/ptc/Windchill_12.1/Windchill/vaults/
```

## 步骤四：目标服务器恢复数据库

```bash
# 1. 复制 expdp 备份到目标服务器
# 2. 创建表空间（如源有自定义表空间）
# 3. 导入数据
impdp system/manager directory=DATA_PUMP_DIR \
  dumpfile=windchill_full_$(date +%Y%m%d).dmp \
  logfile=impdp_$(date +%Y%m%d).log \
  full=y
```

## 步骤五：配置更新（Rehost）

使用 PTC Rehost Utility（官方推荐）：

```bash
cd D:\ptc\Windchill_12.1\Windchill\bin
rehost.bat D:\temp\rehost.properties
```

或手动执行 xconfmanager：

```bash
xconfmanager -s wt.rmi.server.hostname=目标主机名 -t codebase/wt.properties
xconfmanager -p
```

### rehost.properties 配置示例

```properties
target.wc.hostname=新服务器FQDN
target.db.hostname=新数据库主机
target.db.port=1521
target.wt.home=D:/ptc/Windchill_12.1/Windchill
target.httpserver.port=80
```

详细配置项请参考 PTC Rehost Utility Guide。

## 步骤六：重启服务

```bash
# 1. 停止服务
windchill stop

# 2. 启动服务
windchill start
```

## 步骤七：验证

- [ ] 登录 Windchill Web UI
- [ ] 验证 OData API：`curl -u wcadmin:wcadmin "http://目标IP:7380/Windchill/odata/ProdMgmt/Parts?$top=3"`
- [ ] 验证物料、文档、BOM 数据完整性
- [ ] 验证 Worker Agent 是否正常
- [ ] 用 knowagent 测试查询功能

## 常见问题

| 问题 | 原因 | 解决 |
|:-----|:-----|:------|
| ORA-01950 | 表空间不足 | 创建对应的表空间 |
| ORA-00959 | 表空间已存在 | 跳过或重命名 |
| Windchill 无法启动 | hostname 未更新 | 检查 xconfmanager 配置 |
| OData 404 | WRS 未安装 | 检查 WRS 安装状态 |
| 附件无法下载 | vaults 路径不匹配 | 检查 vaults 目录结构和权限 |

## 责任部门
- IT 基础设施部：服务器准备
- PLM 实施团队：克隆执行
- DBA：数据库备份恢复
