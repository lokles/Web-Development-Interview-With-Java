# Nginx问题



## 1、常用配置参数？

```java 
worker_processes number | auto;  // 设置工作线程数，是Nginx服务器实现并发处理服务的关键所在。
```

```tsx
worker_connections number; // 设置最大连接数
```

```Java 
keepalive_timeout timeout [header_timeout]; // 配置连接超时时间
```

## 2、nginx进程模型？（shopee）

Nginx是**多进程**的，启动时会先启动一个 Master 进程，然后由 Master 进程启动 子Worker 工作进程，Master主要作**配置读取**，**维护 Worker 进程启动-销毁**等，Worker进程对请求进行处理，**Worker进程之间通过共享内存进行通信**，启动Nginx时，默认设置Worker进程数为CPU的核心数。、
## 3、nginx负载均衡？（美团）（滴滴）（平安科技）
round robin (轮询)
random (随机)
weight (权重)
// fair (按响应时长，三方插件)
url_hash (url的hash值)
ip_hash (ip的hash值)
least_conn (最少连接数)
