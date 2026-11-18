# 10-Minute Presentation Slides & Speaker Guide (Saturday, Nov 21, 2026)

**Course:** Advanced NLP (Prof. Nagwa El-Makky)  
**Format:** 10 minutes presentation + 5 minutes Q&A  
**Project Objective:** Build an autonomous tool-calling agent (**PV-VAKRA**) to solve the **two core VAKRA multi-hop reasoning tasks** (Task 1: Multi-Hop Structured API Reasoning [API Chaining, Tool Selection, Multi-Hop Execution] and Task 2: Multi-Hop, Multi-Source Reasoning across APIs + Documents under Tool-Use Policies) and evaluate it on the official VAKRA benchmark.

---

## Slide 1: Title Slide (0:00 – 0:30)
* **Title:** Building a Plan-and-Verify Agent with Schema-Grounded Tool Retrieval and Policy Guardrails for VAKRA-Advanced (SemEval-2027 Task 11)
* **Team Members:** [Team Member 1] & [Team Member 2]
* **Supervisor:** Prof. Nagwa El-Makky

---

## Slide 2: The Problem — Building an Agent for VAKRA-Advanced (0:30 – 2:00)
* **Our Goal:** Build an autonomous agent (`PV-VAKRA`) that solves two core multi-hop reasoning tasks and is evaluated on the VAKRA benchmark ($8,000+$ live Python APIs across $62$ domains + RAG corpora):
  1. **Task 1 — Multi-Hop Structured API Reasoning:**
     * *Subtask 1 (API Chaining):* Sequential & nested Business Intelligence API calls.
     * *Subtask 2 (Tool Selection):* Selecting gold APIs from $150+$ candidates per domain slice.
     * *Subtask 3 (Multi-Hop API Reasoning):* $3\text{--}7$ dependent hops passing intermediate JSON outputs as downstream parameters.
  2. **Task 2 — Multi-Hop, Multi-Source Reasoning under Policies:**
     * *Subtask 4:* Combining structured APIs + unstructured document RAG while strictly obeying natural-language tool-use policies.

---

## Slide 3: Baseline Paper Summary — VAKRA ReAct Agent (Naik et al., 2026) (2:00 – 3:45)
* **Paper:** *VAKRA: Evaluating Multi-Hop Reasoning Across APIs and Retrieval Under Tool-Use Policies* (`arXiv:2608.12282`).
* **Official Baseline Agent:** Standard greedy **ReAct** loop ($\text{Thought}_k \rightarrow \text{Action}(t_k, a_k) \rightarrow \text{Observation}(o_k)$) paired with dense retrieval and system-prompt policy conditioning.
* **Trace-Level Replay Evaluator:** Re-executes predicted trajectories against live SQLite databases and RAG indexes (`Exec-Acc`).
* **Baseline ReAct Bottlenecks:**
  * Single-Hop APIs: **70.4%** $\rightarrow$ Compositional BI Chaining: **50%–51%** $\rightarrow$ Task 1 ($3\text{--}7$ Hops): **>50% drop** $\rightarrow$ Task 2 (Policy Unanswerable): **2.4%**.
  * Fails due to **lack of bridge-tool lookahead**, **RAG-to-SQL entity mismatches**, and **policy instruction-forgetting**.

---

## Slide 4: Concise SOTA Survey of 6 Tool-Calling Agent Architectures (3:45 – 5:15)
* **6 Recent Agent Architectures Surveyed (2023–2026):**
  1. **ReAct Baseline Agent (Yao et al., 2023; Naik et al., 2026):** Greedy 1-step horizon; fails on $3\text{--}7$ hops ($>50\%$ drop) and unanswerable policy queries ($2.4\%$).
  2. **ToolLLM / DFSDT Agent (Qin et al., ICLR 2024):** Dense API Retriever + Depth-First Search Decision Tree; flat retrieval misses intermediate bridge tools.
  3. **AnyTool (Du et al., ICML 2024):** 3-Tier Hierarchical Category$\rightarrow$Tool Agent + Self-Reflective Solver; high multi-agent overhead, no cross-source entity linking or pre-execution policy check.
  4. **ReWOO (Xu et al., 2023/2024):** Decoupled Planner (symbolic DAG `#E1`$\rightarrow$`#E2`) + Worker + Solver; static open-loop plan breaks when intermediate RAG outputs require dynamic entity resolution.
  5. **LATS (Zhou et al., ICML 2024):** Monte Carlo Tree Search (MCTS) + Value Function + Reflection; prohibitive rollout cost ($10\text{--}30\times$ calls per query).
  6. **Re-Invoke (EMNLP 2024) & Gorilla (NeurIPS 2024):** Synthetic query rewriting & retriever-aware fine-tuning; retrieves tools matching explicit prompt sub-queries, missing implicit bridge APIs.

---

## Slide 5: Our Proposed Agent Architecture — PV-VAKRA (5:15 – 7:00)
* **4-Module Agent Pipeline:**
  1. **Module 1 — Intent & Policy Decomposer:** Decomposes query $q$ into atomic sub-goals $\{q_1, \dots, q_m\}$ and parses policy $\mathcal{P}$ into formal rules $\{c_1, \dots, c_L\}$.
  2. **Module 2 — Schema-Graph Hybrid Tool Retriever (for Task 1):** Combines BM25 + Dense Embeddings with 1-hop expansion on a pre-computed **Schema Compatibility Graph $\mathcal{G}(\mathcal{T}, \mathcal{E})$** to discover intermediate bridge APIs.
  3. **Module 3 — Adaptive DAG Executor & Entity Scratchpad (for Tasks 1 & 2):** Builds a symbolic DAG (ReWOO style) while maintaining a structured **Entity Scratchpad $\mathcal{M}_k$** that resolves RAG text mentions into canonical SQL database IDs via lookup APIs.
  4. **Module 4 — Pre/Post-Execution Policy Guardrail Verifier (for Task 2):** Audits every proposed tool call $(t_k, a_k)$ against policy rules $\{c_l\}$ before execution; triggers local DFSDT-style backtracking ($B \le 2$) or safe abstention.

---

## Slide 6: Techniques, Neural Models & Original Implementation (7:00 – 8:15)
* **Neural Models Used:**
  * *Retriever & Reranker:* `BAAI/bge-large-en-v1.5` + BM25 + `bge-reranker-v2-m3`.
  * *Agent Backbone LLMs:* `Qwen2.5-14B/32B-Instruct`, `Llama-3.3-70B-Instruct` (+ stretch goal DPO on `Qwen2.5-7B-Instruct`).
* **Original Agent Modules Implemented From Scratch by Our Team:**
  1. Automated **Schema Compatibility Graph Builder $\mathcal{G}(\mathcal{T}, \mathcal{E})$** & 1-hop bridge retriever across all 8,000+ Python APIs.
  2. **Adaptive DAG Controller** with structured Cross-Source Entity Scratchpad $\mathcal{M}_k$.
  3. **Pre/Post-Execution Policy Entailment Verifier** & backtracking controller.

---

## Slide 7: Evaluation Benchmark (VAKRA) & Quantitative Metrics (8:15 – 10:00)
* **Evaluation Benchmark (`ibm-research/VAKRA`):** $8,000+$ Python APIs, $62$ domains, aligned RAG corpora, and $664$ multi-source queries ($420$ unconstrained + $244$ policy-constrained).
* **Quantitative Metrics:**
  1. **Trajectory Execution Accuracy (`Exec-Acc`)** via live database/RAG replay.
  2. **Final Answer Accuracy & Token-Level F1.**
  3. **Tool Selection Recall@$K$ & F1** (Subtask 2).
  4. **Policy Compliance Rate & Abstention F1** (Task 2, targeting the **2.4%** baseline bottleneck).
  5. **Efficiency:** Avg. API calls & tokens per solved trajectory.
