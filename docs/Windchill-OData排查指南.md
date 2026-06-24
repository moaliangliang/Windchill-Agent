# Windchill OData/WRS 常见问题排查

## 1. OData 端点不可见/访问失败

### 现象
- OData API 返回 403 或空结果
- ThingWorx 中看不到 OData 端点

### 排查步骤
| 步骤 | 操作 | 说明 |
|:-----|:------|:------|
| 1 | 浏览器直接访问 OData URL | 确认是否可以访问 |
| 2 | 检查用户权限 | 确认 wcadmin 用户有容器读取权限 |
| 3 | 检查 `codebase/WEB-INF/ODataEntityProvider.xml` | 确认实体集已注册 |
| 4 | 重启 Tomcat | 使配置生效 |

### 常见原因
- 权限不足：用户需要容器的 read 访问权限
- 实体未注册：ODataEntityProvider.xml 中未配置该实体集
- 缓存问题：ThingWorx 需要重新发现端点

## 2. OData 查询返回错误

### GetPartsList 返回空
- 检查 `DefaultConfigSpecView` 首选项是否为空
- 在首选项管理中设置默认视图

### 参数别名导致 NullPointerException
- SPR 15061941
- 避免在复杂查询中使用 `@alias` 参数
- 改用直接参数传递

### 附件上传损坏
- SPR 15229992
- MS Word 文档通过批量请求上传后损坏
- 建议使用三段式上传（Stage1→Stage2→Stage3）

## 3. WRS 性能优化

### 最佳实践
- 使用 `$select` 限制返回字段
- 使用 `$top` 控制返回条数
- 避免深层 `$expand`
- 对大数据量查询使用服务器端分页

## 4. WRS 版本兼容性

### Windchill 12.1 对应 WRS 2.5.x
- WRS 2.5.1 修复了多个 OData 问题
- 升级 WRS 前确认 Windchill 版本兼容性

## 5. 常见 SPR 修复

| SPR | 问题 | 修复版本 |
|:----|:-----|:---------|
| 14324775 | includeHiddenFilter 不生效 | WRS 2.5.1 |
| 14500760 | MadeFromLink $count=false 返回400 | WRS 2.5.1 |
| 14552377 | 别名属性返回错误 | WRS 2.5.1 |
| 14535333 | GetPartStructure 展开空替换件 | WRS 2.5.1 |
| 14939788 | 双下游分支 Parts 查询失败 | WRS 2.5.1 |
| 15061941 | 参数别名 NPE | WRS 2.5.1 |
| 15229992 | Word 上传损坏 | WRS 2.5.1 |

## 参考资料
- [WRS 2.5 Help Center](https://support.ptc.com/help/windchill_rest_services/r2.5/en/windchill_rest_services.html)
- [Windchill REST Services SPR Fixes](https://support.ptc.com/help/windchill/plus/r12.1.2.0/it/Windchill_Help_Center/WCRESTFramework/wccg_restapiaccess_SPRs_fixed_2_5_1.html)
