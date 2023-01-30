服务注册发现：Netflix Eureka -> Alibaba Nacos Service Registry
服务熔断：Netflix Hystrix -> Alibaba Sentinel
动态配置：Netflix Archaius -> Alibaba Nacos Configuration
服务调用：Netflix Ribbon -> Spring Cloud LoadBalancer(Spring Cloud 官方支持，Hoxton 版本后支持)
服务路由：Netflix Zuul -> Spring Cloud Gateway(Spring Cloud 官方支持，Finchley 版本后支持)