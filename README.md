# HELM / ViMach Proposal v2.1
## Deterministic Premium Live Media Platform
### Media-Plane-Centered Architecture with Independent Control Plane

**Document type:** full proposal  
**Version:** v2.1  
**Language:** English  
**Audience:** internal team, executive stakeholders, investors, strategic partners  
**Status:** updated working proposal

---

# 1. Executive Summary

HELM / ViMach should be positioned as a **deterministic premium live media platform** built around a repeatable **deterministic media-core node** architecture.

The defining principle of the platform is simple:

> **Keep the media plane deterministic, keep the control plane independent, and scale the platform by repeating a clean node pattern across regions.**

The proposed architecture is centered on:

- **software protocol boundaries at ingest**
- **narrow, disciplined rendition creation**
- **FPGA-based deterministic media processing**
- **optical media-plane transport**
- **FPGA-assisted egress ingestion**
- **CPU-based delivery semantics**
- **a software control plane completely outside the FPGA fast path**

The first proof case is intentionally concrete:

- **4K ingest**
- **two profiles**
  - original
  - 2K fallback
- **100,000-user planning scenario**

This case study is not the limit of the platform.  
It is the first proof point used to demonstrate that the architecture is sound.

The architecture is intentionally designed so that it can evolve to:
- newer codecs
- newer live profiles
- higher resolutions
- additional renditions
- multi-region footprints
- larger audience classes

That means the platform should not be understood as “a 4K-only system.”  
It should be understood as:

> **a deterministic media platform whose first benchmark case is 4K + 100K, but whose architecture remains open to future profiles, including 8K-class live delivery when the business case and system budget justify it.**

---

# 2. Strategic Positioning

## 2.1 What HELM / ViMach is building
HELM / ViMach is building a **premium live media platform** for sports and entertainment whose competitive advantage comes from:

- deterministic media handling
- strict separation of media plane and control plane
- bounded internal queueing and timing behavior
- package-once / fanout-late discipline
- practical large-audience delivery using software edge semantics
- a staged hardware moat

## 2.2 Core positioning statement
The platform is not defined by a single codec, resolution, or current profile set.  
It is defined by a **deterministic transport and delivery architecture** that can support current and future premium live profiles.

## 2.3 Why this matters
Premium live delivery problems are not only bandwidth problems.  
They are also:

- timing problems
- fanout problems
- repeated media-work problems
- software-stack interference problems
- delivery-boundary problems

ViMach addresses those problems by enforcing a clear architectural split:

- **Media plane:** deterministic, hardware-assisted, optical, low-jitter
- **Control plane:** software-defined, observable, extensible, operationally rich

---

# 3. Architectural Principles

The platform should be built around the following principles.

## 3.1 Deterministic media plane
The media plane should remain:
- short
- bounded
- measurable
- low-jitter
- isolated from generic application traffic

## 3.2 Independent control plane
The control plane should remain:
- software-driven
- operationally rich
- visible and debuggable
- capable of policy, rollout, routing, observability, and failover
- completely outside the deterministic fast path

## 3.3 Package once, fan out late
Media should be transformed into stable packaged objects once, then replicated as late as possible.

## 3.4 Hardware where timing matters, software where semantics matter
- hardware owns timing, packaging, pacing, and object fanout
- software owns manifests, HTTP/TLS, session state, auth, analytics, and operations

## 3.5 Profile-agnostic architecture
The architecture should support current live profiles first, but remain open to:
- higher resolution
- updated video profiles
- additional codecs
- future premium live workflows

This proposal uses a 4K + 2-profile case to prove the architecture, not to define its permanent ceiling.

---

# 4. Product Thesis

The product thesis is:

> **Use software where protocol and business logic belong, use FPGA and optical transport where deterministic media timing and fanout matter most, and preserve an architecture that can grow from current premium live profiles to future profiles, including 8K-class deployments when justified.**

That leads to five commitments:

## 4.1 Software protocol boundary
Protocol-heavy logic remains at the software boundary:
- WHIP
- RTMP
- session entry
- API
- auth / entitlement
- billing / reporting
- dashboards

## 4.2 Narrow initial rendition strategy
Phase 1.5 uses:
- original profile
- one fallback profile

This keeps the proof architecture disciplined.

## 4.3 Deterministic media core
The FPGA media core owns:
- timing alignment
- package once
- pacing / shaping
- late fanout preparation

## 4.4 Dedicated optical media plane into egress hosts
Media remains on a dedicated optical path from Zone 3 into FPGA cards on Zone 4 hosts, then DMA enters host memory.

## 4.5 CPU-only delivery semantics
Zone 4 CPUs do not rebuild media.  
They only complete delivery semantics:
- playlists
- HTTP/TLS
- cache
- session binding

---

# 5. High-Level Reference Architecture

## 5.1 End-to-end data flow

```text
[PUBLISHER / VENUE]
    Camera / encoder / contribution feed
    Output: WHIP or RTMP
                |
                v
====================================================================
ZONE 1 — INGEST BOUNDARY  (software boundary)
====================================================================
    - Receive WHIP / RTMP
    - Terminate protocol
    - Demux / normalize
    - Extract compressed video/audio
    - Produce normalized compressed media stream
                |
                v
====================================================================
ZONE 2 — RENDITION CREATION
====================================================================
    Input:
      - compressed source profile

    Processing:
      - Keep P0 = original profile
      - GPU creates P1 = fallback profile

    Output:
      - P0 compressed
      - P1 compressed
                |
                |  PCIe DMA / local high-speed handoff preferred
                v
====================================================================
ZONE 3 — DETERMINISTIC MEDIA CORE (FPGA)
====================================================================
    Input:
      - P0 compressed
      - P1 compressed

    FPGA pipeline:
      1. Ingress framing
      2. Timing alignment
      3. Package once
      4. Pace / shape
      5. Late fanout preparation

    Output:
      - HLS-ready media objects
      - preferably CMAF / fMP4 chunks
      - aligned to part / segment boundaries
      - tagged with stream / rendition / sequence / timing metadata
                |
                | publish media objects to internal multicast groups
                v
====================================================================
INTERNAL OPTICAL MEDIA FABRIC
====================================================================
    Physical / link:
      - Ethernet over optics
      - 100G / 200G / 400G class as needed

    Distribution:
      - internal multicast distribution for one-to-many fanout
                |
                v
====================================================================
ZONE 4 — EGRESS HOSTS WITH FPGA + CPU
====================================================================

    [FPGA on each egress host]
      - Subscribe to media groups
      - Receive HLS-ready media objects directly from optical fabric
      - Validate sequence / boundary / timing metadata
      - DMA objects into host memory

    [CPU on each egress host]
      - Build / update playlists
      - Publish parts / segments over HTTP/TLS
      - Handle cache and session binding
      - Handle delivery semantics only

                |
                v
====================================================================
END USER DELIVERY
====================================================================
    Protocol:
      - LL-HLS / HLS over HTTP/TLS

                |
                v
[END USERS]
```

## 5.2 Core design intent
This architecture makes the split explicit:

### Media plane
- Zone 1 normalized compressed media
- Zone 2 profile outputs
- Zone 3 deterministic packaging and fanout
- optical fabric
- Zone 4 FPGA receive and DMA

### Control / application plane
- manifest control
- session direction
- auth / entitlement
- rollout
- observability
- HTTP/TLS semantics
- QoE and support

This split is fundamental to the ViMach value proposition.

---

# 6. Zone Architecture

# 6.1 Zone 1 — Ingest Boundary

## Responsibilities
- terminate WHIP / RTMP
- demux and normalize
- extract compressed video/audio
- hand off normalized compressed media

## Design rule
Zone 1 is where protocol complexity ends.

## Notes on profile support
Zone 1 should not be locked to one permanent profile set.  
It should normalize compressed media into a form suitable for downstream deterministic processing, regardless of whether the source is:
- 1080p
- 4K
- future 8K
- newer codecs or updated live profiles

---

# 6.2 Zone 2 — Rendition Creation

## Responsibilities
- preserve the original profile as P0
- create one fallback profile as P1
- output time-compatible compressed renditions

## Initial proof case
For the first case study:
- P0 = original 4K
- P1 = 2K fallback

## Strategic rule
This is the first proof shape, not the permanent limit of the platform.

The architecture should remain open to:
- more modern profiles
- later 8K-class original feeds
- different fallback ladders
- additional codec support

The proposal should therefore describe Zone 2 as **profile-policy-driven**, not hardcoded to one resolution forever.

---

# 6.3 Zone 3 — Deterministic Media Core

## Responsibilities
- ingress framing
- timing alignment
- package once
- pace / shape
- late fanout preparation

## Why this zone matters
Zone 3 is the technical moat of the platform.

It is where ViMach enforces:
- deterministic timing
- bounded queues
- stable package boundaries
- repeatable one-to-many media distribution

## Output semantics
Zone 3 should not output browser-facing HLS semantics directly.  
Instead, it should output:

- HLS-ready media objects
- preferably CMAF / fMP4 chunk objects
- part-aligned or segment-aligned payload units
- enriched with stream/rendition/timing metadata

These are the right media-plane units for clean handoff to Zone 4.

---

# 6.4 Internal Optical Media Fabric

## Responsibilities
- carry HLS-ready media objects from Zone 3 to many Zone 4 hosts
- keep media on a dedicated plane
- support one-to-many fanout cleanly

## Recommended model
- optical Ethernet at physical/link layer
- multicast distribution inside the fabric for one-to-many fanout

## Why this matters
This lets the architecture:
- preserve standard transport/fabric practicality
- avoid building a completely custom physical network
- still keep media off the generic host NIC path
- scale cleanly to many egress hosts and later to multiple regions

---

# 6.5 Zone 4 — FPGA-Assisted Egress Hosts

## Responsibilities of Zone 4 FPGA
- receive media objects directly from the optical media fabric
- validate sequence and timing boundaries
- place objects into deterministic DMA buffers
- DMA into host memory

## Responsibilities of Zone 4 CPU
- generate/update playlists
- publish parts and segments over HTTP/TLS
- cache/session binding
- user-facing delivery semantics

## Critical design rule
Zone 4 CPUs do not rebuild or transform media.  
They only complete delivery semantics.

This is one of the most important architectural claims in the proposal.

---

# 6.6 Zone 5 — Control / Observability

## Responsibilities
- configuration
- routing policy
- manifest policy
- failover policy
- observability
- QoE
- deployment / release control
- support systems

## Design rule
The control plane must be rich, but it must never become part of the hot media path.

This independence should be visible throughout the proposal.

---

# 7. DETERMINISTIC System Behavior

The proposal should use the word **DETERMINISTIC** only where it can be defended.  
The following statements are strong and supportable:

## 7.1 Deterministic inside the platform
Inside the platform, determinism means:
- bounded processing stages
- bounded queue depth targets
- fixed package and fanout behavior
- explicit timing alignment
- controlled media-plane routing
- media-plane independence from general application traffic

## 7.2 Not absolute determinism across the public Internet
The platform cannot claim absolute deterministic user latency across unmanaged Internet, ISP, and device conditions.

The right framing is:

> **ViMach delivers deterministic internal media handling and bounded platform-side latency behavior.**

## 7.3 Why this matters commercially
This still creates real product value:
- better timing predictability
- better premium-event quality
- cleaner failover reasoning
- lower media-path variability
- stronger operational visibility

---

# 8. Media Plane and Control Plane Independence

This proposal should explicitly make media-plane and control-plane independence one of the core product values.

## 8.1 Media plane
Carries:
- compressed media
- packaged media objects
- timed feeds
- deterministic payload movement

Path:
- ingest normalized media
- profile outputs
- FPGA media core
- optical fabric
- FPGA receive on egress hosts
- DMA into host memory

## 8.2 Control plane
Carries:
- auth
- manifests policy
- routing state
- failover state
- telemetry
- deployment
- dashboards
- support workflows

## 8.3 Why the split is strategically important
This split means:
- control-plane richness can grow without polluting the fast path
- media-plane determinism can remain protected
- software operations maturity and hardware determinism can coexist
- multi-region rollout becomes easier to reason about

---

# 9. Support for Current and Future Live Profiles

## 9.1 Proposal language
The proposal should clearly state:

> **ViMach supports the latest practical live-streaming profile classes through an architecture that is profile-agnostic at the deterministic media-core layer.**

## 9.2 What that means in practice
The media core should not be defined as “4K hardware.”  
It should be defined as:

- compressed-profile aware
- timing aware
- packaging aware
- transport-aware
- fanout-aware

## 9.3 Why the 4K + 2-profile case is still useful
The 4K + 100K case is the right first proof because it is:
- technically meaningful
- commercially relevant
- difficult enough to validate the architecture
- easy to explain to investors and internal teams

## 9.4 What happens later
If the business case requires:
- 8K ingest
- new live premium profiles
- updated codecs
- broader ladders

the architecture still holds.

The changes would primarily affect:
- Zone 1 ingest support
- Zone 2 rendition policy
- capacity planning
- hardware density requirements
- egress sizing

The core ViMach design principle does not change.

## 9.5 Explicit answer to the 8K question
Yes, the architecture can be extended toward 8K-class live workflows.

That does **not** mean “8K is free.”  
It means:
- the architecture does not structurally block it
- bandwidth, compute, media-object size, and facility envelopes would scale
- but the deterministic media-plane/control-plane split remains valid

That is exactly why the proposal should describe 4K/100K as a **proof case**, not a permanent limit.

---

# 10. Network Topology and Multi-Region Scale

## 10.1 Single-node view
Each node contains:
- ingest
- rendition creation
- deterministic media core
- optical media fabric
- FPGA-assisted egress
- control / observability

## 10.2 Multi-region scaling principle
The platform should scale by repeating this node pattern into multiple regions.

### Regional model
```text
Region A
  -> deterministic media-core nodes
  -> local egress clusters
  -> local control-plane services or region-aware control agents

Region B
  -> deterministic media-core nodes
  -> local egress clusters

Region C
  -> deterministic media-core nodes
  -> local egress clusters
```

## 10.3 What changes across regions
As the system scales to multiple regions:
- regional ingress and failover policies expand
- control-plane routing and placement logic expands
- audience steering expands
- regional edge and peering strategies expand

## 10.4 What does not change
The node architecture remains the same:
- software ingest boundary
- disciplined profile generation
- deterministic FPGA media core
- optical media-plane fanout
- FPGA-assisted egress ingest
- CPU delivery semantics
- independent control plane

This repeatability is the architectural basis for multi-region scale.

---

# 11. Network Topology Reference

## 11.1 Logical topology

```text
Publishers / Venues
    -> Zone 1 Ingest Boundary
    -> Zone 2 Rendition Creation
    -> Zone 3 Deterministic Media Core
    -> Optical Media Fabric
    -> Zone 4 Egress Hosts
    -> End Users

Zone 5 Control / Observability
    -> connected logically to all zones
    -> never in the hot media path
```

## 11.2 Key transport choices
- WHIP / RTMP at ingest boundary
- local PCIe or local handoff inside node
- optical Ethernet fabric for inter-zone media transport
- multicast object fanout for one-to-many zone-3-to-zone-4 distribution
- LL-HLS / HTTP/TLS to end users

## 11.3 Why this topology is valuable
It is:
- deterministic where it matters
- practical to build
- scalable across racks
- scalable across regions
- compatible with current delivery ecosystems

---

# 12. Compatibility with Existing Live Streaming Systems

## 12.1 Why compatibility matters
The platform must be adoptable without forcing customers to rebuild their entire live stack from scratch.

## 12.2 Compatibility strategy
The architecture remains compatible because:
- ingest starts with familiar protocols such as WHIP and RTMP
- output remains LL-HLS / HLS
- delivery semantics stay HTTP/TLS-compatible
- software control systems remain API-driven
- the platform can sit behind or alongside current operational tooling

## 12.3 Migration story
This gives customers a practical migration path:
- current contribution workflows can continue
- current HLS-oriented player ecosystems can continue
- ViMach improves the internal media path rather than demanding a total client-side protocol rewrite

This compatibility point should be emphasized strongly in the proposal.

---

# 13. Comparative Positioning

## 13.1 Against generic managed live-streaming stacks
Generic cloud live stacks are strong in:
- convenience
- rapid startup
- broad ecosystem familiarity

ViMach should differentiate on:
- deterministic media handling
- premium live quality path
- package-once / fanout-late discipline
- media-plane isolation from software control noise
- hardware-defined media transport

## 13.2 Against generic server/NIC clusters
A well-tuned server cluster can deliver media, but ViMach is not just another server cluster.

ViMach differentiates by:
- terminating media-plane transport on FPGA in Zone 4
- keeping media off the generic host NIC path
- preserving a dedicated optical media plane end-to-end inside the platform

## 13.3 Scale comparison
ViMach should not claim “national-scale dominance on day one.”  
It should claim:
- a repeatable node pattern
- credible multi-region scale-out
- architecture compatible with partner-assisted edge expansion

This is the correct scale message.

---

# 14. Case Study: 4K Ingest + 2 Profiles + 100K Users

## 14.1 Purpose of the case study
This case study exists to prove the architecture, not to define its permanent ceiling.

## 14.2 Scenario
- one 4K live ingest
- two output profiles:
  - original 4K
  - 2K fallback
- 100,000 concurrent viewers
- 50% on original 4K
- 50% on 2K fallback

## 14.3 Planning assumptions
- original 4K profile: 30 Mbps
- 2K profile: 12 Mbps

## 14.4 Payload calculation
- 50,000 × 30 Mbps = 1.5 Tbps
- 50,000 × 12 Mbps = 0.6 Tbps
- total payload = 2.1 Tbps

## 14.5 Provisioned target
Applying:
- 20% protocol / serving overhead
- 20% operational headroom

results in:
- ~3.15 Tbps provisioned target
- rounded planning target: ~3.2 Tbps

## 14.6 Egress fleet planning
Using a planning assumption of ~100 Gbps usable media delivery per pod:
- ~32 active egress pods

## 14.7 Why this case matters
This case forces the architecture to prove:
- deterministic packaging
- optical media-plane fanout
- FPGA-assisted egress ingestion
- CPU-only HLS serving semantics
- practical delivery scale

That is exactly why it is a good proof case.

---

# 15. Hardware Strategy

## 15.1 Track A — Commercial hardware now
Track A is used for:
- proof
- benchmarks
- pilot
- demo
- early customer validation

Suggested roles:
- x86 servers for ingest/control
- GPU servers for fallback-profile creation
- commercial FPGA cards for deterministic media-core proof
- egress hosts with FPGA + CPU

## 15.2 Track B — ViMach custom media card
Track B runs in parallel.

Goals:
- improved watts/Gbps
- improved optical density
- improved FPGA-assisted egress integration
- improved production economics
- stronger long-term moat

## 15.3 Why this matters
Track A proves the architecture fast.  
Track B makes the architecture harder to replace later.

---

# 16. Software and Operations Stack

## 16.1 Required software capabilities
The platform must include:
- public API / ingress services
- auth / entitlement
- routing/session direction
- manifest control
- observability
- deployment tooling
- operator dashboards
- support systems

## 16.2 Design rule
These systems should remain robust and production-shaped, but they must remain outside the deterministic media plane.

## 16.3 Why this matters
This is what allows the proposal to claim both:
- deterministic media handling
- operational maturity

without mixing the two layers in a fragile way.

---

# 17. Roadmap

## 17.1 12-month aggressive path
- workload and architecture freeze
- ingest + profile creation + media-core bring-up
- HLS-ready object fanout proof
- FPGA-assisted egress ingest proof
- LL-HLS delivery proof
- benchmark package
- investor/customer demo readiness

## 17.2 18-month realistic path
- end-to-end internal proof
- queue/jitter instrumentation
- 10K and 100K planning validation
- software/control-plane hardening
- Track B prototype bring-up
- pilot deployment readiness

## 17.3 Roadmap principle
The objective is not to promise a giant national deployment in the base scope.  
The objective is to prove a repeatable deterministic node and make it ready for controlled expansion.

---

# 18. Risk and Mitigation

## Risk 1 — Media and control paths collapse into each other
**Mitigation:** enforce plane separation at architecture, deployment, and operations levels.

## Risk 2 — Zone 4 CPUs start doing media work again
**Mitigation:** keep Zone 4 CPU restricted to HLS delivery semantics only.

## Risk 3 — Zone 3 becomes too application-specific
**Mitigation:** keep Zone 3 focused on deterministic media objects, not full browser-facing HLS semantics.

## Risk 4 — Architecture is mistaken for 4K-only
**Mitigation:** state clearly throughout the proposal that 4K/100K is a proof case, not a permanent ceiling.

## Risk 5 — Multi-region story becomes too vague
**Mitigation:** define scaling as repetition of the deterministic node pattern, not as a completely different later architecture.

---

# 19. Recommended Proposal Language

## Core statement
> HELM / ViMach is building a deterministic premium live media platform defined by an independent media plane and control plane, a repeatable FPGA-centered media-core node, and a practical path from current premium live profiles to future higher-end profiles.

## Architecture statement
> The deterministic media core packages and fans out HLS-ready media objects over a dedicated optical media plane, while egress hosts receive those objects through FPGA cards and complete user-facing HLS delivery semantics in software.

## Scale statement
> The same node pattern can be repeated across multiple regions, allowing the platform to scale without changing its core architecture.

## Profile statement
> The 4K + 100K case study is the first proof of the architecture, not its permanent limit. The platform is intentionally designed to remain open to future premium live profiles, including 8K-class deployments when commercially justified.

---

# 20. Final Recommendation

This updated proposal should be used as the primary internal and external narrative because it makes the core ViMach value proposition explicit:

- **DETERMINISTIC**
- **independent media plane**
- **independent control plane**
- **optical media-plane transport**
- **FPGA-assisted egress ingestion**
- **profile-agnostic platform logic**
- **repeatable multi-region node pattern**

The correct message is:

> **ViMach is not building a fixed-resolution streaming appliance. It is building a deterministic premium live media platform whose first proof case is 4K + 100K, and whose architecture is intentionally designed to support future profiles, future regions, and future scale.**
