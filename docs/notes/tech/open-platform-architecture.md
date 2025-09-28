---
sidebar_position: 3
title: "企业级多进程隔离 + 消息总线的开放平台方案"
---

# 开放平台架构全览

本文系统性讲述开放平台的整体架构设计与关键实现，帮助新同学快速理解项目结构、核心流程与扩展点。内容包含：系统组件、消息总线、插件生命周期、宿主管理、诊断与可观测性、配置与部署等。文档已去除敏感信息与企业信息。

适用对象：后端工程师、平台/插件开发工程师、运维/发布工程师。

---

## 1. 架构目标与设计原则

- 插件安全隔离：插件运行在独立进程（Plugin Host）中，避免影响平台主进程与其它插件。
- 稳定可靠：HostBoot 统一负责插件宿主的生命周期（启动/销毁/复用），平台主进程保持稳定。
- 简洁可扩展：采用统一的消息协议（PlatformMessage）与消息总线，通道（控制/资源/能力/报告）扩展简单。
- 单机多进程通信：基于 ZeroMQ 在本机进程间通信（不是网络分布式），部署与调试低成本。
- 最小侵入：平台编排，宿主执行；平台只发起请求，宿主负责 fork/exec 与运行时细节。

---

## 2. 系统组件与职责

- Platform（平台进程）
  - 职责：API/控制编排、插件生命周期管理、资源代理（HTTP/DB/文件等）、平台能力（定时任务等）、状态维护与消息路由。
  - 通信：ZeroMQ Router（监听本机 TCP），面向 HostBoot/Plugin Host 的 Dealer。
  - 关键位置：
    - 初始化流程：app/init.go
    - 消息端点/协议：app/connection, protocol/proto
    - 消息调度：app/handler/platform_handler.go
    - 生命周期编排：app/domain/lifecycle
    - 插件宿主调度：app/domain/dispatch/hostmode, app/domain/dispatch/hostboot
    - 运行态映射：app/domain/dispatch/hostPlugin

- HostBoot（宿主管理进程，独立二进制）
  - 职责：接收平台的 HostLifeCycleRequest（Start/Delete），根据 Host 描述拉起或回收 Plugin Host 进程，报告状态。
  - 启动脚本示例：devops/remote/host_boot_service.sh（将平台地址与运行时路径写入 HostBoot 配置，然后启动 host_boot 二进制）
  - 注意：HostBoot 的 fork/exec 实现在其自身仓库中，本仓库仅通过消息交互控制。

- Plugin Host（插件宿主进程）
  - 职责：承载插件运行逻辑，与平台交换控制/资源/能力消息。
  - 进程模型：一个或多个插件运行在独立的宿主进程中（具体模式见“宿主模式”一节）。

---

## 3. 消息总线与协议

平台在本机通过 ZeroMQ 搭建 Router/Dealer 消息总线。所有交互封装在统一的 PlatformMessage 中，通过 Header 指定源与目标（Source/Distinct），实现路由。

- 顶层消息协议（节选）

```proto
syntax = "proto3";
package protocol;

message PlatformMessage {
  RouterMessage Header = 1;      // Source（源）/ Distinct（目标）路由信息
  ControlMessage Control = 5;    // 控制流（生命周期/开发模式等）
  PluginMessage Plugin = 10;     // 事件流（HTTP/运行时数据/读文件等）
  ResourceMessage Resource = 20; // 资源访问（DB/HTTP/File）
  AbilityMessage Ability = 30;   // 平台能力（定时任务/插件WebService等）
  HostBootMessage HostBoot = 35; // 宿主管理（生命周期/报告）
  // 其他通道省略
}
```

- HostBoot 通道（节选）

```proto
syntax = "proto3";
package protocol;

message HostBootMessage{
    uint64 Heartbeat = 1;
    HostBootReportMessage Report = 2;

    HostLifeCycleRequestMessage HostLifeCycleRequest = 3;
    HostLifeCycleResponseMessage HostLifeCycleResponse = 4;
}

message HostLifeCycleRequestMessage{
    enum HostActionType { Start = 0; Delete = 1; }
    HostDescriptor Host = 1;   // 语言/版本/运行时要求
    HostActionType Action = 2;
    string HostPID = 3;        // 删除时可带 PID
}

message HostLifeCycleResponseMessage{
    HostDescriptor Host = 1;   // 新宿主信息（含 HostID 等）
    ErrorMessage Error = 2;
}
```

- 平台端 ZeroMQ 端点（Router，监听本机 TCP）

```go
func RunPlatform() {
    addr := "tcp://" + config.Config.Address + ":" + config.Config.TCPPort
    router := connection.NewZmqPlatformEndpoint(
        config.Config.ID,
        connection.Platform,
        connection.RouterDumper,
        config.Config.Version,
        addr,
        connection.SocketTypeRouter,
        log.Logger,
        &connection.PBProto{Log: log.Logger},
        true,  // allowMultiConnection
        true,  // sendIDFrame
        true,  // isBinding（Listen）
    )
    h := &platformHandler{
        sync: connection.NewSynchronized(config.Config.TimeoutSec, log.Logger),
        log:  log.Logger,
    }
    h.Assign(router)
    router.StartTimer(time.Millisecond * time.Duration(config.Config.HeartBeatIntervalMs))
}
```

- ZeroMQ 发送/连接（节选）

```go
func (ep *ZmqEndpoint) Connect() common.PluginError {
    switch ep.socketType {
    case SocketTypeRouter:
        ep.socket = zmq4.NewRouter(ctx, zmq4.WithID(zmq4.SocketIdentity(ep.ID)))
    case SocketTypeDealer:
        ep.socket = zmq4.NewDealer(ctx, zmq4.WithID(zmq4.SocketIdentity(ep.ID)))
    }
    if ep.isBinding { _ = ep.socket.Listen(ep.Address) } else { _ = ep.socket.Dial(ep.Address) }
    // 状态处理省略
    return nil
}

func (ep *ZmqEndpoint) Send(content []byte) common.PluginError {
    target, _ := ep.parser.GetTargetEndpointDesc(content) // 从消息头路由目标
    ep.sndMsgChan <- &ZmqPlatformMessage{ id: target.ID, data: content }
    return nil
}
```

---

## 4. 初始化与模块视图

平台初始化顺序（节选）见 app/init.go，涉及：配置、日志、数据库/Redis、心跳、生命周期处理器、同步中心、诊断工具、OpenAPI 配置等。通过统一 onStart 注册，确保依赖有序。

- 目录结构（简化）

- app/application：对外应用层（OpenAPI、能力装配、权限）
- app/controllers：HTTP 控制器（插件、生命周期、能力等）
- app/domain：领域逻辑（lifecycle、dispatch、host、i18n、plugin_status、types 等）
- app/handler：消息调度（platform_handler）
- app/connection：消息端点（ZeroMQ/Proto）
- app/message：消息发送封装（同步/异步）
- app/utils：通用工具与日志
- protocol/proto：跨进程消息协议（.proto）
- devops/remote：启动脚本（platform/host_boot）

---

## 5. 插件宿主管理（HostBoot）与宿主模式

- 宿主模式
  - highPerformance：一个宿主承载单实例/优先复用/按动作控制是否销毁
  - powerSave：一个宿主可承载多个插件（节能模式，代码中用常量标识）
  - dev：本地开发模式（仅复用已有宿主）

- highPerformance 模式（节选）

```go
func (p *highPerformanceMode) AttachHost(req *types.AttachHostReq) (types.IHost, error) {
    if req.Action==Start || req.Action==Stop {
        hostInfo, err := getHostWithInstanceID(req.InstanceUUID)
        if err != nil { hostInfo, err = startNewHost(req.PluginConfig) }
        return hostInfo, err
    }
    return startNewHost(req.PluginConfig)
}

func (p *highPerformanceMode) DetachHost(plugin types.IPlugin, hostInfo types.IHost, action protocol.ControlMessage_PluginActionType, hasErr bool) {
    needKill := !(action==Start || action==Enable)
    if needKill || hasErr { killHost(hostInfo); plugin.BlankingHost() }
}
```

- 启动新宿主（平台 -> HostBoot -> Plugin Host）

```go
func startNewHost(pConfig plugin_config.PluginConfig) (types.IHost, error) {
    hostInfo, err := hostboot.StartHostByPlugin(pConfig) // 发起 HostBoot Start
    if err != nil { return nil, err }
    return host.SyncWaitGetHostByHostID(hostInfo.GetInfo().ID) // 等宿主上报
}
```

- 平台构造 HostBoot Start 请求并同步等待结果

```go
func StartHostByPlugin(pConfig plugin_config.PluginConfig) (types.IHost, error) {
    req := &protocol.HostLifeCycleRequestMessage{
        Host:   types.PluginToHostPB(pConfig),
        Action: protocol.HostLifeCycleRequestMessage_Start,
    }
    return retryTimes(5, req, start)
}

func start(req *protocol.HostLifeCycleRequestMessage) (types.IHost, error) {
    hostBoot, _ := SyncWaitGetHostBoot()
    msg := build_message.BuildHostBootLifecycleMessage(types.HostBootToPB(hostBoot.GetInfo()), req)
    retMsg, err := message.SyncSendMessage(msg) // 同步等待 HostBoot 响应
    if err != nil { return nil, err }
    if retErr := retMsg.GetHostBoot().GetHostLifeCycleResponse().GetError(); retErr != nil {
        return nil, fmt.Errorf(retErr.GetError())
    }
    return host.NewTmpHost(retMsg.GetHostBoot().GetHostLifeCycleResponse().GetHost()), nil
}
```

- HostBoot 生命周期消息构造（带路由头）

```go
func BuildHostBootLifecycleMessage(hostBootInfo *protocol.HostBootDescriptor, req *protocol.HostLifeCycleRequestMessage) *protocol.PlatformMessage {
    return &protocol.PlatformMessage{
        Header: &protocol.RouterMessage{
            Source:   getPlatformInfo(),          // 平台
            Distinct: setHostBootInfo(hostBootInfo), // 目标：HostBoot
            SeqNo:    utils.CreateCaptcha(),
        },
        HostBoot: &protocol.HostBootMessage{ HostLifeCycleRequest: req },
    }
}
```

- 结束宿主（删除）

```go
func KillHost(h types.IHost) {
    hostBoot, err := SyncWaitGetHostBoot()
    if err != nil { return }
    req := &protocol.HostLifeCycleRequestMessage{
        Action:  protocol.HostLifeCycleRequestMessage_Delete,
        Host:    types.HostToPB(h.GetInfo()),
        HostPID: h.GetInfo().PID,
    }
    msg := build_message.BuildHostBootLifecycleMessage(types.HostBootToPB(hostBoot.GetInfo()), req)
    _, _ = message.SyncSendMessage(msg)
}
```

- HostBoot 启动脚本（说明作用）
  - 将平台地址、插件目录、宿主包路径等写入 HostBoot 配置
  - 启动 host_boot 二进制
  - 注意：文档不包含路径细节和企业信息，实际配置请参考部署环境

---

## 6. 运行态映射与状态管理

- 宿主与插件映射（并发安全）

```go
var hostPlugins sync.Map // map[types.IHost][]types.IPlugin

func BindHostByPlugins(plugins []types.IPlugin, host types.IHost) {
    if len(plugins) == 0 { return }
    if host.GetInfo().ID == "" { return }
    hostPlugins.Store(host, filterBlack(plugins))
}

func GetInstanceListByHost(h types.IHost) []types.IPlugin {
    if list, ok := hostPlugins.Load(h); ok {
        return list.([]types.IPlugin)
    }
    return nil
}
```

- 消息分发入口（平台收到消息后的统一路由）

```go
func (ph *platformHandler) OnMessage(_ *connection.EndpointInfo, content []byte) {
    msg := &protocol.PlatformMessage{}
    _ = proto.Unmarshal(content, msg)

    if msg.GetResource() != nil { /* 资源流处理 */ }
    if msg.GetPlugin()   != nil { /* 事件流回包同步 */ }
    if msg.GetControl()  != nil { /* 生命周期回包/报告/开发模式 */ }
    if msg.GetAbility()  != nil { /* 定时任务/插件WebService等 */ }
    if msg.GetHostBoot() != nil { /* HostBoot 生命周期回包/报告 */ }
}
```

---

## 7. 生命周期动作与编排

- 支持动作（示例）：Install/UnInstall/Enable/Disable/Upgrade/Start/Stop、Org 级别批量动作等（详见 protocol/control.proto 与 domain/lifecycle）
- 编排要点：
  - AttachHost：按模式复用或启动宿主
  - DetachHost：按动作决定是否回收宿主（Stop/Start 等场景不杀宿主）
  - 同步调用：通过内部 Synchronized 机制等待回包（带超时）
  - 错误处理：统一封装 ErrorMessage 与错误堆栈，日志沉淀

---

## 8. 资源访问与平台能力

- 资源通道：DB/HTTP/File 等代理能力，统一在 ResourceMessage 里，通过平台调用，插件端只需发起请求。
- 平台能力：定时任务与插件 WebService 代理（AbilityMessage），支持健康检查与代理同步（PluginServiceProxy）。

---

## 9. 诊断与可观测性

- 日志：平台日志、插件日志、HostBoot 报告/诊断日志（DiagTool）。
- 诊断工具：聚合平台与 HostBoot 的日志/状态（app/domain/diagtool），统一下行给前端或运维工具。
- 心跳：平台与宿主/HostBoot 之间保持心跳或报告机制，失联时触发恢复或剔除。

---

## 10. 配置与部署

- 平台配置：config/config.json（监听地址、端口、超时、心跳、版本等）
- HostBoot：通过脚本写入平台地址与运行时路径，再启动 host_boot
- 插件包/宿主包：由部署系统下发至指定目录（文档不含路径与企业信息）

---

## 11. 常见问题（FAQ）

- Q：HostBoot 在哪里实现 fork/exec 拉起 Plugin Host？
  - A：在 HostBoot 仓库中，本仓库通过 HostLifeCycleRequest: Start 调用 HostBoot 来执行。
- Q：为什么要等待 Plugin Host 上报？
  - A：即使 HostBoot 返回“已启动”，仍需等待 Plugin Host 自身上报，确保宿主已就绪。
- Q：多语言插件如何支持？
  - A：由 HostDescriptor 的语言/版本描述 + HostBoot 的运行时/宿主包决定。

---

## 12. 新人快速上手路径

- 阅读顺序建议：
  1) protocol/proto/*.proto（整体消息协议框架）
  2) app/connection（端点与 ZeroMQ 通信）
  3) app/handler/platform_handler.go（消息分发与同步应答）
  4) app/domain/dispatch/hostmode 与 hostboot（宿主管理）
  5) app/domain/lifecycle（生命周期编排）
  6) app/application 与 app/controllers（API 与业务入口）
- 调试建议：
  - 单机启动 Platform 与 HostBoot，观察 HostBoot 报告与 Plugin Host 启动
  - 通过日志与 DiagTool 诊断宿主状态、消息往返
  - 逐步演练 enable/start/stop/upgrade 典型动作

---

## 13. 术语表

- Platform：平台进程，Router 端点，负责编排与消息路由。
- HostBoot：宿主管理进程，执行宿主生命周期。
- Plugin Host：插件宿主进程，承载插件运行逻辑。
- PlatformMessage：统一消息协议的顶层封包，含多通道。
- HostDescriptor：宿主描述（语言/版本/运行时要求）。
- Synchronized：平台侧同步调用/回包匹配组件。
- 宿主模式：highPerformance/powerSave/dev 等。

---

如需更深入的实现细节（特别是 HostBoot 的进程管理与隔离策略），请在对应仓库查看其进程管理模块与配置解析模块。