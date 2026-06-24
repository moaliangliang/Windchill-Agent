# Windchill Solr 索引排查指南

## 概述
- **用途**: Solr 全文搜索索引问题排查

## 1. Solr 连接失败
- 现象: MethodServer 日志报 `Connection refused` to Solr
- **解决**:
  1. 检查 Solr 服务是否运行
  2. 检查 `wt.solr.server.url` 配置
  3. 重启 Solr 服务

## 2. 搜索无结果
- 原因: 索引未构建或损坏
- **解决**: 重建索引
  ```bash
  windchill wt.index.SolrIndexRebuild
  ```

## 3. 索引构建失败
- 检查 MethodServer 日志中的具体错误
- 确认 Solr schema 与 Windchill 版本匹配

## 参考资料
- PTC Windchill Help Center「Solr Configuration」
- SPR 相关 (MethodServer 日志报错)
