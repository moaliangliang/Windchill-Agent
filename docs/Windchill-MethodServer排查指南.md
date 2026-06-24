# Windchill MethodServer 常见问题排查

## 1. MethodServer 无法启动

### 现象
- `windchill status` 显示 MethodServer 未运行
- 日志中报连接错误

### 排查步骤
| 步骤 | 操作 | 说明 |
|:-----|:------|:------|
| 1 | `windchill status` | 查看当前状态 |
| 2 | 检查 MethodServer 日志 | `<Windchill>/logs/MethodServer*.log` |
| 3 | 检查端口占用 | `netstat -an | findstr 7222` |
| 4 | 检查 JVM 内存 | 查看 `wt.properties` 中的 `wt.java.args` |
| 5 | 检查数据库连接 | 确认 Oracle 数据库正常运行 |

### 常见错误

#### "Failed to connect to the server at tcp://XXX:7222"
- **原因**: MethodServer JMS 端口被占用或 MethodServer 未正确启动
- **解决**: 
  1. 检查端口是否被其他进程占用
  2. 重启 MethodServer: `windchill stop && windchill start`
  3. 检查 `msgservice.properties` 中的 JNDI 配置

#### "NullPointerException"
- **原因**: 参数别名或代码缺陷
- **解决**: 查看详细堆栈，确定具体操作

#### "OutOfMemoryError"
- **原因**: JVM 堆内存不足
- **解决**: 增大 `wt.java.args` 中的 `-Xmx` 参数

## 2. MethodServer 运行缓慢

### 检查项
- `windchill status` 查看空闲/总内存
- 检查 JVM GC 日志
- 检查数据库连接池
- 检查是否有长时间运行的事务

### 优化建议
- 增加 JVM 堆内存
- 调整线程池大小
- 优化慢查询 SQL

## 3. MethodServer 连接问题

### ESI 队列连接失败
- 检查 Tibco JMS 服务是否运行
- 确认端口可访问
- 检查 `msgservice.properties` 配置

## 4. 日志分析

### 关键日志文件
| 文件 | 用途 |
|:-----|:------|
| `<Windchill>/logs/MethodServer-*.log` | 主日志 |
| `<Windchill>/logs/MethodServer-*-GC.log` | GC 日志 |
| `<Windchill>/logs/stdout.log` | 标准输出 |

### 常用日志搜索命令
```bash
findstr /i "ERROR|Exception|ORA-" MethodServer*.log
findstr /i "OutOfMemory|NullPointer" MethodServer*.log
```

## 5. MethodServer 重启流程

### 标准重启
```bash
windchill stop
windchill start
```

### 强制重启
```bash
windchill stop -force
windchill start
```

## 6. knowagent 运维命令

```bash
# 查看 MethodServer 状态
windchill_server_methodserver(action="status")

# 重启 MethodServer
windchill_server_methodserver(action="restart")

# 查看日志
windchill_view_log(filename="MethodServer-*.log", search="ERROR")
```
