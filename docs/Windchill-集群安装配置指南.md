# Windchill 集群安装配置指南

## 概述
- **文档编号**: PISX-SOP-SYS-016
- **版本**: A1
- **用途**: Windchill 集群环境的安装与配置
- **参考**: PTC Advanced Deployment Guide

## 1. 集群架构

```
负载均衡器 (Load Balancer) — Session Affinity
       │
       ├── 主缓存节点 (Main Cache)
       │   ├── MethodServer + Background MS
       │   ├── 队列处理
       │   └── 数据库连接
       │
       ├── 缓存客户端 (Cache Client)
       │   └── MethodServer（无 Background）
       │
       └── 共享 Vault 存储 (NAS/SAN)
```

### 两种集群类型

| 类型 | 说明 |
|:-----|:------|
| **相同节点集群** | 所有节点可协商为主缓存 |
| **专用主节点集群** | 指定特定节点为主缓存（`wt.cache.main.hostname`）|

## 2. 核心配置参数

```properties
wt.rmi.server.hostname=集群统一主机名（负载均衡器地址）
wt.cache.main.hostname=主节点FQDN
wt.cache.main.secondaryHosts=从节点1FQDN,从节点2FQDN
wt.fv.basedir=共享保险库路径
```

## 3. 安装步骤

### 3.1 基础安装
所有节点安装相同版本的 Windchill。

### 3.2 配置集群属性
```bash
xconfmanager -s wt.cache.main.hostname=主节点FQDN -t codebase/wt.properties
xconfmanager -s wt.rmi.server.hostname=集群统一地址 -t codebase/wt.properties
xconfmanager -p
```

### 3.3 负载均衡器
- 必须启用 **Session Affinity**（sticky session）

## 4. 集群 Rehost

```bash
# rehost.properties
cluster.master.gateway=主节点FQDN
cluster.slave.hostnames=从节点1FQDN,从节点2FQDN

# 主节点
rehost.bat conf/rehost_clustermaster.properties
# 从节点
rehost.bat conf/rehost_clusterslave.properties
```

## 5. 常见问题

| 问题 | 解决 |
|:-----|:------|
| 节点间缓存不一致 | 检查 `wt.cache.main.*` 配置 |
| Session 丢失 | 负载均衡器未配置 affinity |
| MethodServer 无法启动 | 检查 `wt.rmi.server.hostname` |

## 6. 参考资料
- PTC Advanced Deployment Guide「Installing and Configuring a Cluster Windchill Environment」
  https://support.ptc.com/help/windchill/r13.0.1.0/fr/Windchill_Help_Center/WCAdvDeployGuide/WCAdvDepClusterConfig_ServerClustConfigOview.html
- PTC Advanced Deployment Guide「Configuring Windchill with Oracle RAC」
  https://support.ptc.com/help/windchill/r13.0.1.0/es/Windchill_Help_Center/WCAdvDeployGuide/WCAdvDepClusterConfig_WCOracleRACConfig.html
- PTC Rehost Utility Guide (Windchill 12.0.2.0)
  https://www.ptc.com/support/-/media/support/refdocs/Windchill_PDMLink/12,-d-,0/WCRehostUtilityGuide_12_0_2_0.pdf
- PTC Rehost Utility Guide (Windchill 13.1.2.0 最新版)
  https://www.ptc.com/en/support/refdoc/Windchill_PDMLink/13_1/WCRehostUtilityGuide_13_1_2_0
