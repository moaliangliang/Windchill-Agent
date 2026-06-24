# Windchill ThingWorx 集成指南

## 概述
- **用途**: Windchill ↔ ThingWorx 集成配置和问题排查

## 1. OData 端点配置
- ThingWorx 中配置 OData 连接器
- URL: `http://服务器:7380/Windchill/servlet/odata/v3`
- **常见问题**: 端点不可见(权限问题)

## 2. ThingWorx Flow 集成
- 创建工作流触发器
- 配置 Windchill 事件订阅
- SPR 14550821: 分类后新建视图版本不触发工作流

## 3. 双因子认证
- ThingWorx Navigate 的 Windchill SSO 配置
- 确认 SSL 证书配置正确

## 参考资料
- PTC ThingWorx Navigate Installation Guide
  https://support.ptc.com/help/thingworx/navigate/r9.7/en/
- PTC WRS Help Center
  https://support.ptc.com/help/windchill_rest_services/r2.5/en/
