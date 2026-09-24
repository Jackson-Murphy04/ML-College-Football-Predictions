# CS 325 Group Project: Predicting College Football Game Outcomes

This repository contains the code and resources for our group project in CS 325, 
which aims to predict the outcomes of college football games using machine learning techniques.

## Project Structure

```
├── data_exploration.ipynb   # Exploratory data analysis
├── datasets/                # Raw / downloaded data
├── docs/                    # Project proposal and write-ups
├── main.py                  # Script entry point
├── pyproject.toml           # Project dependencies (managed by uv)
└── uv.lock                  # Exact pinned versions — commit this
```

## Setup

This project uses [uv](https://docs.astral.sh/uv/) to manage Python and dependencies.

1. **Install uv** (one time):
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
   Or on macOS: `brew install uv`

2. **Install dependencies** (from the repo root):
   ```bash
   uv sync
   ```
   This creates a `.venv/` folder, installs the Python version from `.python-version` if you don't have it,
   and installs the exact package versions from `uv.lock`.

3. **macOS only: install OpenMP** (XGBoost needs it):
   ```bash
   brew install libomp
   ```
   If you skip this, `import xgboost` fails with `libxgboost.dylib could not be loaded`.

## Running Notebooks

**JupyterLab:**
```bash
uv run jupyter lab
```

**VS Code:** Open the `.ipynb` file, click **Select Kernel** (top right), and choose the
`.venv` Python environment in this repo.

## Managing Packages

| Task | Command |
|---|---|
| Add a package | `uv add <package>` |
| Add a dev-only tool | `uv add --dev <package>` |
| Remove a package | `uv remove <package>` |
| Sync after pulling changes | `uv sync` |
| Run a script | `uv run python main.py` |

- **Use `uv add`, not `pip install`.** `pip install` doesn't update `pyproject.toml` or `uv.lock`, so the package won't be installed for anyone else.
- **Commit `pyproject.toml` and `uv.lock` together** whenever you add or remove a package.
- **Run `uv sync` after pulling** if someone else changed the dependencies.
- **Restart the notebook kernel** after adding a package so the notebook can import it.

## Downloading Data from Kaggle

We use [`kagglehub`](https://github.com/Kaggle/kagglehub) to download datasets:

```python
import kagglehub

path = kagglehub.dataset_download("owner/dataset-name")
print(path)  # local folder containing the dataset files
```

Downloads are cached (in `~/.cache/kagglehub/`), so running this again doesn't download the files again.

**Authentication** (needed for some datasets):
1. Go to kaggle.com → **Settings** → **API** → **Create New Token**.
2. Move the downloaded file to `~/.kaggle/kaggle.json`.
3. On macOS/Linux, restrict its permissions: `chmod 600 ~/.kaggle/kaggle.json`.

**Never commit `kaggle.json` or any API keys.** Put secrets in a `.env` file (already gitignored).

## Tips

**Git and notebooks**
- **Clear outputs before committing notebooks** (Jupyter: *Edit → Clear All Outputs*; VS Code: *Clear All Outputs* in the toolbar). Outputs make diffs noisy and cause merge conflicts.
- **Avoid editing the same notebook as a teammate at the same time.** Notebook merge conflicts are hard to resolve. Make your own notebook (e.g. `modeling_kenneth.ipynb`) for experiments.
- **Move reusable code (data loading, feature engineering) into `.py` files** and import it from notebooks, so everyone uses the same logic.

**Modeling**
- **Split data by time, not randomly.** Train on earlier seasons and test on later ones (e.g. train 2015–2022, test 2023). A random split leaks information from future games into training and makes accuracy look better than it really is.
- **Only use information available before kickoff.** Features like final stats, box score numbers, or end-of-season rankings leak the result.
- **Compare against a simple baseline**, such as "the home team wins" or "the higher-ranked team wins". A model is only useful if it beats that.
- **Set `random_state`** in scikit-learn and XGBoost so results can be reproduced.
- **Trained models (`*.pkl`, `*.joblib`, etc.) are gitignored.** Save the code that produces them, not the files.
