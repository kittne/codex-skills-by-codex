# Generalized Best Practices for Building on Radix

## Executive summary

This report synthesizes a “best practices” blueprint for building secure, maintainable, and user-friendly systems on Radix, with a focus on the Babylon-generation stack (Radix Engine v2, Scrypto, manifests, wallet-centric UX, and live protocol updates). It is written for blockchain engineers and architects designing production dApps, integrations, or infrastructure (nodes/gateways). Radix’s differentiators—human-readable transaction manifests, asset-oriented execution, badge-based authorization, rich metadata standards, and increasingly powerful multi-actor transaction flows (subintents / pre-authorizations)—reshape how teams should think about smart contract attack surfaces, UX guarantees, integration architecture, and operational runbooks. citeturn0search4turn1search8turn6view0turn8search0turn3search6

Key takeaways which strongly influence a generalized best-practices document:

- Treat **protocol versioning** as a first-class constraint: Babylon mainnet has had multiple coordinated protocol updates (e.g., Bottlenose, Cuttlefish), and language/tool versions track those changes; pin toolchains per target network and gate features by protocol version. citeturn8search1turn8search2turn8search3turn7search2  
- Use Radix’s **capability-based security model** (badges + AccessRules + AuthZone proofs) systematically; avoid “address-based” authorization habits from EVM-style systems, and explicitly defend against Radix-specific footguns like **proof cloning / “proof amount” misuse**. citeturn1search0turn1search8turn14search0  
- Design UX around **conforming manifests** + metadata standards so the wallet can present accurate, comprehensible previews; use subintents/pre-authorizations and AccountLocker patterns to solve “fee delegation,” deposit acceptance, and safer onboarding flows. citeturn4search3turn8search0turn3search15turn3search6  
- Optimize for today’s execution realities while architecting for future scalability: Radix’s roadmap envisions multi-sharded, massively parallel consensus at the Xi’an upgrade, but current Babylon behavior does not yet expose full parallelization benefits—so build with principled state partitioning and non-contention patterns, without assuming shard-level concurrency is already available on mainnet. citeturn7search1turn7search5turn7search0  
- Operational maturity matters: production integrations are expected to run full nodes (and a hot-swappable backup), monitor health and metrics endpoints (Prometheus/Grafana), and follow coordinated node update and readiness-signaling processes. citeturn13search2turn0search5turn0search1turn8search3turn2search8  

## Scope, assumptions, and versioning

The target Radix “version” is unspecified in the request. In practice, best practices must be version-aware across:

- **Network eras**: Olympia (Radix Engine v1) vs Babylon (Radix Engine v2 + smart contracts) and beyond. citeturn7search15turn7search6turn7search2  
- **Babylon protocol updates**: Radix documents a sequence of enacted updates including `babylon-genesis` (Olympia→Babylon migration), `anemone`, `bottlenose`, and `cuttlefish` (including `cuttlefish-part2`). citeturn8search1turn8search3  
- **Feature gating by protocol update**: Bottlenose introduced AccountLocker and related enhancements; Cuttlefish brought subintents / pre-authorizations and Transaction V2 semantics for builders. citeturn1search23turn8search0turn8search2  

A generalized best-practices document should therefore include an explicit “compatibility matrix” section, minimally mapping:

- **Core smart-contract capability**: Babylon genesis (Radix Engine v2 / Scrypto deployability). citeturn8search1turn7search15  
- **AccountLocker**: Bottlenose. citeturn1search23turn8search1  
- **Subintents / pre-authorizations**: Cuttlefish (launched on mainnet December 2024). citeturn8search0turn8search1turn8search2  

Toolchain adaptation guidance (must be explicit in the best-practices doc):

- Pin **Scrypto/CLI versions** to the target engine/protocol. Scrypto v1.3.0 is documented as adding support for the Cuttlefish protocol update and also constrains the Rust toolchain (e.g., rustc ≤ 1.81 cited for Cuttlefish compatibility). citeturn8search2turn15search10  
- For off-ledger integrations, select the appropriate Radix Engine Toolkit wrapper and favor stable interfaces (e.g., “LTS” surfaces for integrators) when backward compatibility is a requirement. citeturn11search5turn11search17turn7search16  
- For infrastructure operators, treat protocol updates as coordinated network events (stake-threshold readiness signaling within epoch windows), and bake that into maintenance windows and runbooks. citeturn8search3turn1search13  

## Radix primitives that shape best practices

Radix’s best practices are unusually sensitive to its transaction model, authorization model, and wallet-first UX stack.

### Transaction model, manifests, and atomicity semantics

A Radix transaction is built from a **human-readable transaction manifest** which is compiled/encoded and then signed for submission; the explicit design goal is that clients (notably the wallet) can understand “what will happen” from the manifest itself. citeturn0search4turn4search18  

Atomicity is nuance-sensitive:

- From a state-change perspective, failures do not commit application-side changes (“all-or-nothing” semantics for the intended state update). citeturn4search18  
- But a “Committed Failure” still pays fees up to the failure point, even though other state changes are discarded; only committed transactions appear on ledger. citeturn0search8  

These semantics drive best practices for both security (avoid “fail-open” assumptions) and UX (communicate fee exposure, prefer early assertion checks).

### Data encoding and limits: SBOR and payload constraints

SBOR (“Scrypto Binary-Friendly Object Representation”) is used across the Radix Engine for transaction encoding, state storage, and internal communication; best practices must consider SBOR payload structure, client decoding, and protocol-imposed limits. citeturn8search6turn3search37  

### Consensus, epochs, and ledger structure

Babylon’s ledger is described as a **stream/chain of committed transactions** partitioned into epochs (approximately 5 minutes), rather than discrete user-facing blocks; “state version” can serve as an index for integrations needing a monotonically increasing ledger position. citeturn4search9turn4search2  

Implementation detail relevant to ops/integration expectations: the Babylon node repository describes its core consensus as including a **variant of HotStuff BFT-style consensus**, which aligns with Radix’s broader emphasis on BFT finality within its architecture. citeturn11search14  

### Authorization model: badges, AccessRules, AuthZone, and native controllers

Radix emphasizes a badge-based authorization approach where AccessRules validate proofs present in the AuthZone, enabling capability-style permissions and role-based access control. citeturn1search8turn1search4turn1search32  

A major security and UX primitive is the on-ledger **Access Controller** blueprint, which holds badges and defines multi-role logic (Primary/Recovery/Confirmation) for multi-factor recovery and role updates. citeturn8search5turn2search1  

### Developer and integrator stack: toolkit + gateway/core APIs + dApp toolkit + ROLA

A typical system uses:

- The **Radix dApp Toolkit** for wallet connection and transaction requests (including a consistent connect button pattern). citeturn1search5turn1search1  
- The **Radix Engine Toolkit (RET)** for off-ledger construction/compilation/validation/decompilation of manifests and transactions, with multiple supported language wrappers. citeturn11search2turn11search5turn11search7  
- The **Gateway API** to query ledger state and submit transactions. citeturn3search12turn0search8  
- **ROLA (Radix Off-Ledger Auth)** for verifying cryptographic proofs of control of on-ledger identities/accounts (notably for “Persona login” flows). citeturn2search2turn2search22  

### Visual grounding: typical Radix stack artifacts

image_group{"layout":"carousel","aspect_ratio":"16:9","query":["Radix Wallet transaction manifest preview","Radix dApp Toolkit connect button UI","Radix AccountLocker blueprint diagram","Radix node Grafana dashboard Radix monitoring"] ,"num_per_query":1}

### Reference architecture pattern

```mermaid
flowchart LR
  subgraph Client
    UI[Web / Mobile dApp UI]
    RDT[dApp Toolkit (wallet connection)]
  end

  subgraph Wallet
    W[Radix Wallet]
    P[Persona / Identity]
  end

  subgraph Backend
    API[dApp backend API]
    ROLA[ROLA verifier]
  end

  subgraph RadixInfra
    GW[Gateway API]
    N[Full node / validator node]
  end

  UI --> RDT --> W
  W -->|signed tx / signed pre-authorization| RDT
  RDT -->|submit| GW --> N

  UI -->|login challenge| API --> ROLA
  ROLA -->|verify against ledger state| GW
  API -->|session token / app authz| UI
```

This diagram reflects the documented split between (a) transaction creation/signing as a wallet-centric UX and (b) optional off-ledger authentication based on proofs of control of on-ledger identities/accounts. citeturn1search5turn2search2turn3search12turn8search0  

## Best practices across key engineering domains

### Security, smart contract safety, and key management

**Best-practice guidelines**

- Prefer Radix-native **resource/vault flows** over manual “balance accounting” patterns; let the engine enforce asset semantics wherever possible (this is aligned with Radix’s asset-oriented design narrative and should reduce classes of double-accounting style mistakes). citeturn7search12turn6view0  
- Use **badges + AccessRules** per method/function with least privilege; avoid “address-based” authorization and keep a clear mapping from role → required proof(s). citeturn1search0turn1search8turn1search4  
- Treat **proofs as non-consumable evidence**; do not use “proof amount” as a proxy for one-time entitlement or vote weight unless you also prevent proof cloning/double-use. (Radix docs show proof-cloning as a concrete exploit class when proofs are misused.) citeturn14search0  
- Decide explicitly whether entities should have **Fixed** or **Updatable** owner roles; use fixed ownership when governance changes are not required, and if updatable is needed, define and secure the update path from day one. citeturn15search1turn15search2  
- For end-user or operator key management, standardize on **multi-factor smart accounts** via Access Controllers (Primary/Recovery/Confirmation roles, timed recovery) and document recovery ceremonies; avoid single-seed dependency where possible. citeturn8search5turn2search1  
- Build a security engineering workflow around: (a) unit tests with `scrypto-test`, (b) integration tests with a ledger simulator (and/or resim), (c) static validation / manifest semantics checks with RET, and (d) external audits for high-value code. citeturn1search2turn1search6turn11search4turn14search1turn14search5  

**Rationale**

Radix’s authorization and asset model deliberately moves security-critical logic into well-defined primitives (resources, badges, AccessRules, native blueprints). But “capability-style” systems can fail if a project reintroduces unsafe patterns on top—especially where proofs are treated as consumable authority or as stable measures of entitlement. The official “code hardening” example demonstrating proof-clone amplification is a good model for explaining Radix-specific pitfalls in a best-practices document. citeturn14search0turn1search32  

**Concrete implementation patterns**

Pattern: “proofs authenticate, buckets transact.” When a user must *spend* influence/rights (vote weight, limited access, one-time claim), require a **bucket** of a resource (or transform it into a claim token) rather than relying on proof amounts. The Radix “code hardening” example describes this as analogous to designs like XRD/LSU concepts (lock underlying, return claim-like representation). citeturn14search0turn12search0turn0search37  

Illustrative Scrypto sketch (conceptual, not a drop-in template):

```rust
// Conceptual: voting by escrow + claim token, avoiding proof-clone amplification.
pub fn vote(&mut self, proposal_id: u64, voting_power: Bucket) -> Bucket {
    assert!(voting_power.resource_address() == self.dao_token);
    let weight = voting_power.amount();

    // escrow voting tokens to prevent re-use
    self.escrow.put(voting_power);

    // mint a claim NFT representing the escrowed amount+proposal
    let claim = self.claim_nft.mint_non_fungible(&proposal_id, ClaimData { weight });
    self.tally_vote(proposal_id, weight);

    claim
}
```

Pattern: multi-factor admin + recovery using Access Controller. The Access Controller blueprint is explicitly designed to hold a badge and provide role-based proof creation + multi-factor recovery with optional timed recovery delays. citeturn8search5turn2search1  

Pattern: “productionize ownership.” Radix docs recommend that blueprint instantiation functions take an OwnerRole and apply it recursively to created resources and components; they also warn that an owner badge must be storable somewhere proofs can actually be generated (typically an account or access controller) or it becomes unusable. citeturn15search1turn15search8  

**Trade-offs**

- Rich role graphs (multi-badge AccessRules, multi-factor controllers, timed recovery) improve resilience but increase cognitive load for both developers and users; mistakes in role configuration can create either lock-out risk or over-permissive access. citeturn1search4turn8search5  
- Updatable owner roles and mutable governance can be necessary for evolving systems, but introduce an explicit trust and attack surface (key compromise, governance capture). Radix documentation notes that “updatable royalties” may discourage usage—similar logic applies to updatable admin roles in general. citeturn15search15turn15search1  
- Platform audits reduce systemic risk but do not replace per-dApp audits. Radix has published multiple protocol audit summaries (e.g., by entity["company","Hacken","web3 security auditor"] and entity["company","Zellic","security research firm"]), but dApp logic and economic design remain application-specific risk domains. citeturn14search5turn14search1turn14search12  

### Performance and scalability

**Best-practice guidelines**

- Design transactions with explicit cost awareness: Radix costing tracks execution/finalization cost units and applies fees via a fee reserve model; set realistic cost unit limits, fail early, and avoid “try-and-revert” loops which still pay fees. citeturn0search0turn0search8  
- Keep manifests/payloads and state updates small: SBOR payloads and transaction encoding are subject to limits; treat large metadata writes, large events, and “giant manifests” as potential throughput killers. citeturn8search6turn3search37  
- Architect state for low contention: partition application state across components/resources so that (a) today’s execution avoids hotspots and (b) future sharded consensus can parallelize cleanly when available. Radix’s Xi’an narrative emphasizes shardability and the need to know which portions of state are relevant per transaction. citeturn7search1turn7search0turn6view2  
- Be explicit about “where scalability lives today”: Radix’s roadmap targets sharded Cerberus at Xi’an, but Radix itself notes that Babylon does not yet expose parallelization benefits; avoid designing operational and UX expectations that assume shard-level concurrency now. citeturn7search5turn7search1  
- Use transaction-level prioritization intentionally: integrator docs describe “tip multipliers” as a way validators may prioritize transactions under contention. citeturn3search2  
- For high-volume markets or matching engines, consider hybrid architectures where heavy matching occurs off-ledger while settlement remains on-ledger; Radix’s Flash Liquidity design discussion is an example of this pattern using subintents. citeturn14search6turn8search0  

**Rationale**

Performance on Radix is tied to how much work a transaction imposes (execution + finalization + storage). The manifest-based model enables powerful composability but can tempt developers into “one huge transaction that does everything,” which can collide with payload/limit constraints and fee/cost-unit ceilings. Separately, Radix’s sharding roadmap implies that future scalability will reward good state partitioning and non-contention—so best practices should teach “design for sharding” even if current mainnet behavior is not yet fully parallel. citeturn0search0turn3search37turn7search5turn6view2  

**Concrete implementation patterns**

Pattern: “assert early, withdraw early.” In the pre-authorization/subintent documentation, Radix recommends that subintent manifests end with `YIELD_TO_PARENT` and suggests withdrawing/yielding buckets early and depositing buckets at the end, plus using `ASSERT_...` instructions to guarantee user outcomes. This pattern improves predictability and reduces failure late in execution (which still costs fees). citeturn3search1turn8search0turn0search8  

Pattern: “protocol-aware batching.” When designing multi-step operations (e.g., swap + stake + deposit), aim for:
- one atomic transaction when user safety/guarantees require it,
- otherwise segment into pre-authorizations (subintents) + a submitter intent, allowing better operational control and possible fee delegation. citeturn8search0turn3search20  

Pattern: “state sharding alignment.” Treat each high-volume object (e.g., an AMM pool, order book, oracle) as its own component with minimal shared global state; avoid singletons and shared mutable registries unless access frequency is low. This aligns with Radix’s description of needing to “separate the state of accounts and components” across shards for massive parallelism at Xi’an. citeturn7search1turn6view2  

**Trade-offs**

- Aggressive state partitioning can fragment developer ergonomics, increase cross-component call overhead, and complicate migrations.  
- Hybrid off-ledger/on-ledger architectures can increase throughput and reduce on-ledger costs, but add trust boundaries and operational complexity (message ordering, DOS resistance, partial failures). citeturn14search6  
- Shard-optimized design may not yield immediate gains on Babylon if parallelization is not yet active; teams must balance current simplicity with future-readiness. citeturn7search5turn7search1  

### Developer experience, tooling, testing, debugging, and CI/CD

**Best-practice guidelines**

- Standardize on the modern “Radix CLIs” toolchain (`resim`, `scrypto`, `rtmc/rtmd`, `scrypto-bindgen`) and document the project’s exact versions in a reproducible build manifest. citeturn15search10turn1search6  
- Use a three-tier test strategy:
  - Unit tests via `scrypto-test` (TestEnvironment) for fast method-level assertions. citeturn1search2turn1search22  
  - Integration tests via a ledger simulator (LedgerSimulator / resim-style transaction tests) for end-to-end manifests and auth flows. citeturn15search6turn1search6  
  - “Contract-to-wallet” tests where the build produces conforming manifest stubs and validates wallet previews against expected transaction classifications. citeturn4search3turn4search7  
- For off-ledger software, adopt the Radix Engine Toolkit (RET) in the appropriate language wrapper; use it for manifest parsing, transaction construction, decompilation, and static validation (including bucket semantics validation and header checks). citeturn11search2turn11search4turn11search5  
- Use the dApp Toolkit for wallet connection rather than hand-rolled protocols; it wraps wallet + gateway SDKs and provides consistent UX patterns (connect button). citeturn1search5turn1search21  
- Treat protocol upgrades as build-time gates: maintain CI jobs that compile/test against Stokenet and Mainnet target versions, and pre-validate constraints like required rustc toolchain limits for the engine version. citeturn8search2turn7search2  

**Rationale**

Radix’s stack is unusually “full-stack”: the quality of a dApp depends not just on blueprint correctness but also on the transactions it proposes and how wallets interpret them. Tooling like RET is explicitly designed to prevent each integrator from implementing SBOR and transaction-level semantics in an ad hoc, security-critical way. citeturn11search1turn8search6  

**Concrete implementation patterns**

Pattern: “CI pipeline that matches the network.” A generalized pipeline template should include (at minimum):

- Lock compiler + CLI versions matching the target protocol update (e.g., a Cuttlefish-compatible Scrypto release and compatible rustc). citeturn8search2  
- Run unit tests (`scrypto-test`) and transaction-based integration tests (manifest builder + simulator). citeturn1search2turn15search6  
- Compile and statically validate any generated manifests (e.g., with `rtmc` and/or RET static validation). citeturn15search10turn11search4  
- Enforce metadata standards (wallet display + verification metadata) as lint rules in CI for all resources/components you publish. citeturn3search6turn9search11  

Pattern: “RET static validation in frontends/integrations.” The TypeScript RET library explicitly supports static transaction validation including manifest semantic checks (e.g., bucket reuse, tip/cost unit limits, epoch window checks, network matching). citeturn11search4turn7search16  

**Trade-offs**

- Retaining compatibility across protocol updates can constrain your dependency graph (e.g., pinned Rust versions) and complicate developer onboarding, but avoids “works-on-my-machine” failures and production deployment mismatches. citeturn8search2  
- Wallet-first UX reduces the need for custom signing infrastructure but couples your UX to wallet capabilities such as conforming manifest classifications; complex bespoke transaction flows may become “non-conforming” and harder for users to safely review. citeturn4search3turn4search7  

### UX, wallet integration, fee models, and account abstraction

**Best-practice guidelines**

- Build **conforming transaction manifest stubs** whenever possible; Radix explicitly recommends this so wallets can summarize transactions in focused, high-confidence UI. citeturn4search7turn4search3  
- Treat metadata as part of UX and security: implement wallet display metadata (names, icons, descriptions) and verification metadata (dApp definition linking) so the wallet can attribute actions to your dApp and guide users about trust relationships. citeturn3search6turn9search11turn3search15  
- Use subintents/pre-authorizations for advanced UX: delegated fees, multi-party flows, and “submitter pays” patterns; follow the documented constraints (no fee locks in subintent stub, required yields, explicit asserts). citeturn3search1turn8search0turn4search14  
- Use AccountLocker and deposit patterns instead of ad hoc deposit rules for airdrops or deferred claims; link lockers to your dApp Definition so trusted wallets can surface claims and notifications. citeturn3search15turn15search4turn9search11  
- Understand and communicate fee semantics: fees reflect burden (compute + storage + royalties); committed failures still cost fees up to failure. Avoid UX that encourages users to “retry until it works” without explanation. citeturn0search0turn0search8turn0search24turn3search0  
- For “account abstraction”-like UX (recovery, multi-factor authorization, passwordless login), adopt smart accounts + Personas/ROLA: use Access Controllers for multi-factor recovery and ROLA to verify Persona-based logins off-ledger. citeturn8search5turn2search2turn9search20  

**Rationale**

Radix’s wallet-centric UX is not a mere “client integration detail”; it is a primary safety layer. Wallets can only present strong guarantees when manifests are (a) interpretable and (b) supported by metadata standards and transaction-type patterns. Similarly, fee delegation and multi-actor flows are better expressed using first-class transaction model features (subintents) rather than bespoke relayer schemes. citeturn0search4turn4search3turn8search0turn3search6  

**Concrete implementation patterns**

Pattern: “Conforming swap with explicit guarantees.” Radix’s knowledge base describes the concept that a manifest can enforce “trade guarantees” (e.g., the transaction only occurs if at least X tokens are received), improving user protection against front-running or malicious swap logic. citeturn4search22turn9search20  

Pattern: “Delegated fee payment via subintent.” The subintent docs describe that self-contained subintents can be used for delegated fee payment where a dApp pays to submit a transaction containing the user’s signed subintent. citeturn4search14turn8search0  

Pattern: “AccountLocker for deposits/airdrops.” Radix’s account deposit patterns guide recommends a single locker per dApp and verified linking through dApp Definition metadata (`dapp_definition`, `claimed_entities`, and `account_locker`). citeturn3search15turn15search4turn9search11  

**Trade-offs**

- “Conforming manifest” patterns constrain transaction construction; non-conforming flows may still be possible but risk confusing users, increasing consent errors, and reducing wallet UI support. citeturn4search3turn4search7  
- Fee delegation improves onboarding but shifts costs to dApps and can create abuse surfaces (spam, griefing) unless paired with pre-authorization scopes, rate limiting, and clear business rules. citeturn8search0turn3search1  
- Rich metadata improves UX but is public; do not encode secrets or sensitive identifiers in on-ledger metadata. citeturn9search2turn3search17  

### Governance, upgradeability, and backward compatibility

**Best-practice guidelines**

- Separate:
  - **Protocol governance/operations** (validator readiness signaling, coordinated protocol updates, network parameters), from  
  - **Application governance** (who can upgrade components, change fees/royalties, manage treasuries). citeturn8search3turn15search1  
- For application governance, model authority explicitly through badges and AccessRules; adopt well-defined user/admin badge patterns rather than implicit address checks. citeturn1search0turn1search8  
- Prefer **immutable application logic** where feasible (deploy new packages for major upgrades) and use structured migration patterns; if you require mutable control, constrain it with multi-factor auth and time delays. citeturn8search5turn15search1  
- Use owner role variants intentionally:
  - `OwnerRole::Fixed` for strong immutability and minimal trust surface,  
  - `OwnerRole::Updatable` for governed upgrade paths, and ensure the owner role is itself governed securely (account/access controller). citeturn15search1turn8search5  
- For integrator/backward compatibility, prefer “LTS” surfaces where offered:
  - LTS toolkit interfaces,  
  - Core API `/lts/*` for exchange-style flows,  
  - and explicit Olympia→Babylon mapping guidance for legacy addresses. citeturn11search17turn13search2turn7search2  
- For network upgrades, follow the documented readiness signaling + epoch window approach; integrate update checks into CI/CD and maintenance workflows. citeturn8search3turn1search3turn2search8  

**Rationale**

Governance and upgradeability are inseparable from trust. Radix provides flexible on-ledger authorization structures, but that flexibility can undermine safety if upgrade powers are poorly scoped or if users cannot reason about the mutability of critical parameters (e.g., royalties, admin roles). At the protocol layer, coordinated update mechanisms exist to prevent network splits and ensure simultaneous enactment once stake thresholds are met. citeturn8search3turn1search13turn15search15  

**Concrete implementation patterns**

Pattern: “versioned implementation + governance-controlled router.” Publish Package v2, instantiate v2 components, then update a registry/router component (governed by a multi-factor owner role) so new calls route to v2 while v1 is frozen or deprecated.

```mermaid
flowchart TB
  U[Users / dApp UI]
  R[Router Component (stable address)]
  V1[Implementation v1]
  V2[Implementation v2]
  M[Migrator / Admin tooling]

  U --> R
  R --> V1
  M --> V2
  M --> R
  M -. optional freeze .-> V1
```

This pattern is a generalization of badge-governed role updates and owner-role mutability described in Radix docs; it should be paired with clear metadata and user communication. citeturn15search1turn1search8turn3search6  

Pattern: “lock royalty/metadata mutability.” Radix’s role assignment examples show explicit “locker” roles and role-updater denial to make configuration immutable once finalized (e.g., metadata locker updater set to `deny_all`, royalty updater roles denied). The same pattern applies to governance-critical settings. citeturn15search7turn15search11turn15search15  

**Trade-offs**

- Upgradeable routing adds complexity and the risk of governance capture; fixed ownership reduces that risk but forces token migrations and may strand older state without dedicated migration tooling. citeturn15search1turn8search5  
- Backward-compatible integrator APIs (LTS surfaces) reduce operational risk for custodians/exchanges but may lag new protocol capabilities; teams may need dual paths (LTS for core flows, advanced APIs for richer features). citeturn13search2turn11search17  
- Network protocol updates require operator coordination; missing readiness signaling or delayed upgrades can degrade consensus participation or operational reliability for validators. citeturn8search3turn1search3  

### Economic design: tokenomics, incentives, and fee structures

**Best-practice guidelines**

- Model economic flows in three layers:
  - Network-level: XRD for staking security and transaction fees, with fee mechanics designed to limit spam. citeturn9search9turn0search0  
  - Application-level: resource issuance, incentives, governance tokens, and treasury flows.  
  - Developer-level: royalties for reusable packages/components, plus off-ledger business revenue if applicable. citeturn3search2turn3search0turn6view0  
- Account for fee and failure semantics in UX and economics: committed failures still pay fees, and royalties are charged as part of transaction fees (in XRD, possibly set as approximate USD equivalent via protocol-defined multipliers). citeturn0search8turn15search15turn0search24  
- Use royalties deliberately:
  - Keep royalties stable and minimize “surprise changes” (avoid leaving royalties updatable indefinitely; document when/how royalties may change). citeturn15search15turn3search8  
  - Choose XRD vs approximate USD equivalent based on desired end-user cost stability, recognizing that USD-equivalent multipliers are changed via protocol updates. citeturn15search15turn8search3  
- Incorporate explicit anti-exploit incentives: Radix docs explicitly demonstrate governance/voting exploits when proof amounts can be duplicated; economic designs should “spend” voting power or escrow it with claims. citeturn14search0  
- For validator/staking-aware designs, align timing and reward expectations to epoch structure (epochs aim for ~5 minutes; emissions occur each epoch and the protocol aims for roughly 300m XRD per year, per knowledge base). citeturn0search3turn4search2turn0search19  

**Rationale**

Radix’s economic primitives—fee costing, tips, royalties, emissions—are intertwined with protocol update governance (e.g., changing USD royalty multipliers via protocol updates). An economic section in a best practices document should therefore teach both design-time modeling and operational-time “parameter change” readiness. citeturn15search15turn8search3turn0search0  

**Concrete implementation patterns**

Pattern: “royalty configuration with locked roles.” Radix documents explicit roles for setting/locking/claiming royalties; create conservative defaults and lock updaters when you want immutability. citeturn3search8turn15search15  

Pattern: “fee delegation to solve double onboarding.” Ecosystem sources highlight that pre-authorizations can allow dApps to pay fees for users (avoiding the need for users to obtain XRD before using an app), though implementations should include abuse controls. citeturn8search0turn3search5  

**Trade-offs**

- Royalties support sustainable developer ecosystems, but even Radix docs warn that leaving royalties updatable may discourage usage due to trust concerns. citeturn15search15  
- USD-equivalent royalties reduce real-world price volatility but require acceptance that protocol updates adjust the conversion constant; this adds governance coupling to your business model. citeturn15search15turn8search3  
- Fee delegation improves onboarding but shifts cost and spam risk to the dApp; mitigate with scoped pre-authorizations, rate limits, and possibly identity gating for costly flows. citeturn8search0turn2search2  

### Compliance and privacy: KYC/AML, data privacy, and zero-knowledge options

**Best-practice guidelines**

- Treat on-ledger data as public by default; do not store PII, secrets, or regulated identifiers in component state or metadata. Use off-ledger storage with on-ledger references (hashes, pointers) when needed. citeturn9search2turn3search17  
- Use Personas + ROLA for privacy-preserving login and account-control proofs: ROLA verifies proofs of control of Identity or Account components, enabling passwordless login without requiring an on-ledger transaction for login. citeturn2search2turn9search20turn2search30  
- Implement compliance as an **optional capability**, not a protocol assumption:
  - Radix has described an optional single sign-on KYC approach (Instapass) intended for dApps/services that choose to require KYC/AML compliance, while also stating it is not required to use the permissionless network. citeturn10search9turn9search1turn9search16  
  - Newer ecosystem initiatives include identity/privacy collaborations (e.g., idOS-related announcements). citeturn9search3turn9search0turn9search18  
- For “zero-knowledge options,” treat ZK as an integration pattern:
  - Use ZK proofs for selective disclosure or compliance proofs (e.g., “over 18,” “KYC’d”) while keeping raw PII off-ledger; verify proofs either on-ledger (if feasible in your blueprint constraints) or off-ledger with cryptographic attestation and on-ledger anchoring. (This is general cryptographic practice; Radix sources highlight identity/privacy initiatives but do not document a base-layer private transaction system as the default Babylon mode.) citeturn9search3turn10search0turn2search2  

**Rationale**

Immutability and transparency can conflict with privacy regulations if misused. Radix’s identity stack (Personas/ROLA) suggests a direction where user-centric identity proofs can minimize data duplication and improve compliance ergonomics. At the same time, KYC/AML is application- and jurisdiction-specific; a best-practices document should stress modular compliance: build flows that can be compliant where needed without turning the entire dApp into a permissioned system. citeturn2search2turn10search9turn9search3  

**Concrete implementation patterns**

Pattern: “KYC gating via AccessRule.” Implement regulated entry points as methods restricted by `rule!(require(kyc_badge_resource_address))`, where the KYC badge is minted/attested by your compliance provider or identity partner; keep the “permissionless mode” as a separate method path or separate component. This leverages Radix’s documented AccessRule and badge patterns. citeturn1search8turn1search0turn9search1  

Pattern: “ROLA-backed session.” dApp backend issues a login challenge; user signs with Persona/Identity; backend verifies via ROLA against ledger state; backend returns an application session token without persisting sensitive identity documents. citeturn2search2turn2search10turn2search30  

**Trade-offs**

- Compliance providers and identity partners introduce centralization and availability risk; mitigate with multiple providers, credential portability, and clear user consent boundaries. citeturn9search3turn10search9  
- On-ledger gating tokens can leak behavioral metadata (who is KYC’d); consider ZK-style credential proofs or off-ledger checks for sensitive contexts, at the cost of higher engineering complexity. citeturn9search3turn2search2  

### Deployment and operations: node ops, monitoring, backups, and disaster recovery

**Best-practice guidelines**

- For production integrations, run a dedicated full node connected to mainnet, and run a second **backup node for hot-swapping** if the primary fails (explicitly recommended for exchange-style integrations). citeturn13search2  
- Automate node deployment with the supported `babylonnode` CLI and choose a consistent operating mode (Docker is recommended as straightforward); keep configuration reproducible and audited. citeturn13search1turn2search8  
- Secure node identity:
  - back up the node keystore file immediately,  
  - avoid leaking keystore passwords in plaintext configs,  
  - restrict access to Core/System/metrics endpoints via auth and network policy. citeturn13search1turn0search5  
- Monitoring:
  - Use System API health endpoints for liveness/sync status,  
  - export Prometheus metrics from node and gateway services, and visualize/alert via Grafana dashboards. citeturn0search5turn0search9turn0search1  
- Protocol update readiness operations:
  - upgrade node software ahead of the coordinated update window,  
  - verify status via system health and readiness signal endpoints,  
  - signal readiness on-ledger as required for validators. citeturn8search3turn1search3turn2search8  
- Backups and disaster recovery:
  - Back up secrets (keystores) and configuration separately from ledger data,  
  - rehearse restore procedures on fresh hosts,  
  - maintain infrastructure-as-code for fast redeploy.  
  For ledger database snapshots, Radix community resources exist for restoring from snapshots, but a best-practices document should label them as community guidance and require validation in your own environment. citeturn13search1turn2search0  

**Rationale**

Running Radix infrastructure is closer to running a production database + consensus client than “just calling an RPC endpoint.” The official docs emphasize authenticated APIs, metrics endpoints, structured setup and update flows, and validator-specific ownership and key management patterns. Operational negligence (lost keystores, unmonitored sync lag, untested upgrades) becomes a systemic reliability and security risk. citeturn13search1turn0search1turn12search0turn8search3  

**Concrete implementation patterns**

Pattern: “standard monitoring stack.”

entity["organization","Prometheus","metrics monitoring system"] scrapes node/gateway metrics endpoints and stores time-series data; entity["organization","Grafana","observability dashboard tool"] provides dashboards and alert rules. Radix’s node docs explicitly describe this pairing and provide setup guidance. citeturn0search1turn0search9turn0search5  

Pattern: “update readiness runbook.”
- Upgrade binaries/containers. citeturn2search8  
- Validate health + sync. citeturn0search5turn13search1  
- For validators: ensure readiness signaling is correct for target protocol version. citeturn8search3turn1search3  

**Trade-offs**

- Running full nodes and monitoring infrastructure increases cost/complexity (SRE skills required), but reduces dependency on third-party infra and improves integrity and latency control. citeturn13search2turn4search17  
- Aggressive firewalling and auth protects endpoints but complicates automation; mitigate with secure secrets management and least-privilege access for automation agents. citeturn13search1turn0search5  

### Ecosystem growth: community engagement, documentation, and programs

**Best-practice guidelines**

- Treat good metadata and dApp Definitions as ecosystem UX infrastructure: publish clear names/icons and verified links so wallets and explorers can correctly attribute components/resources to your dApp. citeturn3search6turn9search11  
- Build reusable and well-scoped blueprints/components; modular packages improve testability, upgradability, and security by limiting scope, and align with Radix’s “lego bricks” design goals. citeturn15search17turn6view0  
- Contribute examples and libraries back to the ecosystem:
  - publish well-documented open-source packages,  
  - provide integration snippets for manifests, RDT, RET, and ROLA,  
  - and maintain versioned docs aligned to protocol updates. citeturn1search25turn2search10turn11search2turn8search1  
- Use official funding/acceleration programs where relevant:
  - Radix grants programs document funding for projects building in the ecosystem (e.g., “Booster Grants” up to stated caps). citeturn2search3turn2search23  
- Track evolving governance and decentralization: Radix has published 2026 strategy material emphasizing decentralized provision of critical services and an RFP model for ecosystem participation. A best-practices document should reflect that infra dependencies may shift over time (e.g., gateways run by third parties). citeturn10search6turn0search9  

**Rationale**

Ecosystem growth is partly a technical problem: discoverability, safe composability, “works with wallet” UX, and stable libraries drive developer velocity and user trust. Radix explicitly positions developer royalties and reusable blueprints as incentives for modular, reusable building blocks. citeturn6view0turn15search15  

**Concrete implementation patterns**

Pattern: “documentation-as-a-compatibility contract.” For each release of a package/component, publish:
- protocol update compatibility (e.g., “requires Cuttlefish / Transaction V2”), citeturn8search2turn8search1  
- supported conforming manifest types (for wallet previews), citeturn4search3turn4search7  
- metadata keys and verification links (dApp definition, claimed entities), citeturn9search11turn3search6  
- upgrade/migration policy (whether admin roles are fixed or updatable). citeturn15search1turn8search5  

**Trade-offs**

- Publishing reusable components and docs may reduce competitive advantage, but increases trust, composability, and the chance of ecosystem adoption.  
- Grants and ecosystem funding can accelerate delivery but may impose milestone discipline and public scrutiny; teams should align funding strategy with long-term maintainability.

## Comparative tables for tools, testing, and monitoring

### Tooling and SDKs comparison

| Tool / SDK | Primary use | Typical runtime | Strengths / best-fit scenarios | Versioning & compatibility notes | Primary references |
|---|---|---|---|---|---|
| Radix dApp Toolkit (RDT) | Wallet connection, transaction & pre-authorization requests, connect button UX | Web frontend | “Wallet-first” UX; consistent connect UI; wraps wallet + gateway SDKs | Wallet UX depends on conforming manifests and supported flow types | citeturn1search5turn1search1turn1search21turn4search3 |
| Radix Engine Toolkit (RET) core + wrappers | Off-ledger manifest/tx building, SBOR encode/decode, decompile, static validation, derivations | Frontend or backend (language-dependent) | Reduces need to implement SBOR/tx semantics yourself; supports typed primitives and validation | Multiple wrappers (TypeScript, Python, Swift, C#, Kotlin; docs also reference other bindings); TypeScript wrapper is manually written vs UniFFI in others | citeturn11search2turn11search5turn11search4turn11search7 |
| Gateway API | Ledger state queries + transaction submission | Backend / services | Canonical client interface for state queries and submission; describes outcome semantics (success/failure) | API versioning matters for integrators; treat as contract and monitor version bumps | citeturn3search12turn0search8 |
| Core API + exchange-oriented LTS surfaces | Exchange/custodian integration patterns | Backend | Simplified “LTS” UX for fungible transfers; reduces integration surface | Use LTS where backward compatibility is essential | citeturn13search2turn11search17 |
| ROLA libraries and examples | Off-ledger auth: verify Persona / account-control proofs | Backend (plus example dApps) | Passwordless login + account proof flows anchored in ledger state | Couples backend auth to wallet identity semantics; keep updated with protocol changes | citeturn2search2turn2search10turn2search22 |
| Radix CLIs (`scrypto`, `resim`, `rtmc/rtmd`, `scrypto-bindgen`) | Build/test Scrypto; local simulation; manifest compile/decompile; stub generation | Dev/CI environments | Fast local iteration and reproducible manifest workflows | CLI versions track engine features; pin with protocol update target | citeturn15search10turn1search6turn8search2 |
| `babylonnode` CLI | Node installation, update, auth config, operational tasks | Node hosts | Standardized installation/update workflows for Docker/systemd | Requires secure keystore handling; operator runbooks should include protocol update readiness | citeturn13search1turn2search8turn8search3 |

### Testing and quality strategy comparison

| Layer | Tooling / pattern | What it tests best | What it misses / risks | Primary references |
|---|---|---|---|---|
| Unit tests | `scrypto-test` TestEnvironment | Method/function behavior, invariants, fast feedback | Can miss transaction-level auth/fee/manifest semantics | citeturn1search2turn1search22 |
| Integration tests | Ledger simulator + manifest builder (transaction-based tests) | End-to-end manifests, auth zones, deposits/withdrawals, failure behavior | Still not identical to mainnet runtime for every protocol update unless version-pinned properly | citeturn15search6turn1search6turn8search2 |
| Local manual testing | `resim` | Developer iteration, quick scenario exploration | Manual tests don’t scale; risk of skipping edge cases | citeturn1search6turn4search20 |
| Wallet UX tests | Conforming manifest classification checks | Whether wallet can summarize & preview user intent safely | Non-conforming flows may degrade UX and safety | citeturn4search3turn4search7 |
| Off-ledger correctness | RET decompile + static validation | Bucket semantics, header config, network id, cost limits | Doesn’t replace ledger-state-dependent integration tests | citeturn11search4turn11search2 |
| Security regression | “code hardening” adversarial test suite | Known Radix-specific pitfall classes (e.g., proof cloning misuse) | Does not cover economic attacks unless modeled | citeturn14search0 |

### Node and gateway monitoring solutions comparison

| Monitoring approach | What you measure | Setup complexity | When to choose | Trade-offs | Primary references |
|---|---|---:|---|---|---|
| System API health checks | Sync status, node identity, readiness/protocol status | Low | Minimum viable monitoring; CI/CD smoke checks | Limited time-series visibility; not sufficient alone for validators | citeturn0search5turn13search1turn8search3 |
| Prometheus metrics + Grafana dashboards | Time-series performance, resource usage, service-level indicators; dashboards/alerts | Medium | Recommended baseline for nodes/gateways; validator operations | Requires secure endpoint exposure and alert tuning | citeturn0search1turn0search9turn0search5 |
| Community dashboards (e.g., Grafana dashboard + exporters) | Validator-specific dashboard metrics | Medium | Useful when official dashboards don’t meet needs | Community maintenance risk; verify metric definitions and compatibility | citeturn0search21turn0search1 |
| Redundant node + hot-swap design | Operational continuity rather than a metric | Medium–High | Exchanges/custodians; high availability requirements | Doubling infra costs; operational orchestration | citeturn13search2 |

## Primary sources and further reading

This best-practices synthesis relied primarily on official Radix documentation, official Radix blog/whitepapers, and major official/community repositories, including:

- Radix technical docs on manifests, conforming manifest types, pre-authorizations/subintents, metadata standards, AccessRules/AuthZone, Access Controller, AccountLocker, testing, tooling, node monitoring and protocol updates. citeturn0search4turn4search3turn8search0turn3search6turn1search8turn8search5turn15search4turn1search2turn0search1turn8search3  
- Radix whitepapers and peer-reviewed/academic references on sharded consensus concepts and Radix’s scalability vision (Radix DeFi White Paper; Cerberus-related publications and announcements). citeturn6view0turn6view2turn7search0turn7search11  
- Official repositories such as entity["company","GitHub","code hosting platform"] projects for dApp toolkit, RET, ROLA examples, and node software. citeturn1search1turn11search1turn2search10turn11search14  
- Security audit communications and secure development guidance published by Radix and ecosystem audit partners. citeturn14search5turn14search1turn14search12  
- Ecosystem growth and funding resources (developer grants, ecosystem fund updates, 2026 strategy). citeturn2search3turn2search23turn10search6  
- Compliance/identity materials: ROLA docs, Persona/identity narratives, optional compliance via Instapass, and identity/privacy ecosystem initiatives such as entity["organization","idOS","digital identity consortium"] announcements. citeturn2search2turn10search9turn9search3turn9search0