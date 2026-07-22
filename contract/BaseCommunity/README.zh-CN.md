<div align="right">
  <a href="./README.md">English</a> ·
  <a href="./README.zh-CN.md"><strong>简体中文</strong></a>
</div>

# BaseCommunity 与可扩展社群平台

> 本目录是当前 BaseCommunity 架构对外的正式入口。历史 `MerkelGroup` 路径会单独保留，用于兼容旧链接。

> 此处的 “Base” 指抽象合约底座 `BaseCommunity`，不是 Base 区块链网络。

## 状态与范围

UniChat 社群采用分层合约架构：

- `BaseCommunity` 定义所有群类型共享的基础能力；
- 薄 leaf 合约表达具体业务 kind；
- 每种 kind 可使用独立可升级 beacon，使实现可以隔离演进；
- `CommunityKindRegistry` 记录 canonical `kind → beacon` 目录，用于治理与发现。

本文只描述合约架构与扩展模型，不记录逐链部署状态，也不讨论应用层支持情况。

## 开放开发者生态

BaseCommunity 被设计为开放群聊生态的共享底座。第三方团队可以创建新的群聊模式，同时复用平台既有 kind 使用的成员、消息、经济与集成基础能力。

开发保持开放，canonical 注册采用治理审核。团队可以独立设计和实现自己的 kind，负责维护差异化业务能力，并申请将该 implementation 域登记进 `CommunityKindRegistry`。治理审核让 kind 标识、兼容性、权限和升级控制权保持透明，而不是要求所有群都使用同一种产品模式。

标准开发路径为：

1. 定义可长期使用的 kind 与公开规范；
2. 实现一个薄 `BaseCommunity` leaf；
3. 证明 initializer、storage 和接口兼容；
4. 准备独立 implementation 与 beacon；
5. 提交 kind，申请 canonical 治理审核；
6. 提供明确的 creator、实例索引与可选模块集成；
7. 持续维护测试、安全证据与升级策略。

完整流程见[《开发属于你的 Community Kind》](./DEVELOPER_GUIDE.zh-CN.md)，申请 canonical 注册时可直接使用 [Community Kind 提案模板](./KIND_PROPOSAL_TEMPLATE.zh-CN.md)。

## 架构总览

```text
                          BaseCommunity（抽象层）
                  成员 · 消息 · 红包 · 推荐关系 · claim operator
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                    │
                 ▼                    ▼                    ▼
       OfficialCommunity     LauncherCommunity      HooksCommunity
           OFFICIAL               LAUNCHER               HOOKS
                 │                    │                    │
          官方群 beacon        发射器群 beacon        Hook 群 beacon
                 └────────────────────┼────────────────────┘
                                      ▼
                        CommunityKindRegistry
                        canonical kind → beacon 目录
```

每个群实例都是 `BeaconProxy`。已经创建的实例会持续绑定创建时选中的 beacon；修改目录记录不会迁移该实例，也不会升级其 beacon。

## 合约职责

| 组件 | 职责 |
| --- | --- |
| `BaseCommunity` | storage-compatible 抽象基础层，承载成员、Merkle 准入、消息、红包、推荐关系、元数据、暂停控制和跨模块接口。 |
| `OfficialCommunity` | 平台标准官方群；不增加业务 storage，返回 `communityKind() == "OFFICIAL"`。 |
| `LauncherCommunity` | 为发射代币创建的群；返回 `"LAUNCHER"`，token/launcher 发现关系保留在 `TokenLauncherFactory`。 |
| `HooksCommunity` | 流动性社群 leaf；返回 `"HOOKS"`，增加供 Hook 收益系统使用的群主邀请人白名单。 |
| `CommunityFactory` | 通过官方群 beacon 创建并索引官方群。 |
| `CommunityKindRegistry` | owner 治理的 kind、beacon 地址及 active 目录；不建群、不持有群资金，也不升级 beacon implementation。 |
| `CommunityKeyRegistry` | 独立的群公钥与加密会话密钥分发 registry；它不是 kind registry。 |

## BaseCommunity 基础能力

### 成员与 Merkle 准入

- 按 epoch 更新 Merkle root，刷新活跃成员集合；
- proof 绑定群地址、epoch、账号、tier、有效期和 nonce；
- nonce 防止 proof 重放；
- tier 支持群内分层权限；
- 群主可直接邀请成员；
- 授权 claim operator 可通过 `claimJoin` 支持转账或空投后的自动加群。

`BaseCommunity` 内的 Merkle root 证明的是入群资格。代币发放由独立的 `AirdropClaim` 模块承担，群合约本身不铸造或发放领取资产。

### 群消息

- 文本、语音、图片、视频、文件与自定义消息类型；
- 明文或加密消息标记；
- 可配置内容长度与明文策略；
- 序号、内容哈希和可选 CID，便于索引与媒体获取；
- 可选消息费与费用豁免。

### 群红包

活跃成员可创建群红包，并把红包引用作为群消息广播。实际运行需要双向完成配置：

1. 群合约绑定 red-packet 合约；
2. red-packet 合约授权该群为 chat contract。

### 推荐关系 P0

基础层为 invitee 记录唯一 inviter，并提供分页反向查询。它当前只是关系图：

- 忽略自荐；
- 推荐人必须已经是群成员；
- invitee 的推荐人写定后不可替换；
- 无效推荐输入不会阻断原本合法的入群；
- 不代表已实现推荐奖励或自动收益分账。

### 治理与元数据

每个群公开 owner、topic token、最大 tier、名称、头像 CID 与当前 epoch。群主可以调整运维设置、授权 claim operator、绑定红包模块，并暂停或恢复受保护的写操作。

## 群类型

### OFFICIAL

`OfficialCommunity` 是既有标准群的兼容迁移路径。它的业务 storage 与历史 flat Community 字节级对齐，使官方群 beacon 可以在不重建群 proxy 的前提下采用分层架构。

### LAUNCHER

`LauncherCommunity` 复用全部基础能力，并通过 launcher beacon 隔离未来的发射器专属演进。发射记录和 token→community 映射保留在 `TokenLauncherFactory`，不会在群 storage 重复保存。

如果没有设置 launcher beacon，launcher 可以回退使用官方群 beacon；这种回退不等于已经拥有独立可升级的 LAUNCHER kind。

### HOOKS

`HooksCommunity` 为 Uniswap v4 社群经济增加邀请人白名单。外部收益系统在被授权为 claim operator 后可以自动把 LP 加入群，群主与合格邀请人再参与配置好的收益模型。

### 未来类型

外部开发者可以继承 `BaseCommunity`、只增加差异化业务能力，并为新 kind 使用独立 beacon。开发本身保持开放；进入 canonical 目录需要遵循[开发者指南](./DEVELOPER_GUIDE.zh-CN.md)中的治理注册流程。

## 工厂、Beacon 与目录模型

### 建群

- `CommunityFactory` 创建 OFFICIAL proxy，并维护官方群索引；
- `TokenLauncherFactory` 创建发射器群，并维护独立的发射记录与索引；
- Hooks 部署流程为已登记的池集成创建和接线 HOOKS 群。

### 升级隔离

升级某种 kind，指的是升级该 kind 已有 beacon 指向的 implementation。替换 `CommunityKindRegistry` 里的地址只会改变目录记录，不会升级仍挂在旧 beacon 上的 proxy。

### 目录语义

`CommunityKindRegistry` 支持注册 kind、修改目录中的 beacon，以及设置目录项 active/inactive。当前建群方直接持有 beacon 引用，因此 registry 是 canonical 治理与发现记录，不是所有建群流程必经的运行时路由器，也不是自动熔断器。

## 发现规则

链上不存在一张覆盖全部群实例的统一列表。

| 群来源 | canonical 发现方式 |
| --- | --- |
| 官方群 | `CommunityFactory` 列表与 `(topicToken, maxTier)` 索引 |
| 发射器群 | `TokenLauncherFactory` 事件、token 列表、launch record 与 token/community 映射 |
| Hooks 群 | Hook/池集成记录与部署索引 |
| 群类型 | 兼容实例的 `communityKind()`，以及 kind registry 中登记的 beacon |

需要完整实例视图的调用方必须显式合并这些来源。kind registry 编目的是实现域，不负责索引每一个群 proxy。

## 群公钥

群公钥和加密会话密钥分发位于 `CommunityKeyRegistry`，不进入 `BaseCommunity` storage。该 registry 支持：

- 设置或轮换群公钥；
- 撤销 active 公钥并推进 key epoch；
- 发布加密会话密钥包的 Merkle root 与内容寻址位置；
- 为旧 RSA 命名保留兼容别名，同时当前设计采用算法中性、面向 ML-KEM 的术语。

## 跨模块接口

基础层为其他模块提供稳定、最小化的接口：

- 红包读取 `isActiveMember`，创建绑定群的分发；
- 空投和代币流程通过已授权的 `claimJoin` 自动加群；
- 代币发射使用专属 launcher beacon 与独立发现源；
- Hook 收益模块读取群 ownership 与邀请人资格，并在授权后自动把 LP 加入群。

这些集成都需要显式接线和权限配置。继承 `BaseCommunity` 不会自动授权任何外部模块。

## 升级与 Storage 安全

兼容模型遵循四条规则：

1. 既有业务字段保持原顺序、类型和打包边界；
2. 新共享字段通过消耗基础层预留 gap 从尾部追加；
3. kind 专属字段位于完整 base layout 之后，并维护 leaf 自己的 gap；
4. 官方升级在 beacon 切换 implementation 前与历史 layout 做兼容检查。

仓库验证覆盖静态 layout 对比、layout golden snapshot、运行时升级与字段回读、kind 集成测试和 smoke。它们用于降低升级风险，不代表零风险，也不等于外部安全审计认证。

## 旧架构快照说明

历史 [`Community.sol`、`CommunityFactory.sol` 和 `Room.sol`](../MerkelGroup/legacy/v1-clone-room/) 快照描述的是早期 EIP-1167 clone + Room 设计。它们不代表当前社群架构，也不能作为新集成的 source-of-truth。

当前模型已从群核心移除 Room，改用 BeaconProxy 实例，把密钥分发放入独立 registry，并把群类型拆成可独立演进的 kind 域。

## 安全说明

- 建群或升级前核对精确 kind 与 beacon；
- 把 registry owner、beacon owner 与 factory owner 视为不同控制面；
- 不要假设修改目录就会改变既有 proxy；
- Merkle proof 必须绑定群与 epoch，并保留 nonce 防重放；
- 推荐记录必须保持 non-blocking，保护自动加群原子性；
- 红包、空投、launcher 与 Hook 集成都必须显式授权；

---

**架构代际**：BaseCommunity + kind-specific BeaconProxy 域

**Solidity 版本线**：`^0.8.24`，OpenZeppelin Contracts v5.x

**文档状态**：公开合约架构参考
