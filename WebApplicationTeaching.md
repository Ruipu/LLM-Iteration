# 负载均衡器的作用与原理

_Created: 2026-06-15_

---

**You:**

这是什么？Load Balancer
A load balancer is a component that distributes incoming requests across multiple servers or service instances.
Its goal is to prevent any single server from becoming overloaded and to ensure that traffic is handled efficiently. Load balancing improves application performance, reliability, and availability.

---

**Claude:**

A **load balancer** is a component that distributes incoming network requests across multiple servers.

Its main goals are to prevent any single server from being overloaded, improve performance, and ensure reliability/availability. If one server goes down, the load balancer routes traffic to the healthy ones.

---

**You:**

仔细讲解

---

**Claude:**

```
This block is not supported on your current device yet.
```

我来用图解帮你深入理解负载均衡器的工作原理。
```
This block is not supported on your current device yet.
```

下面是详细讲解：

---

**负载均衡器是什么**

简单说，负载均衡器就是站在所有服务器"前面"的一个调度员。所有进来的请求先打到它，再由它决定把请求转发给哪台服务器处理。

---

**核心功能**

**流量分发** — 把请求均匀（或按策略）分摊到多台服务器，避免某一台被压垮，其他台却闲着。

**健康检查** — 负载均衡器会定期"心跳探测"每台服务器（如图中虚线所示）。如果某台服务器挂了，它立刻把流量切走，用户完全无感知。

**高可用保障** — 即使有服务器故障，服务依然能正常运行，实现"零停机"。

---

**三种常见调度算法**

| 算法 | 逻辑 | 适合场景 |
|---|---|---|
| 轮询 | 1→2→3→1→2→3 循环 | 请求处理时间相近时 |
| 最少连接 | 发给当前最"空闲"的服务器 | 请求耗时差异大时 |
| IP 哈希 | 同一用户固定打同一台 | 需要维持会话（Session）时 |

---

**两种部署位置**

- **四层负载均衡（L4）** — 工作在 TCP/IP 层，只看 IP 和端口，速度极快。
- **七层负载均衡（L7）** — 工作在 HTTP 层，能读取 URL、Header、Cookie，可以做更智能的路由（比如把 `/api/` 请求发到后端集群，把 `/static/` 发到 CDN）。

---

点击图中任何组件，可以继续深入了解某个具体细节。

---

**You:**

这和Dispatcherservelet有什么区别？

---

**Claude:**

这是个很好的问题！两者表面上都在"分发请求"，但层次和职责完全不同。
```
This block is not supported on your current device yet.
```


```
This block is not supported on your current device yet.
```

核心区别就一句话：**负载均衡器解决"去哪台机器"，`DispatcherServlet` 解决"进了机器后找谁处理"。**

---

**负载均衡器** 工作在机器与机器之间，完全不知道你的代码里有什么 Controller、什么业务逻辑。它只看：这台服务器还活着吗？现在有多少连接？然后把请求转过去。

**`DispatcherServlet`** 是 Spring MVC 的前端控制器，运行在 JVM 进程内部。请求已经落到某台服务器上了，它才开始工作：解析 URL，找到 `@GetMapping("/users")` 对应的方法，调用 `HandlerAdapter`，执行你的业务代码，最后把 `ModelAndView` 渲染成响应。

---

**用一个比喻理解：**

想象一家有三个分店的餐厅：
- 负载均衡器 = 总部电话客服，决定把你的订单转给哪家分店（选机器）
- `DispatcherServlet` = 进了分店之后的前台，决定叫哪位厨师来做你点的这道菜（选 Controller）

两者串联在一起，请求必须先经过负载均衡器，才会到达某个 Spring 应用实例，然后 `DispatcherServlet` 再接手。

---

**You:**

好那给我重新写一个回答吧

---

**Claude:**

你想让我重新写哪个部分的回答？是：

- 负载均衡器的讲解
- DispatcherServlet 的讲解
- 两者对比的讲解

还是全部重写？另外有什么特别想调整的方向吗，比如更简洁、更深入、或者换个角度？

---

**You:**

What is loader balancer，英文回答

---

**Claude:**

A **load balancer** sits in front of your servers and distributes incoming requests across them, so no single server gets overwhelmed.

---

**You:**

面试回答，写成一个段落

---

**Claude:**

A load balancer is an infrastructure component that sits between clients and a pool of servers, distributing incoming requests across multiple instances to prevent any single server from becoming a bottleneck. It continuously performs health checks on each server, automatically routing traffic away from any instance that goes down, which ensures high availability and fault tolerance. Common distribution algorithms include round robin, which cycles requests evenly across servers, least connections, which sends each new request to the server currently handling the fewest active connections, and IP hashing, which routes the same client to the same server every time to maintain session stickiness. Load balancers can operate at Layer 4, making routing decisions based on TCP/IP information, or at Layer 7, where they can inspect HTTP headers and URLs to enable more intelligent, application-aware routing.

---

