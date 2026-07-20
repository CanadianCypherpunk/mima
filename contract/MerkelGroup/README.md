<div align="right">
  <a href="./README.md"><strong>English</strong></a> ·
  <a href="./README.zh-CN.md">简体中文</a>
</div>

# BaseCommunity and the Extensible Community Platform

> The `MerkelGroup` directory name is retained for link compatibility. The current protocol model is the BaseCommunity architecture described here, not the earlier clone-and-room design.

> Here, “Base” means the abstract `BaseCommunity` contract layer. It does not refer to the Base blockchain network.

## Status and Scope

UniChat communities use a layered contract architecture:

- `BaseCommunity` defines the capabilities shared by every community type.
- Thin leaf contracts identify a business-specific community kind.
- A kind can use its own upgradeable beacon so its implementation can evolve independently.
- `CommunityKindRegistry` records the canonical `kind → beacon` directory for governance and discovery.

This document describes the contract architecture and its extension model. It intentionally does not track chain-by-chain deployment status or application-layer support.

## Architecture Overview

```text
                          BaseCommunity (abstract)
              membership · messages · red packets · referrals
                              claim operators
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                    │
                 ▼                    ▼                    ▼
       OfficialCommunity     LauncherCommunity      HooksCommunity
           OFFICIAL               LAUNCHER               HOOKS
                 │                    │                    │
          official beacon      launcher beacon       hooks beacon
                 └────────────────────┼────────────────────┘
                                      ▼
                        CommunityKindRegistry
                    canonical kind → beacon directory
```

Each community instance is a `BeaconProxy`. An existing instance remains attached to the beacon selected when it was created. Updating a directory entry does not migrate that instance or upgrade its beacon.

## Contract Roles

| Component | Role |
| --- | --- |
| `BaseCommunity` | Abstract storage-compatible base for membership, Merkle admission, messaging, red packets, referral records, metadata, pause controls and cross-module interfaces. |
| `OfficialCommunity` | Standard platform community. It adds no business storage and reports `communityKind() == "OFFICIAL"`. |
| `LauncherCommunity` | Community created for a launched token. It reports `"LAUNCHER"` and keeps token/launcher discovery in `TokenLauncherFactory`. |
| `HooksCommunity` | Liquidity-community leaf. It reports `"HOOKS"` and adds an owner-managed inviter whitelist used by the Hook revenue system. |
| `CommunityFactory` | Creates and indexes official communities through the official beacon. |
| `CommunityKindRegistry` | Owner-governed directory of registered kinds, their beacon addresses and active directory status. It does not create communities, hold community funds or upgrade beacon implementations. |
| `CommunityKeyRegistry` | Separate registry for group public keys and encrypted session-key distribution. It is not the kind registry. |

## BaseCommunity Capabilities

### Membership and Merkle Admission

- Epoch-based Merkle roots refresh the active membership set.
- A proof is bound to the community address, epoch, account, tier, expiry and nonce.
- Nonces prevent proof replay.
- Tiers support community-specific access levels.
- Owners can invite members directly.
- Authorized claim operators can call `claimJoin` for token-transfer and airdrop-driven auto-join flows.

The Merkle root in `BaseCommunity` proves admission eligibility. Token distribution uses the separate `AirdropClaim` module; the community does not mint or distribute claim assets itself.

### Messaging

- Text, voice, image, video, file and custom message kinds.
- Plaintext or encrypted-message flags.
- Configurable content limits and plaintext policy.
- Sequence numbers, content hashes and optional CIDs for indexing and media retrieval.
- Optional message fees and fee exemptions.

### Red Packets

An active member can create a group red packet and broadcast its reference as a community message. Operation requires both sides of the integration to be configured:

1. The community points to the red-packet contract.
2. The red-packet contract authorizes the community as a chat contract.

### Referral Graph P0

The base layer records one immutable inviter for an invitee and exposes paginated reverse lookup. This is a relationship graph only:

- self-referral is ignored;
- an inviter must already be a member;
- an invitee's referrer cannot be replaced;
- invalid referral input never blocks an otherwise valid join;
- no referral reward or automatic revenue distribution is implied.

### Governance and Metadata

Each community exposes its owner, topic token, maximum tier, name, avatar CID and current epoch. Owners can update operational settings, authorize claim operators, bind the red-packet module, and pause or resume guarded writes.

## Community Kinds

### OFFICIAL

`OfficialCommunity` is the compatibility path for existing standard communities. Its business storage is byte-for-byte aligned with the historical flat community implementation, allowing an official beacon to adopt the layered architecture without rebuilding community proxies.

### LAUNCHER

`LauncherCommunity` reuses the base capabilities while isolating future launcher-specific evolution behind a launcher beacon. Launch records and token-to-community mappings remain in `TokenLauncherFactory`; they are not duplicated in community storage.

If no launcher beacon is configured, a launcher may fall back to the official community beacon. That fallback does not provide an independently upgradeable LAUNCHER kind.

### HOOKS

`HooksCommunity` adds an inviter whitelist for the Uniswap v4 community-economy integration. The external revenue system can be authorized as a claim operator to admit LPs, while the group owner and approved inviters participate in the configured revenue model.

### Future Kinds

A new kind can inherit `BaseCommunity`, add only its specific business behavior, use a dedicated beacon, and be registered in the canonical directory. Registration is governed; it is not currently a permissionless deployment guarantee.

## Factory, Beacon and Registry Model

### Creation

- `CommunityFactory` creates OFFICIAL proxies and maintains the official-community indexes.
- `TokenLauncherFactory` creates launcher communities and maintains its own launch indexes.
- The Hooks deployment flow creates and wires HOOKS communities for registered pool integrations.

### Upgrade Isolation

Upgrading a kind means upgrading the implementation referenced by that kind's existing beacon. Replacing a `CommunityKindRegistry` entry only changes the directory record; it does not upgrade proxies attached to the previous beacon.

### Directory Semantics

`CommunityKindRegistry` supports registering a kind, changing its recorded beacon, and marking the directory entry active or inactive. Current creators use direct beacon references, so the registry is a canonical governance and discovery record rather than a universal runtime router or automatic circuit breaker.

## Discovery Rules

There is no single on-chain list of every community instance.

| Community source | Canonical discovery path |
| --- | --- |
| Official communities | `CommunityFactory` lists and `(topicToken, maxTier)` indexes |
| Launcher communities | `TokenLauncherFactory` events, token lists, launch records and token/community mappings |
| Hooks communities | Hook/pool integration records and deployment indexes |
| Community kinds | `communityKind()` on compatible instances and `CommunityKindRegistry` for registered kind beacons |

Consumers that need a complete instance view must combine these sources explicitly. The kind registry catalogs implementation domains; it does not index every community proxy.

## Group Public Keys

Group public keys and encrypted session-key distributions live in `CommunityKeyRegistry`, outside `BaseCommunity`. The registry supports:

- setting or rotating a group's public key;
- revoking an active key and advancing its key epoch;
- publishing Merkle roots and content-addressed locations for encrypted session-key packages;
- compatibility aliases for older RSA-oriented integrations while the current design uses algorithm-neutral, ML-KEM-oriented terminology.

## Cross-Module Interfaces

The base layer exposes stable, minimal interfaces for other modules:

- Red packets read `isActiveMember` and create group-bound distributions.
- Airdrop and token flows use authorized `claimJoin` calls for auto-join.
- Token launching uses a dedicated launcher beacon and separate launch discovery.
- Hook revenue modules read community ownership and inviter eligibility, and can auto-admit LPs when authorized.

These integrations require explicit wiring and permissions. Inheriting `BaseCommunity` does not automatically authorize an external module.

## Upgrade and Storage Safety

The compatibility model follows four rules:

1. Existing business fields keep their order, type and packing boundaries.
2. Shared fields are appended by consuming reserved base storage gap slots.
3. Kind-specific fields are appended after the complete base layout and use a leaf-specific gap.
4. Official upgrades are checked against the historical layout before the beacon implementation changes.

Repository validation includes static layout comparison, a layout golden snapshot, runtime upgrade/readback tests, kind-specific integration tests and smoke checks. These controls reduce upgrade risk but are not a claim of zero risk or an external audit certification.

## Legacy Snapshot Notice

The `Community.sol`, `CommunityFactory.sol` and `Room.sol` files retained in this directory are historical documentation snapshots of the earlier EIP-1167 clone-and-room design. They do not represent the current community architecture and must not be used as the integration source of truth.

The current model removes Room from the community core, uses BeaconProxy instances, places key distribution in a separate registry, and separates community types into independently evolvable kind domains.

## Security Notes

- Validate the exact kind and beacon before creating or upgrading a community.
- Treat registry ownership, beacon ownership and factory ownership as distinct control planes.
- Never assume a directory update changes existing proxies.
- Keep Merkle proofs bound to the community and epoch, and preserve nonce replay protection.
- Keep referral recording non-blocking for auto-join safety.
- Require explicit authorization for red-packet, airdrop, launcher and Hook integrations.

---

**Architecture generation**: BaseCommunity + kind-specific BeaconProxy domains

**Solidity line**: `^0.8.24` with OpenZeppelin Contracts v5.x

**Document status**: public contract-architecture reference
