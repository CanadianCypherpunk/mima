<div align="right">
  <a href="./KIND_PROPOSAL_TEMPLATE.md">English</a> ·
  <a href="./KIND_PROPOSAL_TEMPLATE.zh-CN.md"><strong>简体中文</strong></a>
</div>

# Community Kind 提案模板

使用本模板申请将新的 BaseCommunity 兼容 kind 纳入 canonical 治理审核。所有结论应提供证据，并尽可能链接公开规范、源码与测试结果。

## 1. Kind 身份

- **Kind 名称**：
- **`bytes32` 标识**：
- **一句话用途**：
- **维护者或组织**：
- **公开联系方式**：

## 2. 问题与群聊模型

说明该 kind 提供什么群聊行为、服务哪些用户，以及为什么它应成为可复用 kind，而不是一次性集成。

## 3. BaseCommunity 复用范围

列出未经修改直接复用的 BaseCommunity 能力：

- 成员与 Merkle 准入；
- 消息；
- 红包；
- 推荐关系；
- claim operator；
- 元数据与运维控制；
- 其他共享接口。

## 4. Kind 专属行为

记录 leaf 新增的每一条规则、状态变量、事件和外部函数。

## 5. Initializer 与 Storage 兼容

- **BaseCommunity 版本或源码引用**：
- **Initializer 兼容证据**：
- **Storage layout 对比**：
- **Leaf 专属 storage gap**：
- **升级与字段回读测试结果**：

说明为什么本扩展没有修改继承字段顺序、类型、打包方式或 initializer 语义。

## 6. 权限与信任模型

列出所有特权角色、operator、owner 控制操作与外部合约权限。说明失败行为，以及权限如何转移、撤销或暂停。

## 7. Implementation 与升级域

- **Implementation 源码**：
- **Implementation 引用**：
- **UpgradeableBeacon 引用**：
- **Beacon owner**：
- **升级策略**：
- **紧急处理流程**：

## 8. 创建与发现

说明群实例如何创建和索引：

- creator 或 factory；
- 初始化参数；
- 建群事件；
- 实例查询或索引来源；
- 与 `CommunityKindRegistry` 的关系。

## 9. 可选模块集成

针对红包、空投、launcher、收益系统或其他集成，逐项说明接口、所需授权和信任边界。

## 10. 测试与安全证据

- 单元测试与集成测试；
- 负向路径测试；
- storage 与升级测试；
- 权限测试；
- 已知限制；
- 审计或评审状态，不夸大安全保证。

## 11. 注册请求

说明希望执行的 registry 操作，并确认：

- [ ] kind 标识稳定且非零；
- [ ] 源码与规范引用公开；
- [ ] beacon ownership 已披露；
- [ ] creator 与发现路径已经定义；
- [ ] 已提供 storage 与 initializer 兼容证据；
- [ ] 权限和外部集成已经文档化；
- [ ] 维护者接受后续升级与信息披露责任。

## 12. 维护计划

说明维护者、发布流程、升级通知、事件响应，以及 kind 的弃用或替换流程。
