# Eureka

Eureka是Spring Cloud中用于服务注册和发现的重要组件。

## Eureka 重要的组件

### Eureka Server 

服务注册中心，提供服务注册和发现功能。

### Eureka Client

服务实列，负责将自身注册到注册中心，并定期发送心跳以保证注册状态。

## 基本概念也是它的服务治理功能

### 服务注册

服务向Eureka Server 注册自己的信息（如主机名、端口、健康状态等）。

### 服务发现

客户端从Eureka server查询可用额服务实列，用于负载均衡和调用。

### 心跳机制

服务向注册中心发送心跳，告诉他们我还活着。

### 剔除机制

注册中心会移除长时间没有发送心跳的服务。
