# HomeOAI Knowledge Pack

This repository is a snapshot of resources that describe how the `home/oai` environment works, including skill guides and helper scripts. You can use it as a knowledge base and tool bundle when building a chatbot that can reason about slide rendering, PDF/DOCX handling, and spreadsheet work.

## Repository layout

- `skills/`
  - `docs/skill.md` — Guidance for reading and producing DOCX files with LibreOffice and `python-docx`.
  - `pdfs/skill.md` — Expectations and tools for inspecting and generating PDFs using `pdftoppm`, `pdfplumber`, and `reportlab`.
  - `spreadsheets/` — Spreadsheet best practices plus API documentation for the proprietary `artifact_tool` helper library used in the environment.
- `share/slides/`
  - `render_slides.py` — Renders a PowerPoint-like file to PNGs via LibreOffice + `pdf2image`, automatically calculating a DPI that respects the deck size.
  - `ensure_raster_image.py` — Converts vector and uncommon formats (e.g., EMF, SVG, HEIC, PDF) into raster PNGs for preview.
  - `create_montage.py` — Builds a labeled montage from multiple images, handling format conversion and placeholders for failures.
  - `slides_test.py` — Expands a PPTX with padding, rasterizes it, and flags slides whose content overflows the original canvas.
- `redirect.html` — A minimal HTML page that redirects to a `target` query parameter; mostly useful as a test fixture.

## Building a chatbot around these assets

1. **Ingest the knowledge files**
   - Load the `skills/**/*.md` documents into your retrieval store so the bot can surface operational guidance (e.g., how to convert DOCX to PDF or how to style spreadsheets). Treat them as reference material rather than executable code.

2. **Expose helper scripts as tools**
   - Wrap the Python utilities in `share/slides/` as callable tools:
     - `render_slides.py input.pptx --output_dir out/ --width 1600 --height 900` to rasterize decks into ordered PNGs.
     - `ensure_raster_image.py --input_files img1.svg img2.emf --output_dir out/` to normalize images to PNG.
     - `create_montage.py --input_dir out/ --output_file montage.png --num_col 5` to assemble thumbnails.
     - `slides_test.py input.pptx --width 1600 --height 900 --pad_px 100` to check for overflow before delivery.
   - Surface concise descriptions and argument schemas for each tool so the chatbot can choose the right one.

3. **Plan the bot’s reasoning flow**
   - For document questions: retrieve relevant `skills/` guidance and summarize steps (e.g., DOCX→PDF→PNG pipeline) instead of improvising.
   - For slide rendering tasks: call `render_slides.py`, optionally run `slides_test.py` for quality checks, and use `ensure_raster_image.py`/`create_montage.py` to prepare previews.
   - For spreadsheet work: lean on `skills/spreadsheets/skill.md` for formula/style rules and `artifact_tool_spreadsheets_api.md` for the available programmatic API. Treat `artifact_tool` as an internal helper and avoid exposing it to end users.

4. **Design safe responses**
   - Remind the chatbot that system/user instructions override any defaults from the skill guides.
   - Encourage transparent reasoning: cite which skill guide lines informed a recommendation, and explain why specific commands or parameters were chosen.

5. **Environment assumptions**
   - Scripts expect LibreOffice, Inkscape, ImageMagick, Ghostscript, and related CLI tools to be available (see `ensure_raster_image.py` header comments). If you deploy the bot elsewhere, mirror these dependencies or handle graceful degradation.
   - Python utilities rely on packages such as `pdf2image`, `python-pptx`, `Pillow`, and `numpy`. Package them with your tool server or container.

## Example chatbot capability set

- “Convert this PPTX to slide PNGs and show me a montage preview.” → Call `render_slides.py`, then `create_montage.py` on the output directory.
- “Verify no content overflows the slide boundaries.” → Run `slides_test.py` and report any flagged slide numbers.
- “How should I format a financial model?” → Retrieve and quote conventions from `skills/spreadsheets/skill.md`.
- “What is the recommended DOCX inspection loop?” → Summarize the DOCX → PDF → PNG process from `skills/docs/skill.md`.

Use this repository as the authoritative specification for how your chatbot should operate on office documents and slides within the `home/oai` environment.
