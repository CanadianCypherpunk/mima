<div align="right">
  <a href="./DEVELOPER_GUIDE.md">English</a> ·
  <a href="./DEVELOPER_GUIDE.zh-CN.md"><strong>简体中文</strong></a>
</div>

# 开发属于你的 Community Kind

BaseCommunity 是一个开放扩展模型。第三方团队可以创建具有独特业务能力的链上群聊类型，同时直接复用成员、Merkle 准入、消息、红包、推荐关系和授权自动入群等公共能力，不必重新开发整套群聊底层。

生态遵循一条简单规则：

> **开放开发，治理注册。**

任何开发者或组织都可以设计和实现新的 Community Kind。canonical 注册由治理管理，用于统一审核 kind 标识、storage 兼容性、权限模型与升级控制权。

## 平台提供什么

每个兼容 kind 都可以复用 BaseCommunity 的协议能力：

- 按 epoch 更新的 Merkle 准入与 tier 成员体系；
- 邀请入群和 claim operator 自动入群；
- 文本、媒体与加密消息元数据；
- 群红包集成；
- non-blocking 推荐关系；
- 群元数据与运维控制；
- 稳定的 `communityKind()` 类型标识；
- kind 专属可升级 beacon 隔离；
- 通过 `CommunityKindRegistry` 进行 canonical 发现。

## Kind 开发者负责什么

Kind 开发者定义自己的群聊模式为什么与众不同，例如游戏公会、创作者会员、DAO 协作、研究社群、商业社群或其他垂直场景。

开发者需要负责：

- kind 专属 storage 与业务规则；
- 权限与信任边界；
- implementation 与 beacon 方案；
- 建群方式与实例索引；
- 外部模块接线；
- 测试、安全分析、文档与后续升级。

## 扩展生命周期

### 1. 定义 Kind

选择稳定且非零的 `bytes32` 标识，并说明该 kind 与其他群类型的业务差异。kind 应代表可长期使用的群聊模型，而不是一次活动或某个单独部署。

### 2. 实现薄 Leaf

继承 `BaseCommunity`，复用其 initializer，并覆写 `communityKind()`。公共能力保留在 base 层，只在 leaf 中增加属于该 kind 的差异化行为。

简化结构：

```solidity
contract ExampleCommunity is BaseCommunity {
    constructor() {
        _disableInitializers();
    }

    function initialize(
        address communityOwner,
        address unichatToken,
        address treasury,
        address topicToken,
        uint8 maxTier,
        string calldata name,
        string calldata avatarCid
    ) external initializer {
        __BaseCommunity_init(
            communityOwner,
            unichatToken,
            treasury,
            topicToken,
            maxTier,
            name,
            avatarCid
        );
    }

    function communityKind() public pure override returns (bytes32) {
        return "EXAMPLE";
    }

    // kind 专属 storage 必须追加在完整 base layout 之后并经过审核。
}
```

这段代码只展示扩展边界；生产实现必须使用当前经过审核的 BaseCommunity 版本与 storage 规则。

### 3. 证明兼容性

候选 kind 应证明：

- initializer 签名和初始化安全符合要求；
- `communityKind()` 稳定且唯一；
- kind storage 只追加在完整 base layout 之后；
- 保留经过审核的 leaf 专属 storage gap；
- BaseCommunity 接口与不变量不被破坏；
- 每个外部模块都经过显式授权；
- 已覆盖负向路径、升级和字段回读测试。

修改继承字段的顺序、类型、打包方式或 initializer 语义，不属于兼容扩展。

### 4. 准备升级域

使用独立 `UpgradeableBeacon` 承载 leaf implementation。必须公开 beacon owner 与升级流程，让用户和集成方明确后续 implementation 变更由谁控制。

### 5. 申请 Canonical 注册

向治理提交以下资料：

- kind 名称与 `bytes32` 标识；
- 公开规范与源码引用；
- implementation 与 beacon 信息；
- beacon ownership 与升级策略；
- storage layout 对比；
- 测试与安全证据；
- 权限模型；
- creator 与发现方案；
- 集成与维护联系人。

审核通过后，registry owner 可以调用 `registerKind(kind, beacon)`。注册只会让 kind 出现在 canonical 目录中，不会自动创建群实例，也不会转移 beacon ownership。

### 6. 创建并索引群实例

每个 kind 都需要明确的创建入口。开发者可以提供专属 factory，或采用其他经过审核的 creator，通过该 kind 的 beacon 初始化 `BeaconProxy` 实例。

creator 还需要说明如何索引群实例。`CommunityKindRegistry` 编目的是 implementation 域，不是每一个群实例。

### 7. 接入可选模块

红包、空投、代币发射、收益系统和其他模块通过显式接口与权限接入。新 kind 会继承公共能力，但任何外部模块都不会被自动信任或启用。

## 治理边界

生态对外部开发保持开放，同时采用治理式 canonical 注册：

- 设计或实现新 kind 不需要治理许可；
- 独立实现可以在 canonical 目录之外继续演进；
- 成为平台正式识别的 kind 需要审核和 registry 批准；
- registry active 状态不是直接持有 beacon 引用的 creator 的运行时熔断器；
- 修改 registry 不会升级既有 proxy；
- 已有 proxy 的升级由其当前绑定 beacon 的 owner 控制。

这条边界既保留开放创新，也让正式互操作与升级控制权保持透明。

## 开发者就绪清单

- [ ] kind 拥有稳定名称、标识和公开规范。
- [ ] leaf 继承经过审核的 BaseCommunity 版本。
- [ ] initializer 与 storage layout 兼容。
- [ ] kind 专属权限已经文档化并经过测试。
- [ ] implementation 与独立 beacon 可供审核。
- [ ] beacon ownership 与升级策略已经披露。
- [ ] creator 和实例发现路径已经定义。
- [ ] 可选模块全部采用显式授权。
- [ ] 升级、负向路径和字段回读测试通过。
- [ ] 治理注册资料完整。

## 开始开发

1. 阅读 [BaseCommunity 架构](./README.zh-CN.md)。
2. 为新 kind 编写简短公开规范。
3. 实现最小且兼容的 leaf。
4. 准备 storage、测试、权限和升级证据。
5. 填写 [Community Kind 提案模板](./KIND_PROPOSAL_TEMPLATE.zh-CN.md)，提交 kind 并申请 canonical 审核。

这个生态不是要求所有群都使用同一种模式，而是让每一种新模式都建立在共同、可组合的底座之上。
