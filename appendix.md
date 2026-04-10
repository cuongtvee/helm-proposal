Below is a **ready-to-paste English appendix** for the proposal. I wrote it to match the tone of the v1.3 sections and to stay technically defensible.

---

## Appendix A — Reference Hardware BOM for the 100,000-Viewer Proof Platform

This appendix defines a **reference proof-platform BOM** for the initial benchmark scenario described in the proposal: **1 × 4K live ingest, 2 delivery profiles, and 100,000 concurrent viewers**. It should be read as a **staged proof BOM**, not as the final bill of materials for a full commercial multi-region deployment.

The purpose of this BOM is to support four proof objectives:

* establish a managed deterministic media path,
* validate modular egress through repeatable hardware nodes,
* measure throughput, queue behavior, and power,
* and integrate the hardware platform with the HELM software environment.

### A.1 Hardware Selection Logic

ViMach should use **three hardware classes**, each serving a different role in the proof path.

**1. Bring-up and subsystem validation platforms**
These are suitable for protocol experiments, basic optical and Ethernet bring-up, embedded control, telemetry, and low-rate functional validation.
The **AMD KR260 Robotics Starter Kit** includes **4 × 1GbE RJ45 ports and 1 × 10GbE SFP+ optical interface**, making it useful for low-rate node experiments and control-side prototyping. ([AMD][1])
The **AMD KCU105** is useful for 10G-class optical and packet-processing experiments and is commonly used as a general FPGA development board; the **AMD VCK190** offers higher-end Versal-class development with **2 × SFP28 and 1 × QSFP28** for higher-bandwidth subsystem validation. ([AMD Documentation][2])

**2. Media-core proof platform**
For the managed media core, the strongest evaluation candidate is the **AMD VHK158 Evaluation Kit**. AMD specifies **32 GB HBM**, **819.2 GB/s HBM bandwidth**, **dual Cortex-A72**, **dual Cortex-R5F**, **GTYP 32.75 Gb/s transceivers**, and **GTM transceivers supporting up to 112G PAM4**. AMD also highlights **100G and 600G Ethernet cores**, **400G crypto engines**, and board-level connectivity including **4 QSFP and 2 QSFP-DD connectors**, which makes it a strong fit for deterministic media-core and optical-fabric proof work. ([AMD][3])

**3. Production-reference egress node platform**
For high-throughput modular egress proof, the recommended reference node is the **AMD Alveo V80**. AMD specifies **32 GB HBM2e**, approximately **819–820 GB/s memory bandwidth**, **4 × QSFP56 optical ports**, and **PCIe Gen4 x16 or dual Gen5 x8**, with a passive **up to 190W TDP** card form factor. These characteristics make the V80 far more suitable than starter kits for a serious modular egress proof platform. ([AMD Documentation][4])

### A.2 Recommended Proof BOM

The following configuration is a practical and credible starting point for the 100,000-viewer proof environment.

**Managed media core**

* **2 × AMD VHK158 Evaluation Kit**
  Recommended role: deterministic media core, queue and timing instrumentation, controlled packaging logic, optical-fabric experiments, active/standby or mirrored validation path. ([AMD][3])

**Modular egress nodes**

* **4–8 × AMD Alveo V80 cards**, installed across **2–4 x86 host servers**
  Recommended role: production-reference egress nodes for modular scaling proof, delivery preparation, node-level pacing, and throughput measurement. ([AMD][5])

**Host servers**

* **2 × x86 servers** for Alveo V80 hosting
* **2 × x86 servers** for ingest gateway, control adapters, telemetry collection, and orchestration
* **2–4 × x86 servers** for synthetic traffic generation, session simulation, and benchmark replay
  This layout keeps the proof credible by separating media-node hosting from control, observability, and load-generation functions.

**NICs**

* **NVIDIA ConnectX-7 Ethernet adapters** for high-bandwidth server attachment. NVIDIA specifies support for **up to 400GbE**, **1/2/4-port variants**, **PCIe Gen5 x16/x32**, and interfaces such as **SFP56, QSFP56, QSFP56-DD, and QSFP112**. These are appropriate for V80 hosts, benchmark hosts, and traffic-generation servers. ([NVIDIA][6])

**Switching fabric**

* **1–2 × 100/400GbE fabric switches** for the proof lab
  A practical reference choice is the **NVIDIA Spectrum SN4000 family**, which NVIDIA specifies at **up to 12.8 Tbps switching capacity** with support for **1/10/25/40/50/100/200/400GbE** and built-in timing support including **PTP**. ([NVIDIA][7])

**Optics and cabling**

* QSFP56 / QSFP-DD optics as required by the selected fabric and node topology
* DAC/AOC for short in-rack links
* single-mode optics for cross-rack or zone-level proof paths
  The exact optic mix depends on whether the proof environment is built as a single-rack cell or a multi-rack media zone.

**Timing and observability**

* **1 × PTP grandmaster or equivalent timing source**
* out-of-band management switch
* telemetry collectors and storage for traces, metrics, pcap, and benchmark logs
  PTP-aware switching is supported in the SN4000 family, which is useful for timing discipline and measurement during proof. ([NVIDIA][7])

### A.3 Lower-Cost Bring-Up Configuration

A lower-cost lab path can be used for early subsystem work, but it should **not** be presented as the full 100,000-viewer proof platform.

A reasonable bring-up set is:

* **2 × KR260**
* **2 × KCU105**
* **1 × VCK190**
* **2–3 × x86 servers**
* **1 × 100GbE development fabric switch**

This configuration is useful for software/hardware partition validation, control adapters, protocol compatibility, low-rate pacing experiments, and basic optical tests. However, the KR260’s network I/O and starter-kit form factor make it inappropriate as the primary egress-node representative for the full 100,000-viewer benchmark. ([AMD][1])

### A.4 Hardware Path Recommendation

The proposal should state the hardware path clearly:

* **Bring-up / subsystem validation:** KR260, KCU105, VCK190
* **Managed media-core proof:** VHK158
* **Production-reference egress proof:** Alveo V80
* **Future specialization:** custom board and, only after evidence gates are met, possible ASIC consideration

This staged path reduces technical and capital risk while preserving a credible evolution from development kits to a pilot-ready infrastructure platform.

---

## Appendix B — Reference Architecture Diagrams and Explanatory Notes

### B.1 Hardware Architecture

```text
                           +--------------------------------------+
                           |         CONTROL / OPS ZONE           |
                           |--------------------------------------|
                           | Orchestrator | Routing Policy | QoE  |
                           | Metrics      | Logs           | Auth |
                           +-------------------+------------------+
                                               |
                                      Out-of-band control
                                               |
=========================== Managed Media Platform ===========================

+----------------+   +-------------------+   +-------------------+
| Ingest Gateway |-->| Deterministic     |-->| Optical Fabric    |
| WHIP / RTMP    |   | Media Core        |   | 100/400G Fabric   |
| Normalize      |   | (VHK158-class)    |   | leaf/spine zone   |
+----------------+   | queue/timing/pkt  |   +---------+---------+
                     +-------------------+             |
                                                       |
                              +------------------------+------------------------+
                              |                        |                        |
                     +--------+--------+      +--------+--------+      +--------+--------+
                     | Egress Node 1   |      | Egress Node 2   |      | Egress Node N   |
                     | (Alveo V80 host)|      | (Alveo V80 host)|      | modular repeat  |
                     +--------+--------+      +--------+--------+      +--------+--------+
                              |                        |                        |
                              +------------------------+------------------------+
                                                       |
                                             +---------+---------+
                                             | CDN / ISP / IX    |
                                             | Public Internet   |
                                             +-------------------+
```

**Short explanation**
The hardware architecture is intentionally divided into three layers: **ingest and normalization**, **deterministic media core plus controlled optical fabric**, and **repeatable egress nodes**. The most important system principle is that the **control plane remains out of band**, while the bandwidth-heavy, timing-sensitive media path is contained inside the managed platform. This is consistent with the VHK158’s strength as a high-bandwidth media-core proof device and the V80’s suitability as a high-throughput egress-node reference platform. ([AMD][3])

### B.2 Software Architecture

```text
+--------------------------------------------------------------------+
|                          SOFTWARE CONTROL PLANE                    |
|--------------------------------------------------------------------|
| Session / Entitlement | Routing Policy | Orchestration | QoE       |
| Observability         | Capacity Mgmt  | Failover Ctrl | APIs      |
+--------------------------+----------------+------------------------+
                           |                |
                           | southbound     | telemetry ingest
                           v                ^
+--------------------------------------------------------------------+
|                    MEDIA-PLANE CONTROL ADAPTERS                    |
|--------------------------------------------------------------------|
| FPGA config mgmt | node inventory | health poll | rollout control  |
+--------------------------+----------------+------------------------+
                           |                |
                           v                ^
+--------------------------------------------------------------------+
|                    DETERMINISTIC MEDIA EXECUTION                   |
|--------------------------------------------------------------------|
| Ingest normalize | queue discipline | pacing | egress preparation  |
| optical movement | node-local telemetry | degraded local behavior  |
+--------------------------------------------------------------------+
```

**Short explanation**
The software architecture should be described in three layers: **service and operational control**, **media-plane adapters**, and **hardware media execution**. The adapter layer is especially important because it allows orchestration, observability, and rollout control to interact with FPGA and accelerator nodes without moving general service logic into the timing-critical media path.

### B.3 Data Center and Network Topology

```text
Region A
┌─────────────────────────────────────────────────────────────────────┐
│                         CONTROL / MGMT ZONE                         │
│  mgmt switch 1/10/25G                                               │
│  orchestration servers                                              │
│  observability / logs / PTP / provisioning                          │
└───────────────┬─────────────────────────────────────────────────────┘
                |
                | out-of-band mgmt
                |
┌───────────────┴─────────────────────────────────────────────────────┐
│                         MEDIA FABRIC ZONE                           │
│                                                                     │
│   [Leaf/Fabric Switch 100/400G]  <---->  [Leaf/Fabric Switch]       │
│          |             |                    |                        │
│   [Ingest GW]    [Media Core A]      [Media Core B]                 │
│                    (VHK158)            (VHK158)                      │
│          |             |                    |                        │
│          +-------------+---------100G/400G--+                        │
│                                |                                     │
│         +----------------------+----------------------+              │
│         |                      |                      |              │
│   [Egress Host 1]       [Egress Host 2]       [Egress Host N]       │
│   [Alveo V80 x2]        [Alveo V80 x2]        [Alveo V80 x2]        │
│         |                      |                      |              │
│         +----------------------+----------------------+              │
│                                |                                     │
│                        Border / CDN / IX routers                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Short explanation**
The proposal should present the data center design as two zones: a **Control/Management Zone** and a **Media Fabric Zone**. Inside the media zone, the architecture follows a leaf/spine or equivalent IP-fabric pattern with dedicated media-core devices and modular egress hosts. Juniper’s EVPN-VXLAN data center fabric guidance is consistent with this style of modern scalable fabric design, and the NVIDIA SN4000 family is a practical 100/400G switching reference for such a zone. ([Juniper Networks][8])

---

## Appendix C — Timing, Failure, and Benchmark Assumption Tables

### C.1 Managed-Path Timing and Queue Budget

The proposal should include a table like the following:

| Stage                | Primary Metric                     | Target Type | Observability Source      |
| -------------------- | ---------------------------------- | ----------- | ------------------------- |
| Ingest normalization | stage latency                      | bounded     | gateway + node telemetry  |
| Managed media core   | queue depth / queue growth         | bounded     | on-device counters        |
| Optical fabric       | hop variance / congestion behavior | bounded     | switch telemetry          |
| Egress node          | pacing jitter / throughput         | bounded     | node-local counters       |
| Recovery path        | failover / redistribution time     | controlled  | node + control-plane logs |

This table converts the term **deterministic** into a measurable engineering contract.

### C.2 Failure Domains and Recovery Contract

The proposal should also include a simple fault model:

| Fault Domain            | Detection                      | Initial Response               | Degradation Mode              | Recovery Goal               |
| ----------------------- | ------------------------------ | ------------------------------ | ----------------------------- | --------------------------- |
| Egress node failure     | node health / telemetry loss   | isolate node                   | reduced reserve margin        | controlled redistribution   |
| Optical link impairment | port telemetry / BFD / errors  | isolate or reroute path        | local capacity reduction      | bounded recovery            |
| Control-plane delay     | stale heartbeat / stale policy | continue local media operation | reduced orchestration agility | graceful re-sync            |
| Ingest instability      | ingress monitoring             | normalize or isolate input     | source-quality degradation    | preserve platform stability |

Google’s Cloud Router best-practice guidance supports the use of **BFD** for faster forwarding-path outage detection and recommends **MD5 authentication** for BGP sessions where supported. Those are relevant principles for ViMach’s northbound and interconnect edge design as well. ([Google Cloud Documentation][9])

### C.3 Benchmark Assumptions Table

| Parameter                   | Initial Assumption               |
| --------------------------- | -------------------------------- |
| Ingest                      | 1 × 4K live ingest               |
| Profiles                    | 4K + 2K                          |
| Viewer mix                  | 50% / 50%                        |
| 4K bitrate                  | 30 Mbps                          |
| 2K bitrate                  | 12 Mbps                          |
| Payload throughput          | ~2.1 Tbps                        |
| Provisioned delivery target | ~3.2 Tbps                        |
| Region model                | single-region proof benchmark    |
| Reserve margin              | included in provisioned capacity |

This table should sit next to the benchmark narrative so that throughput and economic claims can be interpreted as part of a defined service model, not as isolated payload math.

### C.4 Hardware Phase Gates

| Phase                 | Hardware Class          | Primary Goal                         |
| --------------------- | ----------------------- | ------------------------------------ |
| Bring-up              | KR260 / KCU105 / VCK190 | subsystem validation                 |
| Core proof            | VHK158                  | managed media-core proof             |
| Production reference  | Alveo V80               | high-throughput modular egress proof |
| Future specialization | custom board / ASIC     | only after proof gates are met       |

---

## Appendix D — Carrier, Interconnect, and Network Equipment Selection for Data Center Ingress and Egress

### D.1 Carrier Selection Principles

ViMach should prefer **carrier-neutral data center environments** wherever possible. Carrier-neutral facilities improve optionality because they allow the platform to connect to multiple network providers and interconnection services in the same location. Equinix describes its facilities as highly connected interconnection environments across many metros, and carrier-neutral interconnection is widely valued because it improves resilience, flexibility, and placement near partners and users. ([US English][10])

For ViMach, carrier selection should be divided into **three roles**:

**1. Inbound contribution / ingest carriers**
These links carry source feeds, venue contribution, partner handoff, or inter-region media transport into the managed media platform.

**2. Outbound transit / edge-delivery carriers**
These links carry traffic from ViMach toward ISP/CDN destinations and major eyeball networks.

**3. Data center interconnect (DCI) providers**
These links connect ViMach regions, backup sites, or parallel media zones.

Each role should be selected independently, even if the same operator can provide more than one service.

### D.2 Recommended Carrier Strategy

A strong initial strategy is:

* **at least two physically and commercially independent upstream transit providers** for outbound Internet reach,
* **one or more peering/IX relationships** for lower-latency and lower-cost exchange with important networks,
* and **separate DCI capacity** for region-to-region replication or backup-site connectivity.

DE-CIX explicitly notes that peering can reduce latency and gives networks more control over where traffic is exchanged, while Equinix positions direct interconnection and DCI as a way to achieve scalable and highly available private connectivity across metros. ([de-cix.net][11])

For premium live delivery, this means ViMach should avoid a single-provider edge design. The proposal should state that **outbound reach, peering, and inter-region connectivity are separate architecture decisions**, even when they terminate in the same facility.

### D.3 Selection Criteria for Inbound Carriers

For inbound contribution and ingest, the carrier decision should prioritize:

* route stability,
* deterministic packet-loss behavior,
* SLA and restoration commitments,
* metro and venue reach,
* support for diverse physical entry paths,
* and handoff options at 10/25/100/400G.

If contribution feeds come from venues or partner broadcasters, diversity of physical path and handoff location is at least as important as raw bandwidth. The objective is to reduce single-path failure risk before traffic enters the managed media platform.

### D.4 Selection Criteria for Outbound Providers

For outbound delivery, the platform should combine:

* **IP transit** for broad Internet reach,
* **peering / IX** for direct exchange with key networks,
* and, where commercially relevant, **private interconnects** to strategic CDN or distribution partners.

DE-CIX’s peering guidance is relevant here: direct peering lowers latency and gives the network more control over where traffic is exchanged. For premium live delivery, that can materially improve user-path quality and reduce dependence on expensive generic transit for every destination. ([de-cix.net][11])

A useful target model is:

* **Transit A + Transit B**
* **IX / peering fabric**
* optional **private partner interconnects**

This gives ViMach pricing leverage, route diversity, and a cleaner path to policy-based traffic engineering.

### D.5 Border and Interconnect Design

ViMach’s border should support **eBGP-based multihoming**, **BFD**, **MD5 where appropriate**, policy-based route control, and rapid failover between transit, peering, and DCI paths. Google’s hybrid-connectivity and Cloud Router guidance explicitly supports BFD for faster failure detection and notes MD5 authentication as a best practice when supported by the peer. ([Google Cloud Documentation][9])

At the proposal level, the key design principle is simple:

* the **media fabric** remains internal and deterministic,
* the **border layer** translates that managed delivery system into a resilient, policy-controlled external network edge.

### D.6 Recommended Network Equipment Roles

The network equipment should be described by **role**, not by a single fixed vendor.

**1. Media-fabric switching inside the data center**
For high-density 100/400G media-zone switching, a practical reference class is the **NVIDIA Spectrum SN4000 family**, which supports **up to 12.8 Tbps** and 100/200/400GbE speeds with timing support such as **PTP**. This makes it a suitable candidate for a media-fabric leaf/spine proof environment. ([NVIDIA][7])

**2. High-density leaf/spine or universal fabric switch/router class**
The **Arista 7280R3 family** is a strong example of a high-performance 100/400G leaf or universal data-center switch/router class, with Arista specifying **up to 24 ports of wire-speed 400G in 1RU** and broader family options scaling to **21.6 Tbps** in 2RU. ([Arista Networks][12])

**3. Border / peering / DCI router class**
For external edge, transit, peering, and DCI roles, the proposal can reference compact high-capacity routing platforms such as the **Juniper MX304**, which Juniper specifies at **4.8 Tbps in 2RU**, or the **Juniper PTX10001-36MR**, which Juniper specifies at **9.6 Tbps throughput** in 1RU for dense 100/400G routing and switching roles. ([Juniper Networks][13])

**4. High-speed server attachment NICs**
For node hosts, benchmark servers, and edge services, **NVIDIA ConnectX-7** remains a strong reference NIC because it supports **up to 400GbE**, multiple optical form factors, and PCIe Gen5 host connectivity. ([NVIDIA][6])

### D.7 Suggested Inbound/Outbound Network Topology

```text
                    +-----------------------------------+
                    |        Upstream Transit A         |
                    +----------------+------------------+
                                     |
                    +----------------+------------------+
                    |        Upstream Transit B         |
                    +----------------+------------------+
                                     |
                    +----------------+------------------+
                    |      IX / Peering Fabric          |
                    +----------------+------------------+
                                     |
                          +----------+----------+
                          | Border / Peering    |
                          | Routers (MX/PTX or  |
                          | equivalent class)   |
                          +----------+----------+
                                     |
                           +---------+---------+
                           | Media Fabric Edge |
                           | Leaf / Spine      |
                           +---------+---------+
                                     |
                         +-----------+------------+
                         | Managed Media Platform |
                         | Core + Egress Nodes    |
                         +-----------+------------+
                                     |
                    +----------------+------------------+
                    |  Inbound Contribution / DCI Links |
                    |  venue feeds / partner ingest /   |
                    |  inter-region replication         |
                    +-----------------------------------+
```

**Short explanation**
This topology makes a useful architectural distinction:

* **Inbound links** bring source media into the platform.
* **The internal media fabric** distributes it through the deterministic managed path.
* **Outbound edge routers and peering/transit links** export delivery toward the Internet and partner networks.

This separation improves operational clarity and allows carrier strategy to evolve without changing the core media-plane architecture.

### D.8 Practical Recommendation for the Proposal

The proposal should avoid naming a single mandatory carrier or a single mandatory equipment vendor for all deployments. Instead, it should state the selection framework:

* deploy in **carrier-neutral facilities** where possible,
* use **at least two independent upstream providers** for outbound reach,
* add **IX/peering** where traffic profile justifies it,
* keep **DCI** independent from basic Internet transit,
* and select border, fabric, and server-attachment equipment by role and density requirements rather than by one fixed brand.

That gives ViMach a more defensible infrastructure position because it emphasizes architecture, resilience, and repeatability over vendor lock-in.

---

## Appendix E — Closing Note

These appendices are intended to show that ViMach is not only an architectural thesis, but a **buildable staged system**.

The hardware path progresses from subsystem bring-up to deterministic media-core proof and then to production-reference egress nodes. The software and network design preserve strict separation between control and managed media execution. Carrier and border strategy are treated as first-class architectural decisions rather than as afterthoughts.

This staged approach reduces technical risk, keeps benchmarking credible, and provides a practical path from laboratory validation to pilot-ready infrastructure.

If you want, I can now turn this into a **single consolidated v1.3 proposal draft** with Sections 1–5 plus Appendices A–E merged and de-duplicated.

[1]: https://www.amd.com/en/products/system-on-modules/kria/k26/kr260-robotics-starter-kit.html?utm_source=chatgpt.com "Kria KR260 Robotics Starter Kit"
[2]: https://docs.amd.com/r/en-US/ug1611-vhk158-eval-bd?utm_source=chatgpt.com "VHK158 Evaluation Board User Guide (UG1611)"
[3]: https://www.amd.com/en/products/adaptive-socs-and-fpgas/evaluation-boards/vhk158.html?utm_source=chatgpt.com "AMD Versal™ HBM Series VHK158 Evaluation Kit"
[4]: https://docs.amd.com/r/en-US/ds1013-v80/Summary?utm_source=chatgpt.com "Summary - DS1013"
[5]: https://www.amd.com/en/products/accelerators/alveo/v80/a-v80-p64g-pq-g.html?utm_source=chatgpt.com "AMD Alveo™ V80 Compute Accelerator Card"
[6]: https://www.nvidia.com/content/dam/en-zz/Solutions/networking/ethernet-adapters/connectx-7-datasheet-Final.pdf?utm_source=chatgpt.com "ConnectX-7 Ethernet Datasheet"
[7]: https://www.nvidia.com/en-au/networking/ethernet-switching/spectrum-sn4000/?utm_source=chatgpt.com "Spectrum SN4000 Open Ethernet Switches"
[8]: https://www.juniper.net/documentation/us/en/software/nce/sg-005-data-center-fabric/index.html?utm_source=chatgpt.com "Data Center EVPN-VXLAN Fabric Architecture Guide"
[9]: https://docs.cloud.google.com/network-connectivity/docs/router/concepts/best-practices?utm_source=chatgpt.com "Best practices for Cloud Router"
[10]: https://www.equinix.com/?utm_source=chatgpt.com "Equinix: Data Center Company & Enterprise Network ..."
[11]: https://www.de-cix.net/en/about-de-cix/news/10-reasons-to-peer-3-peering-lowers-latency?utm_source=chatgpt.com "10 Reasons to peer: 3. peering lowers latency"
[12]: https://www.arista.com/en/products/7280r3-series?utm_source=chatgpt.com "Arista 7280R3 Series"
[13]: https://www.juniper.net/documentation/us/en/hardware/mx304/topics/topic-map/mx304-system-overview.html?utm_source=chatgpt.com "MX304 System Overview"
