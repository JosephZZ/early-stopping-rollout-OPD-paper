# Revision Notes v1 — NeurIPS-spotlight pass on main.tex + related_work_section.tex

## Summary of what changed

### main.tex

1. **Title** replaced with a sharper, assertive title:
   *"Prefix Is All You Need: Position Dominates Token-Level Signal Quality in On-Policy Distillation."*
2. **Abstract** fully rewritten. Now states the cross-task / cross-family / two-regime / 1.7–8B-teacher scope, the surpass-teacher finding, the 20–35× / 4× efficiency numbers, and the "not high-KL in disguise" result. Keeps the entropy and 14.9pp numbers from the prior version.
3. **Introduction** (§1) substantially rewritten:
   - Removed two fake citations that appeared in the prior draft (`lu2025onpolicy`, `yang2026beyond`) — neither was in references_extended.bib.
   - Added the concrete prefix-continuation numbers (62.7 / 56.3 / 51.8 at 100/200/300) from the deck's Part-1 motivation.
   - Foregrounded the three converging lines of evidence, now with their actual failure numbers (HE 40.2 → 26.8; MATH n=16 collapse to 46.3; Gemma coding below baseline).
   - Re-anchored the contribution list as five bullets, matching the deck's actual structure.
4. **Notation.** Unified to `\pi_s, \pi_t` and `y_{<t}^student` throughout intro, §2.1 conditional-scoring, and §Method formulation. The command `\method` now expands to "prefix distillation"; full text reads the same wherever it appeared.
5. **Method (§4)** rewritten for clarity: single Eq. (2) for prefix-distillation loss; a "why truncate the rollout, not just mask the loss" paragraph separating stability (masking) from efficiency (truncation).
6. **Main results table (Table 1)** replaced with the deck's Table 1a structure: Qwen + Gemma × Math/Coding/BFCL, with MATH-500 avg@4 and pass@4, HE, HE+, MBPP, MBPP+, BFCL full_acc, and per-step time. Degradation flagged with `↘`-equivalent `^\downarrow` footnote marker. Now includes Prefix-50 as a robustness sanity row, Prefix-100 as canonical.
7. **New Table 2 (`tab:ft_scaling`)** added: FullFT Qwen + Gemma math, Qwen→4B and Qwen→8B scaling (math / coding / BFCL columns), chained Qwen3-4B→8B BFCL row, and a placeholder row for the 14B experiment. The placeholder uses `\tbd{14B}` so the TODO is visible in the PDF.
8. **Token-selection ablation (§Ablation → §"Prefix Is Not High-KL or High-Entropy in Disguise")** updated:
   - Table rebuilt to mirror deck 3.1 exactly: Prefix-100, Full-seq, Top-RKL k=100 / k=200, Top-H_t, Top-H_s k=100 / k=200, AND, OR, triple-product, soft-weighted joint entropy, top-K-renorm RKL, Random-100.
   - Commentary reordered: Prefix beats every selector; every selector beats nothing; Random beats every smart selector; mean position is the load-bearing variable.
   - Top-K-renorm RKL row flagged with a TODO citation comment for "Fu et al. 2025" because I could not verify the exact paper (web search was down during this pass).
9. **Three-mechanisms subsection (§4.4, `sec:mechanisms`)** added between token selection and the old cascade paragraph. Writes out (a) contamination, (b) cascade (subsumes prior standalone §Cascade Effect), (c) planning-content. Anchored to the concrete experimental signatures in the paper.
10. **Mode-seeking subsection (§4.5, `sec:mode_seek`)** added: the deck's 3.3 hypothesis for surpass-teacher cases, framed conservatively as a hypothesis with an argument, not a proof.
11. **Conclusion & Limitations** rewritten for the new scope. Limitations now acknowledges (i) signal analysis reported mostly on Qwen, (ii) reverse-KL specificity of the mode-seeking story, (iii) 14B-teacher run still pending, (iv) KL-profile diagnostic is suggestive but not yet verified across all six cells.
12. **Appendix C (`app:kl_profile`)** added: a short paragraph on the cumulative-KL-45% rule of thumb for picking N from the deck's Appendix C.
13. Fixed a duplicate `\label{tab:timing}` (the appendix table now labels as `tab:timing_detail`).

### related_work_section.tex

1. **KD-for-LMs paragraph** rewritten: tightened, removed the weaker framing about "building on reverse-KL", replaced with the positioning that our result is orthogonal to the divergence-design axis.
2. **Token-level importance paragraph** rewritten: explicitly contradicts the scalar-saliency premise of prior token-selection work using our §Token-Selection result, notes the apparent conflict with Wang et al. 2025 (Beyond-80/20) and resolves it.
3. **Top-K-renorm RKL (Fu et al. 2025)** — added a `\footnote{}` TODO because I could not verify the paper. Needs resolution before submission.
4. **Reasoning-distillation paragraph** expanded: DeepSeek-R1 + CoT cast as the SFT-distillation precedent; our prefix + on-policy + RKL result explicitly positioned as complementary.

## What is stale / not-yet-touched

- **14B-teacher scaling row** in Table 2 is a `\tbd{14B}` placeholder. Fill in from `scripts/prod_mps_7b_14b_fullseq.sh` output. A `% TODO 14B` marker is NOT used; I used the project's existing `\tbd{}` macro so it renders visibly in red in the PDF.
- **Llama cross-family row** — deck mentions Llama is in Table 1 / Appendix A of the deck; the paper's main results table currently has Qwen + Gemma only. If Llama numbers are stable, a third block could be added; I left it out per the instruction "don't cite specific 14B numbers" interpretation that was tight on new rows.
- **KL-profile appendix (`app:kl_profile`)** is discussion-only (two confirmed rows); six-cell verification is flagged with `\tbd{KL-profile scan}`.
- **Top-K-renorm RKL citation** — exact Fu et al. 2025 paper could not be verified (web search unavailable during this pass). Currently marked with a `[TODO-cite-FuEtal25]` bracket in the table and a `\footnote{}` in related_work. Needs one line in references_extended.bib once identified.
- **Figures** not modified. The generate_figures.py pipeline produces the current `fig_analysis_{1,3,4}.pdf`, `fig_continuation.pdf`. The N-sweep figure (`fig_n_sweep.png`) referenced in the deck is not yet pulled into the paper — left as-is because the paper conveys the same information via Table 4 (pos sweep). Could be converted to a figure for visual impact.

## Open questions for the user

1. **Llama in the main table?** The deck says three model families (Qwen, Gemma, Llama) and has Llama in Appendix A of the deck. Should the paper's main Table 1 grow a Llama block, or stay two-family in the body with Llama moved to a paper appendix?
2. **Top-K-renorm citation** — do you want me to hunt down Fu et al. 2025 with a different search, or do you have the arXiv id offhand?
3. **Prefix-50 vs Prefix-100 as canonical.** The deck open-questions suggest picking Prefix-100 as canonical and mentioning Prefix-50 in one line. I kept both in the main table for robustness; Prefix-100 is used as the ours-row in most prose. Reasonable?
4. **Position-sweep figure.** Deck Part-2 ablation is a figure; paper has Table 4. Convert paper to a figure for visual impact?
5. **Mode-seeking section.** The deck frames mode-seeking as a hypothesis (slide 3.3). I kept that framing in §4.5. Should we commit harder to it (e.g., toy experiment showing a smaller-teacher-mode being selected), or leave as-is?

## What the next revision pass should focus on

1. **Fill 14B row** once `prod_mps_7b_14b_fullseq.sh` finishes, and drop the `\tbd{}` placeholders.
2. **Resolve Fu et al. 2025 citation** and add to `references_extended.bib`. Remove the `[TODO-cite-FuEtal25]` bracket and the `\footnote{TODO:}` in related_work.
3. **Decide on Llama placement** (see open question 1) and insert rows if in main.
4. **Tighten Section 2.** The analysis section is the strongest in the paper empirically but reads a little uneven — especially §2.4 (Position Region Ablation) currently has less prose than its figure deserves. A pass on the prose here would help.
5. **Add a cascade figure** (cascade KL curve from deck 3.2 Reason 2) to §4.4 — currently described but not visualized. `images/kl_cascade_curves.png` exists in the deck's images dir.
6. **Efficiency table vs. deck**. Deck Table 2 splits Generation / Teacher-scoring / Train fwd+bwd and gives per-phase GPU memory. Paper Appendix A has the same, but main-text Table 3 is a condensed version. Consider merging.
7. **Citations sanity check**: I removed `lu2025onpolicy`, `yang2026beyond` because they weren't in the bib. If these are real papers the author wanted to cite, re-add them properly.
