# Repository Guidelines

This repository is for preparing a publication-ready master thesis on AI and web scraping. The primary goal is high-quality LaTeX text in `main.tex`.

## Project Structure

- `main.tex`: main thesis document and single source of truth for final text.
- `references/`: PDFs for cited papers; filenames typically follow original identifiers (e.g., `2509.13305v1.pdf`).
- `references_list*.md` and `relevant_references.md`: working notes and curated reference lists.
- Other root PDFs (`thesis_assignment.md`, `supervisor_requirements.md`, etc.): official requirements; treat as read-only.

## Writing & LaTeX Workflow

- Write and revise thesis content directly in `main.tex` (or in included `.tex` files if you modularize later).
- Compile with `latexmk -pdf main.tex` or `pdflatex main.tex` (run at least twice); fix critical warnings before milestones.
- Use semantic LaTeX commands (`\section`, `\subsection`, `\emph`, `\cite`) instead of manual formatting.

## Style & Language

- Always write in English with a formal, academic tone.
- Follow your faculty’s thesis guidelines for structure (chapters, sections, lists, figures, tables).
- Keep paragraphs focused (one main idea) and sentences clear and concise.

## Citations & References

- Keep bibliographic data in a BibTeX file (e.g., `references.bib`) next to `main.tex`—create it if missing.
- Use `\cite{key}` for all references and ensure each key corresponds to a real entry and, ideally, a PDF in `references/`.
- Keep citation style consistent and aligned with the official thesis template.

## Version Control & Reviews

- Commits: short, imperative summaries (e.g., `Refine related work section`), one logical text change per commit where possible.
- When reviewing changes, focus on argument clarity, structure, and consistency of terminology, not just typos.

## Agent-Specific Instructions

- Always generate text in English.
- When producing thesis content, output LaTeX suitable for direct inclusion in `main.tex` (sections, figures, citations).
- Do not delete or rewrite PDFs in `references/`; only add new files or adjust related metadata.
- Prefer minimal, well-scoped textual changes and avoid large restructures unless explicitly requested.
