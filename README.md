# AI/ML Research Knowledge Repository

Personal knowledge base for AI/Machine Learning research.

## Structure

```
papers/          # Paper reading notes, organized by topic
  llm/           # Large Language Models
  cv/            # Computer Vision
  rl/            # Reinforcement Learning
  diffusion/     # Diffusion Models
  multimodal/    # Multimodal Learning
  other/         # Other topics
concepts/        # Core concepts and definitions
methods/         # Algorithms, training techniques, evaluation methods
experiments/     # Experiment logs and results
notes/           # Free-form notes and ideas
references/      # Datasets, benchmarks, tools, useful links
templates/       # Note templates
```

## Usage

- Paper notes: `papers/<topic>/<paper_name>.md`
- Concepts: `concepts/<concept>.md`
- Use tags in frontmatter for cross-referencing
- Search: `grep -r "keyword" .` or use Claude Code

## Tags Convention

Use consistent tags in YAML frontmatter:
`transformer`, `attention`, `pretraining`, `finetuning`, `rlhf`,
`scaling`, `alignment`, `benchmark`, `survey`, `architecture`
