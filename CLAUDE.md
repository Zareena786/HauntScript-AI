# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Setup**: Install dependencies using UV (recommended) or pip.
  - Install UV: `pip install uv`
  - Create virtual environment: `uv venv .venv`
  - Activate environment: 
    - Windows: `.venv\Scripts\activate`
    - Unix/macOS: `source .venv/bin/activate`
  - Install packages: `uv pip install -r requirements.txt`
  - (Alternative) Using pip directly: `pip install -r requirements.txt`

- **Run the application**: 
  - Ensure you have a valid Gemini API key in `.env` (key named `api_key`).
  - Start the Streamlit app: `streamlit run horror.py`
  - The app will be available at `http://localhost:8501` by default.

- **Environment variables**: 
  - Copy `.env.example` (if present) or create a `.env` file with:
    ```
    api_key=YOUR_GEMINI_API_KEY
    ```
  - The `.env` file is already listed in `.gitignore`.

- **Linting/Formatting**: 
  - No formal linting setup is configured. You may use standard Python tools like `ruff`, `flake8`, or `black` as needed.

- **Testing**: 
  - The project does not currently include automated tests. If you add tests, consider using `pytest` and place them in a `tests/` directory.

## Project Structure & Architecture

- **Entry point**: `app.py` – Streamlit web interface that collects user inputs and displays results.
- **Core logic**: `horror.py` – Contains `generate_horror_story()` which:
  - Builds a prompt (either custom or via `utils/promptmaker.craft_horror_prompt`).
  - Calls the Gemini API (`gemini-2.5-flash`) to generate the story.
  - Optionally generates a cover image via `utils/covergen.generate_cover_image`.
- **Utilities** (`utils/` directory):
  - `promptmaker.py`: Functions to rephrase user prompts, craft horror prompts, and generate visual prompts for cover images (all using Gemini 2.0 Flash).
  - `covergen.py`: Uses Gemini preview image generation (`gemini-2.0-flash-preview-image-generation`) to create cover images from story text.
  - `formatting.py`: Formats the final story with user inputs, prompt, and date for download as a `.txt` file.
- **Configuration**:
  - `pyproject.toml`: Defines project dependencies (Streamlit, Google AI libraries, Pillow, python-dotenv).
  - `requirements.txt`: Mirror of dependencies for pip installation.
  - `uv.lock`: Lock file for reproducible UV environments.
  - `.python-version`: Specifies Python 3.12 (or compatible) for UV.
- **Data flow**:
  1. User enters inputs in Streamlit UI (`app.py`).
  2. On "Generate Story", `app.py` calls `horror.generate_horror_story()`.
  3. `horror.py` builds a prompt (using `promptmaker` or custom text), configures the Gemini model with temperature from UI, and generates the story.
  4. If cover image requested, `horror.py` calls `covergen.generate_cover_image()` which:
     - Creates a visual prompt from the story (`promptmaker.visual_prompt`).
     - Calls Gemini image model to produce a PIL Image.
  5. The story and optional image are stored in Streamlit session state and displayed.
  6. Download buttons use `formatting.format_txt()` to create a downloadable text file.

## Important Notes

- The project relies on the Gemini API; ensure you have quota and access.
- The cover image generation is experimental and uses a preview model.
- Keep your `.env` file secure and never commit it.
- When modifying prompt crafting functions, test changes carefully as they directly affect API calls and cost.
- Streamlit automatically reloads on file edits during development (`streamlit run` with default options).
