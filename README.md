# A Clinical Decision Support System for Prescription Risk Screening
### using Graph-LLM Integration

> 🚧 **Status: FYP1 — Proposal Approved, Implementation Not Yet Started**
> Project conception, literature review, and system architecture are
> complete. FYP2 (prototype build and evaluation) begins next semester.
> See [Roadmap](#-roadmap--timeline) below.

**Author:** Xie Wanen · Programme: CST

## 📋 Overview

Cardiovascular patients are frequently prescribed multiple concurrent
medications (polypharmacy), which creates a high risk of dangerous drug
interactions. Current hospital prescription-checking systems only perform
**single-hop name lookups** — comparing a proposed drug against a fixed
list of known-bad combinations. This misses two categories of risk:

- **Unstructured clinical notes** — doctors record observations (e.g. a
  recent lab result, a new diagnosis) as free text, which fixed-table
  systems cannot read
- **Indirect risk** — a drug can look safe in isolation but become
  dangerous in combination with a specific lab result or secondary
  diagnosis; flat tables cannot connect these scattered pieces of
  information

This project proposes a system that keeps **language parsing** and
**clinical fact-finding** strictly separate, so the parts of the system
that could hallucinate (the LLM) are never the ones making medical
claims.

## 🎯 Aim & Objectives

**Aim:** Build and validate a prototype that combines a lightweight LLM
parser with a Neo4j knowledge graph to detect dangerous drug combinations
and compute a MERIS risk score, using doctor input and synthetic patient
data.

1. **Text-reading module** — an open-source LLM under 10B parameters,
   using prompt engineering only (no additional training), converts
   clinical notes into structured Cypher query parameters
2. **Neo4j knowledge graph** — populated with drug interaction data from
   **DrugBank**, disease-warning data from **SIDER 4.1**, and synthetic
   patient histories from **Synthea**
3. **MERIS risk scoring module** — a Python script that computes the
   Medication Risk Score using the same scoring rules and cutoffs
   published by Saedder et al. (2016)
4. **Evaluation** — test the full system against a standalone LLM to
   compare drug-risk detection accuracy and fabrication ("hallucination")
   rate

## 🏗️ System Architecture

A 3-layer pipeline, strictly separating language parsing from clinical
reasoning:

| Step | Actor | Action |
|---|---|---|
| 1 | Doctor | Inputs prescription & clinical notes |
| 2 | Python Backend | Receives input, requests parsing |
| 3 | **LLM Parser** | Extracts entities (drug, disease, labs) — *parsing only, no medical judgment* |
| 4 | Python Backend | Compiles Cypher query from extracted entities |
| 5 | **Neo4j Database** | Executes multi-hop path search across drugs, patient history, and lab results |
| 6 | Python Backend | Calculates MERIS risk score from graph search results |
| 7 | **LLM Parser** | Formats the score and evidence path into a natural-language alert |
| 8 | Doctor | Sees the high-risk warning, with the full evidence path traceable back to the graph |

**Layer 1 — Parsing (LLM):** text → structured variables. No medical
judgment happens here.
**Layer 2 — Reasoning (Neo4j):** multi-hop Cypher search across drugs,
patient history, and lab results — finds both direct and indirect risks.
**Layer 3 — Output (Python + LLM):** MERIS score → plain-language alert,
with every claim traceable back to a real graph edge.

### Why this design

- **Local, lightweight LLM (<10B params):** patient data stays
  on-premise rather than going to an external API
- **The LLM never makes medical claims** — it only parses input text and
  formats the final output, directly addressing the hallucination risk
  documented in medical LLM benchmarks (e.g. Med-HALT, Pal et al., 2023)
- **Neo4j knowledge graph:** drug interactions are inherently relational,
  which a graph traverses far more naturally than a flat table lookup
- **DrugBank + SIDER 4.1:** established, publicly available drug
  interaction and disease-warning databases
- **Synthea:** synthetic patient data — avoids handling real patient
  records during development
- **MERIS (Saedder et al., 2016):** a validated medication risk scoring
  algorithm, rather than an ad-hoc scoring scheme

## 🔍 Worked Example: Hyperkalemia Risk from Lisinopril

A concrete case the system is designed to catch, illustrating the
multi-hop reasoning a flat-table lookup would miss:

```
Lisinopril (new prescription)
        │ «inhibits»
        ▼
   Aldosterone (hormone)
        │ «promotes»
        ▼
Potassium Excretion (physiological process)
        │ «negatively affects»
        ▼
Serum Potassium: 5.8 (existing patient lab result — already high)

→ Conflict detected: Lisinopril inhibits aldosterone, which reduces
  potassium excretion — a severe risk given this patient's already
  elevated serum potassium.
→ Calculated risk score: 0.92 (Extreme Risk – Hyperkalemia)
```

## 🧪 Scope & Limitations

This is explicitly a **software design proof-of-concept**, not a
clinical-ready tool:

1. **Synthetic data only** — all patient records (Synthea) and drug data
   (DrugBank/SIDER 4.1) are synthetic or public; no real patient data is
   used, and real hospital records are messier than synthetic ones
2. **Scoped to three risk categories** in cardiovascular patients:
   drug–disease cross-reactions (e.g. non-selective beta-blockers in an
   asthma patient), drug–drug interactions (e.g. Warfarin + Aspirin), and
   drug–lab conflicts (e.g. ACE inhibitors with recent hyperkalemia)
3. **Requires reasonably structured input** — very long or illegible
   clinical notes may reduce LLM parsing accuracy

## ⚖️ Ethical Considerations

All patient data comes from Synthea (synthetic) and all drug data from
public databases (DrugBank, SIDER 4.1) — no real patient information is
used. This means the project does not require data ethics approval, but
it also means results cannot be generalized to real-world clinical
accuracy without further validation on real (properly approved) data.

## 🧪 Planned Evaluation

Prescription test cases will be run twice: once through the full
Graph-LLM system, once through a standalone LLM with no graph access.
Comparing the two will measure:

- How often the final warning includes fabricated ("hallucinated") facts
- Whether the computed MERIS score matches hand-calculated correct
  answers for each test case

## 🛠️ Tech Stack

- **Language:** Python
- **Graph database:** Neo4j (Community Edition)
- **LLM:** open-source, local, <10B parameters
- **Data sources:** DrugBank, SIDER 4.1, Synthea
- No special hardware required — runs on a normal personal computer

## 🗺️ Roadmap & Timeline

**FYP1 (current semester)**
- [x] Confirm supervisor
- [x] Finalize scope & literature review
- [x] Draft system architecture
- [x] Proposal submission & presentation

**FYP2 (next semester)**
- [ ] Prepare Neo4j database (DrugBank + SIDER 4.1 + Synthea ingestion)
- [ ] Prompt-engineer the LLM parsing layer
- [ ] Develop Python backend / Cypher query execution
- [ ] Implement MERIS scoring algorithm
- [ ] System integration and testing
- [ ] Baseline vs. Graph-LLM contrastive evaluation
- [ ] Final FYP report & thesis submission
- [ ] Final prototype demonstration & presentation

## 📚 Key References

- Saedder, E. A., et al. (2016). Detection of Patients at High Risk of
  Medication Errors: Development and Validation of an Algorithm.
  *Basic & Clinical Pharmacology & Toxicology*, 118(2), 143–149.
- Pal, A., Umapathi, L. K., & Sankarasubbu, M. (2023). Med-HALT: Medical
  Domain Hallucination Test for Large Language Models.
- Edge, D., et al. (2025). From Local to Global: A Graph RAG Approach to
  Query-Focused Summarization.
- Nygren, S., et al. (2026). RAG-based architectures for drug side
  effect retrieval using compact LLMs. *Scientific Reports*.
- Wishart, D. S., et al. (2018). DrugBank 5.0: A major update to the
  DrugBank database for 2018. *Nucleic Acids Research*, 46(D1).
- Lin, Z., et al. (2023). Risk detection of clinical medication based on
  knowledge graph reasoning. *CCF Transactions on Pervasive Computing and
  Interaction*, 5(1), 82–97.

*(Full reference list and literature review synthesis matrix available
in `docs/proposal.pdf`.)*

## 📄 Documentation

- `docs/Proposal.pdf` — full FYP1 proposal
- `docs/ProposalPresentation.pdf` — FYP1 presentation slides

## ⚠️ Disclaimer

This is an academic Final Year Project. It uses only synthetic patient
data (Synthea) and public drug interaction data (DrugBank, SIDER 4.1) —
no real patient data is used. It is a research prototype, not a
certified medical device or clinical tool.
