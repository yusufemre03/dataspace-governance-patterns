# Challenge 1: Participant Onboarding  

## Pattern 1  

**Name**  
Gaia-X Federated Onboarding  

---

**Problem**  
How can a Gaia-X federation onboard a new organization in a way that:  

1. the applicant proves its legal status by providing at least N verifiable KYB credentials, issued by trusted attesters whose DIDs are included in a quorum-approved Merkle root,  
2. the organization’s signed self-description and connector information are securely linked to its DID and made available through IPFS, and  
3. every approval step is permanently recorded on-chain,  

all without depending on a single central authority?  

---

**Context**  
A Gaia-X data federation works without a central authority. It uses four smart-contract DAOs to manage governance. The blockchain only stores lightweight proofs like Merkle roots and CIDs, while full credentials and JSON-LD metadata are kept on IPFS. All members already have DIDs and use Verifiable Credentials. When a new organization and its connector join, the federation must prove the legal identity, publish trusted metadata, and register endpoints. It is fast, secure, and without centralizing control again.  

---

**Forces**  

- **Legal assurance vs. Decentralization** – New members must provide strong KYB evidence, but the federation rejects a single certificate authority. Trust must come from multiple independent attesters, whose DIDs are themselves governed by a DAO.  
- **Quorum governance vs. Onboarding latency** – Requiring N-of-M votes from anchor members prevents unilateral decisions, but each round of quorum voting slows down onboarding and adds work for busy members.  
- **Transparency & auditability vs. Business confidentiality** – Participants need to review proofs and DAO decisions, but applicants should not expose sensitive corporate data or connector endpoints on-chain.  
- **Minimal on-chain footprint vs. Rich verification evidence** – Storing only CIDs and Merkle roots keeps gas costs low, but verifiers still need enough off-chain data (e.g. via IPFS) to fully check the trust chain.  
- **Scalability vs. Coordination overhead** – As membership grows, more onboarding proposals and votes are required. This risks DAO congestion or voter fatigue.  
- **IPFS reliance vs. Data resilience** – IPFS offers low-cost off-chain storage, but relying on a single content network risks data unavailability if nodes fail.  
- **Root history vs. On-chain bloat** – Keeping full Merkle root history supports auditability, but increases on-chain storage needs over time.  
- **Bootstrap trust vs. Collusion risk** – The federation needs trusted anchors at the start, but they could collude and weaken decentralization from the beginning.  
- **Anchor stability vs. Inclusivity and renewal** – The federation needs stable, trusted anchors for governance, but must also allow rotation and new members to avoid concentration of power.  

---

**Solution**  

1. **Digital Know-Your-Business (KYB).**  
   The applicant must submit at least N Verifiable Credentials (VCs) that prove its legal-entity status. A KYB smart contract verifies two things: first, that each issuer’s decentralized identifier (DID) appears in the Attester Merkle root; and second, that at least N-of-M submitted VCs successfully pass both signature verification and revocation checks.  

   The initial Attester Merkle root is established at network launch by the founding anchor organizations. To avoid circular trust dependency, each founding anchor must provide independent KYB credentials verified by external, well-known attesters. The genesis Attester Merkle root is built from this externally verified anchor set, with a public audit report for transparency.  

   Founding anchors are chosen through an open pre-launch call, where applicants submit externally verified KYB credentials. A public genesis vote by prospective federation members approves the initial set. Each anchor must stake X tokens. Any future change to the attester or anchor set requires approval by either a two-thirds supermajority of anchors or a 51% stake-weighted vote. Malicious anchors can be slashed and replaced during a quarterly rotation window. During these windows, any member may nominate new anchors. Nominations require quorum approval by the Membership DAO, with final inclusion needing a supermajority of existing anchors. Anchors serve renewable one-year terms, with mandatory review at each rotation.  

   To help keep gas costs predictable, each DAO emits an updated Merkle root only once per one-hour epoch. The full issuer metadata is stored off-chain on IPFS. To ensure resilience, issuer metadata is pinned to IPFS and also backed up on redundant storage. Merkle roots are compacted into periodic checkpoint digests to reduce on-chain storage while keeping auditability.  

2. **Publish company self-description.**  
   The company signs a JSON-LD self-description, split into a public part with business facts and an optional private part encrypted using the Gaia-X key-management service for GDPR-sensitive data. Both files are uploaded to IPFS, producing `CID_pub` and `CID_priv`. To enhance data availability guarantees, both CIDs are also pinned to a redundant storage layer. The pair is submitted to the Self-Description Registry DAO. When the quorum approves, the DAO adds a new leaf to its Merkle tree and writes the updated root on-chain at the next batching epoch. If personal data needs to be deleted later, the anchors can issue a Verifiable Credential (VC) that redirects the leaf to a redacted CID and discards the encryption key, making the original data unreachable and meeting the right-to-erasure.  

3. **Obtain quorum-signed participant credential.**  
   The company sends the on-chain KYB-PASSED event and its approved self-description CID to the Membership-Issuer DAO. When the quorum approves, the DAO issues a Participant Verifiable Credential (VC) confirming Gaia-X membership. This VC includes an expiry date and a link to the global Revocation Registry. The DAO stores the VC off-chain and adds its hash to the Membership Merkle tree. The updated Merkle root is written on-chain at the next epoch. Any revocation is handled by setting a flag in the Revocation Registry without changing historical data.  

4. **Register data connector.**  
   The participant deploys a Gaia-X-compatible connector, assigns it a DID, and signs a description of its endpoints and capabilities. The description is uploaded to IPFS, and its CID is included in a self-signed Connector Verifiable Credential (VC) that links the connector DID to the organisation’s DID. The package is submitted to the Connector Registry DAO. When the quorum approves, the DAO adds a new leaf to the Connector Merkle tree, writes the updated root on-chain at the next batching epoch, and makes the connector discoverable. Any party can then resolve the connector DID, retrieve the description via the CID, and verify the signature and registry inclusion without relying on a central authority.  

*Note: Batching adapts to DAO load for timely updates and gas efficiency.*  

---

**Example**  
ACME Analytics, a Spanish SME, applies to join a Gaia-X mobility federation. It submits two KYB Verifiable Credentials (from the commercial registry and its bank), uploads its signed self-description to IPFS, and requests votes. After quorum approval by the KYB, Self-Description, Membership, and Connector DAOs, four new CIDs and Merkle roots are anchored on-chain. Within minutes, any federation member can cryptographically verify ACME’s legal status, metadata, participant credential, and connector, all without relying on a central gatekeeper.  

---

**Resulting Context**  
ACME Analytics now holds four tamper-proof items: KYB check, self-description, participant VC, and connector VC. Their CIDs and Merkle roots are anchored on-chain. Any node can instantly verify the company’s legal identity, metadata, membership, and connector without needing a central gatekeeper. This meets the goals of decentralized trust, auditability, and fast onboarding.  

---

**Rationale**  

The pattern combines four decentralized tools: Merkle roots, CIDs, quorum DAOs, and DIDs, to turn an unverified applicant into a trusted federation member, all without a central gatekeeper.  

- **Separation of concerns (on-chain vs. off-chain)**  
  Big files (VCs, JSON-LD) stay on IPFS; the blockchain only stores cryptographic fingerprints. *Why it works:* keeps gas costs low, while ensuring immutability and global access.  

- **Merkle-root allow-lists**  
  Each DAO keeps one Merkle root to summarize its trusted set (attesters, members, connectors, etc.). *Why it works:* one small hash proves thousands of entries, allowing fast O(1) checks.  

- **Quorum governance**  
  Every key change needs N-of-M anchor votes. *Why it works:* spreads authority, prevents fraud, and moves fast with a small, fixed signer group.  

- **DID-linked credential chain**  
  The applicant’s DID links all proofs (KYB VC, self-description, participant VC, connector VC). *Why it works:* one DID lookup gives all proofs, so peers can automate discovery and access.  

- **Event-driven propagation**  
  DAOs emit events when Merkle roots change; connectors listen and act in seconds. *Why it works:* pushes updates fast without constant polling.  

---
