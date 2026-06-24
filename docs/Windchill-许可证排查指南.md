# Windchill 许可证排查指南

## 概述
- **用途**: Windchill 许可证(License)问题排查

## 1. 许可证过期
- 现象: 登录提示许可证过期
- **解决**: 更新许可证文件到 `<WT_HOME>/license/`

## 2. 许可证不足
- 现象: 用户无法登录，提示许可证不足
- **解决**: 检查同时在线用户数，或购买更多许可证

## 3. FlexNet 服务问题
- 检查 FlexNet License Server 是否运行
- 检查 `lmgrd` 和 `ptc_lichost` 进程
- 检查许可证日志文件

## 参考资料
- PTC License Server Administration Guide
- PTC CS 相关许可证问题
