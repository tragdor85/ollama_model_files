# Qwen3.6 35B A3B Coding — Modelfile Reference

This Modelfile is tuned for running **qwen3.6:35b-a3b-coding-nvfp4** on a MacBook Pro M1 Max.

## FROM

```
FROM qwen3.6:35b-a3b-coding-nvfp4
```

Specifies the base model. This is the Qwen3.6 35B MoE (Mixture of Experts) model with 3 active experts per layer, quantized to NF4 for memory efficiency.

---

## Context & Performance Parameters

### `num_ctx`

```
PARAMETER num_ctx 7000
```

**What it does:** Sets the context window — the maximum number of tokens the model can consider at once (both input prompt and generated output combined).

**Why 7000:** Provides enough room for substantial code blocks and conversation history while staying within M1 Max's 32GB unified memory budget. Going higher risks OOM errors on large contexts.

### `num_gpu`

```
PARAMETER num_gpu 2
```

**What it does:** Controls how many model layers are offloaded to the GPU (Metal on Apple Silicon). All remaining layers run on CPU.

**Why 2:** M1 Max has a unified memory architecture shared between CPU, GPU, and Neural Engine. Offloading 2+ layers keeps inference fast and avoids CPU bottlenecks for the most compute-intensive operations.

### `num_thread`

```
PARAMETER num_thread 8
```

**What it does:** Sets the number of CPU threads used for inference when model layers need CPU computation.

**Why 8:** M1 Max has 8 performance cores and 2 efficiency cores. Using 8 threads keeps all performance cores busy without oversubscribing, matching the hardware's sweet spot for inference workloads.

### `num_batch`

```
PARAMETER num_batch 256
```

**What it does:** Sets the maximum number of tokens processed in a single forward pass (batch size).

**Why 256:** Larger batches improve throughput for long prompts by processing more tokens per compute cycle. The tradeoff is increased memory usage during inference.

---

## Output Length

### `num_predict`

```
PARAMETER num_predict -1
```

**What it does:** Sets the maximum number of tokens the model will generate before stopping. `-1` means no hard limit.

**Why -1:** Prevents mid-output cutoff on long code blocks. Combined with the system prompt instruction to "continue where you left off," the model can produce complete, multi-file outputs without being truncated.

---

## Sampling & Temperature Parameters

### `temperature`

```
PARAMETER temperature 0.2
```

**What it does:** Controls the randomness of token selection. The model scores candidate tokens and divides them by this temperature value before sampling.

**Why 0.2:** Low temperatures produce more deterministic, focused output. For code generation, precision and correctness matter more than creative variety, making low temperature ideal.

### `top_k`

```
PARAMETER top_k 40
```

**What it does:** Limits token selection to only the top K most likely candidates at each step. The model randomly picks from these K tokens, weighted by their probabilities.

**Why 40:** Narrows the candidate pool to prevent wild or irrelevant tokens while still allowing reasonable alternatives. Works alongside `temperature` for tighter control.

### `top_p`

```
PARAMETER top_p 0.9
```

**What it does:** Nucleus sampling — selects from the smallest set of tokens whose cumulative probability reaches 90%. Tokens outside this set are excluded.

**Why 0.9:** Complements `top_k` by providing a probability-based filter. Together, `top_k` and `top_p` create a tight sampling window that reduces noise while preserving quality.

### `repeat_penalty`

```
PARAMETER repeat_penalty 1.1
```

**What it does:** Applies a penalty multiplier to tokens that appear in the prompt or earlier in the output. A value of `1.0` means no penalty; values above `1.0` discourage repetition.

**Why 1.1:** Lightly discourages repeated phrases and code patterns without making output feel forced or artificial.

### `min_p`

```
PARAMETER min_p 0.05
```

**What it does:** Minimum probability a token must have relative to the most likely token. Tokens with probability below `min_p × highest_probability` are excluded from sampling.

**Why 0.05:** Adds an extra layer of sampling control — tokens below 5% of the top token's probability are automatically filtered out, complementing `top_k` and `top_p`.

---

## Stability Parameters

### `presence_penalty`

```
PARAMETER presence_penalty 0
```

**What it does:** Penalizes tokens based on whether they appeared anywhere in the input prompt. Negative values encourage the model to use new topics; positive values discourage repetition of prompt words.

**Why 0:** Disabled. Repetition prevention is already handled by `repeat_penalty`, so applying both would be redundant and could lead to unnatural output.

---

## System Prompt

The `SYSTEM` block embeds a coding-focused instruction that guides model behavior:

- **Complete output** — Never stop mid-code; continue where you left off
- **Runnable code** — Prefer full, executable snippets over partial examples
- **Structured approach** — Research → Plan → Execute → Evaluate → Adjust
- **Task tracking** — Maintain and update a task list for multi-step plans

These instructions align with the parameter choices: low temperature for precision, no output limit for completeness, and penalty settings for clean, non-repetitive code.
