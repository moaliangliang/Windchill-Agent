# Windchill ERP/ESI 集成排查指南

## 概述
- **用途**: ESI/SAP/ERP 集成问题排查
- **相关 CS**: CS364578, CS53764, CS35247, CS111663

## 1. ESI 配置冲突
- CS364578: PSI 安装时报 `Cannot reinclude` 错误
- **解决**: 检查 esi.SearchableTypes.properties.xconf 重复引用

## 2. ERP 发布后无法删除对象
- CS53764: 发布到 ERP 的物料/变更单无法删除
- **原因**: ESI 事务对象仍引用已发布对象

## 3. BOM 行号强制要求
- CS35247: 要求所有组件都要有行号，否则无法发布
- **解决**: 统一设置行号或关闭强制行号检查

## 4. ESI vs ERP Connector
- CS111663: 两者架构差异说明

## 5. SAP 集成常见问题
| 问题 | 原因 |
|:-----|:------|
| 无法创建物料 | ESI workgroup 未启动 |
| 属性不匹配 | TIBCO locale 映射错误 |
| 物料重复发送 | 生命周期/日期变更触发重新发布 |

## 参考资料
- PTC CS364578, CS53764, CS35247, CS111663
- PTC ESI SAP Troubleshooting Guide
  https://support.ptc.com/help/windchill_esi/r2026.0.0.0/es/windchill_esi/esi_users/ESISAPTroubleParts.html
