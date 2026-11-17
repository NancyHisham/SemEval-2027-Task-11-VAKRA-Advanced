# 10-Minute Presentation Slides Outline (Saturday, Nov 21, 2026)

**Course:** Advanced NLP (Prof. Nagwa El-Makky)  
**Format:** 10 minutes presentation + 5 minutes Q&A  
**Required Topics Covered (per Nov 16 email):**
1. The Problem (SemEval-2027 Task 11: VAKRA-Advanced)
2. Brief Summary of the Baseline Paper (*VAKRA*, Naik et al., 2026)
3. Concise SOTA Comparison (5+ Recent Papers)
4. Our Core Idea & Architecture (**PV-VAKRA**)
5. Techniques & Neural Models Used
6. Dataset(s) & Preprocessing
7. Quantitative Performance Evaluation Metrics & Baseline Targets

---

## Slide 1: Title Slide (0:00 – 0:30)
* **Title:** Plan-and-Verify Agent with Schema-Grounded Tool Retrieval and Policy Guardrails for VAKRA-Advanced (SemEval-2027 Task 11)
* **Team Members:** [Team Member 1] & [Team Member 2]
* **Supervisor:** Prof. Nagwa El-Makky

---

## Slide 2: The Problem — Multi-Hop, Multi-Source Tool Calling (0:30 – 2:00)
* **Real-World Challenge:** Enterprise AI agents cannot just make isolated 1-step API calls; they must:
  1. Select tools from **thousands of candidate APIs** (**Subtask 2: Tool Selection**).
  2. Compose sequential/nested API pipelines (**Subtask 1: API Chaining**).
  3. Reason across **3 to 7 dependent hops** where output JSON fields parameterize downstream tools (**Subtask 3: Multi-Hop API Reasoning**).
  4. Combine **structured APIs** with **unstructured document RAG** while obeying **strict natural-language tool-use policies** (**Subtask 4: Multi-Hop, Multi-Source Reasoning**).
* **Illustrative Example:** Finding a lead auditor's name in a compliance PDF $\rightarrow$ resolving their employee/department ID via HR lookup API $\rightarrow$ checking policy rules before querying restricted financial/payroll endpoints.

---

## Slide 3: Baseline Paper Summary — VAKRA (Naik et al., 2026) (2:00 – 3:45)
* **Paper:** *VAKRA: Evaluating Multi-Hop Reasoning Across APIs and Retrieval Under Tool-Use Policies* (IBM Research, `arXiv:2608.12282`).
* **Key Contributions:**
  * **Executable Environment:** 8,000+ locally hosted Python APIs across 62 domains + domain-aligned RAG corpora.
  * **Trace-Level Replay Evaluator:** Re-executes predicted trajectories against live backend databases rather than relying on static string matching or LLM-as-a-judge.
* **Key Baseline Findings (Standardized ReAct Harness):**
  * Single-hop query-aligned APIs: **70.4%** accuracy.
  * Compositional Business Intelligence APIs: drops to **50%–51%**.
  * Multi-hop chains (3–7 steps): drops by **>50%**.
  * Policy-constrained / unanswerable queries: collapses to **2.4%**.
* **Core Insight:** Failures are rarely syntax errors; they stem from **entity disambiguation across sources**, **schema mapping**, and **policy instruction-forgetting**.

---

## Slide 4: Concise SOTA Survey & Research Gap (3:45 – 5:15)
* **6 Recent Papers Surveyed (2024–2026):**
  1. **VAKRA (2026):** 8,000+ APIs + RAG + Policies; greedy ReAct struggles on depth & policies.
  2. **Live API-Bench (EACL 2026):** 2,500+ BIRD-SQL APIs (SLOT/SEL/REST); no unstructured RAG or policies.
  3. **ToolLLM / ToolBench (ICLR 2024):** Dense Retriever + DFSDT tree search over 16k RapidAPIs; flat retrieval misses intermediate bridge tools.
  4. **AnyTool (ICML 2024):** Hierarchical Category$\rightarrow$Tool retriever + self-reflection; high multi-agent overhead, no cross-source entity grounding.
  5. **$\tau$-bench (2024):** Policy-constrained retail/airline dialogues; small toolsets (<20 tools).
  6. **Gorilla (NeurIPS 2024) & Re-Invoke (EMNLP 2024):** Synthetic query rewriting & AST execution; single-turn focus.
* **The Research Gap:** No existing system jointly addresses (a) **schema-graph retrieval of intermediate bridge APIs**, (b) **explicit entity linking between RAG text and SQL-backed APIs**, and (c) **pre-execution policy verification**.

---

## Slide 5: Our Idea — The PV-VAKRA Architecture (5:15 – 7:00)
* **Core Hypothesis:** Replacing greedy ReAct with a **4-Stage Plan-and-Verify Pipeline** will directly eliminate VAKRA's three main failure modes:
  1. **Stage 1 — Query & Policy Decomposer:** Splits complex queries into atomic sub-goals $\{q_1, \dots, q_m\}$ and parses policy $\mathcal{P}$ into formal pre-conditions $\{c_1, \dots, c_L\}$.
  2. **Stage 2 — Schema-Graph Hybrid Retriever:** Combines BM25 + Dense Embeddings with 1-hop expansion on a pre-computed **API Input-Output Compatibility Graph $\mathcal{G}(\mathcal{T}, \mathcal{E})$** to discover intermediate bridge APIs.
  3. **Stage 3 — Entity-Grounded DAG Controller:** Maintains a structured **Entity Scratchpad $\mathcal{M}_k$** that resolves unstructured text mentions into canonical database IDs before downstream calls.
  4. **Stage 4 — Pre/Post-Execution Policy & Schema Guardrail:** Audits every proposed tool call $(t_k, a_k)$ against policy $\mathcal{P}$ and schema types; triggers local backtracking or safe abstention on restricted queries.

---

## Slide 6: Models, Techniques & Implementation Plan (7:00 – 8:15)
* **Pretrained Models Used:**
  * *Retriever & Reranker:* `BAAI/bge-large-en-v1.5` (or `Qwen3-Embedding`) + BM25 + `bge-reranker-v2-m3`.
  * *Agent Backbone LLMs:* Open-weight tool-calling LLMs (`Qwen2.5-14B/32B-Instruct`, `Llama-3.3-70B-Instruct`).
  * *Stretch Goal:* Direct Preference Optimization (DPO) on `Qwen2.5-7B-Instruct` using verified vs. policy-violating execution traces.
* **Original Modules Implemented by Our Team:**
  1. Automated **API Input-Output Compatibility Graph Builder** across all 8,000+ Python docstrings/schemas.
  2. **Entity-Grounded DAG Executor** with structured cross-source scratchpad memory.
  3. **Policy Entailment Verifier** & backtracking controller.

---

## Slide 7: Datasets & Quantitative Evaluation Metrics (8:15 – 10:00)
* **Dataset (`ibm-research/VAKRA`):**
  * **8,000+ executable Python APIs** across **62 enterprise domains** (derived from BIRD-SQL, Live API-Bench, Wikidata5M).
  * **Aligned Document Collections** (ClapNQ + enterprise docs).
  * **664 Multi-Hop Multi-Source Queries** (420 unconstrained + 244 policy-constrained, 90% IAA).
* **Quantitative Evaluation Metrics:**
  1. **Trajectory Execution Accuracy (`Exec-Acc`):** Live database/retrieval state equivalence via VAKRA's replay engine.
  2. **Final Answer Accuracy & Token-Level F1.**
  3. **Tool Selection Recall@$K$ & F1** (Subtask 2, out of 150+ tools per slice).
  4. **Policy Compliance Rate & Abstention F1** (Subtask 4, targeting the **2.4%** baseline bottleneck).
  5. **Efficiency:** Average tool calls and token cost per solved trajectory.
