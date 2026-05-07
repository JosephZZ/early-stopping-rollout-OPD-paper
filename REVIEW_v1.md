# REVIEW_v1 — Adversarial NeurIPS reviewer pass

Paper: *Prefix Is All You Need: Position Dominates Token-Level Signal Quality in On-Policy Distillation*

Verdict up front: the empirical story is strong enough for a spotlight shot, but three things will hurt it in review as written — a load-bearing confound not cleanly ruled out, the `\tbd{14B}` placeholders that currently read as an incomplete experiment section, and a chunk of related work whose 2026 citations will be scrutinized and where the framing of Wang et al. (Beyond 80/20) is too quick.

## 1. Story / positioning

The abstract is packed but the spotlight claim *is* legible: "position is an independent, load-bearing axis — not high-KL, not high-entropy, in disguise." Good. The title is defensible *if* the ablation holds; if a reviewer decides the ablation does not rule out "fewer loss tokens = regularization," the title reads as overclaim. See §2.

Where the narrative over-hedges: L442 ("numbers will be inserted in the camera-ready") and the Table 2 `\tbd{14B}` rows (L293–294). Reviewers will read an unfinished table as "the authors think the scaling story needs 14B but couldn't get it done." Either drop the 14B row from Table 2 entirely and commit to the 1.7B/4B/8B story, or hold the submission until 14B lands. Leaving `\tbd{}` visibly red in the PDF is worst-of-both.

Where it over-claims: "essentially every cell" (abstract, L68). The Qwen FullFT block (L267–269) shows prefix *losing* to full-seq on math avg@4 (56.20 vs 58.20). That is one of only two FullFT math cells and it contradicts "essentially every." Soften to "every cell except Qwen FullFT math, where full-seq and prefix are within 2pp."

## 2. Evidence rigor (the biggest risk)

**Confound A — regularization from fewer loss tokens.** This is the first thing a skeptical reviewer will raise and the paper does not cleanly rule it out. Prefix-100 trains on 100 tokens per rollout; full-seq trains on ~400. Fewer gradient tokens = implicit regularization = potentially better generalization on small data, independent of *which* tokens. The token-selection ablation (Table 4) is the intended defense, but note: Top-$H_t$=100 (63.30), Top-$H_s$=100 (62.70), Random-100 (61.25) are all *close to* full-seq (62.35). None of these match prefix-100 (65.85), but the gap is only 2.5–4.5pp — within the range plausibly explainable by "which 100 tokens" rather than "position per se." The cleanest rebuttal would be a **Suffix-100** row (train only on the *last* 100 tokens): if position is load-bearing, Suffix-100 must be strictly worse than Prefix-100, and ideally worse than Random-100. §2.3 already reports Middle-100 and Last-100 drop below baseline (L159), so the data exist in Figure 3 — promote that comparison into Table 4 with the same selector framing. Right now it is visually separated from the selector ablation and the connection is implicit.

**Confound B — "not high-KL in disguise" is not watertight.** Top-RKL=100 at coverage 93.2% KL but mean-position 193 shows the *tail* is where per-token KL is highest for individual tokens, yet the mean of that bag is in the tail because the *count* of high-KL tokens in the tail dominates. A reviewer could argue: the prefix is high-KL *density* (KL/token), while top-KL is high-KL *mass*. The paper conflates these in L370 ("capture the whole prefix benefit"). Clarify: prefix-100 captures 45.6% of KL in the first 100 tokens (density 0.456/token); top-RKL-100 captures 93.2% of KL but spread across the tail where individual KLs are middling — the selector is chasing rare high-KL late tokens, not the dense early band. This is a subtle point and it deserves a sentence, not just a table.

**Confound C — surpass-teacher = benchmark noise?** MATH-500 avg@4 standard error at n=4, 500 problems is ~1.5pp. Prefix-100 Qwen math 65.85 vs teacher 65.30 is inside noise. BFCL at 600 problems, prefix-100 61.30 vs teacher 54.00 is +7.3pp — that is real. Gemma coding 28.70 vs teacher 20.70 is real because the student is already above the teacher. So **three** cells are genuinely above teacher (BFCL Qwen, BFCL Gemma, Gemma coding); the "Qwen math" claim on L68, L76, L421 is weaker than the paper implies. Say "three cells surpass the teacher" and drop Qwen math from the list, or report confidence intervals. Currently the paper says "several" in one place, "nine cells" in another (L252), "four surpass-teacher" elsewhere — pick one count and be consistent.

Sources: [main.tex §2.3 L147–159](paper/main.tex), [Table 1 L227–250](paper/main.tex), [Table 4 L373–401](paper/main.tex).

## 3. Citation rigor

I verified every citation key used in `main.tex` and `related_work_section.tex` resolves in `references_extended.bib` (the only `.bib` loaded at L451). No broken keys. But:

- **"Fu et al. 2025" top-$K$-renorm RKL** (L396, related_work L33–34). Unresolved. This is the single most dangerous citation — a reviewer familiar with distillation-stability literature will flag "we couldn't find the paper" as a credibility problem far out of proportion to the table row it supports. Two options: (i) drop the Top-$K$-renorm RKL row from Table 4 until the reference is nailed down, or (ii) rename the method by what it does ("top-$K$ vocab renormalized RKL, concurrent work") and cite via footnote *only if* the actual reference is found. Leaving `[TODO-cite-FuEtal25]` as visible bracketed text in Table 4 and a `\footnote{TODO...}` in related work is not acceptable for submission.
- **MiniLLM / GKD framing.** L62 cites both as sources of the "token-level reward" view. MiniLLM's actual contribution is reverse-KL + student-generated-output mixing; GKD introduces the on-policy generation loop with general divergences. The framing "teacher acts as a reward model, scoring each student-generated token via its conditional distribution" is a *reinterpretation* of those works through the current paper's lens, not what they claim. That is fine as reframing, but the double-citation at the end of that sentence (L62) makes it read as if Gu and Agarwal wrote those words. Split: cite MiniLLM for reverse-KL, GKD for on-policy, and state the reward-model reading as our reframing ("we view this as…").
- **DeepSeek-R1** (related_work L59). Cited with the cite-key `deepseekr1` — fine, but the claim that SFT-distillation "is often competitive with direct RL on the small model" is stronger than the R1 report actually says; R1 shows SFT-distilled smaller models outperform Qwen-math-RL baselines on *some* benchmarks. Soften to "can be competitive".
- **Agarwal 2024 / "on-policy"**. The paper uses `agarwal2024onpolicy` throughout — GKD's own naming is "Generalized Knowledge Distillation" and its on-policy variant is one of several modes. Keep the `agarwal2024onpolicy` key for consistency but on first mention (L62, L14) call it "GKD (Agarwal et al., 2024)".

## 4. Related-work completeness

Missing or underweighted, listed by where they belong:

- **Token-level importance sampling in RL.** Schulman's PPO importance-weighted gradient and the whole line on per-token clipping (Liu et al., DPO; Rafailov et al.) — the paper's "position as independent axis" result is in dialogue with "token-level vs sequence-level weighting" in RLHF. Belongs in §Related Work's "Token-Level Importance" paragraph as one sentence of context.
- **Self-distillation / "self-play fine-tuning".** SPIN (Chen et al., 2024) and "Self-Rewarding LMs" (Yuan et al., 2024) are on-policy but with the model itself as judge. Our prefix result has a clean prediction for them (the judge's signal also degrades with position). Belongs in Discussion "Broader implications" (L440) as one sentence.
- **Rejection-sampling distillation.** LLaMA-3 / Qwen-2 technical reports use rejection-sampled SFT for reasoning distillation — the baseline the paper is *implicitly* beating on efficiency. Acknowledging this would strengthen the efficiency claim.
- **NTP position weighting in pretraining.** Recent work on "early-token upweighting" in pretraining (e.g., the "Not All Tokens Are What You Need" line, Lin et al. 2024) — same insight in a different regime. One sentence in Related Work would pre-empt a "this is not new" reviewer complaint.
- **Speculative/rejection-based filters.** `xu2025speculative` is cited but the related-work treatment could make clearer that Speculative KD and our prefix cutoff are doing *orthogonal* things (token-quality filter vs position-window cutoff). Currently L16 of related_work just summarizes it.

## 5. Method / notation clarity

- Eq. (1) and Eq. (2) are clean. One ambiguity: $y_{<t}^{\text{student}}$ appears in §2.1 but Eq. (1) writes $\mathbf{y}_{<t}$ without superscript. Unify notation; the superscript is load-bearing for the paper's argument (the conditioning is *on the student's prefix*) and should not disappear in the loss equation.
- "Prefix" definition (L68): "first $N$ response tokens." Good. But the token-selection ablation (Table 4, L382) lists Prefix-100 with "Mean pos. 49.5" — that's the mean *index* of selected tokens, i.e. the midpoint of [0,99]. A reader coming fresh to that column may not realize the other rows report mean position of *their* selection. One-sentence caption note: "Mean pos. = mean token index across the selected set; for Prefix-$N$ this is $(N-1)/2$."
- Table 4 "Signal" column mixes units (KL, $H_s$, $H_t$, "joint") without a key. Either add a footnote or drop the column — the information is already in the selector name.
- L197 defines Prefix loss with both $\min(N,|\mathbf{y}|)$ in the sum *and* $|\mathbf{y}|\le N$ as the expectation constraint. These are redundant if the rollout is truncated at generation time (as the text says). Remove one.

## 6. Figure / table quality

- **Table 1** (L227) is the spine of the paper. It works but the Time column only has two entries (~280s, ~8s) and most rows are blank — either fill the column or drop it and put timing in a separate narrow table.
- **Table 2** (L256) mixes regimes (FullFT top, LoRA teacher scaling middle, chained scaling bottom, FullFT-14B bottom) in one table with a column set that doesn't fit all rows (HE/MBPP+ blank in 5/7 rows). Split into two smaller tables or promote the chained-4B→8B result to its own paragraph — as is, the table is hard to scan.
- **Table 4** (L373) is good but would be stronger as a figure: a scatter of *mean selected position* (x) vs *avg@4* (y), with each selector a labeled point. The punchline "mean position is the load-bearing variable" is visual in that form and buried in the table.
- **Figure 2** (L120, continuation panel): the concrete-example box and the continuation plot share a caption that tries to carry both stories. Split into two figures or one figure with two clearly labeled subcaptions.
- Position-sweep **Table 3** (L340) with only five rows (50/100/150/200/full-seq/baseline) could be a simple line plot. A plot would also make the "wide plateau" claim visual and strengthen §App. C's cumulative-KL diagnostic argument.
- **Cascade figure.** §4.3 Mechanism 2 describes KL-everywhere drop in text only — a per-position KL curve (base vs distilled) is the obvious supporting figure and the deck evidently has `images/kl_cascade_curves.png`. Promote it.

## 7. Aesthetics / writing

- Abstract is one paragraph of 300+ words and reads as a sprint. Break at "We propose *prefix distillation*…" — the first half is problem/signal, second half is method/results.
- Repetition: the "+14.9pp" number appears in abstract, L66, L159. The "$20$--$35\times$ speedup" appears in abstract, L68, L201, L303, L327, L444. Each section re-introduces them rather than assuming the reader has seen them. Cut the restatements in §4 Method and §6 Discussion — the abstract and intro suffice.
- §3.4 "Training Instability as a Consequence" (L163) and §4.2 "Training Stability" (L300) say the same thing. §4.2 is two sentences and adds nothing — delete it and reference §3.4 from the main results.
- §4.4 "Three Mechanisms" and §4.5 "Mode-seeking" land back-to-back after Table 4 interprets the ablation. The section boundary between "mechanisms that explain the gain" and "a fourth hypothesis about surpass-teacher" is soft. Consider folding §4.5 in as "Mechanism 4 — reverse-KL mode-seeking on planning tokens" for parallelism, or move §4.5 to Discussion.
- The opening of §5 Discussion (L438) re-summarizes the conditional scoring problem that §2.1 already introduced. Trim; the discussion should push forward, not recapitulate.

## 8. Specific line-edits

**L53 (abstract, first sentence after setup).** Original: "Standard practice treats this signal as uniformly useful across positions. We show the opposite: the teacher's per-token signal quality is a strong, monotonic function of position." Replace with: "Standard practice treats this signal as uniformly useful across positions. We show it is strongly position-dependent — the teacher's signal collapses from entropy 1.07 and 42% disagreement with the student at position 0 to entropy 0.14 and 13% disagreement by position 200."

**L62 (intro, first cite block).** Original: "…scoring each student-generated token via its conditional distribution~\citep{gu2024minillm, agarwal2024onpolicy}." Replace with: "…scoring each student-generated token via its conditional distribution (we call this the *reward-model view* of the teacher, a reframing of MiniLLM's reverse-KL objective~\citep{gu2024minillm} and GKD's on-policy generation loop~\citep{agarwal2024onpolicy})."

**L68 ("essentially every cell").** Original: "prefix distillation matches or exceeds full-sequence distillation on essentially every cell…" Replace with: "prefix distillation matches or exceeds full-sequence distillation on every cell except Qwen FullFT math (within 2pp), with $20$–$35\times$ speedup."

**L252 ("In nine cells prefix *surpasses the teacher* outright").** Original counts cells loosely. Replace with: "Three cells surpass the teacher beyond plausible benchmark noise — Qwen BFCL (+7.3pp full\_acc), Gemma BFCL (+6.2pp), and Gemma coding (above both the 20.7 teacher and the 23.8 student). Qwen math prefix is within 0.6pp of the teacher on avg@4; we report this as a match, not a surpass."

**L68 and L421 ("surpasses the teacher" — Qwen math).** Strip the Qwen-math surpass-teacher claim everywhere it appears; cite Qwen BFCL, Gemma BFCL, and Gemma coding instead.

**L293 Table 2 `\tbd{14B}` rows.** Delete the 14B block from Table 2. Add one sentence in §4.1 Main Results: "A 14B-teacher experiment (Qwen2.5-Math-7B → Qwen3-14B) is in progress; we defer those numbers to the camera-ready." Do not leave visibly red `\tbd{}` in the submission PDF.

**L396 Top-$K$-renorm RKL row.** Original: "Top-$K$-renorm RKL [TODO-cite-FuEtal25]". Either remove the row, or rewrite as: "Top-$K$ vocab-renormalized RKL (concurrent, uncited)" and add a one-line footnote explaining why the cite is deferred.

**Related_work L34 `\footnote{TODO…}`.** Delete the footnote; either resolve the cite or drop the reference to the method.

**Related_work L33** ("exact citation to be added in camera-ready"). Do not submit with this phrase visible.

**L442 ("numbers will be inserted in the camera-ready").** Reviewers read this as "the authors are banking on a result they don't have." Rewrite: "A preliminary Qwen2.5-Math-7B → Qwen3-14B scaling experiment at the time of writing shows [X]; full per-step results are deferred to the camera-ready." If you don't yet have an X, omit the sentence.

## 9. Top 3 must-fix items to clear the spotlight bar

1. **Rule out the "fewer loss tokens = regularization" confound.** Add a Suffix-100 (or Last-100) selector row to Table 4, with the same framing as the other selectors. The §2.3 Middle-100 / Last-100 data already exist; surfacing them in the token-selection table is what closes the loop on "position as independent axis." Without this, the central claim is vulnerable.

2. **Purge the `\tbd{}` placeholders and the TODO-cite bracket.** Either land the 14B numbers, or remove the 14B row from Table 2 and hedge once in prose. Either find Fu et al. 2025, or remove the Top-$K$-renorm RKL row and the related-work footnote. A submitted paper with visible red `[TBD: 14B]` text and `[TODO-cite-FuEtal25]` in a main-results table will be scored down regardless of the actual numbers.

3. **Tighten the surpass-teacher count.** Pick one number (three cells, not "several" in one place and "nine cells" elsewhere), restrict the claim to the three genuinely-above-noise cells (Qwen BFCL, Gemma BFCL, Gemma coding), and report standard errors or at least sample sizes in the Table 1 caption. The surpass-teacher result is one of the paper's strongest contributions; underselling it cleanly is better than overselling it loosely.
