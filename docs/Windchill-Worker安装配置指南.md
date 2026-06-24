# Windchill Worker Agent 安装配置指南

## 概述
- **文档编号**: PISX-SOP-SYS-012
- **版本**: A1
- **用途**: Worker Agent 安装、配置（agent.ini）、常见问题排查

## 1. 安装场景

| 场景 | 说明 |
|:-----|:------|
| 本地 Worker | Worker 与 Windchill 同一台 Windows 服务器 |
| 远程 Windows Worker | Worker 在不同 Windows 机器上 |
| 远程 UNIX Worker | Worker 在不同 UNIX 机器上 |

## 2. 配置文件：agent.ini

路径: `<WT_HOME>\conf\wvs\agent.ini`

```ini
[agent]
numworkers=2
port=5600

[worker1]
autostart=true
host=VM73
shapetype=PROE
exe=D:\ptc\creo_view_adapters\proe_setup1\proeworker.bat
maxinstances=1
starttime=60

[worker2]
autostart=true
host=VM73
shapetype=OFFICE
exe=D:\ptc\office_worker\officeworker.bat
maxinstances=1
starttime=60
```

### 关键参数
| 参数 | 说明 | 示例 |
|:-----|:------|:------|
| numworkers | Worker 总数 | 必须 = 配置段数 |
| autostart | 是否自动启动 | true/false |
| host | 主机名 | VM73 |
| shapetype | 形状类型 | PROE/OFFICE/ACAD |
| exe | 执行文件路径 | 启动脚本完整路径 |
| maxinstances | 最大实例数 | 1 |
| starttime | 启动超时(秒) | 60 |
| hosttype | 主机类型 | local(同机)/nt(远程) |
| port | 通信端口 | 5601+ |

## 3. 常见问题

### 3.1 Worker 图标灰色 "Executable Not Safe"
- **现象**: 管理页面 Worker 呈灰色，提示 "Executable not safe"
- **原因**: 安全白名单未包含 exe 路径
- **解决**: 在 `wvs.properties` 中添加：
  ```
  worker.exe.allowlist.prefixes=D:\
  ```

### 3.2 Worker 启动后立即变为"关闭"
- **原因**: `starttime` 超时设置过短
- **解决**: 增大 `starttime` 参数（60-120 秒）

### 3.3 自定义 Worker 配置被覆盖
- **原因**: 使用 Web UI 配置时会重写 agent.ini
- **解决**: 直接编辑 agent.ini，修改前备份

### 3.4 Worker 无法连接到 Windchill
- **排查**:
  1. 确认 MethodServer 运行正常
  2. 检查端口 5600 是否开放
  3. 检查 host 参数是否正确
  4. 查看 Worker 日志文件

## 4. knowagent 工具
```bash
# 查看 Worker 状态
windchill_worker_agent_status()

# 启动/停止 Worker
windchill_worker_control(action="start", name="PROE", host="VM73")

# 添加新 Worker（修改 agent.ini）
windchill_add_worker(name="OFFICE", host="VM73", exe_path="D:\\office\\worker.bat")
```

## 5. 参考资料
- [PTC Configuring Worker Agent](http://support.ptc.com/help/windchill/plus/r13.0.2.0/en/Windchill_Help_Center/WVSWorkAgent_chp/WorkAgentConfigWorkAgent.html)
- [PTC CS140965 Executable Not Safe](https://www.ptc.com/cn/support/article/cs140965)
