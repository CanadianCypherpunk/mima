<div align="right">
  <a href="./README.md">English</a> ·
  <a href="./README.zh-CN.md"><strong>简体中文</strong></a>
</div>

<h1 align="center">密马.com 智能合约</h1>

<p align="center">
  面向去中心化社交平台的区块链通信与社区管理合约套件。
</p>

<p align="center">
  <a href="./README.md">
    <img alt="Language: English" src="https://img.shields.io/badge/Language-English-blue">
  </a>
  <a href="./README.zh-CN.md">
    <img alt="语言：简体中文" src="https://img.shields.io/badge/%E8%AF%AD%E8%A8%80-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-red">
  </a>
</p>

---

## 目录

- [概述](#概述)
- [合约模块](#合约模块)
- [架构](#架构)
- [技术栈](#技术栈)
- [核心功能](#核心功能)
- [快速开始](#快速开始)
- [部署指南](#部署指南)
- [应用场景示例](#应用场景示例)
- [安全](#安全)
- [贡献](#贡献)

## 概述

密马.com 是为 Web3 社交应用设计的模块化智能合约生态。它提供代币门控社区、链上消息、经济激励与去中心化治理等基础能力。

### 设计理念

**模块化**：每个合约模块解决一类问题，可独立使用或组合使用。

**可扩展**：通过 Merkle 成员体系与按社区类型隔离的可升级 beacon 域实现省气扩展。

**灵活**：可配置参数，适配不同业务场景。

**安全**：基于久经考验的 OpenZeppelin 库，具备完善的访问控制。

### 可以构建什么？

- **代币门控社区**：要求用户持有指定代币才能加入
- **DAO 沟通平台**：链上消息与治理集成
- **NFT 持有者社区**：按 NFT 持有情况区分访问等级
- **社交金融（SoFi）应用**：聊天与代币奖励、分配结合
- **去中心化 Discord/Telegram**：抗审查的群组沟通

## 合约模块

### 🎁 RedPacketGroup
**用途**：带经济模型的代币门控社区

代币门控群组管理：入场费分配（9% 群主、21% 推荐人、70% 奖池）、子群层级、红包奖励、链上治理投票及完整管理工具。

**主要特性**：
- 入场费分成（9% 群主 / 21% 推荐人 / 70% 红包池）
- 三种红包类型（入场、普通、定时）
- 民主治理（主群 51%、子群 75% 投票）
- 子群管理，独立群主
- 限流、禁言与罚款
- 推荐追踪与排行榜

**适用场景**：投资俱乐部、DAO 社区、游戏公会、订阅群组

**文档**：[contract/RedPacketGroup/README.md](./contract/RedPacketGroup/README.md)

---

### 📨 DirectMessage
**用途**：私密 1 对 1 加密消息

链上点对点消息合约，对话历史永久存储在链上。支持 RSA 公钥注册实现端到端加密、黑名单、消息统计，以及与红包的无缝集成。

**主要特性**：
- 链上消息存储（单条最大 1KB）
- RSA 公钥注册与默认回退
- 黑名单（双向拉黑检查）
- 按时间段的消息统计
- 红包集成（随消息发送代币）
- 对话对象追踪

**适用场景**：去中心化聊天、协商平台、可验证沟通

**文档**：[contract/DirectMessage/README.md](./contract/DirectMessage/README.md)

---

### 🏘️ BaseCommunity 社群平台
**用途**：共享社群底座与可独立演进的业务类型

此处的 “Base” 指抽象合约底座 `BaseCommunity`，不是 Base 区块链网络。

`BaseCommunity` 集中成员、Merkle 准入、消息、红包、推荐关系、自动加群接口和运维控制。薄 leaf 合约复用这些能力，同时通过链上 kind 表达具体业务类型。OFFICIAL、LAUNCHER、HOOKS 可以使用独立 beacon，把某一类型的实现变更限制在自己的升级域内。

**主要特性**：
- 按 epoch 更新的 Merkle 成员体系，含 tier 与 nonce 防重放
- 文本、媒体与加密消息元数据
- 群红包与经授权的自动加群集成
- non-blocking 推荐关系图（P0）
- 兼容 leaf 的 `communityKind()` 链上自描述
- 面向开发者自有群类型的开放扩展模型
- kind-specific BeaconProxy 升级域
- 用于治理与发现的 canonical `kind → beacon` 目录
- 支持轮换与撤销语义的独立群公钥 registry

**适用场景**：官方群、代币发射群、流动性社群与经治理接入的未来扩展

**文档**：[contract/BaseCommunity/README.zh-CN.md](./contract/BaseCommunity/README.zh-CN.md)

**开发者指南**：[开发属于你的 Community Kind](./contract/BaseCommunity/DEVELOPER_GUIDE.zh-CN.md)

---

### 🧧 RedPacket
**用途**：独立代币发放系统

灵活的红包合约，支持个人与群组代币发放。基于推荐机制的代币白名单（质押 UNICHAT 上架代币）、可配置平台抽成、均分/随机分配算法，以及与聊天合约的集成。

**主要特性**：
- 个人红包（1 对 1 转账）
- 群红包（均分或随机）
- 通过 UNICHAT 质押的代币白名单
- 按代币可配置税率
- 过期与退款
- 与聊天合约集成

**适用场景**：代币空投、社区奖励、营销活动、游戏奖励

**文档**：[contract/redpacket/README.md](./contract/redpacket/README.md)

---

### 🪝 UniChatHooks
**用途**：MIMA Uniswap v4 Hook 社区经济

面向 MIMA/USDT 流动性社区的 Uniswap v4 Hook 模块。它串联邀请绑定、已注册 Hook 池、LP 准入、Swap 记录，以及群主、邀请人、LP 的 20/30/50 收益分账。

**主要特性**：
- canonical MIMA/USDT Hook 池注册
- LP 准入前的邀请绑定
- Hook 校验的流动性与 Swap 事件
- 群主、邀请人、LP 收益分账
- 聚合群、池、LP 仓位与可领取收益状态的只读 Lens

**适用场景**：流动性驱动社区、邀请型 LP 活动、MIMA 池运营

**文档**：[contract/UniChatHooks/README.zh-CN.md](./contract/UniChatHooks/README.zh-CN.md)

## 架构

### 系统概览

```
┌─────────────────────────────────────────────────┐
│           UniChat 智能合约套件                   │
└─────────────────────────────────────────────────┘
                        │
       ┌────────────────┼────────────────┐
       │                │                │
       ▼                ▼                ▼
 DirectMessage    RedPacketGroup    BaseCommunity
                                            │
                         ┌──────────────────┼──────────────────┐
                         ▼                  ▼                  ▼
                    OFFICIAL           LAUNCHER             HOOKS
                   官方群 beacon       发射器群 beacon       Hook 群 beacon
                         └──────────────────┼──────────────────┘
                                            ▼
                              CommunityKindRegistry
                               （kind → beacon 目录）

 共享集成：RedPacket · AirdropClaim · CommunityKeyRegistry
```

### 集成模式

#### 模式 1：独立模块
```
DirectMessage（P2P 聊天）
    ↓ 可选
RedPacket（代币礼物）
```
简单点对点消息，可选代币转账。

#### 模式 2：标准社群
```
OfficialCommunity
    ├─> Merkle / 邀请成员
    ├─> 群消息
    ├─> 推荐关系记录
    └─> RedPacket（显式授权）
```
标准群复用完整的 BaseCommunity 基础能力。

#### 模式 3：经济型群组
```
RedPacketGroup
    ├─> 代币门控
    ├─> 入场费 → 红包
    ├─> 治理投票
    └─> 子群
```
带内置经济模型的代币门控社区。

#### 模式 4：代币发射群
```
TokenLauncherFactory
    ├─> StrictLaunchToken
    └─> LauncherCommunity
            ├─> BaseCommunity 基础能力
            └─> 独立 launcher 发现索引
```
发射器群可在专属 beacon 后演进，不改变官方群。

#### 模式 5：社区流动性经济
```
HooksCommunity
    ├─> LP 自动入群（经授权 claim operator）
    ├─> 邀请人资格
    └─> 外部 Hook 收益系统
```
流动性社群复用 BaseCommunity，同时把池与收益逻辑留在群核心之外。

### 重要边界

- `CommunityKindRegistry` 编目 kind beacon；当前工厂并非统一通过它路由建群。
- 官方群、发射器群与 Hook 群使用不同发现源，不存在一张全局实例列表。
- 修改 registry 目录项不会改变既有 proxy 创建时绑定的 beacon。

## 技术栈

### 智能合约层

- **Solidity**：^0.8.24 - ^0.8.28
  - 现代语言特性
  - 自定义错误以节省 gas
  - 打包结构体优化存储

- **OpenZeppelin Contracts**：v5.x
  - `Ownable`：单所有者访问控制
  - `AccessControl`：基于角色的权限
  - `ReentrancyGuard`：重入保护
  - `Pausable`：紧急暂停
  - `SafeERC20`：安全代币交互
  - `MerkleProof`：白名单验证

### 设计模式

- **BeaconProxy + UpgradeableBeacon**：可升级群实例
  - 用于 OFFICIAL、LAUNCHER 与 HOOKS 社群域
  - 拥有专属 beacon 的 kind 可在自己的升级域内演进
  - 既有群 proxy 切换 implementation 前需要校验 storage 兼容性

- **EIP-1167 最小代理**：仍可用于 GroupFactory 等 legacy 或独立模块，但已不是当前 CommunityFactory 模型

- **工厂模式**：标准化合约创建
  - 可预测地址
  - 一致初始化
  - 与注册表集成

- **Merkle 树验证**：可扩展白名单
  - O(log n) 验证 gas
  - 单根存储
  - 保护隐私

### 存储优化

**打包结构体**：
```solidity
struct Member {
    bool exists;        // 1 字节
    uint64 joinAt;      // 8 字节  
    uint32 subgroupId;  // 4 字节
}                       // 总计：13 字节（一个槽位）
```

**带分页的动态数组**：
- 避免无界循环
- 省气的查询
- 便于前端使用

**基于映射的存储**：
- O(1) 查找
- 无需遍历
- 最小 gas 成本

## 核心功能

### 通信

#### 链上消息
- **永久存储**：消息永久保存在区块链
- **分页**：高效获取长对话历史
- **内容类型**：明文与加密消息
- **大小限制**：依合约不同约 280–2048 字节
- **元数据**：支持 IPFS CID 引用富内容

#### 加密支持
- **RSA 公钥**：链上密钥注册
- **客户端加密**：保护隐私
- **密钥轮换**：更新密钥不丢历史
- **社群群密钥**：独立 registry 支持轮换、撤销与经 Merkle 分发的加密会话密钥包

#### 管理工具
- **黑名单**：用户自主拉黑
- **限流**：防刷（日/周限制）
- **禁言**：管理员临时静音
- **罚款**：经济惩罚（以 USDT 计）
- **踢出/封禁**：移除违规成员

### 代币经济

#### 入场费
```
新成员支付：100 代币
├─> 9% → 群主
├─> 21% → 推荐人（或平台）
└─> 70% → 红包池
    └─> 分发给现有成员
```

#### 红包
1. **入场红包**：加入时自动创建（入场费的 70%）
2. **普通红包**：用户创建，任意代币与金额
3. **定时红包**：每日自动发放（最长 365 天）

#### 分配算法
- **随机**：预分配金额，带随机因子
- **均分**：所有领取者平均分配
- **顺序**：先到先得领取

#### 代币白名单
- **核心代币**：平台批准（USDT、WETH 等）
- **社区代币**：通过 UNICHAT 质押添加
- **抽成**：按代币可配置平台费率

### 访问控制

#### 代币门控
- **余额检查**：须持有群组代币
- **验证合约**：自定义资格逻辑
- **最低持有量**：可配置阈值
- **多代币**：不同群可设不同代币

#### Merkle 树白名单
- **省气**：单根验证
- **隐私**：不暴露成员列表
- **可更新**：按 Epoch 刷新
- **分级**：1–7 级会员

#### 治理
- **主群选举**：需 51% 成员投票
- **子群选举**：75% 投票阈值
- **冷却期**：6 个月 / 4 年
- **即时生效**：达到阈值自动执行

### 层级与组织

#### RedPacketGroup 结构
```
主群（成员不限）
├─> 子群 1（成员子集）
├─> 子群 2（成员子集）
└─> 子群 N（成员子集）

入场费分成：
- 仅主群：9% 主群主
- 含子群：4.5% 主群 + 4.5% 子群
```

#### BaseCommunity 结构
```
BaseCommunity（共享基础能力）
├─> OfficialCommunity（OFFICIAL）
├─> LauncherCommunity（LAUNCHER）
└─> HooksCommunity（HOOKS）

升级边界：拥有专属 beacon 的 kind 使用自己的既有 beacon 域
实例索引：由相应 factory 或集成域分别维护
```

## 快速开始

本仓库是 UniChat 合约对外的白皮书与架构参考，不是可独立运行的 Hardhat 或 Foundry 工程；仓内 Solidity 文件属于说明性快照，不是生产源码的 source of truth。

1. 先阅读 [BaseCommunity 架构说明](./contract/BaseCommunity/README.zh-CN.md)。
2. 再阅读与集成能力对应的模块 README。
3. 集成前确认预期的 factory、kind、beacon 与 registry 关系。
4. 不要根据本仓库的 legacy 快照推导当前生产行为。

## 部署指南

### 1. RedPacketGroup 部署

```solidity
// 步骤 1：部署注册表
UniChatRegistry registry = new UniChatRegistry(
    unichatToken,
    usdtToken,
    bigOwner,
    platformOwner,
    treasury
);

// 步骤 2：部署群组实现
RedPacketGroup implementation = new RedPacketGroup();

// 步骤 3：部署工厂
GroupFactory factory = new GroupFactory(
    address(registry),
    address(implementation)
);

// 步骤 4：配置注册表
registry.setGroupFactory(address(factory));

// 步骤 5：白名单代币
registry.approveTokenListing(myToken, true);

// 步骤 6：创建群组
address group = factory.createGroup(
    myToken,
    100e18,
    "我的社区",
    "请互相尊重"
);
```

### 2. BaseCommunity Kind 部署

```text
1. 部署对应的 BaseCommunity leaf implementation。
2. 为该 kind 部署或选择 UpgradeableBeacon。
3. 把 kind 与 beacon 登记进 canonical 目录。
4. 配置真正使用该 beacon 的创建方：
   - OFFICIAL 使用 CommunityFactory
   - LAUNCHER 使用 TokenLauncherFactory
   - HOOKS 使用 Hook 集成流程
5. 创建并初始化 BeaconProxy 群实例。
6. 显式接线 red-packet、claim operator、launcher 或 Hook 权限。
7. 发布 Merkle root 与集成所需的协议元数据。
```

只修改 registry 目录项不会升级既有 proxy，也不会重设工厂直接持有的 beacon 引用。部署或集成前请先阅读 [BaseCommunity 架构说明](./contract/BaseCommunity/README.zh-CN.md)。

### 3. RedPacket 部署

```solidity
// 部署 RedPacket 合约
UniChatRedPacket redPacket = new UniChatRedPacket(
    unichatToken,
    treasury,
    360000,      // 默认 100 小时过期
    10000e18     // 1 万 UNICHAT 质押量
);

// 配置核心代币
redPacket.setCoreToken(usdtAddr, true, "QmUSDT...");
redPacket.setCoreToken(wethAddr, true, "QmWETH...");

// 设置税率
redPacket.setDefaultTaxRate(usdtAddr, 100);  // 1%
redPacket.setDefaultTaxRate(wethAddr, 50);   // 0.5%

// 白名单聊天合约
redPacket.setChatContract(directMessageAddr, true);
redPacket.setChatContract(communityAddr, true);
```

### 4. DirectMessage 部署

```solidity
// 使用默认 RSA 密钥部署
DirectMessage dm = new DirectMessage(
    platformOwner,
    defaultRSAPublicKey
);

// 关联 RedPacket
dm.setRedPacket(address(redPacket));
```

## 应用场景示例

### 示例 1：投资 DAO

```
组件：RedPacketGroup + RedPacket

配置：
- 群组代币：DAO 治理代币
- 入场费：1000 DAO 代币
- 子群：VIP（大户）、研究、交易

特性：
- 入场费即时奖励现有成员
- 子群内违规交易规则罚款
- 从金库的每日定时奖励
- 治理投票管理金库
```

### 示例 2：NFT 社区

```
组件：OfficialCommunity + RedPacket

配置：
- 主题代币：NFT 合集地址
- 等级：1–5 基于 NFT 持有
- 群消息：文本、加密消息与媒体 CID

特性：
- Merkle 白名单（仅 NFT 持有者）
- 按等级的群成员权限
- 向持有者发放红包空投
- 推荐关系记录与加密群讨论
```

### 示例 3：P2P 市场

```
组件：DirectMessage + RedPacket

配置：
- 用户 A 挂单
- 用户 B 通过 DirectMessage 议价
- 通过个人红包付款
- 链上留存交易证明

特性：
- 永久议价记录
- 集成支付
- 通过消息历史解决纠纷
```

### 示例 4：游戏公会

```
组件：RedPacketGroup + BaseCommunity kinds

配置：
- RedPacketGroup：主公会（入场费）
- OfficialCommunity：按游戏建立标准群
- 子群：战队/小队

特性：
- 入场费注入奖池
- 按游戏的社群
- 随机红包赛
- 按表现升级等级
```

## 安全

### 审计状态

⚠️ **审计前**：本合约尚未经过审计，使用风险自负。

**主网上线前建议**：
1. 专业安全审计
2. 充分测试网部署
3. 漏洞赏金计划
4. 限额渐进上线

### 安全机制

#### 访问控制
- **基于角色**：OWNER、BIG_OWNER、成员等
- **修饰符**：`onlyOwner`、`onlyMember` 等
- **校验**：零地址、金额校验

#### 重入保护
- **nonReentrant**：所有外部代币交互
- **检查-效果-交互**：遵循该模式
- **先更新状态**：再外部调用

#### 输入校验
- **地址**：禁止零地址
- **金额**：禁止零金额（适用处）
- **数组边界**：分页限制
- **状态**：红包状态、成员状态

#### 应急控制
- **Pausable**：所有者可紧急暂停
- **BIG_OWNER**：平台管理员覆盖权限
- **提款**：所有者紧急提款函数

### 已考虑常见漏洞

✅ **重入**：所有代币转账使用 ReentrancyGuard  
✅ **整数溢出**：Solidity 0.8+ 内置检查  
✅ **访问控制**：完整角色检查  
✅ **抢跑**：视为链上特性接受  
✅ **拒绝服务**：分页避免无界循环  

### 用户建议

**群主**：
- 设置合理入场费
- 关注定时红包
- 积极管理
- 明确群规

**成员**：
- 核实群组真实性
- 理解经济模型
- 妥善保管私钥
- 举报异常行为

**开发者**：
- 在测试网充分测试
- 关注漏洞与依赖更新
- 遵循升级流程

## 贡献

欢迎贡献，可以从以下方面参与：

### 开发新的 Community Kind

欢迎外部团队基于 BaseCommunity 构建具有差异化能力的群聊类型。开发本身保持开放；成为平台 canonical kind 时，需要通过透明的治理审核，明确标识、兼容性、权限与升级控制权。

请先阅读 [Community Kind 开发者指南](./contract/BaseCommunity/DEVELOPER_GUIDE.zh-CN.md)，再提交 kind 规范、兼容性证据、creator/发现方案与升级策略进行审核。

提案可以先作为文档贡献提交；implementation 可以在独立公开仓库中维护，并通过 [Community Kind 提案模板](./contract/BaseCommunity/KIND_PROPOSAL_TEMPLATE.zh-CN.md)引用。

### 贡献方向

- **测试**：完善测试用例
- **文档**：改进说明与示例
- **优化**：Gas 优化建议
- **安全**：报告漏洞
- **功能**：提出或实现新功能

### 贡献流程

1. **Fork 仓库**：在 GitHub 创建你的 Fork
2. **创建分支**：`git checkout -b feature/amazing-feature`
3. **编写测试**：新代码需有测试覆盖
4. **遵循风格**：与现有代码风格一致
5. **提交**：使用约定式提交（feat/fix/docs）
6. **测试**：跑完整测试
7. **Pull Request**：提交并写清说明

### 代码风格

```solidity
// 推荐
function createPacket(
    address token,
    uint256 amount
) external nonReentrant returns (uint256 packetId) {
    require(amount > 0, "ZeroAmount");
    // ...
}

// 使用自定义错误
error ZeroAmount();
function createPacket(...) external {
    if (amount == 0) revert ZeroAmount();
}
```

### 测试要求

- 所有函数有单元测试
- 多合约流程有集成测试
- 覆盖边界情况
- Gas 优化测试
- 安全相关测试

## 许可证

MIT License，详见各合约文件。

Copyright (c) 2024 UniChat

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED.

## 链接与资源

- **文档**：见各合约 README
- **GitHub**：<repository-url>
- **官网**：<project-website>
- **Discord**：<discord-invite>
- **Twitter**：<twitter-handle>

## 路线图

### 已交付基础
- [x] 共享的 `BaseCommunity` 存储与行为层
- [x] `OFFICIAL`、`LAUNCHER`、`HOOKS` 三类实现域
- [x] 按类型隔离的可升级 beacon 与兼容性校验
- [x] 推荐关系 P0 与 `CommunityKeyRegistry` v2 合约模型

### 架构演进
- [ ] 基于 `BaseCommunity` 扩展更多经治理接入的群类型
- [ ] 更丰富的 kind 专属治理与管理策略
- [ ] 扩展 claim、密钥、红包与 Hook 模块的跨合约标准
- [ ] 强化 storage layout 与升级不变量的自动验证

### 生态扩展
- [ ] 通过治理接入更多社区类型
- [ ] 统一协议层发现与索引约定
- [ ] 高级治理与管理模块
- [ ] 分析与运行可观测性

## 支持

- **问题**：在 GitHub Issues 反馈
- **安全**：security@unichat.io（漏洞相关）
- **文档**：本 README 与合约文档
- **社区**：加入 Discord

---

**为 Web3 社交而建 ❤️**

**文档修订日期**：2026-07-22

**社区架构**：`BaseCommunity` 与按类型隔离的 `BeaconProxy` 域

**设计目标**：EVM 兼容智能合约系统
