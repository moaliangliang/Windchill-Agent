# Windchill 类型和属性配置 SOP

## 概述
- **文档编号**: PISX-SOP-SYS-003
- **版本**: A1
- **用途**: 配置 Windchill 自定义类型（软类型）、IBA 属性、分类节点

## 方式一：通过 knowagent XML 工具（推荐）

### 1.1 添加 IBA 自定义属性

```
# 在 knowagent 聊天中说：
"为 WTPart 类型添加一个叫"硬度"的 STRING 类型属性"
```

或手动生成 XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<UDFDefinitions>
  <TypeDef name="WTPart" baseType="wt.part.WTPart">
    <AttributeDef name="硬度" dataType="STRING" description="材料硬度值">
      <Property name="label" value="硬度"/>
      <Property name="searchable" value="true"/>
    </AttributeDef>
    <AttributeDef name="供应商代码" dataType="STRING" description="供应商编号">
      <Property name="label" value="供应商代码"/>
      <Property name="searchable" value="true"/>
    </AttributeDef>
  </TypeDef>
</UDFDefinitions>
```

### 1.2 在服务器上导入

```bash
# 将 XML 文件上传到 Windchill 服务器
# 执行导入
windchill LoadFileDefinition 属性定义.xml
```

### 1.3 重启 MethodServer 生效

```bash
windchill stop
windchill start
```

## 方式二：直接通过 Windchill Web UI

| 步骤 | 路径 | 说明 |
|:-----|:-----|:------|
| 1 | 站点 → 实用程序 → 类型和属性管理 | 打开管理界面 |
| 2 | 选择要修改的类型（如 WTPart） | 在树形结构中展开 |
| 3 | 右键 → 新建属性 | 输入属性名称、数据类型 |
| 4 | 配置属性约束 | 合法值、必填、搜索等 |
| 5 | 保存并重新生成 | 等待服务器更新 |

## 方式三：生命周期模板配置

### 3.1 生成 XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LifecycleTemplates>
  <LifeCycleTemplate name="我的状态模板">
    <State name="INWORK" display="工作中"/>
    <State name="UNDERREVIEW" display="审核中"/>
    <State name="RELEASED" display="已发布"/>
  </LifeCycleTemplate>
</LifecycleTemplates>
```

### 3.2 导入

```bash
windchill LoadFileDefinition 生命周期模板.xml
```

## 方式四：分类节点配置

### 4.1 生成 XML

用 knowagent 的 `windchill_generate_class_xml` 工具生成。

### 4.2 导入

```bash
windchill LoadClassification 分类定义.xml
```

## 方式五：OIR（对象初始化规则）配置

### 5.1 生成 XML

用 knowagent 的 `windchill_generate_oir_xml` 工具生成。

### 5.2 导入

```bash
windchill LoadFileDefinition OIR规则.xml
```

## 验证

- [ ] 新建物料/文档时能看到新增的属性字段
- [ ] 分类节点在新建时可选
- [ ] OIR 规则自动填充默认值
- [ ] 生命周期状态转换正常

## 注意事项

| 注意 | 说明 |
|:-----|:------|
| 备份先做 | 修改类型定义前建议 expdp 备份数据库 |
| 不可回滚 | 删除属性/类型后数据会丢失，请谨慎 |
| 生效时间 | 属性修改后可能需要重启 MethodServer |
| 权限 | 需要站点管理员权限才能修改类型定义 |

## 参考资料
- PTC Windchill Help Center「Type and Attribute Management」
- PTC Importing Types and Global Enumerations
  https://support.ptc.com/help/windchill/r13.0.2.0/en/Windchill_Help_Center/TypeAttrChp_ExportImportTypes_TypeAttrChp_ExportImportTypesImporting.html
- PTC Type and Attribute Management Command-Line Tools (PDF)
  https://www.ptc.com/support/-/media/8F780F60391A42D8A76C2BCFEA424EF9.pdf
- `windchill wt.load.LoadFromFile` (导入类型定义)
