<div align="right">
  <a href="./KIND_PROPOSAL_TEMPLATE.md"><strong>English</strong></a> ·
  <a href="./KIND_PROPOSAL_TEMPLATE.zh-CN.md">简体中文</a>
</div>

# Community Kind Proposal Template

Use this template to propose a new BaseCommunity-compatible kind for canonical governance review. Keep claims evidence-based and link to public specifications, source and test results where available.

## 1. Kind Identity

- **Kind name**:
- **`bytes32` identifier**:
- **One-sentence purpose**:
- **Maintainer or organization**:
- **Public contact**:

## 2. Problem and Community Model

Describe the community behavior this kind enables, who it serves and why the behavior should be a reusable kind rather than a one-off integration.

## 3. BaseCommunity Reuse

List the inherited BaseCommunity capabilities used without modification:

- membership and Merkle admission;
- messaging;
- red packets;
- referrals;
- claim operators;
- metadata and operational controls;
- other shared interfaces.

## 4. Kind-Specific Behavior

Document every new rule, state variable, event and external function introduced by the leaf.

## 5. Initializer and Storage Compatibility

- **BaseCommunity version or source reference**:
- **Initializer compatibility evidence**:
- **Storage-layout comparison**:
- **Leaf-specific storage gap**:
- **Upgrade/readback test results**:

Explain why the extension does not alter inherited field order, types, packing or initializer semantics.

## 6. Permission and Trust Model

List every privileged role, operator, owner-controlled action and external contract permission. Describe failure behavior and how privileges can be transferred, revoked or paused.

## 7. Implementation and Upgrade Domain

- **Implementation source**:
- **Implementation reference**:
- **UpgradeableBeacon reference**:
- **Beacon owner**:
- **Upgrade policy**:
- **Emergency process**:

## 8. Creation and Discovery

Describe how community instances are created and indexed:

- creator or factory;
- initialization inputs;
- emitted creation events;
- instance lookup or indexing source;
- relationship to `CommunityKindRegistry`.

## 9. Optional Module Integrations

For each red-packet, airdrop, launcher, revenue or other integration, document the interface, required authorization and trust boundary.

## 10. Test and Security Evidence

- unit and integration tests;
- negative-path tests;
- storage and upgrade tests;
- permission tests;
- known limitations;
- audit or review status, without overstating assurance.

## 11. Registration Request

State the requested registry action and confirm:

- [ ] the kind identifier is stable and non-zero;
- [ ] source and specification references are public;
- [ ] beacon ownership is disclosed;
- [ ] creation and discovery paths are defined;
- [ ] storage and initializer compatibility are evidenced;
- [ ] permissions and external integrations are documented;
- [ ] maintainers accept responsibility for future upgrades and disclosures.

## 12. Maintenance Plan

Identify maintainers, release process, upgrade communication, incident response and the process for deprecating or replacing the kind.
