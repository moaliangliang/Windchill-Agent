# Windchill 系统重托管（Rehost）SOP

## 概述
- **文档编号**: PISX-SOP-SYS-002
- **版本**: A6
- **用途**: 修改 Windchill 系统的主机名/IP 地址和端口配置
- **适用场景**: 服务器迁移、IP 变更、域名变更

## 前置条件
- [ ] MethodServer 正常运行
- [ ] 有管理员权限（windchill 命令行和 xconfmanager）
- [ ] 记录当前配置（建议先执行 `windchill_system_clone` 备份）
- [ ] 准备维护窗口（需要重启 MethodServer）

## 步骤一：使用 PTC Rehost Utility（官方推荐）

PTC 提供专用的 Rehost Utility，位于 `<WT_HOME>/bin/rehost.bat`：

```bash
cd D:\ptc\Windchill_12.1\Windchill\bin
rehost.bat D:\temp\rehost.properties
```

### rehost.properties 配置示例

```properties
# 目标服务器信息
target.wc.hostname=新服务器FQDN
target.wt.home=D:/ptc/Windchill_12.1/Windchill
target.httpserver.port=80
target.httpserver.ssl.port=443

# 数据库信息
target.db.hostname=新数据库主机
target.db.port=1521
target.db.serviceName=ORCL

# Java 路径
target.directory.java=C:/Program Files/Java/jdk11
```

### Rehost Utility 支持场景

| 场景 | 说明 |
|:-----|:------|
| **Rename** | 同机器改主机名，无需新许可证 |
| **Rehost** | 迁移到现有目标服务器 |
| **Clone** | 克隆完整安装+数据到新服务器 |
| **Move** | 更改安装路径（WT_HOME）|

> ⚠️ `windchill rehost` 命令不存在。PTC Windchill 12.x 使用 `rehost.bat` 脚本。

## 步骤二：手动执行（如自动方式不可用）

### 2.1 修改主机名

```bash
xconfmanager -s wt.rmi.server.hostname=新主机名 -t D:/ptc/Windchill_12.1/Windchill/codebase/wt.properties
```

### 2.2 修改 HTTP 端口（可选）

```bash
xconfmanager -s wt.rmi.server.httpPort=新端口 -t D:/ptc/Windchill_12.1/Windchill/codebase/wt.properties
```

### 2.3 修改 RMI 端口（可选）

```bash
xconfmanager -s wt.rmi.server.rmiPort=新端口 -t D:/ptc/Windchill_12.1/Windchill/codebase/wt.properties
```

### 2.4 传播配置

```bash
xconfmanager -p
```

## 步骤三：更新 Tomcat 配置

编辑 `D:/ptc/Windchill_12.1/Windchill/tomcat/conf/server.xml`：

```xml
<!-- 修改 Connector 端口 -->
<Connector port="新HTTP端口" protocol="HTTP/1.1" ... />

<!-- 如主机名变更，更新 SSL 证书相关配置 -->
```

## 步骤四：重启 MethodServer

```bash
# 使用 knowagent
windchill_server_methodserver(action="restart")

# 或手动
windchill stop
windchill start
```

## 步骤五：验证

| 检查项 | 方法 | 预期结果 |
|:-------|:-----|:---------|
| MethodServer 状态 | `windchill status` | 显示运行中 |
| OData API | `curl -u wcadmin:wcadmin "http://新IP:7380/Windchill/odata/v3/ProdMgmt/Parts?$top=3"` | HTTP 200 |
| Web UI | 浏览器访问 `http://新IP/Windchill` | 登录页面正常 |
| Worker Agent | knowagent `windchill_worker_agent_status` | 工作器在线 |
| knowagent 连接 | `windchill_server_status` | OData 正常 |

## 步骤六：更新外部集成

- [ ] 更新 knowagent 的 `.env` 中 `WINDCHILL_HOST` 配置
- [ ] 更新 CAD Worker 配置（agent.ini 中的连接信息）
- [ ] 更新任何引用旧主机名的第三方系统

## 常见问题

| 问题 | 原因 | 解决 |
|:-----|:-----|:------|
| MethodServer 启动失败 | hostname 未正确设置 | 检查 `wt.rmi.server.hostname` 属性 |
| OData 返回 404 | WRS 配置未更新 | 重启 Tomcat |
| 已有连接断开 | RMI 端口变更 | 客户端需要重新连接 |
| SSL 证书错误 | 域名变更 | 重新生成或更新证书 |

## 回滚方案

如果 Rehost 后出现问题：

```bash
# 恢复原主机名
xconfmanager -s wt.rmi.server.hostname=原主机名 -t codebase/wt.properties
xconfmanager -p
windchill stop
windchill start
```

## 责任部门
- PLM 实施团队：执行配置变更
- IT 基础设施部：网络和服务器配置
- 各业务部门：验证系统可用性
