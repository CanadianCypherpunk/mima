# UniChatHooks Contracts

Uniswap v4 Hook integration for the UniChat / MIMA community economy. This module connects group invitation, MIMA/USDT liquidity, swap activity, and revenue sharing.

This folder is the public UniChat-facing module summary and a small integration surface; it does not copy the full Uniswap v4 implementation workspace. In the current community architecture, a Hook-enabled group is a `HOOKS`-kind implementation built on [`BaseCommunity`](../BaseCommunity/README.md).

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Core Flow](#core-flow)
- [Revenue Model](#revenue-model)
- [Contract Reference](#contract-reference)
- [Security Notes](#security-notes)

## Overview

UniChatHooks turns liquidity participation into a community action:

- Members share invitation links through an on-chain share record.
- Invitees bind themselves to an inviter and group before adding liquidity.
- The MIMA/USDT Uniswap v4 pool is registered as the canonical Hook pool.
- LP activity is recorded by the Hook and forwarded to the revenue manager.
- Rewards can be split between the group owner, inviter, and LP.

## Architecture

| Contract | Role |
| --- | --- |
| `HooksCommunity` | `BaseCommunity` specialization for the `HOOKS` kind, including inviter whitelisting and authorized LP auto-join. |
| `CommunityInviteBindingManager` | Creates share IDs and permanently binds an invitee to inviter/group/share. |
| `CommunityPoolRegistry` | Stores the canonical MIMA/USDT Hook pool and maps groups to that pool. |
| `MimaCommunityHook` | Validates registered MIMA pools, records swaps, and forwards LP callbacks. |
| `CommunityHookRevenueManager` | Records LP positions, checks invite binding, and tracks claimable revenue. |
| `CommunityHookLens` | Aggregates group, pool, LP position, and claimable revenue reads. |

```
Community share
    -> Invite binding
    -> MIMA/USDT Hook pool
    -> LP position
    -> Revenue split
    -> Claim
```

## Core Flow

1. A whitelisted member records a share and receives a `shareId`.
2. The invitee calls `bindInvite` for themselves.
3. The owner registers the MIMA Hook and canonical MIMA/USDT pool.
4. The invitee adds liquidity with Hook data: `(groupContract, lp, shareId)`.
5. The Hook validates the registered pool and calls the revenue manager.
6. The revenue manager verifies invite binding, records the LP position, and can add the LP to the group.
7. A revenue operator distributes rewards.
8. Users claim their accumulated rewards by token.

## Revenue Model

Default split:

| Recipient | Share |
| --- | ---: |
| Group owner | 20% |
| Inviter | 30% |
| LP | 50% |

If no inviter exists, the inviter share falls back to the group owner in the source project.

## Contract Reference

`UniChatHooks.sol` is a lightweight integration file for this repository. It avoids importing Uniswap v4 and OpenZeppelin upgradeable dependencies, while documenting the main external surface used by the UniChat app.

Full implementation source in `univ4-hooks`:

- `contracts/hooks/MimaCommunityHook.sol`
- `contracts/pool/CommunityPoolRegistry.sol`
- `contracts/invite/CommunityInviteBindingManager.sol`
- `contracts/revenue/CommunityHookRevenueManager.sol`
- `contracts/lens/CommunityHookLens.sol`

## Security Notes

- MIMA trades should use the Hook pool path, not a generic 1inch route.
- Mainnet swaps must use a non-zero `amountOutMinimum`.
- LP actions should only be enabled after invite binding exists.
- Hook data must be encoded consistently as `(groupContract, lp, shareId)`.
- Critical owners and operators should be controlled by a multisig or similarly protected account.
