# Windchill 移动端配置指南

## 概述
- **用途**: Windchill+ (移动端) 配置和问题排查

## 1. Windchill+ 连接配置
- 需要: Windchill 服务器 + OData API + SSL 证书
- URL 配置: `https://服务器/Windchill/servlet/odata`

## 2. 常见问题
| 问题 | 解决 |
|:-----|:------|
| 无法连接 | 检查 SSL 证书和网络 |
| 数据不显示 | 确认 OData API 可访问 |
| 性能慢 | 检查移动端网络环境 |

## 3. 推荐配置
- 启用 SSL/HTTPS
- OData API 使用 v3/v4
- 优化移动端查询(限制返回数据量)

## 参考资料
- PTC Windchill+ Help Center
- PTC WRS OData API Guide
