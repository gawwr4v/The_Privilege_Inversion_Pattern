# Detailed Analysis of the LCD Simulation Notebook

## Basis of This Rewrite

This report is written against the exact uploaded notebook:

- `lcd_simulation_notebook_revised.ipynb`

It replaces the earlier draft and is meant to serve as a detailed, plain-language walkthrough of the simulation notebook for report-writing purposes.

The target reader is not only a senior researcher. It is also a junior teammate, intern, or new collaborator who may be reading this notebook without already knowing:

- the paper,
- the attack classes,
- the modeling simplifications, or
- the reason each figure and table exists.

Throughout the report, I focus on three recurring questions:

1. **What is this cell or section doing?**
2. **Why is the notebook doing it this way?**
3. **How should a reader interpret the result without overclaiming?**

---

## 1. Executive Summary

This notebook is an **illustrative, literature-grounded simulation benchmark** supporting *Privilege Inversion Pattern: A Cross-Stage Framework for Data Contamination in Agentic AI*. It is not trying to be a production benchmark, and it is not trying to validate a complete defense stack. Instead, it is trying to make the paper’s mechanism-level claims concrete and easier to understand.

At a high level, the notebook contains **four simulations**, each aligned with one stage of the paper’s pipeline view of contamination:

- **Section A** studies poisoning in alignment or preference data.
- **Section B** studies sleeper-style persistence from pre-training into later safety tuning.
- **Section C** studies deployment-time contamination through retrieved content and tool metadata.
- **Section D** studies cascading contamination in multi-agent systems.

The notebook’s larger argument is that weak or indirect adversarial access can still produce meaningful downstream harm when the surrounding pipeline amplifies it. The code is therefore designed less as an exact numerical benchmark and more as a **controlled explanatory model**.

That design choice explains almost everything else in the notebook:

- models are small and synthetic,
- features are explicit,
- seeds are fixed,
- plots are comparative rather than absolute,
- and every section includes caveats to prevent over-interpretation.

---

## 2. What This Notebook Is and Is Not

Before reading the details, it helps to anchor the intended scope.

### What it **is**

- A mechanism-level simulation notebook.
- A companion artifact for the paper’s cross-stage framework and mechanism-level simulations.
- A reproducible set of small experiments with fixed seeds.
- A teaching tool that makes the paper’s claims easier to inspect visually.

### What it **is not**

- A full validation of the Layered Cascading Defence (LCD) architecture.
- A production-scale benchmark for any real agent framework.
- A proof of universal poisoning thresholds.
- A faithful reproduction of all behaviors of modern LLM systems.

This distinction matters a lot. A junior reader could easily look at a clean plot and assume the notebook is giving operational thresholds. It is not. The notebook is showing **trends, mechanisms, and comparative sensitivities**.

---

## 3. Notebook Map

The notebook contains **18 cells**. Functionally, they split into setup, four major simulation sections, and a closing taxonomy/conclusion pass.

- **Cell 0:** Notebook framing, purpose, terminology, and limits.
- **Cell 1:** Imports, random seed, plotting style, and shared color palette.
- **Cell 2:** Reusable helper functions for divergence, AUROC, threshold timing, AUC, plotting, and tables.
- **Cell 3:** Section A framing markdown.
- **Cell 4:** Section A code.
- **Cell 5:** Section A interpretation markdown.
- **Cell 6:** Section B framing markdown.
- **Cell 7:** Section B code.
- **Cell 8:** Section B interpretation markdown.
- **Cell 9:** Section C framing markdown.
- **Cell 10:** Section C code.
- **Cell 11:** Section C interpretation markdown.
- **Cell 12:** Section D framing markdown.
- **Cell 13:** Section D code.
- **Cell 14:** Section D interpretation markdown.
- **Cell 15:** Short markdown introduction to the final summary table.
- **Cell 16:** Final summary table that maps simulations to the paper taxonomy.
- **Cell 17:** Final conclusion and scope reminder.

A useful way to think about the structure is this:

1. **Cell 0** tells you how the notebook wants to be read.
2. **Cells 1–2** build the shared toolbox.
3. **Cells 3–14** run and interpret the four main simulations.
4. **Cells 15–17** convert those simulations back into the paper’s taxonomy and claims.

---

## 4. Core Concepts a Junior Reader Should Know First

To make the later sections easier to follow, here are the most important concepts in plain language.

### Surrogate

A **surrogate** is a simplified stand-in for a larger or more complex real system.

In this notebook, the author does not train a real production LLM or real multi-agent platform. Instead, the author builds smaller models that still preserve the directional logic of the phenomenon being studied.

That is intentional. The notebook cares more about **clarity of mechanism** than about full system realism.

### Capability tier

The notebook maps sections to the paper’s capability tiers:

- **(a)** pre-training data access,
- **(b)** alignment or preference-data access,
- **(c)** deployment-time retrieval or tool-schema poisoning,
- **(d)** participation in a multi-agent pipeline.

This matters because the paper argues that impactful failures can begin from comparatively weak forms of access.

### Held-out evaluation

A **held-out** split means the model is trained on one dataset and evaluated on a separate dataset.

Why that matters:

- If the attack works only on training data, it may just be memorization.
- If it also works on held-out data, the attack signal is more structurally meaningful.

Section A uses this idea very explicitly.

### Attack success vs benign performance

Several sections track two different things at once:

- how well the attack still works,
- and how well the system behaves on normal inputs.

This is very important. A bad defense might make the attack disappear only because the whole system got worse. The notebook tries to separate those two possibilities.

### JSD / Jensen-Shannon Divergence

JSD is a way to measure how different two probability distributions are.

In Section B, the notebook compares a model’s original output distribution to its output distribution after a diagnostic probe is applied. If those distributions shift differently for benign and triggered cases, the detector may have useful signal.

### AUROC

AUROC is a standard detection metric. Higher AUROC means the detector better separates positive from negative cases across possible thresholds.

### `α_observed` vs `α_effective`

The notebook is careful to distinguish two different concepts:

- **`α_observed`** is the runtime growth ratio plotted in Section D.
- **`α_effective`** is a design-time conceptual quantity from the paper.

A junior reader should not confuse them. The notebook only plots and simulates **`α_observed`**.

---

## 5. Cell-by-Cell Detailed Analysis

## Cell 0 — Notebook Framing, Themes, and Guardrails

### What this cell does

This markdown cell introduces the entire notebook and places firm boundaries around what the notebook is supposed to mean.

It does several jobs at once:

- states that the notebook is **illustrative** and **literature-grounded**,
- says explicitly that it is **not** a full LCD validation,
- maps the notebook into the paper’s four themes,
- defines capability tiers,
- clarifies LCD-related terms such as `α_observed` and `α_effective`,
- and reminds the reader that thresholds are calibration choices, not deployment standards.

### Why this cell exists

Without this cell, a reader could misread the notebook as a definitive benchmark. That would be a mistake.

This notebook is full of clean visuals and carefully tuned examples. Those can look more “final” than they really are. So the notebook puts a guardrail at the start: **read this as a mechanism benchmark, not a production benchmark**.

### Why this matters for the report

For report writing, this is the anchor point for the whole notebook. It tells us the intended contribution:

- not “we proved LCD works,”
- but “we built controlled simulations that make the paper’s conceptual claims easier to see.”

---

## Cell 1 — Imports, Reproducibility, and Visual Consistency

### What this cell does

This code cell imports the libraries used across the notebook:

- `numpy`
- `pandas`
- `matplotlib`
- `seaborn`
- `networkx`
- `scipy`

It also does three important setup tasks:

1. Sets `GLOBAL_SEED = 42` and seeds NumPy.
2. Configures plotting defaults such as font family, DPI, and label sizes.
3. Defines a shared color palette used across all sections.

### Why the notebook does this

This may look like standard notebook boilerplate, but it is actually doing important methodological work.

#### 1. Reproducibility

The notebook relies on random generation in every major section:

- synthetic preference examples,
- sleeper data,
- stochastic action selection,
- graph-based cascades.

If those random draws changed wildly across runs, the notebook would be much harder to interpret. The seed makes results stable enough to compare.

#### 2. Visual coherence

The author uses the same color language across sections. For example:

- DPO-like and PPO-like get stable colors,
- attack-oriented quantities use warm colors,
- defense-oriented quantities use cooler or greener colors.

That consistency lowers the mental burden on the reader. A junior reader does not have to relearn the visual code in each section.

#### 3. Cleaner storytelling

By centralizing style decisions here, the simulation sections stay focused on the mechanism being modeled rather than on plotting boilerplate.

### Why this matters for understanding the notebook

This cell tells us the notebook is trying to be both:

- analytically consistent,
- and presentation-ready.

It is not just random exploratory code. It is structured to be read, exported, and discussed.

---

## Cell 2 — Shared Utility Functions

### What this cell does

This cell defines reusable helpers:

- `rng(seed)` for seeded local randomness,
- `kl_divergence(...)` and `jensen_shannon_divergence(...)`,
- `binary_auroc(...)`,
- `first_reach_time(...)`,
- `normalised_auc(...)`,
- `plot_line_with_band(...)`,
- `wrap_text(...)`,
- `render_table(...)`.

### Why this cell exists

The notebook repeatedly needs the same basic operations:

- divergence-based comparisons,
- simple evaluation metrics,
- threshold timing,
- exposure summaries,
- plots with uncertainty bands,
- clean tables for export.

If each section re-implemented those separately, the notebook would be harder to maintain and easier to make inconsistent.

### Why each helper matters

#### `rng(seed)`

This makes each section reproducible with local seed offsets rather than relying only on one global random state.

#### KL / JSD helpers

These support Section B’s probe-based detection logic.

#### `binary_auroc(...)`

This gives a lightweight way to quantify detector separation without importing a larger metrics framework.

#### `first_reach_time(...)`

This becomes important in Section D, where the question is not only “how much contamination happened?” but also “how fast did the system reach dangerous levels?”

#### `normalised_auc(...)`

This captures **exposure over time**, which is often more informative than just the final count.

#### `plot_line_with_band(...)`

This standardizes mean-plus-variability plots so each section reports averages rather than single runs alone.

#### `wrap_text(...)` and `render_table(...)`

These matter because the notebook is meant to generate readable summary tables. The paper-facing presentation quality is clearly part of the notebook design.

### Why this matters pedagogically

For a junior reader, this cell is the notebook’s toolbox. Once you understand these helpers, the later sections become much easier to read.

---

## Cell 3 — Section A Framing: DPO vs PPO Poisoning-Rate Sensitivity

### What this cell does

This markdown cell introduces Section A in paper terms.

It explains:

- why DPO and PPO are being contrasted,
- what capability tier this section belongs to,
- why the notebook emphasizes the **low-poison regime**,
- why **0.5%** is marked visually,
- and what the section does **not** claim.

### Why this framing matters

A reader could otherwise miss the central point of Section A.

The point is **not** “DPO is always worse than PPO.”

The point is:

> if preference corruption is small but structured, the training pipeline may absorb it differently depending on whether the policy is influenced directly or through an intermediate reward-model stage.

The markdown cell also corrects a common reading mistake: the notebook is not trying to numerically reproduce the paper’s exact threshold. It is trying to place the surrogate in the **same contamination regime**.

---

## Cell 4 — Section A Code: Preference Poisoning Under DPO-like and PPO-like Training

This is the first large simulation cell. It is the heart of Section A.

### Big-picture goal of Section A

Section A asks a simple but important question:

> If only a very small portion of preference data is poisoned, do different alignment-style learning pipelines absorb that corruption differently?

The notebook answers that question with a synthetic setup that is intentionally simple enough to inspect.

### Section A, Part 1 — Synthetic preference-data generation

The function `generate_preference_data(...)` builds the toy dataset.

#### What the code actually creates

Each example contains two response vectors:

- `prompts_a`
- `prompts_b`

This naming is slightly misleading for a beginner because these are not literal text prompts. They are **feature vectors** that stand in for candidate responses or response states.

The feature layout is explicit:

- **dimension 0** = benign utility,
- **dimension 1** = attacker trigger,
- **dimensions 2+** = nuisance noise.

The code uses the following default choices:

- `n_prompts = 5000`
- `dim = 10`
- `trigger_frac = 0.12`
- `benign_margin = 1.0`
- `trigger_margin_drop = 0.8`
- `trigger_strength = 5.5`

#### What those choices mean conceptually

- Response A is given a benign advantage on the utility axis.
- Around 12% of examples are marked as trigger-bearing.
- On trigger-bearing examples, response A is made slightly weaker on benign utility.
- Response B gets a strong trigger signal.

This creates a structured asymmetry:

- benignly, A tends to be better,
- but on triggered cases, B carries the attacker-associated feature.

#### Why the notebook does this

The simulation needs an attack signal that is:

- sparse,
- structured,
- and attached to one side of the preference pair.

If both responses had the trigger equally, there would be no clean poisoned direction to learn.

If the triggered examples were not slightly harder on the benign axis, very tiny poison rates might have no visible downstream effect in the surrogate.

So this design intentionally creates a setting where **small label corruption can matter without requiring obviously unrealistic poison rates**.

#### What a junior reader should understand

This part is constructing the attack surface itself. Everything that follows depends on these design choices. The later curves are meaningful only because the synthetic data are shaped to represent a plausible poisoned preference-learning scenario.

### Section A, Part 2 — Poisoning the labels

The function `poison_labels(...)` applies the attack.

#### What the code does exactly

The key line is that it chooses indices only from `trigger_mask`, then sets those selected labels to `0`.

That means the attack is not randomly corrupting the entire dataset. Instead, it is:

- measuring poison rate over the **overall** dataset size,
- but applying the poison only inside the trigger-bearing subset,
- and forcing those chosen cases toward the B-preferred label.

#### Why this is important

This is one of the most important details in the whole notebook.

The x-axis does **not** mean:

- “what fraction of triggered examples are poisoned?”

It means:

- “what fraction of the **entire preference dataset** is corrupted, with poisoning restricted to trigger-relevant examples?”

That is why the 0.5% marker matters. The notebook is trying to model a **small overall contamination rate**, not a large corruption rate hidden inside the trigger subset.

#### Why the notebook does it this way

This aligns the simulation with the paper’s low-end poisoning discussion and avoids a misleading earlier framing where poison intensity could appear much larger than it really was.

### Section A, Part 3 — DPO-like surrogate training

The function `train_dpo_policy(...)` is the DPO-like side.

#### What the code does

- It computes `delta = prompts_a - prompts_b`.
- It orients that difference using the poisoned labels.
- It trains a linear weight vector `w`.
- It uses a sigmoid-style update with `beta = 3.0`, `lr = 0.18`, and `n_epochs = 280`.

#### Why this is DPO-like

Real DPO is more complex than this. But the notebook is preserving the core property the author cares about:

- the policy is influenced **directly** by pairwise preferences.

That directness matters because poisoned preference labels can then reshape the decision boundary without passing through a separate reward-model stage.

#### Why the notebook includes this surrogate

The author wants to isolate a mechanism:

> direct preference fitting may absorb structured label corruption sooner or more strongly than a reward-buffered alternative.

This surrogate is designed to express that mechanism clearly.

### Section A, Part 4 — PPO-like surrogate training

The function `train_ppo_surrogate(...)` models a more buffered pipeline.

#### What the code does

This is a two-stage process:

1. Train a reward-model-like weight vector `w_rm` on the signed preference gaps.
2. Train a policy vector `w_policy` to match the reward-model target while paying a KL-style penalty.

Key defaults are:

- `lr_rm = 0.03`
- `lr_policy = 0.0025`
- `kl_coef = 12.0`
- `n_epochs_rm = 200`
- `n_epochs_policy = 100`
- `temperature = 3.0`

#### Why this matters conceptually

The notebook is not claiming this is real PPO. It is using a surrogate that captures the paper’s qualitative idea:

- first form an intermediate reward signal,
- then fit the policy under a stabilizing constraint.

That gives the notebook a mechanism for “buffering” that can be compared with the direct DPO-like update path.

### Section A, Part 5 — Evaluation

The function `evaluate_policy(...)` computes two numbers:

- **attack success** on trigger-bearing prompts,
- **clean utility** on non-trigger prompts.

#### What attack success means here

The code scores both responses with the learned weight vector. On triggered prompts, if the policy no longer prefers A, that counts toward attack success.

#### Why the notebook tracks both metrics

A system can appear robust for the wrong reason.

For example:

- if attack success drops because the model collapsed,
- that is not real safety.

By also measuring clean utility on off-trigger data, the notebook checks whether the system remains useful on normal examples.

### Section A, Part 6 — Full low-end sweep with held-out evaluation

The function `run_section_a(...)` turns the toy setup into a benchmark-like sweep.

#### What the code runs

It uses:

- `n_seeds = 6`
- `n_train = 5000`
- `n_test = 2500`
- poison rates of `0.0`, `0.25%`, `0.5%`, `1.0%`, and `2.0%`

For each poison rate and each seed, it:

1. generates training data,
2. generates separate held-out test data,
3. poisons the training labels,
4. trains the DPO-like policy,
5. evaluates it on held-out data,
6. trains the PPO-like surrogate,
7. evaluates it on held-out data,
8. records means and standard deviations.

#### Why the notebook does this

This is where the section stops being a one-off demonstration and becomes a comparative sensitivity analysis.

The held-out split is especially important. It strengthens the interpretation that the learned poisoned direction generalizes beyond the exact examples seen during training.

### Section A, Part 7 — Plots and summary table

The code then produces:

- a held-out attack-success curve,
- a held-out clean-utility curve,
- a small summary table.

#### Why the 0.5% marker is drawn

The notebook puts a vertical reference line at **0.5%** and labels it as the regime discussed in the cited paper.

This does **not** mean the notebook reproduced the exact same threshold numerically. It means the author wants the comparison to live in the same low-end contamination neighborhood.

### What Section A is trying to show

The intended message is:

> when preference corruption is small but targeted, a direct preference-learning pipeline can become meaningfully distorted earlier than a reward-buffered pipeline in this surrogate setup.

### What Section A is not trying to prove

- It is not proving a universal threshold.
- It is not claiming real DPO and PPO always behave like this.
- It is not covering multimodal or full production alignment stacks.

---

## Cell 5 — Section A Interpretation

### What this markdown cell adds

This interpretation cell tells the reader how to read the curves responsibly.

It highlights three revisions:

- poison rate now refers to the **overall dataset**,
- the sweep focuses on the **sub-2% regime**,
- evaluation is now **held-out**.

It then states the intended reading:

- the DPO-like policy lifts earlier,
- PPO-like training looks more buffered,
- clean utility remains comparatively stable.

### Why this cell matters

This cell is a built-in anti-overclaiming device. It prevents the plot from being read as “DPO fails at exactly 0.5%.” Instead, it frames the result correctly as a **comparative mechanism-level trend**.

---

## Cell 6 — Section B Framing: Sleeper-Agent Persistence Through SFT

### What this cell does

This markdown cell introduces the logic of Section B.

It connects the section to the literature on:

- persistent pre-training poisoning,
- sleeper agents,
- the difference between concealment and elimination,
- and the BEAT probing idea.

It also marks the capability tier as **(a)**: pre-training data access.

### Why this framing matters

The key conceptual move here is subtle but important.

Section B is not asking only:

- “Can bad behavior be implanted?”

It is asking:

- “If bad behavior is implanted early, what happens after later safety tuning?”

That is a much more practically relevant question because many systems are tuned after pre-training, and a naive reader might assume later tuning automatically cleans earlier contamination.

### Why BEAT is mentioned here

The markdown cell prepares the reader for a probe-based detector later in the code. It also gives an important caveat: the notebook’s probe result should not be read as a faithful reproduction of BEAT’s best-case reported performance.

---

## Cell 7 — Section B Code: Sleeper-Agent Persistence and Validation-Calibrated Probing

This is one of the richest cells in the notebook because it combines training, safety tuning, and detection.

### Big-picture goal of Section B

Section B asks:

> If a harmful trigger-conditioned behavior is planted during pre-training, can benign-only safety tuning remove it? And if not, can a calibrated probe help detect the residual difference?

### Section B, Part 1 — Minimal policy representation

The class `SimpleLinearPolicy` defines a tiny linear policy with a 3-class softmax output.

#### What the code does

It supports:

- `logits(x)`
- `probs(x)`
- `predict(x)`
- `copy()`

#### Why three classes matter

A beginner might notice that the main labels used later are class `0` and class `2`. So why have 3 actions at all?

Because a 3-class distribution gives the notebook richer output geometry for detection. Even if the direct labels emphasize only benign and harmful classes, the probability distribution can still shift in more informative ways than a strict binary classifier would allow.

That makes the later JSD-based probing more meaningful.

### Section B, Part 2 — Sleeper-style dataset construction

The function `make_sleeper_data(...)` creates two groups:

- `X_benign` with `n_benign = 400`
- `X_trigger` with `n_trigger = 100`

It uses `dim = 8` by default.

#### What the feature design does

- Benign examples get a strong signal on feature 0.
- Trigger examples get a strong signal on feature 1.
- Benign labels are all class `0`.
- Trigger examples during pre-training are labeled as class `2`.

#### Why the notebook does this

The author wants the model to learn two coexisting behaviors:

- normal benign behavior on common inputs,
- harmful trigger-conditioned behavior on a rarer subpopulation.

This is the essence of the sleeper-agent intuition: the model can look normal most of the time while preserving a hidden hazardous pathway.

### Section B, Part 3 — Gradient step helper

The function `cross_entropy_step(...)` performs one full-batch gradient update on the policy.

### Why this helper exists

This keeps the training logic simple and transparent. The notebook is not trying to simulate a large optimizer stack. It just needs a stable and readable training step that can be reused in pre-training and safety-tuning phases.

### Section B, Part 4 — Pre-training the hidden behavior

The function `pretrain_policy(...)` trains on a combined dataset:

- benign examples with benign labels,
- trigger examples with harmful labels.

Defaults:

- `n_epochs = 150`
- `lr = 0.08`

### Why the notebook does this

This gives the model a hidden mapping:

- ordinary inputs → benign action,
- triggered inputs → harmful action.

The notebook wants that mapping to be strong enough to survive a later cleanup attempt, otherwise the section would not be able to test persistence.

### Section B, Part 5 — Benign-only safety tuning

The function `sft_safety_tune(...)` fine-tunes only on benign data.

Defaults:

- `n_epochs = 120`
- `lr = 0.04`

The notebook also later creates a stronger variant, **SFT+**, with:

- `n_epochs = 300`
- `lr = 0.06`

### Why this matters

This is where the paper’s core distinction becomes visible.

Benign-only tuning may improve what the model does on ordinary inputs, but that does not necessarily mean the trigger-conditioned mechanism is gone.

By including both SFT and SFT+, the notebook avoids a simplistic yes/no question. Instead it asks:

- does ordinary SFT help?
- does stronger SFT help more?
- does the hidden behavior still remain partly intact?

### Section B, Part 6 — Evaluation of sleeper persistence

The function `evaluate_sleeper(...)` returns:

- **benign safety**: fraction of benign examples predicted as class 0,
- **attack success**: fraction of trigger examples predicted as class 2.

### Why this is the right pair of metrics

This creates a clean separation between:

- outwardly improved safety,
- and residual hidden risk.

That is exactly what Section B is trying to study.

### Section B, Part 7 — Validation-calibrated BEAT-inspired detector

The function `calibrated_beat_detector(...)` is the detection component.

This is one of the most thoughtful parts of the notebook.

#### What the code does step by step

1. Build a probe vector whose first three features are set to `0.9`.
2. Compute the policy’s output probabilities on the original input.
3. Compute the policy’s output probabilities on the probed input.
4. Measure the JSD between those two distributions for every example.
5. Split benign and trigger scores into validation and test partitions using `validation_frac = 0.4`.
6. Search candidate thresholds from the 5th to 95th percentiles of validation scores.
7. Choose the threshold that maximizes validation balanced accuracy.
8. Evaluate detection accuracy, AUROC, TPR, and FPR on held-out test data.

#### Why this is a strong design choice

This is much better than picking a threshold directly on the same data used for reporting.

A junior reader should notice that the author is trying to avoid a common notebook mistake:

- making the detector look artificially strong by tuning the threshold on the test set itself.

The validation/test split makes the detector story more honest.

#### Why JSD is used instead of only predicted class

The probe may change the entire output distribution even if the top predicted action does not flip.

That is important because hidden sleeper behavior may express itself as a **distributional sensitivity pattern**, not just a visible class change.

### Section B, Part 8 — Running the simulation across seeds

The cell runs the full experiment for `6` seeds.

For each seed, it records:

- pre-training benign safety and attack success,
- post-SFT benign safety and attack success,
- post-SFT+ benign safety and attack success,
- validation-calibrated detector metrics on the SFT model.

### Why multiple seeds matter here

Section B is vulnerable to over-reading a single lucky run. Multi-seed aggregation helps show whether the pattern is stable enough to discuss as a phenomenon instead of an accident.

### Section B, Part 9 — Plots

The cell produces three main plots.

#### Plot 1: grouped bars

This compares benign safety and attack success across:

- pre-training,
- after SFT,
- after SFT+.

This answers:

- Is visible safety improving?
- Is hidden attack behavior shrinking at the same rate?

#### Plot 2: per-seed trajectories

This shows how each seed moves across stages for both benign safety and attack success.

This answers:

- Is the observed pattern consistent across runs?

#### Plot 3: histogram of probe scores

This uses seed 0 as an example and overlays:

- benign JSD scores,
- triggered JSD scores,
- the validation-calibrated threshold.

This answers:

- Do benign and triggered cases occupy different score regions?
- How clean or messy is that separation?

### Section B, Part 10 — Summary tables

The cell then renders two tables:

- a stage summary,
- a detector summary across seeds.

### What Section B is trying to show

The intended point is:

> benign-only safety tuning can improve visible behavior more than it removes trigger-conditioned behavior, and a calibrated probe can reveal some residual difference without being perfect.

### What Section B is not trying to prove

- It is not saying all safety tuning only conceals harm.
- It is not saying BEAT always works.
- It is not modeling large-scale adaptive sleeper attacks.

---

## Cell 8 — Section B Interpretation

### What this markdown cell adds

This cell explicitly tells the reader to interpret the detector as **moderate**, not magical.

It reinforces four points:

- pre-training establishes strong harmful trigger behavior,
- ordinary SFT improves benign performance,
- even stronger SFT+ still may not fully erase the trigger pathway,
- detector metrics are based on a validation-calibrated setup rather than on a single hand-picked cutoff.

### Why this matters

This cell is a key honesty check. Without it, a reader might see AUROC in the tables and mentally convert that into “problem solved.” The notebook is carefully pushing back against that reading.

---

## Cell 9 — Section C Framing: Indirect Prompt Injection and MCP Tool-Schema Poisoning

### What this cell does

This markdown cell introduces Section C as a trust-boundary problem.

It explains that the paper sees a common architectural weakness:

- agentic systems often do not cleanly separate trusted instructions from untrusted external content.

It then names the two attack families modeled in this section:

- **Indirect Prompt Injection (IPI)** through malicious retrieved content,
- **MCP tool-schema poisoning** through malicious tool metadata.

It also lists attack variants such as:

- shadowing,
- rug pulls,
- full-schema poisoning.

### Why this framing matters

Section C is not about training-time poisoning. It is about what happens **during deployment** when the model must interpret external context.

That distinction matters because the defenses also change. The right response is no longer “clean the dataset” but rather:

- control trust boundaries,
- constrain how external inputs are interpreted,
- validate metadata,
- and scope tool execution.

### What this section does not claim

The markdown cell is very careful to say this section is **not** modeling:

- OS-level command injection,
- remote code execution,
- or exact behavior of any specific infrastructure implementation.

That matters because tool poisoning can exist at several layers, and this notebook is modeling the **action-selection layer**, not the full infrastructure layer.

---

## Cell 10 — Section C Code: Source-Specific Unsafe Action Selection

### Big-picture goal of Section C

Section C asks:

> If unsafe behavior is driven by malicious retrieval versus malicious tool metadata, do the same defenses help equally against both?

The notebook answers that by building a small action-selection model with three possible outputs.

### Section C, Part 1 — Softmax helper

The function `softmax_logits(...)` turns logits into action probabilities.

### Why this matters

This is the bridge from abstract scores to a concrete chosen behavior.

### Section C, Part 2 — Multi-logit action selector

The function `action_selector(...)` is the heart of the section.

#### What the code does

It constructs three logits:

- `safe_logit`
- `neutral_logit`
- `unsafe_logit`

These depend on:

- trusted system signal,
- user signal,
- benign retrieval signal,
- benign tool signal,
- malicious retrieval signal,
- malicious tool signal,
- defense configuration,
- and random noise.

Then it:

1. adds Gaussian noise,
2. converts logits to probabilities,
3. chooses the highest-probability action,
4. returns both the chosen action and the probabilities.

#### Why this design is important

This is a major improvement over a binary “unsafe score.”

A binary setup can be too blunt. Here, the neutral class gives the model room to behave in more graded ways. That helps the simulation represent competition among signals rather than forcing every case into a simple safe/unsafe flip.

#### How the logits are meant to behave conceptually

- `safe_logit` rises with trusted system signal and benign context.
- `neutral_logit` acts like a middle baseline.
- `unsafe_logit` rises with malicious retrieval or malicious tool influence, scaled by defense settings.

This makes it possible for the same defense to affect malicious retrieval and malicious tool inputs **differently**.

That is exactly what Section C wants to test.

### Section C, Part 3 — Attack-case evaluation

The function `evaluate_attack_case(...)` averages unsafe-action rates across:

- `n_trials = 250` per seed,
- `n_seeds = 6`.

For each defense configuration, it counts how often the chosen action is `"unsafe"`.

### Why this matters

A single noisy action draw would be too fragile. Averaging across many trials and seeds makes the section behave more like a small Monte Carlo benchmark.

### Section C, Part 4 — Defense configurations

The notebook defines five defense setups:

- `no_defense`
- `instruction_sep`
- `metadata_validation`
- `scoped_execution`
- `combined`

These modify parameters such as:

- `system_safe`
- `ret_scale`
- `tool_scale`
- `guard`

### Why these parameters matter

These are how the notebook encodes different defense philosophies.

#### `instruction_sep`

This mainly weakens malicious retrieval influence.

#### `metadata_validation`

This mainly weakens malicious tool influence.

#### `scoped_execution`

This more broadly reduces harmful leverage across both sources.

#### `combined`

This stacks several partial protections and should therefore perform best overall.

This is a useful design because it lets the reader see that defenses are **source-specific**, not universally interchangeable.

### Section C, Part 5 — Attack cases

The notebook defines three attack conditions:

- `combined`: malicious retrieval `0.70`, malicious tool `0.60`
- `ipi`: malicious retrieval `1.25`, malicious tool `0.00`
- `tool`: malicious retrieval `0.00`, malicious tool `1.10`

### Why the notebook separates these cases

If the notebook only ran a mixed attack, the reader would not know whether a defense helped because it blocked retrieval poisoning, tool poisoning, or both.

By separating them, the section can answer a more useful question:

- which defense helps which attack source?

### Section C, Part 6 — Plots

The cell produces two plots.

#### Plot 1: combined retrieval + tool poisoning

This shows the overall unsafe-action rate for each defense configuration under the combined attack.

This answers:

- if both sources are malicious together, how much does each defense help overall?

#### Plot 2: separated source attacks

This places retrieval-only and tool-only attacks side by side.

This answers:

- is instruction separation especially good against retrieval poisoning?
- is metadata validation especially good against tool poisoning?
- does scoped execution help both?

### Section C, Part 7 — Summary table

The cell then renders a compact summary table for:

- combined attack,
- IPI only,
- tool schema only.

### What Section C is trying to show

The intended point is:

> indirect prompt injection and tool-schema poisoning are related trust-boundary failures, but they are not identical, so the same defense should not be expected to help both in exactly the same way.

### What Section C is not trying to prove

- It is not modeling infrastructure-layer exploitation.
- It is not reproducing any one MCP system exactly.
- It is not claiming the defense coefficients are deployment-calibrated.

---

## Cell 11 — Section C Interpretation

### What this markdown cell adds

This cell explains how to read the updated Section C plots.

It highlights that the new action model no longer collapses into a nearly binary unsafe score. Instead, it uses separate safe, neutral, and unsafe logits.

It then states the intended findings:

- without defenses, both attack sources drive high unsafe-action rates,
- instruction-data separation helps retrieval-only attacks most,
- metadata validation helps tool poisoning most,
- scoped execution helps both,
- combined defenses work best overall.

### Why this matters

This is the section’s clearest statement that **trust-boundary failures are source-specific**. The cell makes sure the reader does not flatten IPI and tool poisoning into one generic attack blob.

---

## Cell 12 — Section D Framing: Multi-Agent Cascading Failure with `α_observed`

### What this cell does

This markdown cell introduces the final simulation section.

It defines the runtime metric:

- `α_observed(t) = anomalous_count(t+1) / max(anomalous_count(t), 1)`

It also draws an explicit distinction between:

- runtime `α_observed`,
- and design-time `α_effective` from the paper.

Then it introduces the Layer-3 circuit-breaker logic with three states:

- yellow,
- orange,
- red.

Finally, it links topology to contamination spread:

- dense/shared-pool structures spread faster,
- sparse/decentralized ones provide more isolation.

### Why this framing matters

Section D is the notebook’s system-level section.

The earlier sections were mainly about how contamination enters or persists in a model pipeline. Section D asks what happens when a contaminated state can **propagate through a network of interacting agents**.

That is why the metrics shift from static accuracy-like outcomes to **kinetic** ones such as:

- growth rate,
- threshold crossing,
- exposure over time.

---

## Cell 13 — Section D Code: Multi-Agent Cascade Simulation with Quarantine-Aware Breaking

### Big-picture goal of Section D

Section D asks:

> If contamination starts small in a multi-agent system, how quickly can it spread, and how much can a breaker slow or contain that spread without pretending the system was never affected?

### Section D, Part 1 — Graph construction

The notebook defines two graph generators:

- `build_dense_graph(...)` → Erdős–Rényi graph,
- `build_sparse_graph(...)` → Barabási–Albert graph.

Defaults:

- `n_nodes = 60`
- dense graph uses `edge_prob = 0.12`
- sparse graph uses `m_edges = 2`

### Why the notebook uses two graph families

This is how the notebook makes topology visible.

A dense graph gives many routes for spread. A sparse graph gives fewer links and more natural isolation.

The author wants to show that contamination is not only about the infection probability. It is also about **network shape**.

### Section D, Part 2 — Cascade simulation

The function `simulate_cascade(...)` runs the spread process.

Defaults:

- `spread_prob = 0.16`
- `n_initial = 2`
- `n_steps = 18`
- yellow threshold `= 1.00`
- orange threshold `= 1.10`
- red threshold `= 2.0`
- red quarantine fraction `= 0.35`

#### What the state variables mean

The function tracks:

- `active` contaminated nodes,
- `affected` nodes,
- `quarantined` nodes,
- `alpha` over time,
- breaker states over time.

The distinction between **active** and **affected** is very important.

- **Active** nodes can still spread contamination.
- **Quarantined** nodes are still affected, but isolated.
- **Affected** means active + quarantined.

This is much more realistic than pretending a quarantined node was magically healed.

#### What happens each time step

For each active contaminated node, the simulation checks its neighbors.

A neighbor is eligible for infection only if it is not already:

- contaminated,
- or quarantined.

Then infection happens with probability `current_spread_prob`.

After that, the notebook computes `raw_alpha`, the growth ratio of active contamination from one step to the next.

### Section D, Part 3 — Breaker logic

If breaker mode is on, the section applies tiered responses:

#### Yellow state

If `raw_alpha >= 1.00`, spread probability is reduced to `spread_prob * 0.78`.

This is a light response. It corresponds to increased caution or logging.

#### Orange state

If `raw_alpha >= 1.10`, spread probability is reduced to `spread_prob * 0.55`.

This is a stronger response. It models partial connectivity reduction or privilege revocation.

#### Red state

If `raw_alpha >= 2.0`, the notebook:

- ranks contaminated nodes partly by graph degree,
- quarantines the top fraction,
- moves them out of the active propagating set,
- reduces future spread to `spread_prob * 0.28`.

### Why degree-based quarantine is used

This is a simple but reasonable surrogate for isolating the most structurally influential spreaders first.

It is not meant to be a perfect enterprise policy. It is meant to illustrate a topology-aware control idea.

### Section D, Part 4 — Running all conditions

The notebook runs four conditions:

- dense without breaker,
- dense with breaker,
- sparse without breaker,
- sparse with breaker.

Global choices for this section are:

- `N_NODES = 60`
- `N_STEPS = 18`
- `N_SEEDS = 8`
- `N_INITIAL = 2`
- `SPREAD = 0.16`

### Why this comparison matrix matters

It lets the notebook separate:

- topology effects,
- breaker effects,
- and their interaction.

That is much more informative than showing only one graph or only one breaker setting.

### Section D, Part 5 — Summary metrics

After all runs, the notebook computes:

- mean active contamination over time,
- mean quarantined counts,
- mean `α_observed` over time,
- final active contamination,
- final affected contamination,
- time to 25% contamination,
- time to 50% contamination,
- normalized exposure AUC,
- peak `α_observed`.

### Why these metrics are a strong design choice

This section does not reduce the whole story to “how many nodes were infected at the end?”

That would miss a lot of what matters operationally.

Instead, it measures **kinetics**:

- how fast contamination begins,
- how quickly it crosses dangerous thresholds,
- how long the system spends in a risky state,
- whether the sharpest growth spike is reduced.

That is exactly the right way to think about circuit breakers.

### Section D, Part 6 — Plots

The notebook renders four plots.

#### Plot 1: Active contamination over time

This shows how many nodes are still actively spreading contamination under each condition.

#### Plot 2: `α_observed` over time in the dense topology

This overlays the growth ratio with yellow, orange, and red threshold lines.

This helps the reader understand not just whether spread happens, but **how aggressively it is accelerating**.

#### Plot 3: Threshold-crossing delay

This compares time to 25% and time to 50% contamination.

This is especially useful because one of the main goals of a breaker is not total elimination but **buying time**.

#### Plot 4: Exposure vs peak growth

This puts normalized exposure AUC and peak `α_observed` side by side.

This answers a more operational question:

- does the system spend less time in a contaminated state,
- and does it avoid sharp runaway growth?

### What Section D is trying to show

The intended point is:

> a quarantine-aware breaker is valuable mainly as a kinetic control mechanism. It slows spread, reduces exposure, and often reduces final active cascade size without claiming universal eradication.

### What Section D is not trying to prove

- It is not validating a real deployment policy.
- It is not claiming universal thresholds.
- It is not modeling heterogeneous real-world agent ecologies.

---

## Cell 14 — Section D Interpretation

### What this markdown cell adds

This cell makes the intended reading explicit.

It emphasizes that the revised version now:

- treats red-state response as quarantine rather than cure,
- reports operational metrics like time-to-threshold and exposure AUC,
- aligns threshold bands more closely with the paper.

It then states the key results in plain language:

- dense topologies spread faster,
- breakers delay threshold crossings,
- exposure drops with breaker use,
- the strongest claim is about slowing and containing spread rather than erasing contamination.

### Why this matters

This is the notebook’s strongest realism correction. Many toy contagion simulations accidentally suggest that intervention means instant cleanup. This one explicitly avoids that mistake.

---

## Cell 15 — Final Summary Table Intro

### What this cell does

This is a short markdown transition that tells the reader the notebook is about to summarize how each simulation maps back to the paper taxonomy.

### Why this matters

This is a small cell, but it performs an important narrative function. It turns the notebook from a sequence of disconnected examples into a structured companion to the paper.

---

## Cell 16 — Final Taxonomy Mapping Table

### What this cell does

This cell constructs a four-row summary table, one row per simulation:

- A — DPO vs PPO
- B — Sleeper Persistence
- C — IPI & MCP
- D — Multi-Agent Cascade

For each row, it records:

- attack class,
- pipeline stage,
- capability tier,
- core metric,
- relevant defense,
- main caveat.

### Why this matters

This is the bridge between notebook and manuscript.

Without this table, the notebook would just be four interesting examples.

With this table, the notebook becomes a clean explanatory appendix to the paper’s taxonomy. A reader can immediately answer:

- which stage of the pipeline is being illustrated,
- which threat family it represents,
- which defense idea it connects to,
- what limitation to keep in mind.

### Why the rendering choices matter

The table is clearly designed for readability:

- wrapped columns,
- large figure size,
- carefully chosen widths,
- styled export-friendly formatting.

That tells us the author wants the notebook to serve not just as code, but as a communicative research artifact.

---

## Cell 17 — Final Conclusion and Scope Reminder

### What this cell does

This markdown cell closes the notebook by summarizing:

- what each section demonstrates,
- what the notebook still does **not** prove,
- and why the revisions matter.

### Why this cell is important

This is the notebook’s final scientific hygiene step.

It reminds the reader that the notebook is still:

- illustrative,
- surrogate-based,
- non-production,
- and incomplete with respect to the full threat taxonomy.

It also lists important out-of-scope attack families such as:

- recursive collapse,
- multimodal contamination,
- memory poisoning,
- federated instruction tuning attacks,
- benchmark leakage.

### Why this matters for the report

This cell helps keep the report honest. The notebook is strongest when described as a **mechanism benchmark for comparative trends**, not as a final empirical validation of the paper’s full defense story.

---

## 6. Cross-Cutting Design Choices Across the Notebook

Several design principles appear across multiple sections.

### A. Fixed seeds and repeated runs

The author consistently uses seeded randomness and multiple runs. That improves reproducibility and makes the results more stable.

### B. Small surrogates instead of large black boxes

The notebook chooses simple linear models and graph simulations because they make causal logic easier to inspect.

### C. Separate benign performance from attacker success

The notebook repeatedly avoids the mistake of treating degraded normal behavior as if it were successful defense.

### D. Use held-out or calibrated evaluation when possible

Section A uses train/test separation. Section B uses validation-calibrated thresholding. These are good habits and make the notebook more trustworthy.

### E. Distinguish source-specific and stage-specific threats

The notebook avoids flattening all contamination into one category. It repeatedly asks:

- where did contamination enter,
- what stage is affected,
- which defense is actually relevant there?

### F. Model containment as isolation, not magical deletion

Section D’s quarantine logic is one of the notebook’s most realistic design improvements. It keeps “affected” and “actively spreading” conceptually separate.

### G. Use tables as bridges from code to paper

The notebook repeatedly renders small, readable tables because the author wants the simulation to be report-friendly and manuscript-friendly.

---

## 7. How a Junior Reader Should Read This Notebook

If I were giving this notebook to an intern or a new researcher, I would recommend the following reading order:

1. Read **Cell 0** carefully so the scope is clear.
2. Skim **Cells 1–2** to understand the shared tools.
3. For each section:
   - read the framing markdown first,
   - then read the code,
   - then inspect the plots and tables,
   - then read the interpretation markdown.
4. End with **Cells 16–17** so the simulations reconnect to the paper.

When reading each major section, keep asking four questions:

- What is the attacker allowed to control?
- What part of the pipeline is being stressed?
- What metric indicates success or failure?
- What simplification is being made for the sake of interpretability?

That question set will stop a junior reader from over-reading any one plot.

---

## 8. Main Takeaways for the Report

If this notebook is being summarized in a formal report, the clearest takeaways are the following.

### Section A takeaway

Small, structured corruption in preference data can change downstream policy behavior, and direct preference optimization can show that sensitivity earlier than a reward-buffered surrogate in the notebook’s low-poison regime.

### Section B takeaway

Trigger-conditioned harmful behavior planted during pre-training can survive later benign-only safety tuning to a meaningful degree, and probe-based detection can help reveal the residue without being perfect.

### Section C takeaway

Retrieved-content attacks and tool-metadata attacks are related trust-boundary failures, but they are not identical. Different defenses help different sources of contamination.

### Section D takeaway

In multi-agent settings, the most defensible claim is not “the breaker erases contamination.” It is “the breaker can slow spread, reduce exposure, and buy response time.”

### Overall takeaway

The notebook’s deepest contribution is showing how **small or indirect contamination channels can become important when the surrounding architecture amplifies them**.

That is the core reason the notebook is useful as a companion to the paper.

---

## 9. Final Assessment

This notebook is strong as a **mechanism-level explanatory benchmark**.

Its main strengths are:

- it is clearly organized,
- it uses interpretable surrogates,
- it matches simulations to paper sections explicitly,
- it separates attacks by stage and source,
- it uses held-out or calibrated evaluation where it matters,
- and it repeatedly includes caveats to prevent overclaiming.

Its main limitation is also intentional:

- it is simplified.

But here that simplification is not a flaw. It is what makes the notebook teachable.

For a junior reader, the notebook is best understood as a **guided explanatory model** of how contamination can enter, persist, and propagate across an agentic AI pipeline.

For a report, the most accurate one-sentence description would be:

> This notebook is a reproducible, literature-grounded set of surrogate simulations designed to illustrate mechanism-level contamination risks and defense tradeoffs across alignment, pre-training persistence, deployment-time trust boundaries, and multi-agent cascade control.
