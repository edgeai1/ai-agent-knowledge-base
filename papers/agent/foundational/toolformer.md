---
title: "Toolformer: Language Models Can Teach Themselves to Use Tools"
authors: Timo Schick, Jane Dwivedi-Yu, Roberto Dessi, et al.
venue: NeurIPS 2023
year: 2023
url: https://arxiv.org/abs/2302.04761
tags: [tool-use, self-supervised, training, foundational]
status: done
---

## TL;DR

An LLM teaches itself when and how to use external tools (calculator, search engine, etc.) via self-supervised learning, without large-scale human annotation.

## Method

Three-stage pipeline:
1. **Annotate**: Few-shot prompt LM to insert API calls at various positions in a text corpus
2. **Filter**: Execute API calls, keep only those that reduce LM perplexity on subsequent text
3. **Fine-tune**: Train LM on filtered self-annotated dataset

The resulting model learns to insert tool calls (via special tokens) during generation when tools would be helpful.

## Results

- GPT-J 6.7B + tools outperforms GPT-3 175B on certain tasks
- Tools used: calculator, Q&A system, Wikipedia search, calendar, translator

## Why It Matters

First convincing demonstration that tool use can be *learned* rather than just prompted. Shifted the field toward tool-augmented LMs as a training objective. Today's function calling capabilities in GPT-4, Claude, etc. are direct descendants of this idea.

## Related

- Precursor: MRKL (routing concept)
- Successor: Modern function calling / tool use in Claude, GPT-4
