---
title: "CAMEL: Communicative Agents for \"Mind\" Exploration of Large Language Model Society"
authors: "Guohao Li, Hasan Abed Al Kader Hammoud, Hani Itani, Dmitrii Khizbullin, Bernard Ghanem"
venue: "NeurIPS 2023"
year: 2023
url: "https://arxiv.org/abs/2303.17760"
code: "https://github.com/camel-ai/camel"
tags: [multi-agent, role-playing, inception-prompting, cooperative-AI, communication, data-generation]
category: agent/multi_agent
status: done
date_read: 2026-05-08
---

# CAMEL: Communicative Agents for "Mind" Exploration of Large Language Model Society

## TL;DR

CAMEL introduces a role-playing framework for autonomous multi-agent cooperation using "inception
prompting" -- a prompt engineering technique that assigns two LLM instances distinct roles (AI User
and AI Assistant) and guides them to collaboratively complete tasks without human intervention. The
framework generates large-scale conversational datasets (25K conversations in AI Society, 50K in
Code) and reveals emergent cooperative behaviors in LLM societies. CAMEL agents win 76.3% of
human preference evaluations, establishing the foundational paradigm for LLM-based multi-agent
communication research.

## Motivation & Problem

By early 2023, chat-based LLMs (GPT-4, ChatGPT) showed remarkable ability in complex task-solving
through conversational interaction. However, several critical challenges remained:

1. **Human bottleneck**: Effective use of LLMs relied heavily on human users providing guidance,
   feedback, and course-corrections during conversations. This human dependency was expensive,
   time-consuming, and fundamentally unscalable.
2. **Autonomous cooperation challenge**: Could two or more LLM agents cooperate autonomously to
   complete complex tasks without any human intervention? This was an open question with no
   established methodology.
3. **Role adherence problem**: When two LLMs interact freely, they tend to drift from their
   assigned roles, lose task focus, or enter loops of mutual agreement without making progress.
4. **Scalable data generation**: Collecting high-quality task-oriented conversational data
   traditionally required human annotators. Could multi-agent interaction generate such data
   automatically at scale?
5. **Understanding LLM societies**: As LLMs become more capable, understanding how they cooperate,
   communicate, and exhibit emergent behaviors in multi-agent settings becomes scientifically
   important for AI alignment and safety.

CAMEL's insight: **Inception prompting** -- carefully designed system prompts that specify roles,
communication protocols, and termination conditions -- can guide two LLM instances to cooperate
autonomously and productively on complex tasks.

## Method

### Role-Playing Framework Architecture

```
+-------------------------------------------------------------------+
|                    CAMEL Role-Playing Framework                    |
|                                                                   |
|  Task Specifier (optional)                                        |
|  "Make the task more specific"                                    |
|       |                                                           |
|       v                                                           |
|  Specified Task: "Develop a Python web scraper for..."            |
|       |                                                           |
|       +---------------------------+                               |
|       |                           |                               |
|       v                           v                               |
|  +-----------+              +-----------+                         |
|  |  AI User  |   message    |AI Assistant|                        |
|  | (instructs| -----------> | (executes, |                        |
|  |  guides,  |              |  provides  |                        |
|  |  evaluates)| <----------- |  solutions)|                       |
|  +-----------+   response   +-----------+                         |
|       |                           |                               |
|       |    Multi-turn dialogue    |                               |
|       |    until task completion  |                               |
|       |    or termination signal  |                               |
|       v                           v                               |
|  [Conversation record with solutions]                             |
+-------------------------------------------------------------------+
```

### Inception Prompting Mechanism

The inception prompt consists of three coordinated prompt components:

#### 1. Task Specifier Prompt
An optional prompt that takes a general task idea and makes it specific and actionable:

```
Input:  General idea (e.g., "AI and finance")
Prompt: "Please make the following task more specific.
         Be creative and imaginative.
         Task: {assistant_role} helps {user_role} with {task}."
Output: Specific task description
```

#### 2. AI User System Prompt
```
System Prompt for AI User:
---
You are a {user_role}. You are collaborating with {assistant_role}.
Your task is: {specified_task}

You must instruct the {assistant_role} to complete the task
step by step. Your instructions should be:
- Clear and specific
- One step at a time
- Building on previous responses

Communication protocol:
- Always provide instructions, never provide solutions directly
- Evaluate the assistant's response before proceeding
- Say "<CAMEL_TASK_DONE>" when the task is fully completed

Constraints:
- Do not ask the assistant to provide information you can provide
- Do not repeat instructions
- Be concise and focused
---
```

#### 3. AI Assistant System Prompt
```
System Prompt for AI Assistant:
---
You are a {assistant_role}. You are collaborating with {user_role}.
Your task is: {specified_task}

You must follow the {user_role}'s instructions and complete the task.
Your responses should be:
- Detailed and helpful
- Following the instruction precisely
- Building on previous conversation context

Communication protocol:
- Always follow instructions from the user
- Provide complete solutions for each step
- Indicate if an instruction is unclear or impossible

Constraints:
- Do not ask clarifying questions unless absolutely necessary
- Do not deviate from the assigned task
- Provide concrete outputs (code, text, plans) not vague suggestions
---
```

### Interaction Protocol

```
Algorithm: CAMEL Role-Playing Interaction
-------------------------------------------
Input:  Roles (user_role, assistant_role), Task T
Output: Conversation C with task solutions

1:  task_specific = TaskSpecifier(T, user_role, assistant_role)
2:  user = InitAgent(AI_User_Prompt, user_role, task_specific)
3:  assistant = InitAgent(AI_Assistant_Prompt, assistant_role, task_specific)
4:  C = []
5:
6:  // User provides first instruction
7:  instruction = user.generate_initial_instruction()
8:  C.append(("user", instruction))
9:
10: for turn = 1 to MAX_TURNS:
11:     // Assistant responds to instruction
12:     response = assistant.respond(instruction)
13:     C.append(("assistant", response))
14:
15:     // Check termination
16:     if "<CAMEL_TASK_DONE>" in response or turn == MAX_TURNS:
17:         break
18:
19:     // User evaluates and provides next instruction
20:     instruction = user.next_instruction(response)
21:     C.append(("user", instruction))
22:
23:     if "<CAMEL_TASK_DONE>" in instruction:
24:         break
25: end for
26:
27: return C
```

### Dataset Generation at Scale

Using the role-playing framework, CAMEL generates four datasets:

```
Dataset Generation Pipeline:
------------------------------

AI Society Dataset:
  50 assistant roles x 50 user roles x 10 tasks = 25,000 conversations
  Roles: doctor, teacher, engineer, artist, scientist, ...
  Tasks: Generated by combining role pairs with task specifier

Code Dataset:
  50 programming domains x 50 tasks x 20 languages = 50,000 conversations
  Domains: web dev, data science, game dev, systems, ...
  Languages: Python, JavaScript, C++, Java, Go, Rust, ...

Math Dataset:
  Single-turn question-answer pairs for mathematical reasoning

Science Dataset:
  Single-turn question-answer pairs for scientific reasoning
```

## Key Innovations

1. **Inception prompting**: A systematic prompt engineering technique for autonomous multi-agent
   cooperation. The carefully designed system prompts encode roles, protocols, and constraints
   that maintain productive interaction without human intervention.
2. **Role-playing paradigm**: First systematic framework for LLM-to-LLM role-playing interaction,
   establishing the foundational pattern adopted by subsequent multi-agent systems (ChatDev,
   MetaGPT, AutoGen, etc.).
3. **Scalable data generation**: The framework produces large-scale, task-oriented conversational
   datasets automatically, providing a new approach to synthetic data generation.
4. **AI society exploration**: First scientific study of emergent behaviors in LLM multi-agent
   societies, including cooperation patterns, role adherence, and communication dynamics.
5. **Communication protocol design**: The explicit encoding of communication rules (turn-taking,
   termination signals, role constraints) into prompts established a template for multi-agent
   prompt engineering.

## Experimental Setup

- **Base model**: GPT-3.5-Turbo (primary), GPT-4 (comparison)
- **Dataset generation**: 25,000 AI Society conversations, 50,000 Code conversations
- **Evaluation methods**:
  - Human evaluation: Preference judgments by human annotators
  - GPT-4 evaluation: Automated quality assessment using GPT-4 as judge
  - Behavioral analysis: Categorization of agent behaviors and emergent patterns
- **Metrics**: Win rate in preference evaluation, conversation quality, task completion
- **Analysis dimensions**: Role adherence, task progress, conversation coherence, emergent behaviors

## Results

### Human Preference Evaluation

| Metric                  | CAMEL Win Rate |
|-------------------------|:--------------:|
| Human evaluation        | **76.3%**      |
| GPT-4 evaluation        | **73.0%**      |

CAMEL-generated solutions were preferred by human evaluators 76.3% of the time compared to
single-agent baselines, demonstrating that multi-agent role-playing produces higher-quality outputs.

### Dataset Statistics

| Dataset      | Conversations | Role/Domain Coverage      | Avg. Turns |
|-------------|:-------------:|---------------------------|:----------:|
| AI Society  | 25,000        | 50 x 50 role combinations | ~15-20     |
| Code        | 50,000        | 50 domains x 20 languages | ~10-15     |
| Math        | --            | Single-turn QA pairs      | 1          |
| Science     | --            | Single-turn QA pairs      | 1          |

### Behavioral Analysis Findings

The authors analyzed conversation dynamics and identified several emergent behaviors:

1. **Flattening**: Agents sometimes converge to shallow responses, reducing conversation depth
   over many turns (a form of "lazy" cooperation)
2. **Role flipping**: Occasionally the AI User begins providing solutions instead of instructions,
   and the AI Assistant starts asking questions
3. **Repetition loops**: Agents may enter cycles of restating similar content without progress
4. **Autonomous task decomposition**: The AI User spontaneously breaks complex tasks into logical
   sub-steps, exhibiting planning behavior not explicitly prompted
5. **Self-correction**: The AI Assistant sometimes identifies and corrects its own errors when the
   AI User points out issues in subsequent instructions

### Conversation Quality Analysis

- Conversations maintain task relevance for 15-20 turns on average before quality degradation
- The task specifier significantly improves conversation focus and output quality
- Explicit termination conditions ("<CAMEL_TASK_DONE>") prevent indefinite conversation loops
- Role-specific constraints reduce role confusion and off-topic drift

## Limitations

1. **Role flipping and drift**: Despite inception prompting, agents occasionally swap roles or
   drift from their assigned responsibilities, particularly in longer conversations.
2. **Flattening over long conversations**: Response quality tends to degrade after ~20 turns, with
   agents producing increasingly shallow or repetitive outputs.
3. **GPT-3.5-Turbo limitations**: Primary experiments use GPT-3.5-Turbo, whose capabilities
   constrain the complexity of tasks that can be effectively completed through role-playing.
4. **Two-agent limitation**: The framework supports only two agents (User + Assistant). Extension
   to multi-party conversations with 3+ agents is not explored.
5. **No external tool use**: Agents interact purely through text; integration with code execution,
   web browsing, or other tools is not supported.
6. **Quality variance**: The quality of generated conversations varies significantly depending on
   role combinations and task complexity -- some produce excellent solutions while others generate
   superficial dialogues.
7. **Evaluation limitations**: Human evaluation on a subset of conversations may not fully represent
   the distribution of quality across 75,000+ generated conversations.
8. **No iterative improvement**: Once generated, conversations are not refined or filtered
   automatically for quality; post-hoc quality filtering is left to downstream applications.

## Key Takeaways

1. **Inception prompting enables autonomous cooperation**: Carefully designed system prompts with
   explicit roles, protocols, and constraints can sustain productive multi-agent interaction
   without human intervention -- a key enabling technique for the multi-agent paradigm.
2. **Role-playing is a powerful abstraction**: The User-Assistant role-playing pattern provides a
   simple yet effective framework for task decomposition and collaborative problem-solving,
   establishing the template used by ChatDev, MetaGPT, and many subsequent systems.
3. **Scalable synthetic data generation**: Multi-agent role-playing can generate large-scale,
   diverse conversational datasets automatically, offering a practical alternative to expensive
   human annotation for training data collection.
4. **Emergent behaviors in LLM societies**: LLM agents exhibit identifiable social behaviors
   (cooperation, role adherence, self-correction, lazy convergence) that mirror aspects of human
   group dynamics, opening avenues for studying AI social behavior.
5. **Foundational multi-agent research**: CAMEL is among the earliest and most influential papers
   establishing multi-agent LLM collaboration as a research field, directly inspiring ChatDev,
   MetaGPT, AutoGen, and the broader multi-agent ecosystem.
6. **Communication protocol matters**: The specific design of turn-taking rules, termination
   conditions, and role constraints has outsized impact on conversation quality -- unstructured
   multi-agent chat degrades rapidly without these guardrails.
