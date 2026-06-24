# Windchill 版本升级/迁移指南

## 概述
- **用途**: Windchill 版本升级和系统迁移

## 1. 升级前检查
- [ ] 确认当前版本和补丁级别
- [ ] 检查目标版本的兼容性矩阵
- [ ] 备份数据库 (expdp)
- [ ] 备份配置 (xconfmanager -dump)
- [ ] 确认磁盘空间充足

## 2. 升级步骤
```bash
# 1. 停止服务
windchill stop

# 2. 运行 PSI (PTC Solution Installer)
# 选择 Upgrade 模式

# 3. 执行数据库升级脚本
windchill db_upgrade

# 4. 启动服务
windchill start

# 5. 验证
windchill version
```

## 3. 常见问题
| 问题 | 解决 |
|:-----|:------|
| 升级后 OData 404 | 重新安装 WRS |
| 自定义代码不兼容 | 检查自定义代码兼容性 |
| 数据库升级失败 | 检查数据库日志 |

## 参考资料
- PTC Windchill Upgrade Guide (随版本提供)
- PTC Release Notes
- PTC CS364578 (PSI 安装失败)
