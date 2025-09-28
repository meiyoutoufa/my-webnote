---
sidebar_position: 3
title: "企业级权限管理方案"
---

# 权限管理方案

本方案面向中大型业务系统，提供一套通用、可扩展、可公开分享的权限管理设计：清晰模型 + 高性能评估 + 分层缓存 + 版本戳一致性控制。文档不包含任何企业或敏感信息，适合学习与参考，也可按需定制字段命名后直接落地。

---

## 摘要

- 统一模型：权限码（Action）、上下文（Context）、资源（Resource）、主体（Subject/用户域）
- 混合规则：RBAC（角色授权） + ABAC（属性约束），默认拒绝、拒绝优先
- 高性能评估：批量检查优先、本地 + Redis 双层缓存、规则索引化
- 一致性控制：版本戳（Stamp）驱动的主动失效与自然过期
- 可运维与安全：审计与可观测、穿透与击穿防护、输入与键规范校验

---

## 目录

1. 设计目标与范围  
2. 概念与术语  
3. 权限码目录范式  
4. 数据模型与存储  
5. 规则 DSL（示例）  
6. 权限评估流程（单次/批量）  
7. 分层缓存架构  
8. 版本戳（Stamp）与主动失效  
9. 并发与穿透保护  
10. 服务接口（REST）  
11. SDK 设计与错误处理  
12. 可观测性与审计  
13. 安全与合规  
14. 性能优化与容量规划  
15. 落地步骤（SOP）  
16. 常见问题（FAQ）  
17. 术语表  
18. 键规范与伪代码  
19. 定制映射模板  
20. 改造清单  

---

## 1. 设计目标与范围

- 目标
  - 统一表达：权限码与上下文模型一致，覆盖系统/组织/项目/资源
  - 高性能：支撑高并发，缓存友好，可批量评估
  - 易扩展：支持 RBAC + ABAC + 附加检查（可插拔）
  - 可运维：主动失效、版本戳控制、可观测与审计
- 范围
  - 适用于 B/S 与微服务架构
  - 多租户/多团队/多项目场景
  - 用户-资源-操作三元权限模型

---

## 2. 概念与术语

- 权限码（Action/Permission Code）：如 `system.admin`、`team.manage`、`project.read`、`task.update`
- 上下文（Context）：System / Team(Org/Tenant) / Project / Resource
- 资源（Resource）：任务、文件、文档等，包含用于 ABAC 的关键字段
- 主体（Subject/用户域）：角色、成员关系、动态关系（创建者/负责人/指派对象等）
- 规则（Rule）：RBAC + ABAC 混合；支持 `effect`（allow/deny）、`priority`
- 版本戳（Stamp）：teamStamp、projectStamp、rulesStamp、membershipStamp、resourceStamp

---

## 3. 权限码目录范式

- 命名建议：`{域}.{动作}`，如 `project.read`、`task.update`
- 属性：`code`、`label`、`contextTypes`、`description`
- 组织方式：分层维护（System / Team / Project / Resource），便于运营管理与展示

---

## 4. 数据模型与存储

- 角色与绑定
  - 表：`roles`、`role_bindings`（user/group ↔ role，含 ctxType、ctxId）
  - 索引：`ctxType, ctxId, role` 与 `ctxType, ctxId, userId`
- 成员关系
  - 表：`team_membership`、`project_membership`
  - 索引：`ctxType, ctxId, userId`
- 权限规则（简要）
  - 表：`rules`，JSON 结构，含 `actions`、`subjects`、`resourceSelector`、`constraints`、`effect`、`priority`
  - 加载阶段按 `contextType, ctxId, action` 建倒排索引，便于快速筛选

---

## 5. 规则 DSL（示例）

```json
{
  "id": "rule-1",
  "contextType": "Project",
  "actions": ["project.read", "task.create", "task.update"],
  "subjects": [
    { "type": "role", "value": "project-admin" },
    { "type": "member", "value": "project-members" }
  ],
  "resourceSelector": { "type": "task", "projectField": "$projectId" },
  "constraints": [
    { "field": "task.status", "op": "in", "value": ["Open", "InProgress"] },
    { "field": "task.assigneeId", "op": "equals", "valueFrom": "currentUserId", "optional": true }
  ],
  "effect": "allow",
  "priority": 100
}
```

- 约束（Constraints）操作符示例：`equals`、`in`、`notIn`、`contains`、`gt`、`lt`、`exists`
- 冲突与优先级：默认拒绝；`deny` > `allow`；优先级数字大者优先

---

## 6. 权限评估流程（单次/批量）

- 输入：`userId`、`action`、`context`（teamId/projectId 等）、`resource`（可选，含关键字段）
- 单次检查
  1) 超管/白名单短路允许  
  2) 构建决策缓存键，尝试命中  
  3) 未命中 → 读取/计算用户域缓存（Subject 集合）  
  4) 加载并索引规则缓存，按 `action` 过滤候选规则  
  5) 按约束与优先级评估（`deny` > `allow`）  
  6) 执行附加检查（可插拔），得到最终结果  
  7) 写入决策缓存（带版本摘要）
- 批量检查（推荐）
  - 同一用户 + 上下文 + 多动作/多资源
  - 先复用用户域，再按动作倒排规则筛选，最后仅对需要资源约束的动作迭代资源
  - 显著提升吞吐与缓存命中

---

## 7. 分层缓存架构

- 决策缓存（Permission Decision Cache）
  - Key：`userId + ctxType + ctxId + action + resourceKey + versionDigest`
  - Value：`allow/deny + matchedRuleId + ts`
  - 存储：Redis（建议） + 本地 LRU（二级缓存）
- 用户域缓存（User Domain Cache）
  - Key：`userId + ctxType + ctxId + domainVersion`
  - Value：主体集合（角色、成员、动态域）
- 规则缓存（Rule Cache）
  - Key：`ctxType + ctxId + rulesVersion`
  - Value：已索引规则结构（按 action 倒排）
- 版本戳（Stamp）
  - 键：`teamStamp`、`projectStamp`、`rulesStamp`、`membershipStamp`、`resourceStamp`
  - 用途：构建 `versionDigest`，驱动主动失效

- TTL 建议
  - 决策：30s~5min（±10%抖动）
  - 用户域：5~30min
  - 规则：10~60min

---

## 8. 版本戳（Stamp）与主动失效

- 递增时机
  - 成员加入/移除、角色绑定变化 → `membershipStamp`
  - 规则新增/修改/删除 → `rulesStamp`
  - 团队/项目属性影响权限 → `teamStamp / projectStamp`
  - 资源关键字段（参与约束）变化 → `resourceStamp` 或纳入 `versionDigest`
- 版本摘要（versionDigest）
  - `hash(teamStamp, projectStamp, rulesStamp, membershipStamp, resourceStamp, ...)`
  - “新写旧失”：新摘要写入键后，旧键自然过期，无需逐键删除

---

## 9. 并发与穿透保护

- singleflight 或分布式锁，避免缓存击穿
- 空值短缓存（Negative Cache）抵御穿透
- 批量接口优先，降低热点键竞争
- 本地 + Redis 双层缓存，减少跨进程压力

---

## 10. 服务接口（REST）

- `POST /permission/check`
  - 入参：`{ userId, action, context: {...}, resource: {...} }`
  - 返回：`{ allow: boolean, reason?: string, matchedRuleId?: string }`
- `POST /permission/batchCheck`
  - 入参：`{ userId, actions: [...], context: {...}, resources: [...] }`
  - 返回：`{ results: [{ action, resourceKey, allow }] }`
- `GET /permission/evaluated?userId&context`
  - 返回用户在上下文下的已评估权限集合（用于展示或调试）

---

## 11. SDK 设计与错误处理

- API
  - `Can(userId, action, ctx, resource)`
  - `Must(userId, action, ctx, resource)` → 不允许时抛业务错误（可携带国际化消息）
- 错误码建议
  - `PERM_DENIED`、`PERM_RESOURCE_MISSING`、`PERM_RULE_INVALID`、`PERM_INTERNAL`

---

## 12. 可观测性与审计

- 指标（Metrics）
  - 决策/域/规则缓存命中率
  - P50/P95/P99 耗时
  - Redis QPS、热点键排行
- 审计（Audit）
  - 记录敏感操作评估链路：`ruleId`、约束、最终结论、操作者
- 评估 Trace（采样）
  - 灰度/排障时开启，以追踪规则命中与约束决策

---

## 13. 安全与合规

- 默认拒绝，拒绝优先
- 防权限提升：规则冲突时以拒绝为先
- 输入校验：`resourceKey` 与字段摘要由后端计算/校验
- 超管绕过受控：限定调用面与操作清单
- 缓存投毒防护：键规范不可被外部注入；版本戳来源可信

---

## 14. 性能优化与容量规划

- 性能要点
  - 批量优先，减少重复加载与评估
  - 规则索引化（按 action 倒排），快速筛选
  - 双层缓存 + TTL 抖动（防雪崩）
  - 附加检查轻量化（避免阻塞）
- 容量建议
  - 键空间控制：版本戳参与键，减少删除需求
  - TTL 与抖动：分散过期时间，平滑负载
  - 热点优化：为热点 action/资源加本地快速缓存

---

## 15. 落地步骤（SOP）

1) 梳理权限码目录与上下文边界  
2) 固化角色/成员模型与数据来源  
3) 设计规则表与运营后台（JSON/DSL 录入与校验）  
4) 接入评估 SDK（先单次，后批量优化）  
5) 引入分层缓存与版本戳，接入事件驱动失效  
6) 建立指标、审计与评估 Trace（采样）  
7) 压测与灰度，建立变更/回滚 SOP  

---

## 16. 常见问题（FAQ）

- 规则变更如何快速生效？  
  → 更新规则时递增 `rulesStamp`；成员变化递增 `membershipStamp`；评估键包含 `versionDigest`，自然“新写旧失”。

- 资源字段频繁变更导致缓存不稳？  
  → 仅将参与约束的关键字段纳入 `resourceKey`/摘要；必要时为该资源引入局部 `resourceStamp`。

- 批量检查资源很多怎么办？  
  → 分片处理；先做“用户域 + 动作”初筛，仅对需资源约束的动作落到资源循环。

---

## 17. 术语表

- RBAC：基于角色的访问控制  
- ABAC：基于属性的访问控制  
- Subject（主体）：规则中的选人集合（角色/成员/动态关系）  
- Additional Check（附加检查）：可插拔的业务函数  
- Stamp（版本戳）：用于缓存键的一致性控制

---

## 18. 键规范与伪代码

- Redis 键示例
  - `permission:decision:Project:prj_1:u_42:task.update:task_1001:v_d1e2f3`
  - `permission:domain:Project:prj_1:u_42:v_27`
  - `permission:rules:Project:prj_1:v_13`
  - `permission:stamp:Project:prj_1:rules` → `13`

- 单次评估伪代码
```plaintext
function Can(userId, action, ctx, resource):
  if IsSuperAdmin(userId): return allow

  v = GetVersionDigest(ctx)  // 组合 team/project/rules/membership/resource 等
  key = buildDecisionKey(ctx, userId, action, resource.key, v)

  if cache.exists(key): return cache.get(key)

  domains = loadUserDomains(userId, ctx)   // 带缓存
  rules = loadCompiledRules(ctx)           // 带缓存（按 action 倒排）
  allowed = evalRules(domains, action, resource, rules)
  result = runAdditionalChecks(action, resource, userId, ctx, allowed)

  cache.set(key, result, ttlWithJitter())
  return result
```

- 版本戳与摘要
```plaintext
function BumpStamp(ctx, type):
  // type: membership/rules/project/team/resource
  redis.incr("permission:stamp:" + ctx.type + ":" + ctx.id + ":" + type)

function GetVersionDigest(ctx):
  teamVer   = redis.get("permission:stamp:Team:"    + ctx.teamId    + ":v") or 0
  projVer   = redis.get("permission:stamp:Project:" + ctx.projectId + ":v") or 0
  rulesVer  = redis.get("permission:stamp:Project:" + ctx.projectId + ":rules") or 0
  memberVer = redis.get("permission:stamp:Project:" + ctx.projectId + ":membership") or 0
  return hash(teamVer, projVer, rulesVer, memberVer)
```

---

## 19. 定制映射模板

将以下“通用名”替换为你的系统字段：

- 上下文与主体
  - `Team` → `team` / `org` / `tenant`
  - `Project` → `project`
  - `Resource` → `task` / `file` / `doc`（按资源类型）
  - `userId` → `uid` / `user_uuid`
  - `teamId` → `team_id` / `org_id`
  - `projectId` → `project_id`
- 权限码命名
  - `system.admin`、`team.manage`、`project.read`、`task.update`（可按你的命名风格调整）
- 版本戳（建议保留）
  - `teamStamp`、`projectStamp`、`rulesStamp`、`membershipStamp`、`resourceStamp`
- 键模板（保持结构，替换字段名）
  - `permission:decision:{ctxType}:{ctxId}:{userId}:{action}:{resourceKey}:{versionDigest}`
  - `permission:domain:{ctxType}:{ctxId}:{userId}:{domainVersion}`
  - `permission:rules:{ctxType}:{ctxId}:{rulesVersion}`
  - `permission:stamp:{ctxType}:{ctxId}:{stampType}`

---

## 20. 改造清单（一步步落地）

1) 列出权限码目录与说明，按 `System/Team/Project/Resource` 分层  
2) 统一上下文字段命名与来源（`teamId/projectId` 等）  
3) 将角色/成员模型映射到 `subjects`（role/member/group）  
4) 梳理需资源字段参与约束的动作（如 `task.update`），定义 `resourceKey` 与关键字段摘要  
5) 建立版本戳写入点：成员变更、角色变更、规则更新、资源关键字段更新  
6) 设计缓存 TTL 与抖动，开启本地 + Redis 双层  
7) 为热点动作加批量评估接口与本地短缓存优化  
8) 接入附加检查（仅挂在必须的权限码），保持函数轻量  
9) 打通审计日志与指标观测，添加评估 trace（采样）  
10) 灰度发布，建立变更与回滚 SOP  

---

## 结语

这是一套可以直接面向公众分享与学习的权限管理参考实现。核心在于“RBAC+ABAC 混合模型、分层缓存、版本戳主动失效、批量评估与并发保护”。你可以基于本方案进行二次定制，将字段与键命名替换为你的系统习惯，即可快速落地。