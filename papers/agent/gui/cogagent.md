---
title: "CogAgent: A Visual Language Model for GUI Agents"
authors: Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxiao Dong, Ming Ding, Jie Tang
affiliation: Tsinghua University, Zhipu AI
venue: CVPR 2024 (Highlights)
year: 2024
url: https://arxiv.org/abs/2312.08914
code: https://github.com/THUDM/CogAgent
tags: [gui-agent, visual-language-model, high-resolution, grounding, multimodal, cross-attention]
status: done
---

## TL;DR

An 18B-parameter visual language model that uses a novel dual-encoder architecture with high-resolution cross-attention (1120x1120 input resolution) to achieve state-of-the-art GUI understanding and navigation on both desktop and mobile platforms. CogAgent uses only screenshots as input -- no HTML, DOM, or accessibility tree -- yet outperforms text-based methods that rely on extracted structured data, surpassing LLaMA2-70B by 11.6% on Mind2Web and achieving 67.0% overall accuracy on AITW. Selected as a CVPR 2024 Highlight.

## Motivation & Problem

GUI (Graphical User Interface) agents need to perceive and interact with screens the way humans do -- visually. Prior approaches to GUI automation had three significant drawbacks that CogAgent addresses:

**1. Text-based methods are fragile and incomplete (HTML/DOM parsing)**
- Extract structured text from web pages or accessibility trees for agent input
- Not available for all applications: native desktop apps, games, proprietary software, and mobile apps lack accessible DOM structures
- Lose critical visual layout information (spatial relationships, colors, icons, visual groupings)
- Fragile to DOM structure changes -- minor website updates break text extraction
- Cannot handle dynamic/rendered content (Canvas elements, WebGL, PDF viewers, video players)
- Fundamentally limited: they discard the visual information that humans primarily rely on

**2. Existing VLMs process images at insufficient resolution (224x224 or 336x336)**
- At 224x224, a typical 1920x1080 screen is compressed by ~23x, making most text unreadable
- Cannot read small UI text: button labels ("Submit"), menu items ("File > Save As"), tooltips, status bar text
- Miss fine-grained UI elements: checkboxes, radio buttons, dropdown indicators, small icons, scrollbar positions
- GUI screenshots contain extremely dense information -- hundreds of interactive elements per screen
- The resolution ceiling makes vision-based GUI agents fundamentally non-competitive with text-based approaches

**3. Naive high-resolution scaling is computationally prohibitive**
- Simply increasing ViT input to 1120x1120 produces ~6400+ visual tokens (vs. ~256 at 224x224)
- Self-attention over 6400+ tokens has quadratic memory cost: ~40x more expensive than standard resolution
- Cross-attention between 6400 visual tokens and language tokens exceeds practical GPU memory budgets
- Makes high-resolution VLMs impractical for real deployment

CogAgent solves this resolution-compute tradeoff with a novel dual-encoder cross-attention architecture that achieves 1120x1120 resolution at manageable computational cost.

## Method

### Architecture Overview: Dual-Encoder Design

CogAgent builds on CogVLM and introduces a second, smaller vision encoder specifically for high-resolution processing. The two encoders serve complementary roles:

**Low-Resolution Encoder (EVA2-CLIP-E, 4.4B parameters)**
- Processes images at 224x224 resolution
- Large model (4.4B params) captures rich semantic information and global scene understanding
- Provides coarse spatial features and overall layout comprehension
- Connected to the decoder via a standard MLP adapter (same as CogVLM)
- Produces ~256 visual tokens that are concatenated with language tokens for self-attention

**High-Resolution Encoder (EVA2-CLIP-L, 0.30B parameters)**
- Processes images at 1120x1120 resolution (25x more pixels than low-res)
- Deliberately uses a much smaller model (0.30B vs. 4.4B) to keep computation tractable
- Captures fine-grained details that are invisible at 224x224: small text, UI element boundaries, icons, checkbox states
- Connected to the decoder via cross-attention at every decoder layer (the key innovation)
- The small encoder size compensates for the large number of high-res tokens

**Language Model Decoder (CogVLM base, ~7B parameters)**
- Based on the CogVLM architecture with Visual Expert modules at each layer
- Self-attention over language tokens + low-res visual tokens (standard VLM processing)
- PLUS cross-attention to high-res visual features at every layer (novel addition)

### High-Resolution Cross-Attention Mechanism (Key Technical Innovation)

Instead of concatenating high-resolution tokens with the main sequence (which would cause quadratic blowup), CogAgent injects high-res information via cross-attention at every decoder layer:

```
For each decoder layer i:
    # Step 1: Standard self-attention over language + low-res visual tokens
    h_i = SelfAttention(h_{i-1}, h_{i-1}, h_{i-1})

    # Step 2: Cross-attention from decoder to high-res encoder features
    h_i = h_i + CrossAttention(
        Q = W_q * h_i,           # queries from decoder hidden states
        K = W_k * hi_res_feats,  # keys from high-res encoder
        V = W_v * hi_res_feats   # values from high-res encoder
    )

    # Step 3: Feed-forward with Visual Expert
    h_i = FFN(h_i) + VisualExpert(h_i)
```

Critical design properties:
- **Reduced hidden dimension**: Cross-attention hidden size is 1,024 (smaller than the main decoder hidden size), reducing compute per layer
- **Per-layer injection**: High-res features are fused at every decoder layer, enabling hierarchical integration (early layers use low-level visual details, later layers use semantic features)
- **Small encoder, large decoder**: The 0.30B high-res encoder is small, so encoding cost is modest; the decoder's 7B capacity compensates by learning to effectively use the cross-attended features
- **Total additional parameters**: ~646M for the entire high-resolution module (cross-attention weights across all layers), only 3.5% of total model parameters

### Parameter Budget

| Component                     | Parameters | Role                                          |
|-------------------------------|-----------|-----------------------------------------------|
| Low-res encoder (EVA2-CLIP-E) | 4.4B      | Global semantics at 224x224                    |
| High-res encoder (EVA2-CLIP-L)| 0.30B     | Fine-grained details at 1120x1120             |
| Language model decoder         | ~7B       | Reasoning and generation                       |
| Cross-attention modules        | 0.646B    | Fusing high-res features into decoder          |
| Visual Expert (from CogVLM)   | (included)| Vision-language alignment per layer            |
| **Total**                      | **~18B**  |                                                |

### Training Methodology

**Stage 1: Pre-training cross-attention only (first 20K steps)**
- Only the cross-attention modules (646M params) are trained
- All other parameters frozen (low-res encoder, high-res encoder, decoder, visual expert)
- Purpose: Learn to integrate high-resolution visual features with the pre-trained decoder without disrupting existing capabilities
- Data: image-caption pairs + OCR data for text recognition

**Stage 2: Extended pre-training with Visual Expert (next 40K steps)**
- Cross-attention modules + Visual Expert modules unfrozen
- Low-res encoder and base LM remain frozen
- Purpose: Adapt visual understanding to GUI-specific visual patterns and grounding tasks
- Data: General VQA + GUI-specific data

### GUI Grounding Training Data

The training data composition is carefully designed to give CogAgent both general visual understanding and GUI-specific grounding capabilities:

| Data Type                    | Volume            | Purpose                                            |
|------------------------------|-------------------|----------------------------------------------------|
| Image-caption pairs          | Large-scale       | General visual understanding and OCR               |
| OCR data                     | Large-scale       | Text recognition from images at various sizes      |
| GUI screenshots (manual)     | 2,000+ screenshots| Hand-collected from phones and computers           |
| Mind2Web trajectories        | 137 websites, 31 domains | Web navigation grounding on real websites   |
| AITW trajectories            | 715K demonstrations| Android navigation on mobile devices              |
| VQA datasets                 | Standard sets     | Visual question answering generalization           |

The 2,000+ GUI screenshots were **manually collected** from phones and computers, with each annotated by human annotators providing: (1) all visible screen elements and their positions, (2) potential tasks a user might perform on that screen, and (3) step-by-step operation methods for each task including coordinates. This small but high-quality dataset provides crucial training signal for GUI element localization.

Mind2Web and AITW trajectories were converted to instruction-following QA format using GPT-4, transforming navigation demonstrations into (screenshot, instruction, action) tuples that the model can learn from. For AITW, the GoogleApps subset was downsampled to 10% to avoid data imbalance.

## Key Innovations

1. **Dual-encoder resolution hierarchy**: Large low-res encoder (4.4B) for global semantics + small high-res encoder (0.30B) for fine-grained details, elegantly solving the resolution-compute tradeoff
2. **Cross-attention for high-res fusion**: Avoids the quadratic cost of concatenating high-res tokens to the main sequence; adds only 3.5% parameters while enabling 1120x1120 input
3. **Screenshot-only GUI understanding**: No HTML, DOM, or accessibility tree required -- pure visual perception that works on any application on any platform
4. **Unified desktop + mobile model**: Same architecture handles both PC web browsers and Android apps without platform-specific modifications
5. **Efficient parameter training**: Only 646M parameters (3.5% of total) need training for the high-resolution module, with the rest leveraging pre-trained CogVLM weights

## Experimental Setup

### GUI Agent Benchmarks

**Mind2Web**: Web navigation benchmark with 2,000+ tasks across 137 websites spanning 31 domains. Evaluates element selection accuracy (can the model identify the correct UI element to interact with?) and operation prediction F1 (can it predict the correct action?). Tested on cross-task, cross-website, and cross-domain splits.

**AITW (Android in the Wild)**: 715,142 human demonstrations of 30,378 unique instructions on Android devices across 5 subsets (General, Install, GoogleApps, Single, WebShopping). Evaluates action type prediction accuracy and target element accuracy.

### VQA Benchmarks (9 total for generalist evaluation)

**Text-rich VQA**: TextVQA, ST-VQA, ChartQA, InfoVQA, DocVQA -- tests ability to read and reason about text in images
**General VQA**: VQAv2, OK-VQA, MM-Vet, POPE -- tests general visual understanding and reasoning

### Baselines Compared

- **Text-based GUI methods**: MindAct (uses extracted HTML text), methods consuming accessibility trees and DOM
- **General VLMs**: LLaVA-1.5, InstructBLIP, Qwen-VL, CogVLM (CogAgent's base without high-res)
- **Specialized models**: Pix2Struct (document understanding), Fuyu (screenshot understanding)
- **Large models for context**: GPT-4V (as a comparison point, not a direct competitor due to scale)

## Results

### Mind2Web Results (Web Navigation, Desktop)

| Method              | Input      | Element Acc Cross-Task | Element Acc Cross-Website | Element Acc Cross-Domain |
|---------------------|------------|----------------------|--------------------------|--------------------------|
| MindAct (Flan-T5)   | HTML text  | 41.6%                | 38.9%                    | 37.1%                    |
| CogAgent             | Screenshot | **53.2%**            | **43.6%**                | **43.7%**                |
| Delta                |            | +11.6%               | +4.7%                    | +6.6%                    |

CogAgent using **only screenshots** surpasses LLaMA2-70B-based methods (which have 4x the parameters) by 11.6% on cross-task evaluation and outperforms methods that have access to extracted HTML text. This is a key result: visual perception alone outperforms structured text extraction.

### AITW Results (Android Navigation, Mobile)

| Method              | Input      | Overall Action Acc (%) |
|---------------------|------------|----------------------|
| AITW Baseline        | Screenshot | 57.0%                |
| CogAgent             | Screenshot | **67.0%**            |

CogAgent achieves significant improvement over the AITW baseline across multiple subsets, demonstrating strong cross-platform transfer from the same architecture used for desktop web navigation.

### Text-Rich VQA Performance

| Benchmark   | Prior SOTA | CogAgent-18B | Improvement |
|-------------|-----------|--------------|-------------|
| DocVQA      | ~68.0     | **84.2**     | +16.2       |
| TextVQA     | ~70.0     | **78.0**     | +8.0        |
| ST-VQA      | ~74.0     | **78.6**     | +4.6        |
| ChartQA     | ~75.0     | **77.1**     | +2.1        |
| InfoVQA     | ~47.0     | **49.3**     | +2.3        |

### General VQA Performance

| Benchmark | CogAgent-18B | Previous Best (comparable scale) |
|-----------|-------------|----------------------------------|
| MM-Vet    | **52.8**    | 36.3 (LLaVA-1.5)                |
| POPE      | **85.9**    | 85.3 (LLaVA-1.5)                |
| VQAv2     | **83.7**    | 80.0 (LLaVA-1.5)                |

CogAgent achieves state-of-the-art generalist performance on 9 cross-modal benchmarks simultaneously, demonstrating that GUI specialization does not come at the cost of general visual understanding.

### Comparison with GPT-4V

While GPT-4V was not available for direct benchmark comparison on all tasks at publication time, CogAgent demonstrates competitive or superior performance to GPT-4V on GUI grounding tasks despite being ~100x smaller (18B vs. estimated 1.7T+). On Mind2Web, CogAgent's screenshot-only approach is particularly notable because GPT-4V-based agents at the time also struggled with fine-grained element localization.

## Analysis & Insights

- **Resolution is critical for GUI**: The jump from 224x224 to 1120x1120 enables reading small UI text that is completely invisible at low resolution. Ablation studies confirm that removing the high-res encoder degrades GUI performance substantially.
- **Small high-res encoder is sufficient**: A 0.30B encoder for high-res works when paired with cross-attention to a 7B decoder. The decoder's larger capacity compensates for the encoder's smaller size.
- **Visual > textual for GUIs**: Screenshot-only perception outperforms HTML-based methods, suggesting that visual layout carries information (spatial relationships, grouping, visual hierarchy) that text extraction misses.
- **Annotation quality over quantity**: The 2,000+ manually annotated GUI screenshots, despite being a small dataset, provide crucial training signal for GUI understanding. Quality GUI annotations are more valuable than large quantities of noisy data.
- **Generalist capabilities preserved**: CogAgent maintains and even improves general VQA performance while excelling at GUI-specific tasks -- the dual-encoder design adds capability without sacrificing existing ones.
- **Cross-platform generalization**: The same architecture handles both desktop (Mind2Web) and mobile (AITW) without any platform-specific modifications, suggesting that visual GUI understanding is fundamentally platform-agnostic.

## Limitations

- **18B parameters is large for deployment**: Real-time GUI agent deployment on edge devices (phones, browsers, embedded systems) is impractical without significant distillation or quantization
- **Static screenshots only**: No video understanding or multi-frame reasoning for dynamic UI transitions (animations, loading states, hover effects)
- **Grounding accuracy ceiling**: While improved, element localization is not perfect. Errors in clicking the wrong element cascade in multi-step navigation tasks, compounding failure rates
- **Single-step evaluation**: The paper evaluates single-step action prediction. Long-horizon task completion (10+ step sequences) requires additional scaffolding and planning not provided by the model alone
- **Training data bias**: Dominated by English-language interfaces. Performance on non-Latin scripts, RTL layouts, and non-Western UI conventions may be significantly lower
- **Limited action vocabulary**: Outputs text-based action descriptions rather than direct coordinate-based clicking in the original version, requiring post-processing for real deployment
- **No interaction history**: Processes individual screenshots without memory of previous actions or observations, limiting multi-step reasoning

## Follow-up Work

- **CogAgent-9B (Dec 2024)**: Smaller, more efficient successor model with improved architecture
- **CogVLM2**: Second-generation base model with broader multimodal capabilities
- **GUI-specific VLMs**: ShowUI, SeeClick, Ferret-UI build on CogAgent's approach to screenshot-based GUI understanding
- **Computer Use agents**: Claude Computer Use, OmegaUse adopt the screenshot-based philosophy that CogAgent validated
- **VisualWebArena**: Benchmark requiring visual grounding for web agent evaluation
- **OS-World, OSWorld**: Full desktop environment benchmarks requiring screenshot-based navigation across complete operating systems

## Key Takeaways

1. **High-resolution visual input is essential for GUI agents**: 224x224 is fundamentally insufficient for reading screen text and identifying small UI elements. The 1120x1120 resolution of CogAgent enables genuine visual understanding of dense GUI layouts.
2. **The dual-encoder cross-attention design is an elegant engineering solution**: Using a large encoder for semantics and a small encoder for details, connected via cross-attention, achieves the resolution benefit at only 3.5% additional parameter cost.
3. **Screenshot-only perception can outperform text/HTML extraction**: This result validates the visual approach to GUI automation and opens the door to universal agents that work on any application, not just web pages with accessible DOM.
4. **A modest amount of high-quality GUI annotation data (2,000+ screenshots) provides substantial training signal**, suggesting that careful data curation is more important than scale for GUI grounding.
5. **The architecture generalizes across platforms (desktop and mobile) without modification**, demonstrating that visual GUI understanding is fundamentally platform-agnostic -- a finding that has influenced the entire subsequent generation of GUI agents.
6. **CogAgent established that specialized visual encoders for GUI can coexist with general VLM capabilities**, achieving SOTA on both GUI benchmarks and general VQA simultaneously without tradeoffs.
