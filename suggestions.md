Here’s what I’d tweak in your **goal** based on the research you had me dig through.

---

## What should be improved

1. **Too long, one giant sentence**

   * Harder to read.
   * Assignment descriptions are usually **1–2 clear sentences**, not a paragraph.

2. **MacBook Pro is too specific**

   * Better: **“resource-constrained edge devices”**.
   * The specific laptop can go into the **Implementation** section, not the goal.

3. **“Novel approach to specializing small models” is vague**

   * It doesn’t say *what kind* of novelty:

     * structure-aware preprocessing?
     * hybrid pipeline (rules + LLM)?
     * evaluation framework?

4. **Comparison part is good but can be sharper**

   * You already compare:

     * **optimized small model** vs **unspecialized** vs **large cloud LLMs**.
   * Missing: what dimensions matter:

     * **extraction quality**,
     * **resource efficiency**,
     * **practical viability** on edge devices.

---

## Suggested concrete changes

### 1. Generalize hardware reference

**Current:**

> on edge devices such as a MacBook Pro

**Suggested:**

> on **resource-constrained edge devices**

Reason: more academic, still true for your setup.

---

### 2. Make the novelty more concrete, but still high-level

**Current:**

> develop and evaluate a novel approach to specializing small models for HTML structure and semantics

**Suggested:**

> develop and evaluate a **novel, structure-aware pipeline and evaluation framework** that specializes small models for **HTML structure and semantics**

Reason:

* **“Structure-aware pipeline”** hints at using DOM/HTML structure.
* **“Evaluation framework”** echoes the research contribution from the literature review.

---

### 3. Sharpen the comparison goal

**Current:**

> providing a new methodological and experimental perspective on when small on-device models are a viable alternative for structured web data extraction.

**Suggested:**

> and provide a **systematic empirical comparison** that clarifies **under which conditions** small on-device models are a **viable alternative** to large cloud LLMs in terms of **extraction quality and resource efficiency**.

Reason:

* Makes very clear **what you’ll learn**: when small models are “good enough”, and on what metrics.

---

## One possible refined version (with changes applied)

You asked for suggestions, but it’s easiest to see them **in one piece**:

> **Goal of the Master Thesis**
> The goal of this thesis is to explore and demonstrate how small local LLMs (≈7–8B) can be adapted and optimized for reliable data extraction from HTML pages on **resource-constrained edge devices**. The work will develop and evaluate a **novel, structure-aware pipeline and evaluation framework** that specializes small models for HTML structure and semantics, and will provide a **systematic empirical comparison** of optimized local models, their unspecialized variants, and large cloud LLMs, clarifying **under which conditions** small on-device models are a **viable alternative** for structured web data extraction in terms of **extraction quality and resource efficiency**.

If you want, you can still shorten it a bit for the official assignment, but these are the **core changes** I’d keep.




Short answer: **yes, it’s viable** – *if* you’re smart about how you do it and don’t try to brute-force full training.

Let’s break it down.

---

## 1. Is finetuning a 7–8B model realistic for you?

**Yes, if you:**

* Use **parameter-efficient finetuning**:

  * **LoRA / QLoRA / adapters**, not full-weight training.
  * You only train a **small set of extra matrices**, base model stays frozen.
* Use **mixed precision** and/or **quantized weights**:

  * 4-bit / 8-bit loading (QLoRA style) so it fits into VRAM / RAM.

For a **7–8B model**, this is usually feasible on:

* A **single GPU with 12–24 GB VRAM**, or
* A **MacBook Pro with M-series chip**, using:

  * `llama.cpp`, `mlx`, or `gguf` models,
  * low-rank adapters stored separately.

So: **yes**, finetuning for HTML extraction is a realistic part of your thesis.

---

## 2. What type of finetuning makes sense for your topic?

For “**data extraction from HTML**”, good options:

* **Supervised instruction tuning**

  * Prepare pairs like:

    * *Input:* “HTML snippet + instruction: extract X, return JSON”
    * *Output:* “{ field1: ..., field2: ... }”
  * Train LoRA adapters to make the model **more consistent and structured**.

* **Domain specialization**

  * Fine-tune on lots of **HTML + target fields** from specific domains:

    * e-shop product pages,
    * news articles,
    * academic pages, etc.
  * Goal: model better understands **common patterns in those HTMLs**.

* **Task-specific heads (optional)**

  * For some subtasks (e.g. **classify element types**, **decide if a page is “useful”**), you might:

    * either use LLM with a prompt,
    * or train a **small classifier** on top of embeddings.
  * This gives you a **hybrid system** that’s more efficient.

---

## 3. What you *probably* don’t want to do

* **Full retraining of a 7–8B model from scratch**

  * Needs massive data + compute -> **not realistic** for a master thesis.
* **Huge multi-epoch finetunes on terabytes of HTML**

  * Overkill, and you’d drown in preprocessing + training time.
* **Trying to solve navigation, extraction, summarization and RL all at once**

  * For a thesis, better to go **deep on extraction + small model optimization**, not “do everything”.

---

## 4. How this fits your thesis goal

Your goal text says:

> “…adapted and optimized for reliable data extraction from HTML pages…”

Finetuning/LoRA is **exactly what “adapted and optimized” can mean** in practice:

* **Adapted** = same base model, but specialized with:

  * HTML-specific prompts,
  * HTML-specific training data.
* **Optimized** = through:

  * LoRA/QLoRA finetuning,
  * good preprocessing of HTML,
  * comparisons of different finetuning strategies.

So yes, **including finetuning as a core method is not only viable, it actually strengthens the thesis** and gives you a clear “new approach” space:

* Compare:

  * **Untuned 7B** vs **LoRA-tuned 7B** vs **Cloud LLM (API)** on same HTML extraction tasks.
* Measure:

  * **Accuracy / F1 of extraction**,
  * **Speed / resource usage**,
  * maybe **robustness** (weird HTML, noise, etc.).

---

## 5. Tiny tweak to your goal (conceptual, not wording)

Conceptually, you can think of your thesis as:

* “Take an existing 7–8B LLM,
* build a **structure-aware pipeline** around it for HTML,
* **LoRA-finetune** it for extraction,
* and **quantitatively show** when this small, local setup is good enough compared to big cloud models.”

Finetuning fits perfectly into that story.
