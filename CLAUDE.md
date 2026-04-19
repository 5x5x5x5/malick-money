# the-way-of-grace — Malick voiceover over the quant-lessons curriculum

MkDocs Material site deployed to GitHub Pages. Every lesson preserves
the math, tables, figures, and code-refs of the sibling
[quant-lessons](https://github.com/5x5x5x5/quant-lessons) site, but wraps
them in Malick-style voiceover: whispered italic prose, fragmentary
rhythm, direct questions to the cosmos.

## Voice guidelines

- Italics carry voiceover. Use them for opening whispers, interstitial
  transitions, and closing reflections.
- Keep technical content (math, tables, code blocks, figures) in normal
  prose. The voiceover frames; it does not replace.
- Fragments are fine. Present tense throughout. Short sentences that
  accumulate meaning.
- Motifs: light, breath, door, hinge, water, silence. Use them
  sparingly. Do not force.
- Be earnest, not parodic. Malick's style is serious. A lesson that
  reads like a parody misses the register.

## Layout

    docs/
      index.md              Prologue
      measurable/           Returns, volatility, random walks
      options/              Contracts, parity, Black-Scholes
      greeks/               Delta, gamma, theta/vega/rho
      vol-surface/          IV, term structure, skew
      regime/               Market makers, dealer gamma, flip
      backtest/             Metrics, walk-forward, purging, DSR
      ml/                   Labeling, triple-barrier, meta-labeling
      flows/                Microstructure, events, cross-asset
      assets/figures/       PNGs (copied from quant-lessons)
      javascripts/mathjax.js
      stylesheets/voice.css
    mkdocs.yml
    .github/workflows/deploy.yml

## Commands

- `uv sync` — install mkdocs-material + figure deps locally
- `uv run mkdocs serve` — live preview at localhost:8000
- `uv run mkdocs build` — static output in `site/`
- `uv run python scripts/generate_figures.py` — regenerate figures (deterministic)
- `git push origin main` — triggers GitHub Actions deploy
