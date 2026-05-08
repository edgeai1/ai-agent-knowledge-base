---
title: "AgentBench: Evaluating LLMs as Agents"
authors: Xiao Liu, Hao Yu, Hanchen Zhang, Yifan Xu, Xuanyu Lei, Hanyu Lai, Yu Gu, Hangliang Ding, Kaiwen Men, Kejuan Yang, Shudan Zhang, Xiang Deng, Aohan Zeng, Zhengxiao Du, Chenhui Zhang, Sheng Shen, Tianjun Zhang, Yu Su, Huan Sun, Minlie Huang, Yuxiao Dong, Jie Tang
affiliation: Tsinghua University, The Ohio State University, UC Berkeley
venue: ICLR 2024
year: 2024
url: https://arxiv.org/abs/2308.03688
code: https://github.com/THUDM/AgentBench
tags: [benchmark, agent-evaluation, multi-environment, LLM-as-agent, decision-making]
status: done
---

## TL;DR

The first systematic benchmark evaluating LLMs as autonomous agents across 8 diverse interactive environments (OS, Database, Knowledge Graph, Digital Card Game, Lateral Thinking Puzzles, HouseHolding, Web Shopping, Web Browsing), testing 29 models and revealing a stark performance gap: GPT-4 achieves an overall score of 4.01 while the best open-source model (CodeLlama-34B) scores only 0.96. Poor long-term reasoning, decision-making, and instruction following are identified as key capability bottlenecks preventing LLMs from functioning as effective autonomous agents.

## Motivation & Problem

By mid-2023, LLMs were increasingly deployed as autonomous agents -- systems that take actions in environments to achieve goals, rather than simply generating text. Yet evaluation remained fragmented and inadequate:

1. **Domain-specific benchmarks only**: Existing evaluations tested narrow capabilities (coding only in HumanEval, web browsing only in Mind2Web) without a unified framework that could assess general agent competence across diverse task types.
2. **Static evaluation paradigm**: Most LLM benchmarks used single-turn QA or text generation metrics (MMLU, HellaSwag), completely missing the multi-turn, interactive, stateful nature of agent tasks where each action changes the environment.
3. **No cross-domain comparison**: Impossible to determine whether a model strong at coding was also capable at database management, game-playing, or web navigation. No unified scoring framework existed.
4. **Open-source vs. commercial opacity**: Open-source models increasingly claimed GPT-4-level performance on static benchmarks, but whether this parity extended to complex agentic tasks was entirely unknown.
5. **Missing capability diagnosis**: No systematic study had identified which specific capabilities (long-term reasoning, instruction following, decision-making, tool use) were the primary bottlenecks for LLM agents.

AgentBench addresses all of these by providing a unified evaluation framework across 8 environments spanning code execution, strategic games, web interaction, and knowledge reasoning, with standardized scoring to enable direct cross-model and cross-domain comparison.

## Method

### The 8 Evaluation Environments in Detail

AgentBench comprises 8 distinct environments organized into three categories. Five environments (OS, DB, KG, DCG, LTP) are original contributions created by the research team; three (HH, WS, WB) adapt existing benchmarks into the unified framework.

#### Category 1: Code-Grounded Environments

**1. Operating System (OS) -- NEW**
- **Setup**: Agent interacts with a real Ubuntu Docker container via an interactive bash shell
- **Task type**: System administration and file manipulation tasks requiring multi-step command-line operations
- **Example tasks**: Recursively set all files in a directory to read-only; find and kill processes matching a pattern; configure a cron job with specific timing; parse log files and extract statistics
- **Interaction**: Multi-turn bash command execution. Agent sends commands, receives stdout/stderr feedback, and iterates
- **Turns per task**: 5-15 on average
- **Evaluation metric**: Success Rate (SR) -- binary pass/fail based on whether the final system state matches the expected state via automated validation scripts
- **Difficulty factors**: Requires knowledge of Linux commands, file system operations, permissions model, process management, scripting, and multi-step reasoning about command outputs
- **Why it matters**: Tests practical system-level reasoning that real-world coding agents and DevOps assistants need

**2. Database (DB) -- NEW**
- **Setup**: Agent interacts with relational databases via SQL queries on real MySQL/PostgreSQL instances
- **Task type**: Data retrieval, analysis, and manipulation tasks requiring increasingly complex query construction
- **Example tasks**: "Find the top 5 customers by total order amount from the orders table and return their names and total amounts"; join multiple tables to answer business intelligence questions
- **Interaction**: Agent issues SQL queries, receives query results as tables, can iteratively refine queries based on output
- **Turns per task**: 3-10 on average
- **Evaluation metric**: Success Rate -- correctness of final query results compared to ground truth answers
- **Difficulty factors**: Requires SQL proficiency (joins, aggregations, subqueries, window functions), schema comprehension from description alone, and iterative debugging of query errors
- **Why it matters**: Database interaction is a core competency for data analysis agents and business intelligence applications

**3. Knowledge Graph (KG) -- NEW**
- **Setup**: Agent navigates large-scale knowledge graphs based on FREEBASE (45M+ entities, 3B+ facts) via structured queries
- **Task type**: Complex question answering requiring multi-hop reasoning over entity relationships in the graph
- **Example tasks**: "What films did the director of Inception also produce that won an Academy Award?" -- requires traversing director -> filmography -> awards paths
- **Interaction**: Agent issues SPARQL-like queries to explore entity relationships, receives partial results, must navigate the massive graph structure to find answers
- **Turns per task**: 5-15 on average
- **Evaluation metric**: F1 score on extracted answers (partial credit for partially correct answer sets)
- **Difficulty factors**: Operating with incomplete information over a graph with 45M+ entities; managing inherent uncertainties; constructing valid structured queries; multi-hop inference chains
- **Why it matters**: Tests structured reasoning and information retrieval capabilities essential for knowledge-intensive agent applications

#### Category 2: Game-Grounded Environments

**4. Digital Card Game (DCG) -- NEW**
- **Setup**: Agent plays a simplified digital card game (Aquaducts, inspired by trading card games) against rule-based opponents of varying difficulty
- **Task type**: Strategic game-playing requiring resource management, card selection, timing, and opponent modeling across extended multi-turn games
- **Interaction**: Turn-based. Agent receives the current game state (hand, board, opponent visible state, resources), selects actions from legal moves (play card, attack, use ability, pass)
- **Turns per task**: 20-50 per game
- **Evaluation metric**: Win Rate against rule-based opponents
- **Difficulty factors**: Long-horizon strategic planning (decisions 20+ turns ago affect endgame), imperfect information (opponent hand unknown), resource optimization, and need to anticipate opponent strategy
- **Why it matters**: Tests genuine strategic reasoning under uncertainty -- a capability qualitatively different from text generation. The extended game length (20-50 turns) particularly stresses long-term planning.

**5. Lateral Thinking Puzzles (LTP) -- NEW**
- **Setup**: Agent plays a situation puzzle game. A host presents a mysterious scenario (e.g., "A man walks into a bar and asks for water. The bartender pulls out a gun. The man says thank you and leaves."), and the agent must deduce the full explanation
- **Task type**: Creative deductive reasoning through sequential yes/no questioning
- **Interaction**: Agent asks yes/no questions ("Did the man have hiccups?"); host responds with "yes," "no," or "irrelevant"; agent must reconstruct the hidden scenario through efficient questioning
- **Turns per task**: 10-30 per puzzle
- **Evaluation metric**: Game Progress (0-1 scale based on key elements of the puzzle deduced) and Success Rate
- **Difficulty factors**: Requires creative/lateral thinking (not just logical deduction), hypothesis generation, efficient information gathering through heavily constrained questioning, and the ability to synthesize disparate clues into a coherent narrative
- **Why it matters**: Tests a fundamentally different reasoning mode -- creative/divergent thinking -- that is orthogonal to the logical/procedural reasoning tested by other environments

#### Category 3: Web-Grounded Environments

**6. HouseHolding (HH) -- adapted from ALFWorld**
- **Setup**: Agent controls a virtual household robot in a simulated home environment (TextWorld-based)
- **Task type**: Household tasks requiring navigation, object interaction, and multi-step physical reasoning
- **Example tasks**: "Put a clean mug on the desk" requires: go to kitchen -> find mug -> pick up mug -> go to sink -> clean mug -> go to bedroom -> go to desk -> put mug on desk
- **Interaction**: Text-based commands with environment state descriptions as feedback
- **Turns per task**: 10-30 per task
- **Evaluation metric**: Success Rate -- whether the task goal state is achieved
- **Difficulty factors**: Spatial reasoning (which rooms connect), object affordance understanding (mugs can be cleaned at sinks), and executing multi-step plans in correct order
- **Why it matters**: Tests embodied agent capabilities in a grounded physical world simulation; benefits heavily from common-sense reasoning

**7. Web Shopping (WS) -- adapted from WebShop**
- **Setup**: Agent shops on a simulated e-commerce website to find products matching natural language specifications
- **Task type**: Product search, comparison, and selection tasks
- **Example tasks**: "Find a red cotton t-shirt under $30 with good reviews and size L"
- **Interaction**: Web navigation actions (search, click product, scroll, select options like color/size, add to cart)
- **Turns per task**: 5-15 per task
- **Evaluation metric**: Reward score (0-1) based on attribute matching between the purchased item and the target specification
- **Difficulty factors**: Understanding product attributes, comparing across multiple options, applying multiple simultaneous criteria (color + material + price + rating + size)
- **Why it matters**: Tests practical consumer-facing web interaction with real-world relevance

**8. Web Browsing (WB) -- adapted from Mind2Web**
- **Setup**: Agent performs tasks on diverse real websites by predicting correct action sequences on actual website HTML/screenshots
- **Task type**: General web browsing tasks spanning many websites and domains (booking flights, managing accounts, finding information, filing forms)
- **Interaction**: At each step, predict the correct web element to interact with and the action to take (click, type, select)
- **Turns per task**: 5-20 per task
- **Evaluation metric**: Step Success Rate (per-step element and action accuracy) and task completion rate
- **Difficulty factors**: Must generalize across extremely diverse website layouts with no prior exposure; requires understanding forms, navigation menus, dynamic content, and multi-step web workflows
- **Why it matters**: Tests open-web generalization -- the ability to navigate arbitrary websites, not just specific pre-trained applications

### Unified Evaluation Protocol

- All environments use multi-turn interaction where each action changes the environment state
- Environment-specific metrics are normalized to produce the **Overall AgentBench Score (OA)** as a weighted average
- Standardized prompts ensure fair comparison: each model receives identical system prompts per environment
- Budget limits (maximum turns, maximum tokens) prevent runaway costs
- Environments run in isolated Docker containers for reproducibility

## Key Innovations

1. **Multi-dimensional agent evaluation**: First benchmark to span 8 environment types (code, games, web, knowledge) in a unified framework with standardized scoring
2. **Five new environments**: OS, DB, KG, DCG, and LTP are original contributions testing previously unevaluated agent capabilities
3. **Scale of comparison**: 29 models evaluated, covering the full spectrum from GPT-4 to small open-source models, enabling definitive capability stratification
4. **Capability diagnosis**: Goes beyond aggregate scores to identify three specific failure modes (long-term reasoning, decision-making, instruction following)
5. **Commercial vs. open-source analysis**: First systematic, quantitative comparison revealing the true agent capability gap between model tiers

## Experimental Setup

### Models Tested (29 total)

**Commercial API models**: GPT-4 (0613), GPT-3.5-turbo (0613), text-davinci-003, Claude (v1.3), Claude-2, Claude-instant, chat-bison (Google PaLM)

**Open-source models**: LLaMA-2-70B/13B/7B (chat), CodeLlama-34B/13B/7B, Vicuna-33B/13B/7B, LLaMA-33B/13B, and additional variants -- all tested at their largest available sizes

All models tested zero-shot with identical per-environment prompts. No fine-tuning or agent-specific training.

## Results

### Overall AgentBench Scores

| Model                | Overall Score (OA) | Tier       |
|----------------------|-------------------|------------|
| GPT-4 (0613)         | **4.01**          | Top tier   |
| Claude-2             | 2.49              | Strong     |
| GPT-3.5-turbo        | 2.32              | Strong     |
| Claude (v1.3)        | 1.64              | Moderate   |
| text-davinci-003     | 1.52              | Moderate   |
| chat-bison (PaLM)    | 1.32              | Moderate   |
| CodeLlama-34B        | 0.96              | Weak       |
| LLaMA-2-70B-chat     | ~0.6              | Weak       |
| Vicuna-33B           | ~0.5              | Weak       |
| Most OSS models      | 0.2-0.8           | Very weak  |

### GPT-4 Environment-Specific Highlights

| Environment          | GPT-4 Score | Best OSS      | Notes                                     |
|----------------------|-------------|----------------|-------------------------------------------|
| HouseHolding         | **78% SR**  | ~20-30% SR     | Easiest; benefits from common-sense        |
| Digital Card Game    | **74.5% WR**| Near zero      | Strong strategic reasoning; OSS fails      |
| Web Shopping         | 62.6%       | Low            | Moderate; structured criteria matching     |
| Knowledge Graph      | 57.2% F1    | Near zero      | Multi-hop graph reasoning                  |
| Operating System     | 42.4% SR    | Near zero      | System knowledge + multi-step scripting    |
| Database             | 32.5% SR    | Low            | Complex SQL remains challenging            |
| Web Browsing         | 5.3%        | Near zero      | Near-zero for all models                   |
| Lateral Thinking     | 5.0%        | Near zero      | Creative reasoning is fundamentally hard   |

GPT-4 achieves the best performance in **6 out of 8** environments.

### Environment Difficulty Analysis (Why Some Are Hardest)

**Hardest environments (near-zero for all models):**
1. **Web Browsing (5.3%)**: Requires generalizing across arbitrary website layouts with no prior exposure. The diversity of real websites makes this an open-domain generalization problem that current models cannot solve.
2. **Lateral Thinking (5.0%)**: Creative reasoning requires generating novel hypotheses, not just applying learned patterns. This is orthogonal to the pattern-matching strength of LLMs.

**Challenging environments:**
3. **Database (32.5%)**: Multi-table joins, aggregations, and subqueries push beyond simple SQL generation into genuine data reasoning.
4. **Operating System (42.4%)**: Complex shell operations (piping, scripting, process management) require system-level knowledge that goes beyond text manipulation.

**Relatively accessible environments:**
5. **HouseHolding (78%)**: Constrained action space, strong common-sense priors from pre-training, and relatively short episodes make this the most tractable environment.
6. **Digital Card Game (74.5%)**: GPT-4 shows genuine strategic ability, but open-source models essentially play randomly, creating the largest per-environment gap.

### Commercial vs. Open-Source Gap

The performance gap is dramatic and consistent:
- GPT-4 (4.01) outperforms the best open-source model by **4.2x** (CodeLlama-34B at 0.96)
- This gap is far larger than what static benchmarks suggest -- on MMLU and HumanEval, open-source models appear competitive, but on agentic tasks the gap is qualitative, not just quantitative
- Open-source models score near zero on the most demanding environments (DCG, WB, KG), indicating complete incapability rather than just reduced accuracy
- Even GPT-3.5-turbo (2.32) outperforms every open-source model tested by more than 2x

## Limitations

- **Temporal snapshot**: Results reflect model capabilities as of mid-2023. The open-source gap has substantially narrowed since publication (Qwen-2.5, DeepSeek-V3, LLaMA-3 have improved significantly).
- **Aggregate score simplification**: A single overall score hides dramatic per-environment variation. A model strong in coding but weak in games gets a similar aggregate to one with opposite strengths.
- **Prompt sensitivity**: Agent performance is highly sensitive to system prompt design. Different prompting strategies (few-shot, chain-of-thought) could substantially change rankings.
- **No fine-tuned agent baselines**: All models tested zero-shot. Agent-specific fine-tuning (as later demonstrated by CodeAct and others) can dramatically improve performance.
- **No tool-use augmentation**: Models tested in their base form without tool augmentation (code interpreters, retrieval systems), which may underestimate practical capabilities.
- **English-only**: All environments and tasks use English exclusively.
- **Limited task diversity per environment**: Each environment has a fixed set of tasks that may not capture the full diversity of real-world agentic scenarios.
- **Binary task evaluation in most environments**: No partial credit for agents that complete some steps correctly but fail at the final step.

## Key Takeaways

1. **LLMs as agents expose a hidden capability gap**: Models that appear competitive on static benchmarks (MMLU, HumanEval) show dramatic performance differences when used as interactive agents. AgentBench reveals capabilities that standard evaluation entirely misses.
2. **The commercial-open-source gap is severe for agentic tasks**: GPT-4's 4.01 vs. CodeLlama's 0.96 is a 4x gap that represents a qualitative capability difference -- open-source models are not just worse, they completely fail on the hardest environments.
3. **Long-term reasoning is the primary bottleneck**: Poor performance correlates strongly with environments requiring many sequential decisions (DCG at 20-50 turns, LTP at 10-30 questions), more than with knowledge or basic instruction following.
4. **Three specific failure modes identified**: (a) poor long-term reasoning over extended interactions, (b) weak decision-making even when correct actions are available, and (c) instruction following deficits (wrong output format, ignored constraints) -- particularly acute in open-source models.
5. **Environment diversity matters**: Performance varies dramatically across environments (78% on HouseHolding vs. 5% on Web Browsing for GPT-4), making single-environment benchmarks fundamentally misleading for assessing general agent capabilities.
6. **Improving instruction following and training on high-quality multi-round alignment data are the most promising paths** to improving agent performance -- a finding validated by subsequent work on agent-specific fine-tuning (CodeAct, Qwen-Agent).
7. **The benchmark established a model capability hierarchy** -- GPT-4 >> Claude-2 ~ GPT-3.5 >> open-source -- that held remarkably consistent across all environments and has guided subsequent model development priorities.
