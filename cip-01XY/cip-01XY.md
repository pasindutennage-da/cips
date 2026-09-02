# CIP-01XY

<pre>
Number: CIP-01XY
Title: Traffic-based validator onboarding for making sequencers public on the Global Synchronizer
Author(s):  
  Pasindu Tennage
  Moritz Kiefer
Type: Standards Track  
Status: Pending
Created: 2026-08-26 
Approved: 2026-xx-xx 
License: CC0-1.0
</pre>


## Abstract

- What does "Public Synchronizer" mean for the Canton Network.

- The transition from IP-whitelist-based access control to traffic-based access.

- Sybil Resistance; explain what sybil resistence mean; buying minimum MemberTraffic replaces IP whitelisting as the Sybil resistance mechanism.

- Validator Onboarding and offboarding now require 2f+1 majority SV consensus, removing the unilateral "sponsor SV" model.

- The proposal includes a one-time manual network switchover to support the enforcement of traffic-based onboarding.


## Specification

### Architecture

- Brief comparison of existing unilateral SV sponsor model with the new decentralized 2f+1 SV on-chain voting model.

- How MemberTraffic triggers the automated issuance of ParticipantSynchronizerPermission.

- Two types of revocation (temporary vs. permanent) utilizing the ValidatorUnpermission contract.

- Repermissioning validators once permemently unPublic 


### Deployment Security

- Deployment security details are discussed and implemented in CIP 01XZ.

### Network Transition Timeline

- Prerequisites: Canton 3.X and Splice 0.x.x releases.

- Switching to new onboarding flow
- Deprecating the old onboarding flow
- Phase 1 (Topology Submission): SVs use a feature flag (submitSynchronizerPermission: true) to automatically submit permissions for all existing validators holding MemberTraffic.
- Phase 2 (Switchover): SVs set a second flag (requireSynchronizerPublic: true) to change the Canton DynamicSynchronizerParameters to RestrictedOpen.
- Handling UnPublic Nodes: existing validators without MemberTraffic will lose access and must purchase MemberTraffic via standard channels to trigger automatic permissioning.
- Drop whitelists

### DevNet Exception

- specific DevNet endpoint that allows automated self-onboarding
- use of aggressive IP-based rate limiting applied to it.

### Rollback

- manual, off-chain SV coordination required to switch the network back to UnrestrictedOpen in case of an emergency.


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