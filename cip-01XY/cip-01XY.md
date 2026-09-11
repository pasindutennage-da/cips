# CIP-01XY

<pre>
Number: CIP-01XY
Title: Traffic-based validator onboarding for making sequencers public on the global synchronizer
Author(s):  
  Pasindu Tennage
  Moritz Kiefer
Type: Standards Track  
Status: Pending
Created: 2026-xx-xx 
Approved: 2026-xx-xx 
License: CC0-1.0
</pre>


## Abstract

This CIP proposes the changes required to make the sequencers public on the global synchronizer.

Previously, validator onboarding relied on two manual processes: (1) static IP whitelisting step by SV operators and (2) the unilateral generation of an onboarding secret by a single sponsor SV.

This CIP replaces both of these dependencies with an automated, decentralized process. By leveraging Canton protocol features, the need for IP whitelists and sponsor-generated secrets is removed. Instead, a validator is automatically granted synchronizer access upon purchasing a minimum amount of traffic, ensuring a baseline financial commitment to the network.

Finally, this CIP includes the steps to realize the necessary transition on the global synchronizer, which includes coordinated and lock-step sequence of tasks.

## Specification

### Onboarding Flow

The existing validator onboarding model relies on a single sponsor SV to unilaterally onboard a new validator by generating an onboarding secret. This CIP replaces the sponsor model with a decentralized flow.

Instead of secrets, new validator onboarding is now driven by traffic purchases. The high-level onboarding flow works as follows:

- A prospective validator operator spins up their validator node to generate their cryptographic keys and obtain a unique participant ID.

- An existing party on the network holding Canton coins purchases traffic for that new participant ID.

- This traffic purchase automatically triggers the permissioning process, allowing the validator to connect to the global synchronizer.

Concretely, when the `MemberTraffic` contract is created with sufficient traffic (as publicly defined in the ledger), SV automation observes this contract and automatically submits a `ParticipantSynchronizerPermission` topology transaction for the validator's participant ID. Once confirmed by a majority of SVs, the validator's connection is accepted.

In the event that SVs detect network abuse by a validator, SVs can collectively decide to offboard the offending validator. To handle validator offboarding, SVs use a new `ValidatorUnpermission` contract to vote on revoking a validator's synchronizer access. `ValidatorUnpermission` contract supports two modes of revocation:

- Temporary: Suspend the validator's access until a specific time by setting the `loginAfter` parameter on the `ParticipantSynchronizerPermission`.
- Permanent: Fully revoke the `ParticipantSynchronizerPermission` topology state.

If a validator is permanently unpermissioned, it can be repermissioned at a later time. SVs must vote to issue a new `ParticipantSynchronizerPermission`.

### Deployment Security

The deployment security details required to expose the sequencer APIs to the public internet are discussed and implemented in CIP-01XZ.

### Network Transition Timeline

- Prerequisites: Canton 3.X and Splice 0.x.x releases.

Making sequencers public requires a coordinated, 5-step transition process by Super Validator operators;

1. Switch to the new onboarding flow: The network enables the new traffic-based onboarding automation, using `MemberTraffic` contracts to issue permissions to validators.
2. Deprecate the old onboarding flow: The legacy secret-based sponsor SV onboarding flow is deprecated and, eventually, disabled.
3. Topology submission: SV operators set the `submitSynchronizerPermission: true` feature flag on the SV application. This triggers a one-time decentralized automation to submit `ParticipantSynchronizerPermission` topology transactions for all existing validators that hold a valid `MemberTraffic` contract, with minimal required traffic, as specified in the ledger.
4. Network switchover: Once the topology submission is complete, SV operators set the `requireRestrictedOpen: true` feature flag. This automatically converts the network to `RestrictedOpen` mode.
5. Drop whitelists: Once the synchronizer is successfully running in `RestrictedOpen` mode, SV operators remove the IP whitelists from their infrastructure.

### Easy Onboarding to DevNet

On DevNet, `/v0/devnet/onboard/validator/purchase-traffic` endpoint on the SV application allows for automated self-onboarding. This endpoint uses test tokens to automatically generate `MemberTraffic` for joining validators. To prevent denial-of-service attacks on this free onboarding mechanism, aggressive IP-based rate limiting is applied to the `/v0/devnet/onboard/validator/purchase-traffic`.

### Rollback

In the event of unexpected issues with the new onboarding mode, rolling back the network back to `UnrestrictedOpen` requires manual coordination among Super Validator operators. The SVs must coordinate to manually switch back to `UnrestrictedOpen` in the `DynamicSynchronizerParameters`.

## Motivation

### A Public Network

A decentralized network must be public and open, and the participation should not require manual, off-ledger gatekeeping like IP whitelisting. This CIP ensures anyone can join the network seamlessly based on protocol-level rules (`MemberTraffic`) rather than manual administrative approval.

### Governance

The existing secret-based onboarding model makes a single sponsor SV unilaterally responsible for admitting a new validator. This CIP ensures that onboarding and offboarding validators is a decentralized process.

### Operational Overhead

The manual generation of onboarding secrets by a sponsor SV is a time-consuming process that does not scale as the network expands. 
Furthermore, maintaining static IP whitelists creates a significant operational bottleneck, severely delaying the speed at which new validators can join the network.
 
## Rationale

- Why Canton 3.X: This version supports the `RestrictedOpen` synchronizer state and the `ParticipantSynchronizerPermission` topology transaction required to make sequencers public.

- Why `MemberTraffic`: To prevent attackers from misusing the global synchronizer, joining the network must have a cost. Since validators already purchase `MemberTraffic` to transact, we reuse this existing financial requirement as the economic barrier to entry rather than inventing a new mechanism.

- Why drop IP whitelists: Because access control is now securely handled by the on-chain logic, network-layer firewalls are no longer necessary to block unauthorized validators. This allows operators to run sequencers on the open internet.

## Backwards Compatibility

- Existing Validator and Scan APIs remain unchanged for applications using them, ensuring no disruption for downstream applications.

- Making sequencers public imposes a hard operational impact on existing validators that do not hold a valid `MemberTraffic` contract with minimal required traffic, at the time of the network switchover. These validators will experience synchronizer downtime until new traffic is purchased for their participant ID.

## Reference Implementation

Development of the code enabling Public Sequencer was funded in part by the Canton Foundation Development Fund, via a grant XXX.

The Splice 0.X.X code can be found at https://github.com/canton-network/splice/release-line-0.X.X

The OSS Canton 3.X code can be found at https://github.com/DACH-NY/canton/tree/release-line-X.X


## Copyright

This CIP is licensed under [CC0-1.0: Creative Commons CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

## Changelog

- 2026-XX-XX: Approved
- 2026-XX-XX: Initial draft v1
