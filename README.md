# AI-assisted Molecular Docking Triage Platform

> A triage workbench for early-stage drug discovery: it ranks compounds by binding affinity, combines the ranking with ADMET and interaction analysis, and helps researchers decide **which molecules to look at first**. Every number is traceable.

> [!NOTE]
> **Source code is private.** This repository is a showcase of a platform under active development. Demo available upon request.

**中文簡介**：為小型生技實驗室打造的 AI 分子對接分流平台。對接、ADMET 與交互作用分析全部是確定性計算，AI 只負責解讀結果；所有驗證都使用公開基準資料集，先寫死判讀標準再跑數據，結果不好也照實揭露。原始碼不公開，可提供 demo。

---

## Design principle: AI interprets, it does not compute

- Docking, ADMET, interaction analysis and ranking are **all deterministic computations**. None of them passes through an LLM.
- The LLM only interprets results when the user asks, and every answer shows its data sources. If the model suggests a compound it has not actually verified, the interface flags it.
- A **local model** is used by default, so data never leaves the deployment environment.
- Reports record methodology parameters, engine versions and random seeds, so every run is **reproducible and auditable**.

## When to use it, and when not to

| ✅ Suitable for | ❌ Not suitable for |
|---|---|
| Predicting binding poses and interactions in a known or hypothesised pocket | Reliably picking active compounds out of a large library (see VP-2) |
| Triaging compounds on affinity, ADMET and interactions together to set a review order | Replacing wet-lab validation or medicinal chemistry judgement |
| Producing reproducible, auditable reports | Using AlphaFold-predicted structures as the basis for poses (see VP-3) |

---

## Validation: success criteria fixed before running the data

Every validation uses **public benchmark datasets**. Success criteria are written down and version-controlled before any data is run, and unfavourable results are reported as they are.

| ID | What was tested | Result |
|---|---|---|
| **VP-1** Pose accuracy | Redocking; success = RMSD < 2 Å | Astex Diverse Set **64.7%**, PoseBusters **51.4%**, on par with AutoDock Vina (57.6% / 52.3%) and the commercial GOLD (67.1% / 51.2%) as reported in the PoseBusters paper. Re-checked in CI on every push |
| **VP-2** Virtual screening enrichment | DEKOIS 2.0: 24 targets, 29,760 molecules | Mean AUROC **0.62**, consistent with the literature, but early enrichment is no better than a molecular-size baseline. **Per the pre-set criteria, no virtual screening capability is claimed** |
| **VP-3** AlphaFold structures | PoseBusters, 396 pairs: the same ligand docked into the AlphaFold model and the crystal structure | AlphaFold **11.4%** vs crystal 52.3% → in the product, AlphaFold structures are reference-only and always carry a warning |
| **VP-5** Pocket prediction | Pocket prediction tool vs blind docking | Failed the pre-set threshold → **not integrated into the product** |

Further validation of dual-engine agreement and cross-docking scenarios has been carried out; details are not yet public.

---

## Core features

- **Batch docking**: manual entry, CSV upload, or molecules drawn in a 2D structure editor
- **3D visualisation**: receptor and ligand in 3D, with drag-and-drop binding-box setup; switch between multiple binding modes per molecule
- **Custom receptor upload**: supports private, unpublished structures for targets under NDA
- **Confidence check card**: automatic checks on structure quality, missing residues near the pocket, a co-crystal ligand redocking positive control, and more
- **ADMET rules**: Lipinski Ro5, QED, hERG, GI absorption, synthetic accessibility and more, clearly labelled as empirical rules rather than predictive models
- **Interaction analysis**: H-bonds, hydrophobic contacts and salt bridges, with a 2D interaction diagram in the report
- **Ask AI**: multi-turn questions about a run, with access to PubMed, a personal knowledge base and patent data
- **Patent lookup**: advanced EPO keyword search, PubChem CID lookup and chemical structure matching, positioned as early novelty leads
- **Formal reports**: PDF reports go through a draft → review → finalise workflow with version history; export to Excel, CSV and PDBQT

## AI governance: the LLM as a system to be managed

| Element | Approach |
|---|---|
| Rules | Version-controlled rule files with an explicit list of prohibited behaviours |
| Tool boundaries | Each tool states when it may and may not be used; high-risk tools are removed entirely in read-only contexts |
| Safeguards | Timeouts, masked error messages, database-level tenant isolation |
| Post-hoc verification | Numbers and compounds in an answer are checked against their sources; failing answers are sent back for a rewrite |
| Evaluation | Behavioural evals run against real APIs, locking in fixes for past bugs |
| Observability | Token usage and cost logged for every call; production errors tracked |

## Built for real deployment

- **On-premise Docker deployment**: data stays inside the customer's network
- **Multi-tenant isolation**: database-level Row-Level Security plus application-level filtering, verified by tests
- **Audit logs** and per-tenant API keys, stored encrypted
- **Automatic resume** of unfinished jobs after a service restart
- **CI test suite** covering tenant isolation, validation workflows and report generation

## Licensing and data compliance

- Every directly used dependency is **BSD, MIT or Apache 2.0 only**; GPL / AGPL / LGPL are excluded so the platform can be licensed commercially
- Several components were replaced for this reason, including OpenBabel, Redis 7.4 and MinIO
- Licences and citation requirements reviewed for each external database (PDB, AlphaFold DB, PubChem, EPO and others)
- Data-use terms reviewed for each LLM service; **free tiers that train on user data are never used for customer data**

---

## Screenshots

**Binding-box setup**: drag in the 3D view to define the docking region

![Binding box setup](binding-box.png)

**Interaction analysis**: Benzamidine bound to trypsin, including the salt bridge with ASP 189A

![Benzamidine–trypsin interactions: salt bridge with ASP 189A](report.png)

## Tech overview

React · FastAPI · PostgreSQL (pgvector) · Prefect · RDKit · AutoDock Vina · LangChain · Docker

---

Built by [Tzu-Yu Huang](https://github.com/Tzuyuu0716) · Related work: [Medi-Agent MCP](https://github.com/Tzuyuu0716/medi-agent-mcp), my master's thesis on a hybrid RAG + MCP medical dialogue system
