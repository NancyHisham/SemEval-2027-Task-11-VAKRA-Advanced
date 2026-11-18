# Building a Plan-and-Verify Agent with Schema-Grounded Tool Retrieval and Policy Guardrails for VAKRA-Advanced (SemEval-2027 Task 11)

**Advanced Natural Language Processing — Course Project Proposal**  
**Instructor:** Prof. Nagwa El-Makky  
**Submission Deadline:** Thursday, November 19, 2026  
**Project Discussion:** Saturday, November 21, 2026  

---

## 1. Key Information

* **Project Title:** Building a Plan-and-Verify Agent with Schema-Grounded Tool Retrieval and Policy Guardrails for VAKRA-Advanced (SemEval-2027 Task 11)
* **Team Members:**
  1. **[Team Member 1 Name]** (`member1@email.edu`)
  2. **[Team Member 2 Name]** (`member2@email.edu`)
* **Project Type:** Custom Shared-Task Project ([SemEval-2027 Task 11: VAKRA-Advanced](https://sites.google.com/view/vakrasemeval2027))
* **Mentor / Course Supervision:** Supervised by Prof. Nagwa El-Makky
* **Project Objective:** Design and build an autonomous tool-calling agent (**PV-VAKRA**) to solve the **two core VAKRA multi-hop reasoning tasks** (Task 1: Multi-Hop Structured API Reasoning [API Chaining, Tool Selection, and Multi-Hop Execution] and Task 2: Multi-Hop, Multi-Source Reasoning across APIs + Documents under Tool-Use Policies) and evaluate it on the official VAKRA benchmark.

---

## 2. Research Paper Summary (Baseline Agent & Task Paper — Max 2 Pages)

### Bibliographical Information
* **Title:** *VAKRA: Evaluating Multi-Hop Reasoning Across APIs and Retrieval Under Tool-Use Policies*
* **Authors:** Ankita Rajaram Naik, Anupama Narasimha Murthi, Benjamin T. Elder, Siyu Huo, Praveen Venkateswaran, Abdulhamid Adebayo, Sara Rosenthal, and Danish Contractor (IBM Research)
* **Publication Venue & Year:** arXiv Preprint (`arXiv:2608.12282`), 2026 (Official Baseline Agent & Evaluation Framework for SemEval-2027 Task 11)
* **URLs:** Paper: https://arxiv.org/abs/2608.12282 | Baseline Agent Code: https://github.com/IBM/vakra | Dataset: https://huggingface.co/datasets/ibm-research/VAKRA

### Background and Motivation
Autonomous Large Language Model (LLM) agents deployed in enterprise workflows must solve two intertwined reasoning problems: (1) **Multi-Hop Structured API Reasoning**, where an agent selects relevant tools from thousands of candidate APIs and chains $3\text{--}7$ dependent function calls by passing intermediate JSON outputs as downstream parameters, and (2) **Multi-Hop, Multi-Source Reasoning under Tool-Use Policies**, where the agent dynamically combines structured API executions with unstructured document retrieval (RAG) while obeying strict natural-language organizational policies (e.g., data privacy or source-authority constraints). Existing agent architectures such as standard **ReAct** (Yao et al., 2023) interleave single-step reasoning and tool execution greedily. Naik et al. (2026) formalize these two multi-hop tasks in a realistic executable environment ($8,000+$ Python APIs across $62$ domains) and implement a standardized ReAct baseline agent to diagnose why current LLM agents fail on long-horizon, policy-constrained workflows.

### Summary of Contributions
1. **Formulation of Two Core Multi-Hop Agent Tasks (Spanning Four Subtasks):** Given a user query $q$, natural-language tool-use policy $\mathcal{P}$, candidate API library $\mathcal{T}$, and unstructured document corpus $\mathcal{D}$, the agent must synthesize an execution trajectory $\tau = \langle (t_1, a_1, o_1), \dots, (t_K, a_K, o_K), \hat{y} \rangle$. This formulation encompasses **Task 1: Multi-Hop Structured API Reasoning** (covering Subtask 1: API Chaining over compositional Business Intelligence APIs, Subtask 2: Tool Selection from $150+$ APIs per slice, and Subtask 3: $3\text{--}7$ hop API reasoning) and **Task 2: Multi-Hop, Multi-Source Reasoning** (Subtask 4: integrating API calls and document RAG across $664$ queries, including $244$ policy-constrained queries).
2. **Standardized ReAct Baseline Agent & Deterministic Trajectory Evaluator:** The authors implement a unified ReAct agent harness that iteratively generates $\text{Thought}_k \rightarrow \text{Action}(t_k, a_k) \rightarrow \text{Observation}(o_k)$. Because multiple API sequences can validly solve a query, the agent's predicted trajectory $\tau$ is evaluated by re-executing calls against live databases and verifying state equivalence and policy compliance:
   $$\text{Score}(\tau, q, \mathcal{P}) = \mathbb{I}\!\left[\text{ExecState}(\tau) \equiv \text{ExecState}(\tau^*)\right] \cdot \mathbb{I}\!\left[\forall k \in \{1..K\},\ (t_k, a_k) \models \mathcal{P}\right] \cdot \text{Acc}(\hat{y}, y^*)$$
3. **Empirical Diagnosis of ReAct Agent Bottlenecks:** Benchmarking frontier and open-weight LLMs within the ReAct baseline reveals severe architectural weaknesses: while the baseline agent achieves up to **70.4%** accuracy on 1-hop query-aligned APIs, accuracy drops to **50%–51%** on compositional BI API chaining, degrades by **>50%** across $3\text{--}7$ reasoning hops, and collapses to **2.4%** on policy-constrained unanswerable queries. Trace-level error analysis proves that failures are rarely JSON syntax errors; rather, the ReAct agent fails due to *cross-source entity disambiguation errors*, *intermediate schema misalignments*, and *policy instruction-forgetting*.

### Limitations of the Baseline Agent and Critical Discussion
* **Myopic Greedy Execution Without Lookahead or Backtracking:** The baseline ReAct agent selects one tool at a time without constructing a global dependency plan or backtracking when an intermediate API lacks required downstream schema fields.
* **Lack of Explicit Cross-Source Entity Grounding Memory:** When transitioning from an unstructured document hop (`Retrieve`) to a structured database API hop, the baseline agent passes unnormalized surface strings (e.g., `"Big Blue"`) into strict SQL-backed API parameters expecting canonical IDs (`company_id="IBM_01"`).
* **Passive Prompt-Only Policy Conditioning:** The baseline agent simply appends policy $\mathcal{P}$ to the system prompt. As multi-hop JSON observations accumulate ($>8\text{k}$ tokens), attention dilution causes the LLM to ignore policy restrictions ($97.6\%$ failure rate on unanswerable policy queries).

### Why This Paper?
We selected Naik et al. (2026) because it defines the exact problem formulation, evaluation metrics, and official **ReAct Baseline Agent** for **SemEval-2027 Task 11 (VAKRA-Advanced)**. Analyzing the paper's failure taxonomy gave us the blueprint for our own agent design: replacing greedy ReAct with a *Plan-and-Verify Agent* featuring schema-graph retrieval, structured entity-linking memory, and an active pre-execution policy verifier.

### Wider Research Context
Building agents for multi-hop, multi-source tool use advances *neuro-symbolic program synthesis* and *grounded compositional reasoning*, where natural language serves as a control interface to orchestrate symbolic computation (SQL/Python APIs) alongside unstructured reading comprehension (RAG).

---

## 3. Concise SOTA Survey of Tool-Calling Agent Architectures (Max 2 Pages)

Because our project focuses on **building an agent to solve multi-hop structured API and multi-source reasoning tasks** (and evaluating it on the VAKRA benchmark), we survey **six recent state-of-the-art agent architectures and tool-retrieval methods (2023–2026)**.

### Summary of Surveyed Agent Architectures
1. **VAKRA Baseline ReAct Agent (Naik et al., 2026; Yao et al., ICLR 2023):** Greedy interleaved Thought-Action-Observation agent with dense retrieval. Suffers $>50\%$ degradation over $3\text{--}7$ hops and $2.4\%$ accuracy on unanswerable policy queries.
2. **ToolLLM & DFSDT Agent (Qin et al., ICLR 2024):** Combines a contrastive dense *Neural API Retriever* with a *Depth-First Search-based Decision Tree (DFSDT)* controller that allows backtracking from failed tool calls.
3. **AnyTool (Du et al., ICML 2024):** Introduces a *3-tier Hierarchical Retriever Agent* (Meta $\rightarrow$ Category $\rightarrow$ Tool agents) paired with a *Self-Reflective Solver* that audits trajectories and re-invokes search on failure.
4. **ReWOO: Decoupled Plan-then-Execute Agent (Xu et al., 2023/2024):** Replaces step-by-step ReAct with a decoupled (*Planner* $\rightarrow$ *Worker* $\rightarrow$ *Solver*) architecture that generates a symbolic dependency DAG (`#E1`, `#E2`) upfront, cutting token usage by $5\times$.
5. **LATS: Language Agent Tree Search (Zhou et al., ICML 2024):** Unifies reasoning, acting, and planning via *Monte Carlo Tree Search (MCTS)* guided by an LLM value function $V(s_k)$ and verbal self-reflection.
6. **Re-Invoke (Chen et al., EMNLP 2024) & Gorilla (Patil et al., NeurIPS 2024):** Re-Invoke enriches API indexing with synthetic queries and decomposes multi-intent user prompts at inference, while Gorilla trains retriever-aware LLMs for schema-faithful parameter generation.

### Comparison Table of SOTA Agent Architectures

| Agent / Method | Agent Architecture & Technique | Evaluated Datasets | Key Metrics | Limitations on Multi-Hop & Multi-Source Tasks |
| :--- | :--- | :--- | :--- | :--- |
| **ReAct Baseline** (Yao 23; Naik 26) | Greedy interleaved Thought $\rightarrow$ Action $\rightarrow$ Observation + prompt policy | **VAKRA** (8,000+ APIs, 62 domains, 664 multi-source queries) | Trace Replay `Exec-Acc`, Answer Acc. | Myopic 1-step horizon (>50% drop on 3–7 hops); prompt-only policy fails (2.4% on unanswerable queries). |
| **ToolLLM (DFSDT)** (Qin et al., ICLR 2024) | Contrastive Dense API Retriever + Depth-First Search Decision Tree | **ToolBench** (16,464 REST APIs), APIBench | Pass Rate & Win Rate | Flat retriever misses intermediate bridge APIs; lacks cross-source entity linking between RAG text and SQL APIs. |
| **AnyTool** (Du et al., ICML 2024) | 3-Tier Hierarchical Retriever Agent + Self-Reflective DFSDT Solver | **AnyToolBench** & ToolBench (16k+ APIs) | Rubrics Pass Rate, Recall@$K$ | High multi-agent token cost; self-reflection checks final answer completeness but not strict tool-use policies. |
| **ReWOO** (Xu et al., 2023/2024) | Decoupled Planner (symbolic DAG `#E1`$\rightarrow$`#E2`) + Worker + Solver | HotpotQA, TriviaQA, Tool QA benchmarks | Exact Match, F1, Token Efficiency | Open-loop static plan breaks if an intermediate RAG/API output requires dynamic entity disambiguation or fallback. |
| **LATS** (Zhou et al., ICML 2024) | Monte Carlo Tree Search (MCTS) + LLM Value Function + Verbal Reflection | HotpotQA, WebShop, Programming tasks | Success Rate, Exact Match | Prohibitive inference latency (10–30$\times$ rollouts per query); does not model API I/O schema graphs. |
| **Re-Invoke** (EMNLP 24) & **Gorilla** (NeurIPS 24) | Synthetic query-enriched indexing + Multi-intent query decomposition | ToolBench, ToolE, APIBench (1.6k–16k APIs) | Recall@$K$, NDCF, AST Match | Retrieves tools matching explicit user sub-queries, missing implicit bridge APIs whose schemas link hop $k$ to hop $k+1$. |

---

## 4. Project Description (1–2 Pages)

### 4.1 Main Goals and Scientific Question
Our goal is to design and build an autonomous agent, **PV-VAKRA** (*Plan-and-Verify Agent*), to solve the two core multi-hop reasoning tasks in **SemEval-2027 Task 11 (VAKRA-Advanced)** and evaluate it on the official VAKRA benchmark:
> *Can an agent architecture that integrates (1) Schema-Graph-Guided Tool Retrieval, (2) Adaptive DAG Planning with an Explicit Cross-Source Entity Scratchpad, and (3) Pre/Post-Execution Policy Guardrail Verification significantly outperform the official VAKRA ReAct baseline agent across both Multi-Hop Structured API Reasoning (Task 1) and Policy-Constrained Multi-Hop Multi-Source Reasoning (Task 2)?*

### 4.2 NLP Tasks Solved by Our Agent
1. **Task 1 — Multi-Hop Structured API Reasoning (Subtasks 1, 2, & 3):** Select gold tools from $150+$ domain candidates (*Subtask 2: Tool Selection*), compose sequential/nested calls (*Subtask 1: API Chaining*), and execute $3\text{--}7$ dependent hops (*Subtask 3: Multi-Hop API Reasoning*).
2. **Task 2 — Multi-Hop, Multi-Source Reasoning under Policies (Subtask 4):** Interleave structured API calls with unstructured document retrieval (`Retrieve`) while resolving entities across sources and strictly satisfying natural-language tool-use policy $\mathcal{P}$.

### 4.3 Evaluation Benchmark Dataset (`ibm-research/VAKRA`) & Preprocessing
* **Evaluation Benchmark:** $8,000+$ executable Python APIs across $62$ domains + aligned document collections, including single-hop/compositional BI splits, multi-hop API splits ($2\text{--}7$ hops), and $664$ multi-source queries ($420$ unconstrained + $244$ policy-constrained).
* **Offline Agent Indexing:** (1) Parse all $8,000+$ API docstrings/schemas to build a directed **Schema Compatibility Graph** $\mathcal{G}(\mathcal{T}, \mathcal{E})$, and (2) index enriched API schemas and document chunks with BM25 + `BAAI/bge-large-en-v1.5`.

### 4.4 Proposed Agent Architecture (`PV-VAKRA`)
1. **Intent Decomposer & Schema-Graph Hybrid Retriever:** Decomposes query $q$ into sub-goals $\{q_1, \dots, q_m\}$, scores tools via hybrid BM25 + dense similarity, expands top-$k_0$ candidates via 1-hop neighbors on $\mathcal{G}(\mathcal{T}, \mathcal{E})$ to discover bridge APIs, and reranks with `bge-reranker-v2-m3`.
2. **Adaptive DAG Planner & Cross-Source Entity Scratchpad:** Builds a symbolic execution DAG (ReWOO style) while maintaining a structured **Entity Scratchpad** $\mathcal{M}_k = \{(\text{mention}, \text{id}, \text{source}, \text{type})\}$ that maps unstructured RAG mentions to canonical SQL IDs via lookup APIs.
3. **Pre/Post-Execution Policy Guardrail Verifier:** Parses policy $\mathcal{P}$ into atomic rules $\{c_1, \dots, c_L\}$ and verifies entailment $P_{\text{verify}}(\mathcal{P} \models \neg(t_k, a_k))$ before executing each tool and before emitting $\hat{y}$, triggering local DFSDT-style backtracking ($B \le 2$) or safe abstention.

### 4.5 Baselines & Quantitative Evaluation on VAKRA
* **Baselines:** Official `IBM/vakra` ReAct Baseline Agent, Flat Dense Retrieval + ReAct Agent, and Module Ablations.
* **Quantitative Metrics:** Trajectory Execution Accuracy (`Exec-Acc`), Final Answer F1, Tool Selection Recall@$K$ & F1, Policy Compliance & Abstention F1, and Token/Call Efficiency.

### 4.6 Ethical Challenges, Societal Risks, and Mitigation Strategies
Addresses unauthorized tool execution/policy bypass and compounded cross-source hallucinations via calibrated abstention thresholds ($\tau_{\text{safe}}$), sandboxed read-only database execution, and verifiable provenance traces.
