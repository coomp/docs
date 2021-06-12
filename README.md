# docs
global docs

## 整体计划

+ 全局设计
+ 功能模块设计
+ kubernetes 环境搭建
+ 协议设计
+ 模块设计
+ 模块开发测试
+ 集成测试

## 全局设计

+ 是否全部依赖 k8s ？
[y] k8s based

+ 是否支持负载均衡
[y] ingress (nginx or traefik)

+ 基本的数据流
[y] Internet -> LB -> Ingress -> Biz Gateway -> Bus -> Service(n)^*

+ 事件驱动的协议
[y] 哪个快用哪个

+ 扩展性
[y] k8s auto scale

+ 监控
[y] Prometheus + grafana

+ 客户端
[y] Go/Java

+ 瓶颈
[y] 一套业务配置一套MQ （租户隔离）

+ 可靠性
[x] 尽可能做到最高

+ 功能
[x] 实现 go 、java 的脚手架接口
[y] 支持串行，分支
[x] 可视化的配置
