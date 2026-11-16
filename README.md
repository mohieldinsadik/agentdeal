# AgentDeal (draft v0.1)

An open, neutral format for **contracts, proof of delivery, independent verification and arbitration**
between AI agents of different organisations. It is meant to sit on top of A2A and MCP.

- 📄 [SPEC.md](SPEC.md) — the specification
- 🧾 [schemas/](schemas/) — JSON Schemas for the Contract and the Receipt
- 🧪 [examples/](examples/) — illustrative examples (hashes and signatures are **placeholders** and do not verify)

**Status:** an early draft by one person. Expect mistakes. Please open an issue with criticism,
missing cases, or prior art I should read.

**Not legal advice.** This is a technical record format, not a court.
## Related work

AgentDeal does not replace existing protocols. It tries to fill a gap next to them.

- **A2A (Agent2Agent)**: how agents of different organisations discover each other and exchange tasks, using Agent Cards. https://github.com/a2aproject/A2A
- **MCP (Model Context Protocol)**: how an agent accesses tools and data.
- **AP2 (Agent Payments Protocol)**: payment mandates for agents. AgentDeal only *references* a mandate and never holds funds.
- **Independent trust scoring of agents**: several projects publish trust scores or ratings for agents (for example AgentRisk, AgentScore, Open Ratings). They score an agent; AgentDeal records a specific deal between two agents.
- **Dispute handling for agents**: T54 Labs has reportedly published an "Agentic Risk Standard" for disputes when agents cause financial loss. I have not reviewed it in detail.
- **Research on the gap**: a paper on governance gaps in agent interoperability protocols (arXiv 2606.31498), and a 2026 paper from Microsoft Research, Columbia and Google DeepMind on a "guarantee gap" between probabilistic AI reliability and commercial guarantees.

If you know of prior art that overlaps with this draft, please open an issue. Corrections to this list are welcome.
Licence: Apache-2.0 (see the LICENSE file).
