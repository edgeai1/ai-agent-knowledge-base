---
title: "ChatDev: Communicative Agents for Software Development"
authors: "Chen Qian, Wei Liu, Hongzhang Liu, Nuo Chen, Yufan Dang, Jiahao Li, Cheng Yang, Weize Chen, Yusheng Su, Xin Cong, Juyuan Xu, Dahai Li, Zhiyuan Liu, Maosong Sun"
venue: "ACL 2024"
year: 2024
url: "https://arxiv.org/abs/2307.07924"
code: "https://github.com/OpenBMB/ChatDev"
tags: [multi-agent, software-development, role-playing, chat-chain, waterfall-model, code-generation]
category: agent/multi_agent
status: done
date_read: 2026-05-08
---

# ChatDev: Communicative Agents for Software Development

## TL;DR

ChatDev presents a virtual software development company powered by LLM-based agents playing
specialized roles (CEO, CTO, Programmer, Art Designer, Tester) that collaborate through natural
language conversation to produce complete software systems. A "chat chain" mechanism decomposes the
software development lifecycle (waterfall model: designing, coding, testing, documenting) into
sequential atomic subtasks, each resolved through multi-turn dialogue between role-playing agents.
"Communicative dehallucination" techniques address LLM-specific issues like code hallucination.
ChatDev completes full software projects in under 7 minutes at less than $1 cost, with 86.66% of
generated systems being directly executable.

## Motivation & Problem

By mid-2023, LLMs demonstrated strong code generation ability on isolated functions (HumanEval),
but building complete, multi-file software systems remained an open challenge:

1. **Single-function vs. whole-system**: Existing LLM coding tools generated individual functions
   or snippets, not complete software systems with multiple interacting components, UIs, and
   test suites.
2. **Missing software engineering process**: Real software development follows established
   methodologies (waterfall, agile) with distinct phases and specialized roles. LLM-based code
   generation ignored this organizational structure entirely.
3. **Hallucination in code generation**: LLMs frequently hallucinate non-existent APIs, libraries,
   or functions when generating code, producing plausible-looking but non-functional software.
4. **No iterative refinement**: Single-pass code generation lacks the iterative design-code-test
   cycle that real software development relies on for quality assurance.
5. **Knowledge integration challenge**: Building complete software requires integrating knowledge
   from multiple domains (requirements analysis, system design, programming, testing, UI design)
   that exceeds any single prompt's scope.

ChatDev's insight: **Software development is fundamentally a collaborative, multi-role process.**
Encoding distinct roles (CEO, CTO, Programmer, Tester) with specialized knowledge and structured
communication protocols produces more complete and reliable software than single-agent generation.

## Method

### Virtual Software Company Architecture

```
+-------------------------------------------------------------------+
|                    ChatDev Virtual Company                         |
|                                                                   |
|  Phase 1: DESIGNING                                               |
|  +--------+  "What should     +--------+                          |
|  |  CEO   |  we build?"       |  CTO   |                          |
|  | (needs |  <------------>   |(tech    |                          |
|  | analysis)|                 | choices)|                          |
|  +--------+                   +--------+                          |
|       |                                                           |
|  Phase 2: CODING                                                  |
|  +--------+  "Implement       +--------+                          |
|  |  CTO   |  this design"     |Programmer|                        |
|  |(design  |  <------------>  |(code     |                        |
|  | spec)  |                   | writing) |                        |
|  +--------+                   +--------+                          |
|       |                            |                              |
|  Phase 3: TESTING                  |                              |
|  +----------+  "Test this    +--------+                           |
|  |Programmer |  code"        | Tester  |                          |
|  |(code fixes|  <--------->  |(bug     |                          |
|  | debugging)|               | reports)|                          |
|  +----------+                +--------+                           |
|       |                                                           |
|  Phase 4: DOCUMENTING                                             |
|  +--------+  "Document       +--------+                           |
|  |  CTO   |  the system"     |Programmer|                        |
|  |(review) |  <------------>  |(docs,    |                        |
|  +--------+                   | README) |                        |
|                               +--------+                          |
+-------------------------------------------------------------------+
```

### Chat Chain Mechanism

The chat chain is the core organizational structure that decomposes the waterfall development
process into a sequence of atomic chat-based subtasks:

```
Algorithm: Chat Chain Execution
---------------------------------
Input:  User requirement R
Output: Complete software system S

ChatChain = [
  Phase("Designing", [
    Chat(CEO, CTO, "Determine modality: web/CLI/GUI?"),
    Chat(CEO, CTO, "Choose programming language"),
    Chat(CEO, CTO, "Define main features and scope"),
  ]),
  Phase("Coding", [
    Chat(CTO, Programmer, "Generate system architecture"),
    Chat(CTO, Programmer, "Implement main functionality"),
    Chat(ArtDesigner, Programmer, "Create GUI/visual assets"),
  ]),
  Phase("Testing", [
    Chat(Programmer, Tester, "Write and run test cases"),
    Chat(Programmer, Tester, "Debug and fix reported bugs"),
    Chat(Programmer, Tester, "Review code for quality"),
  ]),
  Phase("Documenting", [
    Chat(CTO, Programmer, "Write user manual"),
    Chat(CTO, Programmer, "Generate environment dependencies"),
  ]),
]

1: context = {requirement: R}
2: for each phase P in ChatChain:
3:     for each chat C in P.chats:
4:         dialogue = MultiTurnDialogue(
5:             agent_1 = C.role_1,
6:             agent_2 = C.role_2,
7:             topic = C.topic,
8:             context = context
9:         )
10:        context.update(dialogue.outputs)
11:    end for
12: end for
13: S = AssembleSoftware(context)
14: return S
```

Each chat in the chain:
- Involves exactly two role-playing agents
- Has a specific subtask topic
- Produces structured outputs (design decisions, code, test results)
- Receives accumulated context from all previous chats

### Role Definitions

| Role           | Phase(s)              | Responsibilities                                      |
|----------------|----------------------|-------------------------------------------------------|
| CEO            | Designing            | Requirements analysis, scope definition, modality choice |
| CTO            | Designing, Coding, Documenting | Technology selection, architecture design, review   |
| Programmer     | Coding, Testing, Documenting | Code implementation, bug fixing, documentation     |
| Art Designer   | Coding               | GUI design, visual asset creation, UI specifications  |
| Tester         | Testing              | Test case design, bug detection, quality assurance    |

### Communicative Dehallucination

ChatDev introduces specific techniques to combat LLM hallucination during code generation:

```
Technique 1: Cross-Role Verification
--------------------------------------
When Programmer generates code:
  1. Tester independently reviews for hallucinated imports/APIs
  2. If hallucinated module found:
     - Tester reports: "Module X does not exist"
     - Programmer revises using only verified libraries
  3. Repeat until no hallucinated dependencies remain

Technique 2: Thought Instruction Mechanism
--------------------------------------------
Agents are prompted to:
  1. First state their reasoning ("I think we should use Flask because...")
  2. Then provide the concrete output (actual code)
  3. Separate "thinking" from "output" reduces hallucination in final artifacts

Technique 3: Self-Reflection Prompting
-----------------------------------------
After each code generation step:
  1. Programmer reviews own code: "Check for bugs, missing imports, undefined variables"
  2. Agent identifies potential issues before passing to Tester
  3. Pre-filters obvious errors before the testing phase
```

### Waterfall Development Process

The four phases map directly to the classic waterfall software development model:

```
Designing -----> Coding -----> Testing -----> Documenting
   |               |              |               |
   v               v              v               v
- Modality      - Architecture  - Unit tests    - User manual
- Language      - Main code     - Integration   - README
- Features      - GUI assets    - Bug fixes     - Dependencies
- Scope         - File struct.  - Code review   - Setup guide
```

Key property: Each phase must complete before the next begins, with accumulated outputs serving
as context for subsequent phases. This sequential constraint ensures design consistency but
prevents iterative refinement across phases.

## Key Innovations

1. **Chat chain for software engineering**: Decomposing the waterfall model into a chain of
   atomic two-agent chats, each with a specific subtask, provides structured yet flexible
   control over the development process.
2. **Role-playing software company**: Assigning LLM instances specialized software engineering
   roles (CEO through Tester) with distinct knowledge and responsibilities parallels real-world
   team structures.
3. **Communicative dehallucination**: Cross-role verification and thought instruction techniques
   specifically address code hallucination -- a critical problem for LLM-generated software.
4. **End-to-end software generation**: From a one-sentence requirement to deployable software
   with code, tests, documentation, and dependency specifications in under 7 minutes.
5. **Cost-effective development**: Complete software systems generated for less than $1 in API
   costs, demonstrating the economic viability of LLM-powered software development.

## Experimental Setup

- **Base model**: GPT-3.5-Turbo (primary)
- **Evaluation**: 70 software development tasks across diverse categories
- **Baselines**: GPT-Engineer (single-agent), MetaGPT (structured multi-agent)
- **Metrics**:
  - Executability: Whether generated software runs without errors
  - Completeness: Coverage of specified requirements
  - Consistency: Alignment between design and implementation
  - Quality: Code quality and documentation completeness
  - Human evaluation and GPT-4 evaluation (preference judgments)
- **Cost tracking**: API token usage and dollar cost per project

## Results

### Software Generation Statistics (70 tasks)

| Metric                      | ChatDev         |
|-----------------------------|:---------------:|
| Tasks evaluated             | 70              |
| Executability rate          | **86.66%**      |
| Average development time    | < 7 minutes     |
| Average cost per project    | < $1.00         |
| Code files per project      | Multiple (3-8)  |

86.66% of generated software systems were directly executable without manual modification.

### Comparison with Baselines

| Metric              | GPT-Engineer | MetaGPT  | ChatDev      |
|---------------------|:------------:|:--------:|:------------:|
| Completeness        | Low          | High     | **High**     |
| Executability       | Moderate     | High     | **86.66%**   |
| Human preference    | Low          | High     | **High**     |
| GPT-4 preference    | Low          | High     | **High**     |

ChatDev and MetaGPT both significantly outperform GPT-Engineer (a single-agent approach),
demonstrating that complex software development benefits from multi-agent collaboration.

### Multi-Agent vs. Single-Agent Analysis

The multi-agent paradigm produces:
- **More code files**: Multi-agent generates more modular, multi-file software systems
- **Larger codebases**: Greater total lines of code per project
- **Better structure**: Clearer separation of concerns across files
- **Higher quality**: More comprehensive error handling and documentation

Trade-off: Multi-agent is slower and consumes more tokens than single-agent, but the quality
improvement justifies the overhead for complex software projects.

### Ablation: Impact of Chat Chain Components

Removing components of the chat chain degrades performance:
- Without testing phase: Executability drops significantly (bugs go undetected)
- Without designing phase: Code lacks coherent architecture, more ad-hoc
- Without art designer: GUI applications lack proper visual design
- Without dehallucination: Hallucinated imports and APIs increase error rates

## Limitations

1. **Waterfall rigidity**: The sequential phase structure does not support iterative development.
   Design flaws discovered during testing cannot trigger a redesign phase -- only local code
   fixes are possible.
2. **GPT-3.5-Turbo ceiling**: Primary evaluation uses GPT-3.5-Turbo, which limits the complexity
   of software that can be generated. More capable models would likely improve results.
3. **Simple project scope**: The 70 evaluation tasks involve relatively simple software (games,
   utilities, small tools). Scaling to enterprise-level software with complex architectures,
   databases, and external integrations is unvalidated.
4. **No persistent state**: Each ChatDev run starts from scratch; there is no mechanism for
   incremental development, version control, or building on previous projects.
5. **Limited debugging depth**: The testing phase catches surface-level bugs but cannot perform
   deep semantic testing, integration testing, or performance testing.
6. **Two-agent per chat limitation**: Each chat involves exactly two agents. Real software
   development often requires multi-party discussions (design reviews, sprint planning with
   full team).
7. **Art Designer limitations**: GUI and visual asset generation is constrained by text-based
   LLM capabilities; actual graphic design requires multimodal generation.
8. **No user feedback loop**: The process runs end-to-end without user checkpoints, unlike real
   development where clients review prototypes and provide feedback.
9. **Comparison scope**: Direct comparison with MetaGPT in the paper is limited; MetaGPT's own
   publication shows stronger results with its structured document approach.

## Key Takeaways

1. **Multi-agent > single-agent for software**: Both ChatDev and MetaGPT demonstrate that multi-
   agent collaboration produces more complete and reliable software than single-agent approaches,
   validating the multi-agent paradigm for software engineering.
2. **Chat chain provides structured control**: Decomposing a complex process into a sequence of
   focused two-agent conversations offers a practical middle ground between free-form multi-agent
   chat (chaotic) and rigid pipelines (inflexible).
3. **Dehallucination through verification**: Cross-role verification (Tester checking Programmer's
   code for hallucinated APIs) is an effective technique for reducing a critical LLM failure mode
   in code generation.
4. **Software development is communication-centric**: The success of ChatDev validates the
   insight that software development is fundamentally about communication between specialized
   roles, and this communication can be effectively conducted by LLM agents.
5. **Remarkably cost-effective**: Full software systems for under $1 in under 7 minutes represents
   a paradigm shift in the economics of software prototyping, even if quality is below production
   standards.
6. **Chat-based vs. document-based communication**: ChatDev (dialogue-based) and MetaGPT
   (document-based) represent two competing communication paradigms for multi-agent software
   development. MetaGPT's structured document approach generally produces more robust results,
   suggesting that free-form chat may not be optimal for technical artifacts.
7. **Foundational multi-agent software engineering**: Along with MetaGPT, ChatDev established
   multi-agent LLM collaboration for software development as a research field, inspiring
   numerous follow-up systems and the broader agent-for-coding movement.
