# Reactome-ESM3 Pathway Game — Design Notes

Working title: **Pathway Rush** (evolves from Ribosome Rush).

Captured 2026-10-05 from the drive-home thread. This file was first committed on branch `main` by mistake; it now lives on `master` so it shows up with the rest of the repo.

## Core loop

1. Player types an mRNA sequence (DNA template scrolls in, player types the RNA complement — A→U, T→A, C→G, G→C).
2. Every three correct bases clicks a codon into the ribosome and adds an amino acid bead to the growing chain.
3. A stop codon ends the gene. The finished amino acid sequence is handed to **ESM3-small** (1.4B open checkpoint on Hugging Face) for function annotation — predicted function keywords and InterPro tags per residue.
4. The annotated protein is mapped to Reactome pathways via the free REST Content Service and Analysis Service (no auth needed).
5. The player sees: which pathway their protein belongs to, and what the model thinks it does.

## The twist — mutations and disease pathways

- A **missense** mutation (wrong bead color) can change the model's predicted function.
- A **nonsense** mutation truncates the protein — the model says it no longer looks like anything.
- A **silent** mutation leaves everything unchanged — the player was just lucky.
- The deeper hook: Reactome has ~400 disease proteins and ~5,500 disease variants. A mutation can push the protein *out of* its normal pathway into a **disease pathway**, or knock it out of its normal one entirely.

Teaching genes discussed: HBB sickle cell (c.20A>T, Glu6Val), CFTR delta F508 (three-base deletion, so the codon-typing mechanic has to handle a deletion), TP53 Li-Fraumeni missense (R175H, R248Q) as loss of function in a tumor suppressor.

## Architecture

- **Gradio Space** on Hugging Face with the **ZeroGPU** tier for ESM3 inference.
- Call Reactome REST endpoints directly — do **not** host the Neo4j dump (~432 MB) on a Space. The dump is for local analysis only.
- **Cache** pathway mappings locally. The Analysis Service is token-based and results expire after 7 days.
- Keep sequences short — the teaching genes already are.

## Open questions / caveats

- Sanity-check ESM3 function predictions against known proteins before trusting them as teaching material. Some multimodal bio-reasoning models barely use their biological inputs.
- Wobble pairing (third base allowed to be wrong in the biologically legal way) is still the best second mechanic.
- Input direction fix from the original prototype: the player should type RNA directly, not DNA mapped to RNA.

## Context

- David's prior work with Reactome is relevant.
- Related prototype: `ribosome-rush/index.html` in this repo.
- Also logged with InfiniteProjectIdeaTracker on 2026-10-05.
