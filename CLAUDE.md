<!-- ARIS:BEGIN -->
## ARIS Skill Scope
ARIS skills installed in this project: 83 entries.
Manifest: `.aris/installed-skills.txt`
ARIS repo root: `E:\ARISProject\Auto-claude-code-research-in-sleep`
Project skill path: `.claude/skills/<skill-name>`
For ARIS workflows, prefer the project-local skills under `.claude/skills/`.
Do not edit or delete junctioned skills in place; update upstream or rerun:
`powershell -NoProfile -ExecutionPolicy Bypass -File "E:\ARISProject\Auto-claude-code-research-in-sleep\tools\install_aris.ps1" "E:\ResearchProject" -Platform claude -Reconcile`
<!-- ARIS:END -->

## Zotero Paper Full Text (MinerU Cache)
To read a paper stored in Zotero, read its local MinerU markdown first.
Prefer it over `get_content`, `zotero_get_item_fulltext`, or parsing the PDF.

Cache directory: `E:\Zotero\llm-for-zotero-mineru\<itemID>\`
- `full.md`: the paper text.
  `manifest.json` lists figure and table captions with page numbers:
  `page` is 0-based, while `figureBlocks[].pageStart` and `pageEnd` are 1-based.
  `images/` holds extracted figures and is absent when there are none.
- `_llm_source.json`: records `attachmentKey`, `parentItemKey`, and `sourceFilename`.
- Other files (`content_list.json`, `_llm_sync_state.json`) can be ignored.
- `<itemID>` is the PDF attachment's internal integer ID in `zotero.sqlite`.
  None of the Zotero MCP tools used below, pyzotero, or the Zotero API returns it,
  so never build this path;
  find the folder by key.

Lookup:
1. Get the PDF attachment key, referred to below as `<PDF_KEY>`:
   - Known title: call zotero-mcp-v1 `search_library` with `title`, `mode: "minimal"`, and `limit: 3`.
     Check `title` in `results` to confirm the target paper
     (raise `limit` if the target is not among them),
     then take the `key` of the entry in its `attachments` whose `contentType` is `application/pdf`.
     Each result lists at most 3 attachments, so a PDF can be missing.
     If a result has `attachmentsTruncated: true`,
     call zotero-mcp-v2 `zotero_get_item_children` with its `key` to list all of them.
     - If `results` is empty, retry with a shorter, distinctive fragment of the title.
       If it is still empty, the PDF may be standalone with no parent item
       (its `title` is usually the filename without extension, not the paper title):
       search again with `itemType: "attachment"`, `includeAttachments: "true"`, and a fragment of that title in `q`.
       `q` matches titles only (case-insensitive substring), so leave out `.pdf`.
       Don't use the `title` parameter: this mode ignores it and returns standalone attachments unfiltered.
       Confirm the result's `title`; its `key` is `<PDF_KEY>`.
   - Topic or keywords only: find papers with zotero-mcp-v2 `zotero_semantic_search` or `zotero_search_items`,
     and confirm their titles first
     (when `zotero_search_items` finds no match, it falls back to loosely related papers).
     Then call `zotero_get_item_children` once with an array of the chosen parent keys,
     and take the keys of attachments whose type is `application/pdf`.
     For standalone PDFs, use the v1 attachment search above
     (v2 `zotero_semantic_search` misses them, and v2 `zotero_get_attachment_path` returns nothing for them).
   - If a paper has several PDFs, tell them apart by filename.
     If the filenames are identical, run step 2 for each,
     then Grep `^#` in each `full.md` and compare the first few headings
     (supplementary material, for example, starts with `# Supplementary Materials`).
2. Find the `_llm_source.json` that contains the key, with PowerShell:
   `Select-String -Path 'E:\Zotero\llm-for-zotero-mineru\*\_llm_source.json' -SimpleMatch -List -Pattern '"attachmentKey": "<PDF_KEY>"' | ForEach-Object Path`
   (Bash: `grep -l '"attachmentKey": "<PDF_KEY>"' /e/Zotero/llm-for-zotero-mineru/*/_llm_source.json`).
   The folder of the matching file contains `full.md`.
   Don't use the Grep tool for this search:
   it walks every `images/` folder (tens of thousands of files on a hard disk) and can time out after 20 seconds.
   A timeout does not mean there is no match.
3. For long papers, Grep `^#` in `full.md` to get section line numbers,
   then Read only the needed range with `offset`/`limit`.

Rules:
- Match on `attachmentKey` only, never `parentItemKey`.
  A PDF cached while it was standalone has no `parentItemKey`,
  and the plugin never adds one later;
  one parent can also have several cached PDFs.
- Always pass `limit` to v1 `search_library`.
  In minimal mode the default is 30 results, each with its attachment list,
  which wastes tokens.
- Don't query `zotero.sqlite` directly.
  Zotero locks it while running,
  and forcing a read with `immutable=1` can miss recent writes.
  Never hardcode itemIDs.

Fallback (when step 2 finds no match):
- Read the PDF with zotero-mcp-v2 `zotero_read_pdf_pages`,
  passing `item_key: <PDF_KEY>`, `start_page`, and `end_page` (1-based).
  It needs no file path and also works for standalone PDFs,
  but text from multi-column pages can interleave, and figures are omitted.
- To see the figures of a PDF with at most 10 pages, Read the PDF file without `pages`.
  Read with `pages` currently fails with "PDF is password-protected", even for unencrypted PDFs,
  because the TeX Live `pdftoppm` on PATH cannot write the JPEG pages that Read needs.
  Get the PDF path for Read as follows:
  - Has a parent item: call zotero-mcp-v2 `zotero_get_attachment_path` with the **parent item key**
    (passing `<PDF_KEY>` returns "No attachments found"),
    and take the `Local path` of the entry headed by `<PDF_KEY>`.
  - Standalone PDF: reuse the zotero-mcp-v1 `search_library` attachment search results from step 1 (`itemType: "attachment"`),
    and take `attachments[0].filePath` from the result whose `key` is `<PDF_KEY>`.
    Plain title searches don't return `filePath`.

## IEEE-Elsevier-CNKI-mcp

When using the `ieee-sciencedirect-download` MCP to search for literature,
if the user specifies a journal, publication year, or any other search constraint,
include those constraints directly in the initial search query or retrieval parameters.
Do not perform a broad search first
and then filter the returned results by journal, year, or other specified criteria afterward.


## Repository Layout
- `Experiments/`: Experiment code and logs.
- `Manuscripts/IEEE_Template/`: Manuscripts for submission (IEEEtran two-column, pdfLaTeX + BibTeX). Run LaTeX build commands from this directory; see [LaTeX Compilation](#latex-compilation).
- `Manuscripts/Notes_and_Outlines/`: Rough drafts, outlines, and writing notes. Not part of the manuscript LaTeX build.
- `Materials/`: Reference materials, including papers in Markdown and PDF formats. Treat this directory as read-only; do not create, modify, or delete files here.
- `research-wiki/`: Persistent knowledge base for ARIS, covering papers, ideas, experiments, and claims. Update this directory only through the `research-wiki` skill.


## LaTeX Compilation

Always `cd Manuscripts/IEEE_Template` first — the project references `./figs/` and `./reference.bib` by relative path, so building from anywhere else loses figures and bibliography.

Default to latexmk (decides whether BibTeX is needed and reruns until cross-references converge):

```bash
latexmk -pdf -synctex=1 main.tex
```

`-synctex=1` is not optional — latexmk's built-in engine call is bare `pdflatex %O %S`, so without it no `main.synctex.gz` is written and editor forward/inverse search breaks. latexmk forwards the flag to pdflatex unchanged.

When an explicit pdfLaTeX chain is required, all four steps are needed — pass 1 records `\citation` in `.aux`, BibTeX reads that and writes `.bbl` from `reference.bib`, pass 2 typesets the bibliography, and pass 3 resolves the in-text `[n]` numbers:

```bash
pdflatex -interaction=nonstopmode -file-line-error -synctex=1 main.tex
bibtex   main
pdflatex -interaction=nonstopmode -file-line-error -synctex=1 main.tex
pdflatex -interaction=nonstopmode -file-line-error -synctex=1 main.tex
```

pdflatex exits 0 even on errors in `nonstopmode` — verify by grepping `main.log`, not the exit code. Baseline: 0 errors / 0 warnings / 0 bad boxes.

`latexmk -c` clears the intermediate files and keeps `main.pdf`; `-C` deletes `main.pdf` too. Either unsticks a build blocked by stale state — e.g. a truncated `main.aux` from an interrupted run makes BibTeX report `I found no \citation commands`, which neither reruns nor `-g` clear. Prefer `-c`.

