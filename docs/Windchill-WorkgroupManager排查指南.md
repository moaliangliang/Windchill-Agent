# Windchill Workgroup Manager (WWGM) 排查指南

## 概述
- **用途**: CAD 集成和工作组管理问题排查
- **相关 CS**: CS384354, CS246687, CS218844, CS131176, CS50545, CS43373

## 1. WFS 服务不可用
- CS384354: WFS 服务灰色不可用
- **解决**: 检查 Windows 服务中 PTC WFS Controller 是否运行

## 2. CAD 无法识别 WWGM
- CS246687(SolidWorks), CS218844(通用)
- **解决**: 重新注册 CAD、检查 appregistry.xml、清除 Java 缓存

## 3. 工作区缓存问题
- CS131176: 缓存损坏导致工作区无法初始化
- **最佳实践**: 不跨会话共享缓存，正确设置 PTC_WF_ROOT

## 4. 防病毒软件冲突
- CS50545: creoagent.exe 被防病毒拦截
- **解决**: 排除 WGM 缓存目录和进程文件

## 5. 服务器注册失败
- CS43373: Windchill 服务器无法连接
- **解决**: 检查 SSL 证书、URL 格式、用户权限

## 参考资料
- PTC CS384354, CS246687, CS218844, CS131176, CS50545, CS43373
- PTC WWGM Release Notes (12.1-13.0)
