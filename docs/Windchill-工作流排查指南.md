# Windchill 工作流常见问题排查

## 1. 工作流任务无法完成

### 常见原因
| 原因 | 现象 | 解决 |
|:-----|:------|:------|
| 权限不足 | 审批按钮灰色 | 检查用户是否有审批权限 |
| 任务已分配他人 | 无法操作 | 检查任务所有者 |
| 工作流模板配置错误 | 路由不正确 | 检查工作流模板设计 |

## 2. 工作流触发失败

### 生命周期状态变更触发
- 确认生命周期模板已关联工作流
- 检查触发条件配置
- 确认对象类型匹配

### 事件触发工作流
- SPR 14550821：分类后新建视图版本不触发工作流
- **解决**: 手动触发或升级修复版本

## 3. OData 工作流 API

### 可用操作
| 操作 | 说明 | knowagent 工具 |
|:------|:------|:----------------|
| 查询任务列表 | 按状态过滤 | `windchill_query_workitems` |
| 审批任务 | 通过路由事件完成 | `windchill_approve_task` |
| 驳回任务 | 拒绝路由事件 | `windchill_reject_task` |
| 转派任务 | 转给其他用户 | `windchill_reassign_task` |
| 暂存任务 | 保存备注不提交 | `windchill_save_workitem` |
| 查询可转派用户 | 获取用户列表 | `windchill_get_workitem_reassign_user_list` |

### 任务状态
- `POTENTIAL` — 待分配
- `ACTIVE` — 活跃（可处理）
- `COMPLETED` — 已完成

## 4. 工作流模板配置

### 模板设计注意点
- 投票规则：多个审批人时的通过条件
- 路由条件：基于变量的条件路由
- 截止日期：超时自动处理规则
- 通知：任务分配时的通知设置

### 常见配置错误
- 缺少出口条件：路由没有定义默认出口
- 变量未定义：条件中引用了不存在的变量
- 循环路由：路由形成死循环

## 5. 工作流性能

### 大并发问题
- 大量工作流实例同时运行可能影响 MethodServer 性能
- 建议合理设计工作流模板，避免不必要的并行分支
- 定期清理已完成的工作流实例

### 排查方法
```sql
-- 查活跃工作流
SELECT COUNT(*) FROM wnc12.wfworkitem WHERE status='ACTIVE';

-- 查超时任务
SELECT COUNT(*) FROM wnc12.wfworkitem WHERE deadline < SYSDATE AND status='ACTIVE';
```
