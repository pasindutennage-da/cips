# CIP-01XY

<pre>
Number: CIP-01XY
Title: Traffic-based validator onboarding for making sequencers public on the Global Synchronizer
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

This CIP proposes the changes required to make the sequencers public on the Global Synchronizer, allowing anyone on the internet to talk to them.

Previously, we used IP whitelisting to allow validators to connect to the global synchronizer. This CIP removes the need for IP whitelisting, and instead uses the Canton protocol to ensure that only validators with correct authorization can connect to the synchronizer.

To achieve this, we use existing `MemberTraffic` contracts to enable validators to connect to the sequencers. Therefore, only validators that have purchased sufficient traffic will receive the necessary authorization to join the global synchronizer, providing built-in Sybil resistance.

Furthermore, in the previous validator onboarding flow, a validator must ask a sponsor SV for a secret, making onboarding dependent on a single SV. We replace this model so that new validator onboarding and offboarding are now approved by a majority (2f+1) of SVs.

Finally, this CIP includes the steps to realize this transition on the Global Synchronizer, which includes coordinated, lock-step manual processes.

## Specification

### Protocol

The existing validator onboarding model relies on a single sponsor SV to unilaterally onboard a new validator by generating an onboarding secret. This CIP replaces the sponsor model with a decentralized flow, where granting synchronizer access requires on-chain consensus from a 2f+1 majority of SVs.

Instead of secrets, permissioning is now driven by traffic purchases. When a `MemberTraffic` contract is created for a validator, with sufficient traffic, as publicly defined in the ledger, SV automation observes the MemberTraffic contract and automatically submits a `ParticipantSynchronizerPermission` topology transaction for the validator's participant ID. Once confirmed by a majority of SVs, the validator is permitted to connect to the synchronizer.

To handle offboarding, SVs use a new `ValidatorUnpermission` contract to vote on revoking a validator's synchronizer access. `ValidatorUnpermission` contract supports two modes of revocation:

* Temporary: Suspends the validator's access until a specific time by setting the `loginAfter` parameter on the `ParticipantSynchronizerPermission`.
* Permanent: Fully revokes the `ParticipantSynchronizerPermission` topology state.

If a validator is permanently unpermissioned, it can be repermissioned at a later time. SVs must vote to issue a new `ParticipantSynchronizerPermission`.


### Deployment Security

- The deployment security details required to expose the sequencer APIs to the public internet are discussed and implemented in CIP-01XZ.

### Network Transition Timeline

- Prerequisites: Canton 3.X and Splice 0.x.x releases.

Making global synchronizer sequencers public requires a coordinated, 6-step transition process by Super Validator operators;

1. Switch to the new onboarding flow: The network enables the new traffic-based onboarding automation, using `MemberTraffic` contracts to issue permissions to validators.
2. Deprecate the old onboarding flow: The legacy secret-based sponsor SV onboarding flow is officially deprecated and disabled.
3. Topology Submission: SV operators set the `submitSynchronizerPermission: true` feature flag on the SV application. This triggers automation to automatically submit `ParticipantSynchronizerPermission` topology transactions for all existing validators that hold a valid `MemberTraffic` contract, with minimal required traffic, as specified in the ledger.
4. Network Switchover: Once the topology submission is complete, SV operators set the `requireSynchronizerPublic: true` feature flag. This automatically converts the network  `RestrictedOpen` mode.
5. Handle unpermissioned nodes: Any existing validators who didn't have a `MemberTraffic` contract, with enough traffic, at the time of Phase 2 will lose synchronizer access. To regain access, they must purchase `MemberTraffic` via standard channels, which will trigger the automated permissioning flow.
6. Drop whitelists: Once the synchronizer is successfully running in `RestrictedOpen` mode, SV operators remove the IP whitelists from their infrastructure.

### DevNet Exception

On DevNet, `/v0/devnet/onboard/validator/purchase-traffic` endpoint on the SV application allows for automated self-onboarding. This endpoint uses test tokens to automatically generate `MemberTraffic` for joining validators. To prevent denial-of-service attacks of this free onboarding mechanism, aggressive IP-based rate limiting is applied to the `/v0/devnet/onboard/validator/purchase-traffic`.

### Rollback

- In the event of an emergency, rolling back the network back to `UnrestrictedOpen` requires manual, off-chain coordination among Super Validator operators. The SVs must coordinate to manually switch the Canton `DynamicSynchronizerParameters` back to `UnrestrictedOpen`.


## Motivation

### Governance

- Onboarding and offboarding validators must be a decentralized network decision, not a unilateral one by a single sponsor.

### Onboarding Speed

- Manual secret generation by sponser SV doesn't scale as network grows.
- IP whitelisting is a manual bottleneck, and doesn't scale when network grows
 
## Rationale

- Why Canton 3.X: It natively supports the RestrictedOpen synchronizer state and ParticipantSynchronizerPermission topologies.

- Why MemberTraffic: It leverages an existing network mechanic for Sybil resistance

- Why drop IP whitelists: Because access control is now handled by the on-chain logic, allowing for true public sequencers.

## Backwards Compatibility

- APIs: Existing Validator and Scan APIs remain unchanged for applications.

- Operational impact: Hard impact on existing validators if they do not hold MemberTraffic at the time of the network switchover, they will experience downtime until someone purchase traffic for them.

## Reference Implementation

Development of the code enabling Public Synchronizer was funded in part by the Canton Foundation Development Fund, via a grant XXX  for Public Synchronizer.

The Splice 0.X.X code can be found at https://github.com/canton-network/splice/release-line-0.X.X

The OSS Canton 3.X code can be found at https://github.com/DACH-NY/canton/tree/release-line-X.X


## Copyright

This CIP is licensed under [CC0-1.0: Creative Commons CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/).

## Changelog

[//]: # (- 2026-XX-XX: Approved)
- 2026-XX-XX: Initial draft v1