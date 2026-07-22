<div align="right">
  <a href="./DEVELOPER_GUIDE.md"><strong>English</strong></a> ·
  <a href="./DEVELOPER_GUIDE.zh-CN.md">简体中文</a>
</div>

# Build Your Own Community Kind

BaseCommunity is an open extension model for teams that want to create a distinct type of on-chain community without rebuilding membership, Merkle admission, messaging, red packets, referral records and authorized auto-join from the ground up.

The ecosystem follows one simple rule:

> **Build openly. Register through governance.**

Any developer or organization can design and implement a community kind. Canonical registration is governed so that shared identifiers, storage compatibility, permissions and upgrade ownership remain reviewable across the ecosystem.

## What the Platform Provides

Every compatible kind can reuse the BaseCommunity protocol surface:

- epoch-based Merkle admission and tiered membership;
- invited and claim-operator membership flows;
- text, media and encrypted-message metadata;
- group red-packet integration;
- non-blocking referral relationships;
- community metadata and operational controls;
- a stable `communityKind()` identity;
- isolation through a kind-specific upgradeable beacon;
- canonical discovery through `CommunityKindRegistry`.

## What Kind Developers Own

Kind developers define the behavior that makes their community model unique. Examples may include gaming guild rules, creator memberships, DAO coordination, research groups, commerce communities or other domain-specific systems.

The developer remains responsible for:

- kind-specific storage and business rules;
- permission and trust boundaries;
- the implementation and beacon proposal;
- creation and instance-indexing strategy;
- external module wiring;
- tests, security analysis, documentation and future upgrades.

## Extension Lifecycle

### 1. Define the Kind

Choose a stable, non-zero `bytes32` identifier and document the behavior that distinguishes the kind. A kind should represent a durable community model rather than a single campaign or deployment.

### 2. Implement a Thin Leaf

Inherit `BaseCommunity`, reuse its initializer and override `communityKind()`. Keep shared capabilities in the base layer and add only behavior that belongs to the new kind.

Simplified shape:

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

    // Append reviewed kind-specific storage after the complete base layout.
}
```

This sketch shows the extension boundary; production code must use the current reviewed BaseCommunity version and storage rules.

### 3. Prove Compatibility

A candidate kind should demonstrate:

- the expected initializer signature and initialization safety;
- a stable, unique `communityKind()` value;
- append-only kind storage after the complete base layout;
- a reviewed leaf-specific storage gap;
- preserved BaseCommunity interfaces and invariants;
- explicit authorization for every external module;
- negative-path and upgrade/readback tests.

Changing inherited field order, type, packing or initializer semantics is not a compatible extension.

### 4. Prepare the Upgrade Domain

Deploy the leaf implementation behind a dedicated `UpgradeableBeacon`. The proposed beacon owner and upgrade process must be declared so users and integrators can understand who controls future implementation changes.

### 5. Apply for Canonical Registration

Submit the kind for governance review with:

- kind name and `bytes32` identifier;
- public specification and source reference;
- implementation and beacon information;
- beacon ownership and upgrade policy;
- storage-layout comparison;
- test and security evidence;
- permission model;
- creator and discovery design;
- integration and maintenance contacts.

After approval, the registry owner can call `registerKind(kind, beacon)`. Registration makes the kind discoverable in the canonical directory; it does not deploy community instances or transfer beacon ownership.

### 6. Create and Index Instances

Each kind needs an explicit creation path. A developer may provide a dedicated factory or another reviewed creator that initializes `BeaconProxy` instances from the kind's beacon.

The creator should also define how instances are indexed. `CommunityKindRegistry` catalogs implementation domains, not every community instance.

### 7. Connect Optional Modules

Red packets, airdrops, token launchers, revenue systems and other modules use explicit interfaces and permissions. A new kind inherits the shared capability surface, but no external module is automatically trusted or enabled.

## Governance Boundary

The ecosystem is open to external development but uses governed canonical registration:

- developers do not need permission to design or implement a kind;
- independent implementations can evolve outside the canonical directory;
- canonical platform recognition requires review and registry approval;
- registry activation is not a runtime kill switch for creators that hold direct beacon references;
- registry updates do not upgrade existing proxies;
- the owner of each existing beacon controls upgrades for proxies attached to that beacon.

This boundary keeps experimentation open while making official interoperability and upgrade authority explicit.

## Developer Readiness Checklist

- [ ] The kind has a stable name, identifier and public specification.
- [ ] The leaf inherits the reviewed BaseCommunity version.
- [ ] The initializer and storage layout are compatible.
- [ ] Kind-specific permissions are documented and tested.
- [ ] The implementation and dedicated beacon are reviewable.
- [ ] Beacon ownership and upgrade policy are disclosed.
- [ ] A creator and instance-discovery path are defined.
- [ ] Optional module integrations use explicit authorization.
- [ ] Upgrade, negative-path and readback tests pass.
- [ ] Governance registration materials are complete.

## Start Building

1. Read the [BaseCommunity architecture](./README.md).
2. Write a short specification for the new kind.
3. Implement the smallest compatible leaf.
4. Produce storage, test, permission and upgrade evidence.
5. Complete the [Community Kind Proposal Template](./KIND_PROPOSAL_TEMPLATE.md) and submit the kind for canonical review.

The goal is not to force every community into one model. It is to give every new model a common, composable foundation.
